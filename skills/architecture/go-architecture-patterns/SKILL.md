---
name: go-architecture-patterns
description: >
  Define e aplica padrões de arquitetura e design em projetos Go: Hexagonal
  Architecture (Ports & Adapters), Repository Pattern com abstração de SQL,
  Injeção de Dependência (DI manual e Wire) e Functional Options Pattern.
  Use esta skill ao estruturar um novo projeto Go, revisar organização de
  pacotes, criar abstrações de banco de dados, ou configurar dependências.
---

# Go Architecture Patterns

Esta skill define os principais padrões arquiteturais e de design para projetos
Go idiomáticos, produtíveis e testáveis.

---

## Padrão 1 — Hexagonal Architecture (Ports & Adapters)

### Conceito

A Arquitetura Hexagonal divide o sistema em três zonas:

```
         ┌──────────────────────────────────────┐
         │             Adapters                 │
         │  ┌────────────────────────────────┐  │
         │  │      Application Core          │  │
         │  │  ┌──────────────────────────┐  │  │
         │  │  │       Domain             │  │  │
         │  │  │  (Entities + Services)   │  │  │
         │  │  └──────────────────────────┘  │  │
         │  │       Ports (Interfaces)        │  │
         │  └────────────────────────────────┘  │
         │   HTTP │ gRPC │ CLI │ DB │ Queue      │
         └──────────────────────────────────────┘
```

- **Core (Domain + Application)**: lógica de negócio pura, zero dependência de infra
- **Ports**: interfaces Go que o core define e expõe
- **Adapters**: implementações concretas (HTTP handlers, repositórios SQL, clientes gRPC)

### Estrutura de Pacotes

```
myapp/
├── cmd/
│   └── api/
│       └── main.go              # Composition Root
├── internal/
│   ├── domain/
│   │   ├── user.go              # Entidade
│   │   ├── user_service.go      # Domain Service
│   │   └── errors.go
│   ├── application/
│   │   ├── user_usecase.go      # Use Case (orquestração)
│   │   └── ports/
│   │       ├── user_repository.go  # Port (interface de saída)
│   │       └── email_service.go    # Port (interface de saída)
│   └── adapters/
│       ├── http/
│       │   ├── handler/
│       │   │   └── user_handler.go
│       │   └── middleware/
│       │       └── auth.go
│       ├── postgres/
│       │   └── user_repository.go  # Adapter (impl. do port)
│       └── email/
│           └── smtp_service.go     # Adapter (impl. do port)
├── pkg/                         # Código público reutilizável
└── go.mod
```

### Port (Interface de Saída — definida pelo core)

```go
// internal/application/ports/user_repository.go
package ports

import (
    "context"
    "myapp/internal/domain"
)

// UserRepository é o port de saída para persistência de usuários.
// O core define; os adapters implementam.
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    Save(ctx context.Context, user *domain.User) error
    Delete(ctx context.Context, id string) error
}

// EmailService é o port de saída para envio de emails.
type EmailService interface {
    SendWelcome(ctx context.Context, to, name string) error
}
```

### Use Case (Application Core)

```go
// internal/application/user_usecase.go
package application

import (
    "context"
    "myapp/internal/application/ports"
    "myapp/internal/domain"
)

type UserUseCase struct {
    repo         ports.UserRepository  // port — não o adapter concreto
    emailService ports.EmailService
}

func NewUserUseCase(repo ports.UserRepository, email ports.EmailService) *UserUseCase {
    return &UserUseCase{repo: repo, emailService: email}
}

type CreateUserInput struct {
    Name     string
    Email    string
    Password string
}

func (uc *UserUseCase) Create(ctx context.Context, input CreateUserInput) (*domain.User, error) {
    existing, err := uc.repo.FindByEmail(ctx, input.Email)
    if err != nil {
        return nil, err
    }
    if existing != nil {
        return nil, domain.ErrEmailAlreadyTaken
    }

    user, err := domain.NewUser(input.Name, input.Email, input.Password)
    if err != nil {
        return nil, err
    }

    if err := uc.repo.Save(ctx, user); err != nil {
        return nil, err
    }

    _ = uc.emailService.SendWelcome(ctx, user.Email, user.Name) // best-effort

    return user, nil
}
```

### Adapter HTTP (Driving Adapter)

```go
// internal/adapters/http/handler/user_handler.go
package handler

import (
    "encoding/json"
    "net/http"
    "myapp/internal/application"
)

type UserHandler struct {
    uc *application.UserUseCase // depende do use case, não do domínio diretamente
}

func NewUserHandler(uc *application.UserUseCase) *UserHandler {
    return &UserHandler{uc: uc}
}

func (h *UserHandler) Create(w http.ResponseWriter, r *http.Request) {
    var input application.CreateUserInput
    if err := json.NewDecoder(r.Body).Decode(&input); err != nil {
        http.Error(w, "invalid body", http.StatusBadRequest)
        return
    }

    user, err := h.uc.Create(r.Context(), input)
    if err != nil {
        mapErrorToHTTP(w, err)
        return
    }

    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}
```

---

## Padrão 2 — Repository Pattern (Abstração de SQL)

### Interface do Repositório

Sempre defina a interface no lado do **consumidor** (application/ports), não no lado da implementação.

```go
// internal/application/ports/user_repository.go
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    Save(ctx context.Context, user *domain.User) error
    Delete(ctx context.Context, id string) error
    ListByRole(ctx context.Context, role domain.Role, limit, offset int) ([]*domain.User, error)
}
```

### Implementação com `database/sql` puro

```go
// internal/adapters/postgres/user_repository.go
package postgres

import (
    "context"
    "database/sql"
    "errors"
    "myapp/internal/domain"

    _ "github.com/lib/pq"
)

type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

func (r *UserRepository) FindByID(ctx context.Context, id string) (*domain.User, error) {
    const query = `
        SELECT id, name, email, role, password_hash, created_at
        FROM users
        WHERE id = $1 AND deleted_at IS NULL
    `

    row := r.db.QueryRowContext(ctx, query, id)
    return r.scan(row)
}

func (r *UserRepository) Save(ctx context.Context, user *domain.User) error {
    const query = `
        INSERT INTO users (id, name, email, role, password_hash, created_at)
        VALUES ($1, $2, $3, $4, $5, $6)
        ON CONFLICT (id) DO UPDATE SET
            name         = EXCLUDED.name,
            email        = EXCLUDED.email,
            password_hash = EXCLUDED.password_hash
    `
    _, err := r.db.ExecContext(ctx, query,
        user.ID, user.Name, user.Email,
        user.Role, user.PasswordHash, user.CreatedAt,
    )
    return err
}

// scan mapeia sql.Row → domain.User (separação entre modelo de BD e entidade)
func (r *UserRepository) scan(row *sql.Row) (*domain.User, error) {
    var u domain.User
    err := row.Scan(&u.ID, &u.Name, &u.Email, &u.Role, &u.PasswordHash, &u.CreatedAt)
    if errors.Is(err, sql.ErrNoRows) {
        return nil, nil // not found → nil, nil (padrão Go)
    }
    if err != nil {
        return nil, err
    }
    return &u, nil
}
```

### Implementação com `sqlx` (mais ergonômica)

```go
// internal/adapters/postgres/user_repository_sqlx.go
package postgres

import (
    "context"
    "database/sql"
    "errors"
    "myapp/internal/domain"

    "github.com/jmoiron/sqlx"
)

type userRow struct {
    ID           string    `db:"id"`
    Name         string    `db:"name"`
    Email        string    `db:"email"`
    Role         string    `db:"role"`
    PasswordHash string    `db:"password_hash"`
    CreatedAt    time.Time `db:"created_at"`
}

type UserRepositorySqlx struct {
    db *sqlx.DB
}

func (r *UserRepositorySqlx) FindByID(ctx context.Context, id string) (*domain.User, error) {
    var row userRow
    err := r.db.GetContext(ctx, &row, `
        SELECT id, name, email, role, password_hash, created_at
        FROM users WHERE id = $1 AND deleted_at IS NULL
    `, id)
    if errors.Is(err, sql.ErrNoRows) {
        return nil, nil
    }
    if err != nil {
        return nil, err
    }
    return rowToDomain(&row), nil
}

func (r *UserRepositorySqlx) ListByRole(ctx context.Context, role domain.Role, limit, offset int) ([]*domain.User, error) {
    var rows []userRow
    err := r.db.SelectContext(ctx, &rows, `
        SELECT id, name, email, role, password_hash, created_at
        FROM users WHERE role = $1 AND deleted_at IS NULL
        ORDER BY created_at DESC
        LIMIT $2 OFFSET $3
    `, role, limit, offset)
    if err != nil {
        return nil, err
    }

    users := make([]*domain.User, len(rows))
    for i, row := range rows {
        row := row // capture
        users[i] = rowToDomain(&row)
    }
    return users, nil
}

func rowToDomain(r *userRow) *domain.User {
    return &domain.User{
        ID:           r.ID,
        Name:         r.Name,
        Email:        r.Email,
        Role:         domain.Role(r.Role),
        PasswordHash: r.PasswordHash,
        CreatedAt:    r.CreatedAt,
    }
}
```

### Transações

```go
// internal/application/ports/tx.go

// TxManager é o port para gerenciar transações de banco de dados.
type TxManager interface {
    RunInTx(ctx context.Context, fn func(ctx context.Context) error) error
}

// internal/adapters/postgres/tx_manager.go
type TxManager struct {
    db *sql.DB
}

func (m *TxManager) RunInTx(ctx context.Context, fn func(ctx context.Context) error) error {
    tx, err := m.db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }

    // Injeta a tx no contexto para que repositórios a usem
    ctx = context.WithValue(ctx, txKey{}, tx)

    if err := fn(ctx); err != nil {
        _ = tx.Rollback()
        return err
    }
    return tx.Commit()
}
```

### Mock em Testes

```go
// internal/adapters/mock/user_repository.go  (ou usando testify/mock)
package mock

type UserRepository struct {
    FindByIDFn    func(ctx context.Context, id string) (*domain.User, error)
    FindByEmailFn func(ctx context.Context, email string) (*domain.User, error)
    SaveFn        func(ctx context.Context, user *domain.User) error
}

func (m *UserRepository) FindByID(ctx context.Context, id string) (*domain.User, error) {
    return m.FindByIDFn(ctx, id)
}

func (m *UserRepository) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    return m.FindByEmailFn(ctx, email)
}

func (m *UserRepository) Save(ctx context.Context, user *domain.User) error {
    return m.SaveFn(ctx, user)
}

// No teste:
func TestCreateUser_EmailTaken(t *testing.T) {
    repo := &mock.UserRepository{
        FindByEmailFn: func(_ context.Context, _ string) (*domain.User, error) {
            return &domain.User{Email: "taken@example.com"}, nil
        },
    }
    uc := application.NewUserUseCase(repo, &mock.EmailService{})

    _, err := uc.Create(context.Background(), application.CreateUserInput{
        Email: "taken@example.com",
    })
    require.ErrorIs(t, err, domain.ErrEmailAlreadyTaken)
}
```

---

## Padrão 3 — Dependency Injection em Go

Go não tem container de DI nativo, mas o padrão idiomático é **DI manual com
Composition Root** em `main.go`. Para projetos maiores, use **Wire** (Google).

### DI Manual — Composition Root

```go
// cmd/api/main.go
package main

import (
    "database/sql"
    "log"
    "net/http"

    _ "github.com/lib/pq"

    "myapp/internal/adapters/email"
    "myapp/internal/adapters/http/handler"
    "myapp/internal/adapters/postgres"
    "myapp/internal/application"
)

func main() {
    // 1. Infraestrutura
    db, err := sql.Open("postgres", mustEnv("DATABASE_URL"))
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // 2. Adapters (implementações dos ports)
    userRepo := postgres.NewUserRepository(db)
    emailSvc := email.NewSMTPService(mustEnv("SMTP_HOST"))

    // 3. Use Cases (recebem ports, não implementações)
    userUseCase := application.NewUserUseCase(userRepo, emailSvc)

    // 4. HTTP Handlers
    userHandler := handler.NewUserHandler(userUseCase)

    // 5. Rotas
    mux := http.NewServeMux()
    mux.HandleFunc("POST /users", userHandler.Create)
    mux.HandleFunc("GET /users/{id}", userHandler.GetByID)

    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

### Wire (Google) — DI com geração de código

Wire gera o código de wiring automaticamente a partir de **providers**.

```bash
go install github.com/google/wire/cmd/wire@latest
```

```go
// cmd/api/wire.go  (arquivo de configuração do Wire — não vai para prod)
//go:build wireinject

package main

import (
    "database/sql"
    "github.com/google/wire"
    "myapp/internal/adapters/email"
    "myapp/internal/adapters/postgres"
    "myapp/internal/application"
    "myapp/internal/adapters/http/handler"
)

// ProviderSet agrupa os providers de cada camada
var infraSet = wire.NewSet(
    postgres.NewUserRepository,
    email.NewSMTPService,
)

var appSet = wire.NewSet(
    application.NewUserUseCase,
)

var handlerSet = wire.NewSet(
    handler.NewUserHandler,
)

// InitializeApp é a função que o Wire vai gerar
func InitializeApp(db *sql.DB, smtpHost string) (*App, error) {
    wire.Build(infraSet, appSet, handlerSet, NewApp)
    return nil, nil // Wire substitui isso
}
```

```bash
wire gen ./cmd/api/  # gera wire_gen.go automaticamente
```

### Regras de DI em Go

- **Nunca** use `init()` para inicializar dependências globais
- **Nunca** use variáveis globais mutáveis para injetar dependências
- Prefira `*ConcreteType` no retorno dos construtores, interface no parâmetro de quem consome
- Construtores retornam `(*Type, error)` quando a inicialização pode falhar

```go
// ✅ Correto — retorna concreto, recebe interface
func NewUserUseCase(repo ports.UserRepository, email ports.EmailService) *UserUseCase

// ❌ Incorreto — retorna interface (desnecessário e obscuro)
func NewUserUseCase(...) ports.UserUseCase
```

---

## Padrão 4 — Functional Options Pattern

Usado para configurar structs com muitos parâmetros opcionais de forma elegante,
sem proliferar construtores ou usar structs de config complexas.

### Implementação Básica

```go
// pkg/httpserver/server.go
package httpserver

import (
    "net/http"
    "time"
)

type Server struct {
    addr         string
    readTimeout  time.Duration
    writeTimeout time.Duration
    maxBodyBytes int64
    handler      http.Handler
    tls          *tlsConfig
}

// Option é o tipo funcional — cada opção é uma função que modifica o Server
type Option func(*Server)

// Opções públicas (funções construtoras de Option)

func WithAddr(addr string) Option {
    return func(s *Server) {
        s.addr = addr
    }
}

func WithReadTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.readTimeout = d
    }
}

func WithWriteTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.writeTimeout = d
    }
}

func WithMaxBodyBytes(n int64) Option {
    return func(s *Server) {
        s.maxBodyBytes = n
    }
}

func WithTLS(certFile, keyFile string) Option {
    return func(s *Server) {
        s.tls = &tlsConfig{certFile: certFile, keyFile: keyFile}
    }
}

// New é o construtor principal com defaults sensatos
func New(handler http.Handler, opts ...Option) *Server {
    s := &Server{
        addr:         ":8080",                // default
        readTimeout:  5 * time.Second,        // default
        writeTimeout: 10 * time.Second,       // default
        maxBodyBytes: 1 << 20,               // 1 MB default
        handler:      handler,
    }

    for _, opt := range opts {
        opt(s) // aplica cada opção
    }

    return s
}

func (s *Server) Start() error {
    srv := &http.Server{
        Addr:         s.addr,
        Handler:      s.handler,
        ReadTimeout:  s.readTimeout,
        WriteTimeout: s.writeTimeout,
    }
    if s.tls != nil {
        return srv.ListenAndServeTLS(s.tls.certFile, s.tls.keyFile)
    }
    return srv.ListenAndServe()
}
```

### Uso

```go
// cmd/api/main.go

// Simples — só defaults
srv := httpserver.New(mux)

// Com customizações seletivas
srv := httpserver.New(mux,
    httpserver.WithAddr(":9090"),
    httpserver.WithReadTimeout(10*time.Second),
    httpserver.WithTLS("/etc/ssl/cert.pem", "/etc/ssl/key.pem"),
)

srv.Start()
```

### Functional Options com Validação

```go
// Para casos onde a opção pode falhar (ex: parsing de URL, certificado inválido)

type Option func(*Server) error

func WithAddr(addr string) Option {
    return func(s *Server) error {
        if addr == "" {
            return errors.New("addr cannot be empty")
        }
        s.addr = addr
        return nil
    }
}

func New(handler http.Handler, opts ...Option) (*Server, error) {
    s := &Server{
        addr:    ":8080",
        handler: handler,
    }
    for _, opt := range opts {
        if err := opt(s); err != nil {
            return nil, fmt.Errorf("server option: %w", err)
        }
    }
    return s, nil
}
```

### Quando Usar Functional Options vs Alternativas

| Situação | Padrão recomendado |
|----------|-------------------|
| 0–2 parâmetros opcionais | Parâmetros diretos na função |
| 3–5 parâmetros opcionais simples | Struct de configuração (`Config struct`) |
| 5+ parâmetros opcionais, alguns com validação | **Functional Options** |
| API pública de biblioteca/SDK | **Functional Options** (extensível sem breaking change) |
| Configuração complexa hierárquica | Builder pattern |

---

## Estrutura de Projeto Completa (Go + Hexagonal)

```
myapp/
├── cmd/
│   ├── api/
│   │   ├── main.go          # Composition Root
│   │   ├── wire.go          # Wire providers (build tag: wireinject)
│   │   └── wire_gen.go      # Gerado pelo Wire (não editar)
│   └── worker/
│       └── main.go
├── internal/
│   ├── domain/
│   │   ├── user.go          # Entidade + regras de negócio
│   │   ├── order.go
│   │   └── errors.go        # Sentinel errors do domínio
│   ├── application/
│   │   ├── ports/
│   │   │   ├── user_repository.go
│   │   │   ├── email_service.go
│   │   │   └── tx_manager.go
│   │   └── user_usecase.go
│   └── adapters/
│       ├── http/
│       │   ├── handler/
│       │   │   └── user_handler.go
│       │   ├── middleware/
│       │   └── router.go
│       ├── postgres/
│       │   ├── user_repository.go
│       │   ├── tx_manager.go
│       │   └── migrations/
│       ├── email/
│       │   └── smtp_service.go
│       └── mock/              # Mocks para testes
│           ├── user_repository.go
│           └── email_service.go
├── pkg/
│   ├── httpserver/            # Functional Options aplicado aqui
│   │   └── server.go
│   └── postgres/
│       └── client.go
├── go.mod
└── go.sum
```

---

## Regras de Importação de Pacotes

| Pacote | Pode importar | Nunca pode importar |
|--------|--------------|---------------------|
| `domain` | stdlib, `pkg` | `application`, `adapters`, `cmd` |
| `application` | `domain`, stdlib | `adapters`, `cmd`, SQL drivers |
| `adapters` | `domain`, `application`, `pkg` | `cmd` |
| `cmd` | tudo | — |
| `pkg` | stdlib | `internal/` |

---

## Anti-padrões a Evitar em Go

| Anti-padrão | Problema | Solução |
|-------------|----------|---------|
| `sql.DB` passado diretamente para use case | Viola o princípio de inversão de dependência | Use interface `UserRepository` |
| `interface{}` / `any` como dependência | Perde type-safety | Defina interface específica |
| Variável global para DB: `var db *sql.DB` | Estado global, difícil de testar | Injete via construtor |
| `init()` abrindo conexão de banco | Ordem de execução imprevisível | Abra em `main()` |
| Struct de config com 15+ campos | Difícil de usar e manter | Functional Options |
| Mock com `//go:generate mockgen` em `domain/` | Mock gerado no lugar errado | Coloque em `adapters/mock/` |
| Retornar interface no construtor | Anti-padrão Go (Go Proverbs: "Accept interfaces, return structs") | Retorne `*ConcreteType` |

---

## Checklist de Arquitetura Go

- [ ] `domain` e `application` não importam drivers de banco/frameworks?
- [ ] Interfaces estão definidas no lado do **consumidor** (application/ports)?
- [ ] Construtores recebem interfaces e retornam tipos concretos?
- [ ] O Composition Root (`main.go`) é o único lugar que instancia adaptadores concretos?
- [ ] Repositórios retornam `nil, nil` para "não encontrado" (padrão Go)?
- [ ] Erros de domínio são `sentinel errors` ou tipos customizados (não strings)?
- [ ] Functional Options usados onde há 5+ parâmetros opcionais?
- [ ] Mocks estão em `adapters/mock/` e não misturados com código de produção?
- [ ] Testes de use case usam mocks manuais (sem container de DI)?

---

## Referências

- [Hexagonal Architecture — Alistair Cockburn](https://alistair.cockburn.us/hexagonal-architecture/)
- [Go Proverbs — Rob Pike](https://go-dev.appspot.com/talks/2015/proverbs.slide)
- [Google Wire](https://github.com/google/wire)
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout)
- [sqlx](https://github.com/jmoiron/sqlx)
- [Effective Go — Interfaces](https://go.dev/doc/effective_go#interfaces)
