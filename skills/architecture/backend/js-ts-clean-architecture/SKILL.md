---
name: backend-js-ts-clean-architecture
description: >
  Define e aplica a Arquitetura em Camadas (Layered Architecture) seguindo os
  princípios da Clean Architecture exclusivamente no contexto de Backend (Node.js,
  Express, NestJS). Use esta skill ao estruturar um novo projeto de API, revisar 
  organização de pastas, decidir onde uma classe/função deve residir, ou verificar
  se dependências entre camadas do servidor estão corretas.
---

# Backend JS/TS Clean Architecture — Arquitetura em Camadas

Esta skill define a estrutura de camadas, regras de dependência, convenções de
nomenclatura e padrões de código para projetos de Backend JavaScript e TypeScript
seguindo os princípios da Clean Architecture (Robert C. Martin).

---

## Fundamentos: A Regra de Dependência

> **Dependências de código-fonte só podem apontar para dentro (em direção às camadas mais internas).**

Camadas externas conhecem as internas. Camadas internas **nunca** conhecem as externas.

```
┌─────────────────────────────────────────────┐
│           Presentation / Drivers            │  ← Controllers, Routes, CLI, GraphQL
│  ┌──────────────────────────────────────┐   │
│  │         Application (Use Cases)      │   │  ← Orquestração de negócio
│  │  ┌───────────────────────────────┐   │   │
│  │  │          Domain               │   │   │  ← Entidades, regras puras
│  │  │  (Entities + Domain Services) │   │   │
│  │  └───────────────────────────────┘   │   │
│  └──────────────────────────────────────┘   │
│           Infrastructure                    │  ← DB, HTTP clients, email, cache
└─────────────────────────────────────────────┘
```

---

## As Camadas

### 1. Domain (Núcleo — sem dependências externas)

A camada mais interna. Contém as **regras de negócio puras**, independentes de
frameworks, banco de dados ou qualquer detalhe de infraestrutura.

**O que vive aqui:**
- **Entities**: objetos com identidade e ciclo de vida (`User`, `Order`, `Invoice`)
- **Value Objects**: objetos imutáveis sem identidade (`Email`, `Money`, `CPF`)
- **Domain Services**: lógica de negócio que não pertence a uma única entidade
- **Repository Interfaces**: contratos (interfaces TS) — **não implementações**
- **Domain Events**: eventos que representam fatos do negócio
- **Domain Errors**: erros específicos do domínio

**Regras:**
- Zero imports de bibliotecas externas (exceto utilitários puros como `uuid`)
- Zero imports das camadas Application, Infrastructure ou Presentation
- Código **puro**: sem side effects, sem I/O, sem framework

**Estrutura de pastas:**
```
src/
└── domain/
    ├── entities/
    │   ├── User.ts
    │   └── User.spec.ts
    ├── value-objects/
    │   ├── Email.ts
    │   └── Money.ts
    ├── services/
    │   └── PasswordHashService.ts     ← interface
    ├── repositories/
    │   └── UserRepository.ts          ← interface
    ├── events/
    │   └── UserCreatedEvent.ts
    └── errors/
        ├── DomainError.ts
        └── UserNotFoundError.ts
```

**Exemplo — Entity:**
```typescript
// domain/entities/User.ts
export class User {
  private constructor(
    public readonly id: string,
    public readonly email: Email,
    private passwordHash: string,
    public readonly createdAt: Date,
  ) {}

  static create(props: { email: string; passwordHash: string }): User {
    return new User(
      crypto.randomUUID(),
      Email.create(props.email),   // Value Object valida o formato
      props.passwordHash,
      new Date(),
    );
  }

  changePassword(newHash: string): void {
    if (!newHash) throw new DomainError('Password hash cannot be empty');
    this.passwordHash = newHash;
  }
}
```

**Exemplo — Value Object:**
```typescript
// domain/value-objects/Email.ts
export class Email {
  private constructor(public readonly value: string) {}

  static create(raw: string): Email {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(raw)) {
      throw new DomainError(`Invalid email: ${raw}`);
    }
    return new Email(raw.toLowerCase());
  }
}
```

**Exemplo — Repository Interface:**
```typescript
// domain/repositories/UserRepository.ts
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: Email): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}
```

---

### 2. Application (Use Cases)

Orquestra as entidades e serviços de domínio para realizar **casos de uso** do sistema.
Não contém lógica de negócio — apenas coordenação.

**O que vive aqui:**
- **Use Cases / Interactors**: um arquivo por caso de uso (`CreateUserUseCase`, `AuthenticateUserUseCase`)
- **DTOs**: objetos de transferência de dados (entrada e saída dos use cases)
- **Application Services**: serviços de aplicação (ex: envio de email após criação)
- **Port Interfaces**: contratos para serviços externos (email, storage, cache)

**Regras:**
- Pode importar apenas a camada Domain
- Não conhece HTTP, banco de dados, frameworks
- Cada use case tem **uma única responsabilidade**
- Recebe e retorna DTOs (nunca entidades do domínio diretamente para fora)

**Estrutura de pastas:**
```
src/
└── application/
    ├── use-cases/
    │   ├── create-user/
    │   │   ├── CreateUserUseCase.ts
    │   │   ├── CreateUserDTO.ts
    │   │   └── CreateUserUseCase.spec.ts
    │   └── authenticate-user/
    │       ├── AuthenticateUserUseCase.ts
    │       └── AuthenticateUserDTO.ts
    └── ports/
        ├── EmailService.ts        ← interface
        ├── CacheService.ts        ← interface
        └── TokenService.ts        ← interface
```

**Exemplo — Use Case:**
```typescript
// application/use-cases/create-user/CreateUserUseCase.ts
export interface CreateUserInput {
  name: string;
  email: string;
  password: string;
}

export interface CreateUserOutput {
  id: string;
  email: string;
  createdAt: Date;
}

export class CreateUserUseCase {
  constructor(
    private readonly userRepository: UserRepository,     // Domain interface
    private readonly passwordHasher: PasswordHashService, // Domain interface
    private readonly emailService: EmailService,          // Application port
  ) {}

  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    const existingUser = await this.userRepository.findByEmail(
      Email.create(input.email),
    );

    if (existingUser) {
      throw new UserAlreadyExistsError(input.email);
    }

    const passwordHash = await this.passwordHasher.hash(input.password);
    const user = User.create({ email: input.email, passwordHash });

    await this.userRepository.save(user);
    await this.emailService.sendWelcomeEmail(user.email.value);

    return {
      id: user.id,
      email: user.email.value,
      createdAt: user.createdAt,
    };
  }
}
```

---

### 3. Infrastructure (Adaptadores — detalhes externos)

Implementa as **interfaces definidas nas camadas internas** usando tecnologias concretas
(banco de dados, HTTP, cache, email, filesystem).

**O que vive aqui:**
- **Repository Implementations**: implementações concretas dos repositórios (`PrismaUserRepository`, `MongoUserRepository`)
- **ORM / Query Builders**: Prisma, TypeORM, Knex
- **External Service Adapters**: cliente de email (Nodemailer, SendGrid), cache (Redis), storage (S3)
- **Config**: carregamento de variáveis de ambiente
- **Database**: migrations, seeders, connection setup

**Regras:**
- Pode importar Domain e Application (para implementar as interfaces)
- Não é importada por Domain ou Application
- Cada implementação é **intercambiável** sem mudar o restante do sistema

**Estrutura de pastas:**
```
src/
└── infrastructure/
    ├── database/
    │   ├── prisma/
    │   │   ├── schema.prisma
    │   │   └── migrations/
    │   └── repositories/
    │       └── PrismaUserRepository.ts
    ├── services/
    │   ├── BcryptPasswordHashService.ts
    │   ├── NodemailerEmailService.ts
    │   └── JwtTokenService.ts
    ├── cache/
    │   └── RedisCache.ts
    └── config/
        └── env.ts
```

**Exemplo — Repository Implementation:**
```typescript
// infrastructure/database/repositories/PrismaUserRepository.ts
export class PrismaUserRepository implements UserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const record = await this.prisma.user.findUnique({ where: { id } });
    if (!record) return null;
    return this.toDomain(record);
  }

  async save(user: User): Promise<void> {
    await this.prisma.user.upsert({
      where: { id: user.id },
      create: this.toPersistence(user),
      update: this.toPersistence(user),
    });
  }

  private toDomain(record: PrismaUser): User { /* mapper */ }
  private toPersistence(user: User): PrismaUser { /* mapper */ }
}
```

---

### 4. Presentation (Entrypoints — drivers externos)

Recebe requisições do mundo externo e as traduz em chamadas aos Use Cases.

**O que vive aqui:**
- **Controllers / Route Handlers**: Express, Fastify, NestJS controllers
- **Middlewares**: autenticação, validação de schema, rate limiting
- **Validators / Schemas**: Zod, Joi, class-validator
- **Presenters / View Models**: transformação da saída dos Use Cases para o formato da resposta
- **GraphQL Resolvers**
- **CLI Commands**

**Regras:**
- Pode importar Application (Use Cases e DTOs)
- Não contém lógica de negócio
- Toda validação de input de usuário acontece aqui (antes de chegar ao Use Case)
- Mapeia erros de domínio para status HTTP adequados

**Estrutura de pastas:**
```
src/
└── presentation/
    ├── http/
    │   ├── routes/
    │   │   └── user.routes.ts
    │   ├── controllers/
    │   │   └── UserController.ts
    │   ├── middlewares/
    │   │   ├── auth.middleware.ts
    │   │   └── error-handler.middleware.ts
    │   └── validators/
    │       └── create-user.schema.ts
    └── graphql/
        └── resolvers/
            └── user.resolver.ts
```

**Exemplo — Controller:**
```typescript
// presentation/http/controllers/UserController.ts
const createUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  password: z.string().min(8),
});

export class UserController {
  constructor(private readonly createUserUseCase: CreateUserUseCase) {}

  async create(req: Request, res: Response): Promise<void> {
    const input = createUserSchema.parse(req.body); // valida aqui, não no use case

    const output = await this.createUserUseCase.execute(input);

    res.status(201).json(output);
  }
}
```

---

## Estrutura de Projeto Completa

```
src/
├── domain/
│   ├── entities/
│   ├── value-objects/
│   ├── services/          ← interfaces
│   ├── repositories/      ← interfaces
│   ├── events/
│   └── errors/
├── application/
│   ├── use-cases/
│   └── ports/             ← interfaces para serviços externos
├── infrastructure/
│   ├── database/
│   ├── services/          ← implementações concretas
│   ├── cache/
│   └── config/
├── presentation/
│   ├── http/
│   └── graphql/
└── main.ts                ← Composition Root (wiring de DI)
```

---

## Anti-padrões a Evitar

| Anti-padrão | Problema | Solução |
|-------------|----------|---------|
| Use Case importando `PrismaClient` diretamente | Viola a regra de dependência | Use a interface `UserRepository` |
| Entidade com `@Column()` do TypeORM | Domain acoplado a infra | Separe entity de model de persistência |
| Controller com lógica de negócio | Dificulta testes e reuso | Mova para Use Case |
| `import from '../../../infrastructure/...'` no Domain | Inversão da regra | Remova; o Domain nunca aponta para fora |
| Use Case retornando entity do domínio diretamente | Vaza detalhes internos | Retorne DTO mapeado |
| Fat repository (queries complexas de negócio no repositório) | Lógica de negócio fora do domínio | Mova para Domain Service ou Use Case |

---

## Regras de Importação (resumo)

| Camada | Pode importar | Nunca pode importar |
|--------|--------------|---------------------|
| Domain | nada (apenas stdlib/utils) | Application, Infrastructure, Presentation |
| Application | Domain | Infrastructure, Presentation |
| Infrastructure | Domain, Application | Presentation |
| Presentation | Application | Domain diretamente, Infrastructure diretamente |

---

## Checklist de Arquitetura

- [ ] Cada arquivo está na camada correta?
- [ ] As importações seguem a regra de dependência (só para dentro)?
- [ ] Repositórios e serviços externos estão atrás de interfaces?
- [ ] Use Cases recebem e retornam DTOs (não entidades)?
- [ ] Domain contém apenas lógica de negócio pura?
- [ ] Controllers fazem apenas validação + delegação?
- [ ] Infrastructure implementa, não define, as interfaces?
- [ ] O Composition Root (main.ts) é o único lugar que conhece todas as camadas?

---

## Referências

- [Clean Architecture — Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design Quickly](https://www.infoq.com/minibooks/domain-driven-design-quickly/)
- [Hexagonal Architecture (Ports & Adapters)](https://alistair.cockburn.us/hexagonal-architecture/)
