---
name: js-ts-dependency-injection
description: >
  Define e aplica padrões de Injeção de Dependência (DI) em projetos JavaScript
  e TypeScript. Use esta skill ao criar classes que dependem de serviços externos,
  ao configurar um container de DI, ao escrever testes unitários com mocks, ou ao
  revisar se acoplamento entre módulos está correto. Complementa a skill js-ts-clean-architecture.
---

# JS/TS Dependency Injection — Injeção de Dependência

Esta skill define como aplicar DI de forma consistente em projetos JS/TS,
desde o padrão manual até containers como TSyringe e InversifyJS.

---

## O que é e Por que Usar

**Dependency Injection** é o princípio pelo qual uma classe **recebe** suas
dependências em vez de criá-las internamente.

```typescript
// ❌ Sem DI — acoplamento rígido
class CreateUserUseCase {
  private repo = new PrismaUserRepository(); // ← difícil de testar, difícil de trocar
}

// ✅ Com DI — acoplamento fraco
class CreateUserUseCase {
  constructor(private readonly repo: UserRepository) {} // ← recebe a abstração
}
```

**Benefícios:**
- **Testabilidade**: substitua dependências por mocks/stubs em testes unitários
- **Intercambialidade**: troque `PrismaUserRepository` por `MongoUserRepository` sem tocar no Use Case
- **Coesão**: cada classe tem uma única responsabilidade — não cria seus próprios colaboradores
- **Aderência ao DIP**: dependa de abstrações (interfaces), não de implementações concretas

---

## Padrão 1 — DI Manual (Composition Root)

A forma mais simples: instancie tudo em um único ponto de entrada (`main.ts` / `app.ts`).
**Recomendado para projetos pequenos/médios.**

### Estrutura do Composition Root

```typescript
// src/main.ts  ← único lugar que "sabe" de tudo

import { PrismaClient } from '@prisma/client';
import { PrismaUserRepository } from './infrastructure/database/PrismaUserRepository';
import { BcryptPasswordHashService } from './infrastructure/services/BcryptPasswordHashService';
import { NodemailerEmailService } from './infrastructure/services/NodemailerEmailService';
import { CreateUserUseCase } from './application/use-cases/create-user/CreateUserUseCase';
import { UserController } from './presentation/http/controllers/UserController';
import { createRouter } from './presentation/http/routes/user.routes';

// 1. Instanciar infraestrutura
const prisma = new PrismaClient();

// 2. Instanciar repositórios e serviços (implementações concretas)
const userRepository = new PrismaUserRepository(prisma);
const passwordHasher = new BcryptPasswordHashService();
const emailService = new NodemailerEmailService(process.env.SMTP_HOST!);

// 3. Instanciar use cases (recebem interfaces)
const createUserUseCase = new CreateUserUseCase(
  userRepository,
  passwordHasher,
  emailService,
);

// 4. Instanciar controllers (recebem use cases)
const userController = new UserController(createUserUseCase);

// 5. Montar rotas
const router = createRouter(userController);
```

### Regras do Composition Root

- **Um único arquivo** faz o wiring — nenhuma outra camada instancia dependências
- As camadas internas (Domain, Application) **nunca** fazem `new ConcreteClass()`
- A ordem de instanciação segue as dependências: infra → use cases → controllers

---

## Padrão 2 — Factory Functions

Alternativa ao `new` direto; útil para encapsular a criação e configuração.

```typescript
// src/infrastructure/factories/makeCreateUserUseCase.ts
export function makeCreateUserUseCase(): CreateUserUseCase {
  const prisma = getPrismaClient(); // singleton
  const userRepo = new PrismaUserRepository(prisma);
  const hasher = new BcryptPasswordHashService();
  const email = new NodemailerEmailService(env.SMTP_HOST);
  return new CreateUserUseCase(userRepo, hasher, email);
}

// src/presentation/http/routes/user.routes.ts
router.post('/users', async (req, res) => {
  const useCase = makeCreateUserUseCase(); // ou cache o singleton
  const controller = new UserController(useCase);
  return controller.create(req, res);
});
```

**Quando usar:** quando o Composition Root único ficaria muito grande, ou quando
diferentes contextos (HTTP vs CLI) precisam de composições ligeiramente diferentes.

---

## Padrão 3 — Container com TSyringe

Para projetos maiores onde DI manual fica verboso. **TSyringe** é leve e integra bem
com TypeScript sem exigir decoradores complexos.

### Setup

```bash
npm install tsyringe reflect-metadata
```

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

```typescript
// src/main.ts — importar reflect-metadata PRIMEIRO
import 'reflect-metadata';
```

### Registrando dependências

```typescript
// src/infrastructure/container/index.ts
import { container } from 'tsyringe';
import { PrismaUserRepository } from '../database/PrismaUserRepository';
import { BcryptPasswordHashService } from '../services/BcryptPasswordHashService';

// Registrar implementações para as interfaces (tokens)
container.registerSingleton<UserRepository>(
  'UserRepository',
  PrismaUserRepository,
);

container.registerSingleton<PasswordHashService>(
  'PasswordHashService',
  BcryptPasswordHashService,
);
```

### Injetando com decoradores

```typescript
// application/use-cases/create-user/CreateUserUseCase.ts
import { inject, injectable } from 'tsyringe';

@injectable()
export class CreateUserUseCase {
  constructor(
    @inject('UserRepository')
    private readonly userRepository: UserRepository,

    @inject('PasswordHashService')
    private readonly passwordHasher: PasswordHashService,
  ) {}

  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    // ...
  }
}
```

### Resolvendo no entrypoint

```typescript
// src/main.ts
import 'reflect-metadata';
import './infrastructure/container'; // carrega os registros

import { container } from 'tsyringe';
import { CreateUserUseCase } from './application/use-cases/create-user/CreateUserUseCase';

const createUserUseCase = container.resolve(CreateUserUseCase);
```

---

## Padrão 4 — Container com InversifyJS

Mais robusto e verboso; preferido para projetos enterprise com muitas dependências.

```bash
npm install inversify reflect-metadata
```

```typescript
// src/infrastructure/container/types.ts
export const TYPES = {
  UserRepository: Symbol.for('UserRepository'),
  PasswordHashService: Symbol.for('PasswordHashService'),
  CreateUserUseCase: Symbol.for('CreateUserUseCase'),
};

// src/infrastructure/container/container.ts
import { Container } from 'inversify';
import { TYPES } from './types';

const container = new Container();
container.bind<UserRepository>(TYPES.UserRepository).to(PrismaUserRepository).inSingletonScope();
container.bind<PasswordHashService>(TYPES.PasswordHashService).to(BcryptPasswordHashService).inSingletonScope();
container.bind<CreateUserUseCase>(TYPES.CreateUserUseCase).to(CreateUserUseCase).inTransientScope();

export { container };

// src/application/use-cases/create-user/CreateUserUseCase.ts
@injectable()
export class CreateUserUseCase {
  constructor(
    @inject(TYPES.UserRepository) private readonly repo: UserRepository,
    @inject(TYPES.PasswordHashService) private readonly hasher: PasswordHashService,
  ) {}
}
```

---

## Scopes de Vida das Dependências

| Scope | Significado | Quando usar |
|-------|-------------|-------------|
| **Singleton** | Uma única instância para toda a aplicação | Conexões de DB, clientes HTTP, caches |
| **Transient** | Nova instância a cada resolução | Use Cases, objetos stateful |
| **Scoped** | Uma instância por request (HTTP scope) | Unit of Work, contexto de transação |

```typescript
// TSyringe
container.registerSingleton('PrismaClient', PrismaClientWrapper);  // Singleton
container.register('CreateUserUseCase', { useClass: CreateUserUseCase }); // Transient (padrão)

// InversifyJS
container.bind(TYPES.Prisma).to(PrismaWrapper).inSingletonScope();
container.bind(TYPES.CreateUser).to(CreateUserUseCase).inTransientScope();
container.bind(TYPES.UnitOfWork).to(UnitOfWork).inRequestScope();
```

---

## DI em Testes Unitários

O principal benefício da DI é a **facilidade de mockar dependências** em testes.

### Com Jest — mock manual

```typescript
// application/use-cases/create-user/CreateUserUseCase.spec.ts
import { CreateUserUseCase } from './CreateUserUseCase';

// Mock da interface UserRepository
const mockUserRepository: jest.Mocked<UserRepository> = {
  findById: jest.fn(),
  findByEmail: jest.fn(),
  save: jest.fn(),
  delete: jest.fn(),
};

const mockPasswordHasher: jest.Mocked<PasswordHashService> = {
  hash: jest.fn(),
  compare: jest.fn(),
};

const mockEmailService: jest.Mocked<EmailService> = {
  sendWelcomeEmail: jest.fn(),
};

describe('CreateUserUseCase', () => {
  let sut: CreateUserUseCase;

  beforeEach(() => {
    jest.clearAllMocks();
    // Injeção manual — simples, sem container
    sut = new CreateUserUseCase(
      mockUserRepository,
      mockPasswordHasher,
      mockEmailService,
    );
  });

  it('should create user when email is not taken', async () => {
    mockUserRepository.findByEmail.mockResolvedValue(null);
    mockPasswordHasher.hash.mockResolvedValue('hashed-pw');
    mockUserRepository.save.mockResolvedValue();
    mockEmailService.sendWelcomeEmail.mockResolvedValue();

    const result = await sut.execute({
      name: 'John Doe',
      email: 'john@example.com',
      password: 'secret123',
    });

    expect(result.email).toBe('john@example.com');
    expect(mockUserRepository.save).toHaveBeenCalledTimes(1);
    expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith('john@example.com');
  });

  it('should throw when email is already taken', async () => {
    mockUserRepository.findByEmail.mockResolvedValue(fakeUser());

    await expect(
      sut.execute({ name: 'Jane', email: 'taken@example.com', password: '123' }),
    ).rejects.toThrow(UserAlreadyExistsError);

    expect(mockUserRepository.save).not.toHaveBeenCalled();
  });
});
```

### Boas práticas em testes com DI

- **Nunca** use o container de DI em testes unitários — instancie manualmente
- Use o container de DI em testes de integração/e2e para testar o wiring real
- Prefira `jest.Mocked<Interface>` para type-safety nos mocks
- Crie **factories de teste** (`makeUser()`, `makeOrder()`) em `test/factories/` para reduzir boilerplate

---

## Anti-padrões de DI

| Anti-padrão | Problema | Solução |
|-------------|----------|---------|
| `new ConcreteClass()` dentro de Use Case | Acoplamento rígido, difícil de testar | Receba via construtor |
| Service Locator: `container.resolve()` dentro de Use Case | Dependência escondida, viola transparência | Injete no construtor |
| Injetar o container inteiro como dependência | Container vira dependência global | Injete apenas o que a classe precisa |
| Singleton de classes stateful/mutáveis | Compartilhamento de estado entre requests | Use Transient ou Scoped |
| Circular dependency (A → B → A) | Erro em runtime ou comportamento indefinido | Extraia interface intermediária ou use lazy injection |
| Injetar mais de ~5 dependências em uma classe | Indício de violação de SRP | Divida em classes menores |

---

## Escolha do Padrão por Tamanho de Projeto

| Tamanho | Recomendação |
|---------|-------------|
| Pequeno (< 10 use cases) | DI manual no Composition Root |
| Médio (10–50 use cases) | Factories por módulo + Composition Root |
| Grande (50+ use cases) | TSyringe (mais simples) ou InversifyJS (mais robusto) |
| NestJS | DI nativo do framework (não adicionar outro container) |

---

## Checklist de DI

- [ ] Classes recebem dependências pelo construtor (não criam internamente)?
- [ ] Use Cases dependem de interfaces, não de implementações concretas?
- [ ] O Composition Root é o único lugar que instancia implementações concretas?
- [ ] O scope de vida está correto (Singleton para conexões, Transient para use cases)?
- [ ] Testes unitários instanciam as classes manualmente com mocks?
- [ ] Não há Service Locator escondido (`container.resolve()` dentro de classes de negócio)?
- [ ] Nenhuma classe tem mais de 5 dependências injetadas?
- [ ] Dependências circulares foram eliminadas?

---

## Referências

- [Dependency Inversion Principle — Martin Fowler](https://martinfowler.com/articles/injection.html)
- [TSyringe — Microsoft](https://github.com/microsoft/tsyringe)
- [InversifyJS](https://inversify.io/)
- [SOLID em TypeScript](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
