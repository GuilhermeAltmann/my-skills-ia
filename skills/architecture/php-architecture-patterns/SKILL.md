---
name: php-architecture-patterns
description: >
  Define e aplica padrões de arquitetura, PSRs (PHP Standards Recommendations),
  e código limpo em projetos PHP. Use esta skill ao estruturar um novo projeto PHP,
  revisar aderência às PSRs, aplicar SOLID e Design Patterns, organizar autoloading,
  tratar erros com PSR-3, ou padronizar HTTP com PSR-7/PSR-15.
---

# PHP Architecture Patterns — PSRs, SOLID e Código Limpo

Esta skill define os padrões de qualidade, arquitetura e convenções para projetos
PHP modernos (PHP 8.1+), cobrindo PSRs obrigatórias, princípios SOLID, Design
Patterns e boas práticas de código limpo.

---

## Parte 1 — PSRs (PHP Standards Recommendations)

As PSRs são padrões definidos pelo [PHP-FIG](https://www.php-fig.org/) para
interoperabilidade entre frameworks e bibliotecas.

### PSR-1 — Basic Coding Standard

Regras mínimas de formatação:

```php
<?php

// ✅ Namespace e use declarations
namespace App\Domain\User;

use App\Domain\Shared\Entity;
use App\Domain\Shared\ValueObject\Email;

// ✅ Classes em StudlyCaps
class UserService
{
    // ✅ Constantes em UPPER_SNAKE_CASE
    public const MAX_LOGIN_ATTEMPTS = 5;

    // ✅ Métodos em camelCase
    public function findByEmail(Email $email): ?User
    {
        // ...
    }
}
```

### PSR-2 / PSR-12 — Extended Coding Style (use PSR-12)

PSR-12 é a evolução da PSR-2. Regras principais:

```php
<?php

declare(strict_types=1); // ← obrigatório em todo arquivo

namespace App\Application\UseCase;

use App\Domain\User\UserRepository;
use App\Domain\User\User;

// ✅ Abertura de chave de classe na mesma linha do nome
class CreateUserUseCase
{
    public function __construct(
        private readonly UserRepository $userRepository, // ← propriedades no construtor (PHP 8.0+)
        private readonly PasswordHasher $hasher,
    ) {}

    // ✅ Visibilidade sempre declarada
    public function execute(CreateUserInput $input): CreateUserOutput
    {
        // ✅ Operador ternário em linha separada se longo
        $existing = $this->userRepository->findByEmail($input->email)
            ?? throw new UserAlreadyExistsException($input->email);

        // ...
    }
}
```

**Regras PSR-12 importantes:**
- `declare(strict_types=1)` logo após `<?php`
- 4 espaços de indentação (sem tabs)
- Chave de abertura de classe/interface na **mesma linha**
- Chave de abertura de método **na própria linha** (linha abaixo da assinatura)
- Uma linha em branco entre métodos
- Sem espaço antes de `(` em chamadas de função
- `use` após `namespace`, separados por linha em branco

### PSR-3 — Logger Interface

```php
// ✅ Sempre dependa de Psr\Log\LoggerInterface, nunca de implementação concreta

use Psr\Log\LoggerInterface;

class PaymentService
{
    public function __construct(
        private readonly LoggerInterface $logger, // ← PSR-3 interface
    ) {}

    public function process(Payment $payment): void
    {
        try {
            // ...
            $this->logger->info('Payment processed', [
                'payment_id' => $payment->id,
                'amount'     => $payment->amount,
            ]);
        } catch (\Throwable $e) {
            $this->logger->error('Payment failed', [
                'payment_id' => $payment->id,
                'error'      => $e->getMessage(),
                'trace'      => $e->getTraceAsString(),
            ]);
            throw $e;
        }
    }
}
```

**Níveis de log PSR-3 e quando usar:**

| Nível       | Quando usar |
|-------------|-------------|
| `emergency` | Sistema inutilizável |
| `alert`     | Ação imediata necessária (ex: banco down) |
| `critical`  | Condição crítica (ex: componente indisponível) |
| `error`     | Erros de runtime que não param o sistema |
| `warning`   | Situações excepcionais, mas não erros |
| `notice`    | Eventos normais mas significativos |
| `info`      | Eventos de interesse (login, criação de registro) |
| `debug`     | Informações detalhadas para debugging |

### PSR-4 — Autoloading

```json
// composer.json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    }
}
```

**Mapeamento namespace → arquivo:**

| Namespace | Arquivo |
|-----------|---------|
| `App\Domain\User\User` | `src/Domain/User/User.php` |
| `App\Application\UseCase\CreateUser` | `src/Application/UseCase/CreateUser.php` |
| `Tests\Unit\Domain\UserTest` | `tests/Unit/Domain/UserTest.php` |

**Regras:**
- Um arquivo PHP = uma classe/interface/trait/enum
- Nome do arquivo = nome da classe + `.php`
- Estrutura de diretório espelha o namespace exatamente

### PSR-7 — HTTP Message Interface

```php
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

// ✅ Imutabilidade — sempre use `with*` que retorna nova instância
class UserController
{
    public function create(ServerRequestInterface $request): ResponseInterface
    {
        $body = $request->getParsedBody(); // dados do POST

        // PSR-7 é imutável — with* retorna nova instância
        return $this->response
            ->withStatus(201)
            ->withHeader('Content-Type', 'application/json')
            ->withBody($this->streamFactory->createStream(
                json_encode(['id' => $userId], JSON_THROW_ON_ERROR)
            ));
    }
}
```

### PSR-11 — Container Interface

```php
use Psr\Container\ContainerInterface;

// ✅ Dependa da interface PSR-11, não do container concreto (Laravel, Symfony, PHP-DI)
class SomeFactory
{
    public function __construct(
        private readonly ContainerInterface $container,
    ) {}

    public function make(string $id): mixed
    {
        return $this->container->get($id);
    }
}
```

### PSR-14 — Event Dispatcher

```php
use Psr\EventDispatcher\EventDispatcherInterface;

class CreateUserUseCase
{
    public function __construct(
        private readonly UserRepository $repo,
        private readonly EventDispatcherInterface $dispatcher, // ← PSR-14
    ) {}

    public function execute(CreateUserInput $input): User
    {
        $user = User::create($input->email, $input->name);
        $this->repo->save($user);

        $this->dispatcher->dispatch(new UserCreatedEvent($user));

        return $user;
    }
}
```

### PSR-15 — HTTP Server Request Handlers & Middleware

```php
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

// ✅ Middleware PSR-15
class AuthMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler,
    ): ResponseInterface {
        $token = $request->getHeaderLine('Authorization');

        if (!$this->tokenService->validate($token)) {
            return $this->responseFactory->createResponse(401);
        }

        // Injeta o usuário autenticado no request
        $request = $request->withAttribute('user', $this->tokenService->decode($token));

        return $handler->handle($request);
    }
}
```

---

## Parte 2 — SOLID em PHP

### S — Single Responsibility Principle

```php
// ❌ Viola SRP — User faz tudo
class User
{
    public function save(): void { /* SQL aqui */ }
    public function sendWelcomeEmail(): void { /* SMTP aqui */ }
    public function generateReport(): string { /* HTML aqui */ }
}

// ✅ Cada classe tem uma única razão para mudar
class User { /* apenas dados e regras de negócio */ }
class UserRepository { /* apenas persistência */ }
class WelcomeEmailNotifier { /* apenas email */ }
class UserReportGenerator { /* apenas relatório */ }
```

### O — Open/Closed Principle

```php
// ✅ Aberto para extensão, fechado para modificação
interface PaymentGateway
{
    public function charge(Money $amount, string $token): PaymentResult;
}

class StripeGateway implements PaymentGateway { /* ... */ }
class PayPalGateway implements PaymentGateway { /* ... */ }
class MercadoPagoGateway implements PaymentGateway { /* ... */ }

// PaymentService não muda ao adicionar novo gateway
class PaymentService
{
    public function __construct(
        private readonly PaymentGateway $gateway, // ← depende da abstração
    ) {}
}
```

### L — Liskov Substitution Principle

```php
// ✅ Subclasse substituível sem alterar comportamento
interface Shape
{
    public function area(): float;
}

class Rectangle implements Shape
{
    public function area(): float
    {
        return $this->width * $this->height;
    }
}

class Circle implements Shape
{
    public function area(): float
    {
        return M_PI * $this->radius ** 2;
    }
}

// Funciona com qualquer Shape sem if/switch
function printArea(Shape $shape): void
{
    echo $shape->area();
}
```

### I — Interface Segregation Principle

```php
// ❌ Interface gorda — força implementações desnecessárias
interface UserInterface
{
    public function getName(): string;
    public function getEmail(): string;
    public function getAdminPermissions(): array; // nem todo user é admin
    public function getBillingAddress(): Address; // nem todo user tem cobrança
}

// ✅ Interfaces coesas e específicas
interface Identifiable
{
    public function getId(): string;
}

interface HasEmail
{
    public function getEmail(): string;
}

interface AdminUser extends Identifiable
{
    public function getPermissions(): array;
}

interface BillableUser extends HasEmail
{
    public function getBillingAddress(): Address;
}
```

### D — Dependency Inversion Principle

```php
// ❌ Depende de implementação concreta
class OrderService
{
    private MySQLOrderRepository $repo; // ← acoplamento rígido

    public function __construct()
    {
        $this->repo = new MySQLOrderRepository(); // ← cria a dependência
    }
}

// ✅ Depende de abstração; recebe via construtor
class OrderService
{
    public function __construct(
        private readonly OrderRepository $repo, // ← interface
    ) {}
}
```

---

## Parte 3 — Design Patterns em PHP

### Repository Pattern

```php
// src/Domain/Order/OrderRepository.php  ← interface no domínio
namespace App\Domain\Order;

interface OrderRepository
{
    public function findById(string $id): ?Order;
    public function findByCustomer(string $customerId): OrderCollection;
    public function save(Order $order): void;
    public function delete(string $id): void;
}

// src/Infrastructure/Persistence/DoctrineOrderRepository.php
namespace App\Infrastructure\Persistence;

use App\Domain\Order\{Order, OrderCollection, OrderRepository};
use Doctrine\ORM\EntityManagerInterface;

final class DoctrineOrderRepository implements OrderRepository
{
    public function __construct(
        private readonly EntityManagerInterface $em,
    ) {}

    public function findById(string $id): ?Order
    {
        return $this->em->find(Order::class, $id);
    }

    public function save(Order $order): void
    {
        $this->em->persist($order);
        $this->em->flush();
    }
}
```

### Value Objects

```php
// src/Domain/Shared/ValueObject/Money.php
namespace App\Domain\Shared\ValueObject;

use InvalidArgumentException;

final class Money
{
    private function __construct(
        private readonly int $amount,    // centavos — evita float para dinheiro
        private readonly string $currency,
    ) {}

    public static function of(int $amount, string $currency): self
    {
        if ($amount < 0) {
            throw new InvalidArgumentException('Amount cannot be negative');
        }
        if (!in_array($currency, ['BRL', 'USD', 'EUR'], true)) {
            throw new InvalidArgumentException("Unsupported currency: {$currency}");
        }
        return new self($amount, $currency);
    }

    public function add(Money $other): self
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException('Cannot add different currencies');
        }
        return new self($this->amount + $other->amount, $this->currency);
    }

    public function equals(Money $other): bool
    {
        return $this->amount === $other->amount && $this->currency === $other->currency;
    }

    public function getAmount(): int { return $this->amount; }
    public function getCurrency(): string { return $this->currency; }
}
```

### Service Layer (Use Cases)

```php
// src/Application/UseCase/CreateOrder/CreateOrderUseCase.php
namespace App\Application\UseCase\CreateOrder;

use App\Domain\Order\{Order, OrderRepository};
use App\Domain\Product\ProductRepository;
use App\Domain\Shared\ValueObject\Money;
use Psr\Log\LoggerInterface;

final class CreateOrderUseCase
{
    public function __construct(
        private readonly OrderRepository $orderRepository,
        private readonly ProductRepository $productRepository,
        private readonly EventDispatcherInterface $dispatcher,
        private readonly LoggerInterface $logger,
    ) {}

    public function execute(CreateOrderInput $input): CreateOrderOutput
    {
        $this->logger->info('Creating order', ['customer_id' => $input->customerId]);

        $items = array_map(
            fn(OrderItemInput $item) => $this->resolveItem($item),
            $input->items,
        );

        $order = Order::create($input->customerId, $items);
        $this->orderRepository->save($order);
        $this->dispatcher->dispatch(new OrderCreatedEvent($order));

        $this->logger->info('Order created', ['order_id' => $order->getId()]);

        return CreateOrderOutput::fromOrder($order);
    }

    private function resolveItem(OrderItemInput $item): OrderItem
    {
        $product = $this->productRepository->findById($item->productId)
            ?? throw new ProductNotFoundException($item->productId);

        return OrderItem::create($product, $item->quantity);
    }
}
```

### Factory Pattern

```php
// src/Domain/Notification/NotificationFactory.php
namespace App\Domain\Notification;

enum NotificationChannel: string
{
    case Email = 'email';
    case SMS   = 'sms';
    case Push  = 'push';
}

interface NotificationFactory
{
    public function create(NotificationChannel $channel): NotificationSender;
}

// src/Infrastructure/Notification/ConcreteNotificationFactory.php
final class ConcreteNotificationFactory implements NotificationFactory
{
    public function __construct(
        private readonly EmailSender $emailSender,
        private readonly SmsSender $smsSender,
        private readonly PushSender $pushSender,
    ) {}

    public function create(NotificationChannel $channel): NotificationSender
    {
        return match ($channel) {
            NotificationChannel::Email => $this->emailSender,
            NotificationChannel::SMS   => $this->smsSender,
            NotificationChannel::Push  => $this->pushSender,
        };
    }
}
```

### Strategy Pattern

```php
interface DiscountStrategy
{
    public function apply(Money $price, int $quantity): Money;
}

final class NoDiscount implements DiscountStrategy
{
    public function apply(Money $price, int $quantity): Money
    {
        return $price;
    }
}

final class PercentageDiscount implements DiscountStrategy
{
    public function __construct(private readonly float $percent) {}

    public function apply(Money $price, int $quantity): Money
    {
        $discount = (int) round($price->getAmount() * ($this->percent / 100));
        return Money::of($price->getAmount() - $discount, $price->getCurrency());
    }
}

final class BulkDiscount implements DiscountStrategy
{
    public function __construct(
        private readonly int $minQuantity,
        private readonly float $percent,
    ) {}

    public function apply(Money $price, int $quantity): Money
    {
        if ($quantity < $this->minQuantity) {
            return $price;
        }
        $discount = (int) round($price->getAmount() * ($this->percent / 100));
        return Money::of($price->getAmount() - $discount, $price->getCurrency());
    }
}
```

---

## Parte 4 — Código Limpo em PHP

### Typed Properties e Enums (PHP 8.1+)

```php
// ✅ Use enums em vez de constantes mágicas
enum OrderStatus: string
{
    case Pending   = 'pending';
    case Confirmed = 'confirmed';
    case Shipped   = 'shipped';
    case Delivered = 'delivered';
    case Cancelled = 'cancelled';

    public function canTransitionTo(self $next): bool
    {
        return match ($this) {
            self::Pending   => $next === self::Confirmed || $next === self::Cancelled,
            self::Confirmed => $next === self::Shipped   || $next === self::Cancelled,
            self::Shipped   => $next === self::Delivered,
            default         => false,
        };
    }
}

// ✅ Typed properties — sem "magic" strings/arrays
final class Order
{
    private OrderStatus $status = OrderStatus::Pending;
    private \DateTimeImmutable $createdAt;
    private ?string $cancelReason = null;

    public function cancel(string $reason): void
    {
        if (!$this->status->canTransitionTo(OrderStatus::Cancelled)) {
            throw new InvalidOrderTransitionException($this->status, OrderStatus::Cancelled);
        }
        $this->status = OrderStatus::Cancelled;
        $this->cancelReason = $reason;
    }
}
```

### Exceções de Domínio

```php
// src/Domain/Shared/Exception/DomainException.php
abstract class DomainException extends \RuntimeException {}

// Exceções específicas (não use \Exception genérica)
final class UserNotFoundException extends DomainException
{
    public function __construct(string $id)
    {
        parent::__construct("User not found: {$id}");
    }
}

final class InvalidEmailException extends DomainException
{
    public function __construct(string $email)
    {
        parent::__construct("Invalid email address: {$email}");
    }
}

// Mapeamento para HTTP no controller
final class ExceptionHandler
{
    public function handle(\Throwable $e): ResponseInterface
    {
        return match (true) {
            $e instanceof UserNotFoundException       => $this->json(['error' => $e->getMessage()], 404),
            $e instanceof DomainException            => $this->json(['error' => $e->getMessage()], 422),
            $e instanceof \InvalidArgumentException  => $this->json(['error' => $e->getMessage()], 400),
            default                                  => $this->json(['error' => 'Internal server error'], 500),
        };
    }
}
```

### Named Arguments e Nullsafe Operator (PHP 8.0+)

```php
// ✅ Named arguments — legibilidade em funções com muitos parâmetros
$order = Order::create(
    customerId: $input->customerId,
    items: $items,
    currency: 'BRL',
    discount: DiscountStrategy::percentage(10),
);

// ✅ Nullsafe operator — evita if aninhados
$city = $user?->getAddress()?->getCity()?->getName() ?? 'Unknown';

// ✅ match em vez de switch
$label = match ($order->getStatus()) {
    OrderStatus::Pending   => 'Aguardando confirmação',
    OrderStatus::Confirmed => 'Confirmado',
    OrderStatus::Shipped   => 'Enviado',
    OrderStatus::Delivered => 'Entregue',
    OrderStatus::Cancelled => 'Cancelado',
};
```

### Readonly Classes e Propriedades (PHP 8.2+)

```php
// ✅ DTOs como readonly — imutáveis por definição
final readonly class CreateUserInput
{
    public function __construct(
        public string $name,
        public string $email,
        public string $password,
    ) {}
}

// ✅ Value Objects como readonly
final readonly class Email
{
    public function __construct(public string $value)
    {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException($value);
        }
    }
}
```

---

## Parte 5 — Estrutura de Projeto

```
src/
├── Domain/
│   ├── Order/
│   │   ├── Order.php                  # Entidade
│   │   ├── OrderItem.php
│   │   ├── OrderRepository.php        # Interface (port)
│   │   ├── OrderStatus.php            # Enum
│   │   └── Event/
│   │       └── OrderCreatedEvent.php
│   └── Shared/
│       ├── Exception/
│       │   └── DomainException.php
│       └── ValueObject/
│           ├── Money.php
│           └── Email.php
├── Application/
│   └── UseCase/
│       └── CreateOrder/
│           ├── CreateOrderUseCase.php
│           ├── CreateOrderInput.php   # DTO (readonly)
│           └── CreateOrderOutput.php  # DTO (readonly)
├── Infrastructure/
│   ├── Persistence/
│   │   ├── DoctrineOrderRepository.php
│   │   └── migrations/
│   ├── Http/
│   │   ├── Controller/
│   │   │   └── OrderController.php
│   │   └── Middleware/
│   │       └── AuthMiddleware.php     # PSR-15
│   ├── Notification/
│   │   └── SmtpEmailSender.php
│   └── Container/
│       └── dependencies.php           # Composition Root (PHP-DI)
└── Shared/
    └── Http/
        └── ExceptionHandler.php
tests/
├── Unit/
│   └── Domain/
│       └── OrderTest.php
├── Integration/
│   └── Infrastructure/
│       └── DoctrineOrderRepositoryTest.php
└── Feature/
    └── Http/
        └── CreateOrderTest.php
```

---

## Parte 6 — Ferramentas de Qualidade

| Ferramenta | Propósito | Configuração |
|------------|-----------|-------------|
| `PHP_CodeSniffer` | Verifica PSR-12 | `phpcs.xml` |
| `PHP-CS-Fixer` | Corrige PSR-12 automaticamente | `.php-cs-fixer.php` |
| `PHPStan` | Análise estática (nível 8+) | `phpstan.neon` |
| `Psalm` | Análise estática com tipos | `psalm.xml` |
| `PHPUnit` | Testes unitários/integração | `phpunit.xml` |
| `Rector` | Upgrades automáticos de código | `rector.php` |

```bash
# Verificar PSR-12
./vendor/bin/phpcs --standard=PSR12 src/

# Corrigir automaticamente
./vendor/bin/php-cs-fixer fix src/

# Análise estática nível máximo
./vendor/bin/phpstan analyse src/ --level=9

# Rodar testes com cobertura
./vendor/bin/phpunit --coverage-html coverage/
```

---

## Anti-padrões a Evitar

| Anti-padrão | Problema | Solução |
|-------------|----------|---------|
| `new Dependency()` dentro de classe | Acoplamento rígido | Injete via construtor |
| `static` e métodos globais | Difícil de testar | Instâncias com DI |
| Arrays associativos como DTOs | Sem type-safety | Classes `readonly` |
| Lógica de negócio no Controller | Viola SRP | Mova para Use Case |
| Entidade com métodos `save()`/`delete()` | Active Record — viola SRP | Use Repository separado |
| `catch (\Exception $e) {}` vazio | Engole erros silenciosamente | Log + rethrow ou tratamento |
| `var_dump()` / `die()` em código | Debug não removido | Use logger PSR-3 |
| `mixed` ou sem tipos em PHP 8+ | Perde type-safety | Declare sempre os tipos |
| Constantes mágicas: `'status' => 'active'` | Propenso a typos | Use `enum` |

---

## Checklist PHP

- [ ] `declare(strict_types=1)` em todos os arquivos?
- [ ] Namespace e PSR-4 configurados no `composer.json`?
- [ ] Todas as dependências injetadas via construtor (sem `new` interno)?
- [ ] Interfaces definidas no lado do consumidor (Domain/Application)?
- [ ] DTOs usando `readonly` e tipos explícitos?
- [ ] Enums em vez de constantes string/int?
- [ ] Logger PSR-3 em vez de `error_log()` ou `var_dump()`?
- [ ] Exceções de domínio tipadas (não `\Exception` genérica)?
- [ ] PHPStan/Psalm no nível 8+ sem erros?
- [ ] PHP_CodeSniffer passando com PSR-12?

---

## Referências

- [PHP-FIG — PSRs](https://www.php-fig.org/psr/)
- [PHP The Right Way](https://phptherightway.com/)
- [PHPStan](https://phpstan.org/)
- [PHP-CS-Fixer](https://cs.symfony.com/)
- [Rector](https://getrector.com/)
- [Clean Code PHP — jupeter](https://github.com/jupeter/clean-code-php)
