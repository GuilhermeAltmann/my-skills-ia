---
name: endpoint-test-automation
description: >
  Define padrões e automatiza a criação de testes de endpoints (API integration/e2e tests).
  Use esta skill para criar, estruturar ou validar suítes de testes de endpoints, garantindo
  que as requisições, métodos HTTP, URLs base, payloads JSON e retornos esperados estejam corretos.
---

# Endpoint Test Automation

Esta skill padroniza a criação de testes automatizados de integração/E2E para endpoints de API, garantindo robustez na validação de payloads, contratos HTTP, status codes e comportamento sob falha.

---

## 1. Especificação Padrão do Endpoint

Sempre que documentar ou planejar um teste de endpoint, utilize a seguinte especificação:

```markdown
### [Nome Amigável do Caso de Teste]

- **Funcionamento**: [O que o endpoint faz, regras de negócio principais]
- **Método HTTP**: `GET` | `POST` | `PUT` | `DELETE` | `PATCH`
- **URL Base**: `{{base_url}}/api/v1/...`
- **Headers Requeridos**:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
- **Request JSON**:
```json
{
  "campo": "valor"
}
```
- **Retorno Esperado**:
  - **Status Code**: `200 OK` | `201 Created` | `400 Bad Request` | etc.
  - **Response JSON**:
```json
{
  "sucesso": true,
  "id": "123"
}
```
```

---

## 2. Estratégias de Automação por Tecnologia

A automação deve validar:
1. **Contrato**: Estrutura e tipos de dados do JSON de retorno (ex. Zod, JSON Schema).
2. **Status HTTP**: O código HTTP apropriado.
3. **Efeito colateral**: Se um registro foi criado ou modificado no banco (se aplicável).
4. **Resiliência (Edge Cases)**: Payload corrompido, tipos errados, campos ausentes e tokens inválidos.

---

### Exemplo 1: JavaScript / TypeScript (Jest + Supertest)

```typescript
import request from 'supertest';
import { app } from '../src/app'; // Seu express/fastify app

describe('POST /api/v1/users', () => {
  const baseUrl = '/api/v1/users';

  it('deve criar um novo usuário com payload válido', async () => {
    const payload = {
      name: 'John Doe',
      email: 'john.doe@example.com',
      password: 'StrongPassword123!'
    };

    const response = await request(app)
      .post(baseUrl)
      .set('Content-Type', 'application/json')
      .send(payload);

    // Assert Status
    expect(response.status).toBe(201);

    // Assert Contract
    expect(response.body).toEqual(
      expect.objectContaining({
        id: expect.any(String),
        email: 'john.doe@example.com',
        createdAt: expect.any(String)
      })
    );
  });

  it('deve retornar 400 Bad Request ao enviar e-mail inválido', async () => {
    const payload = {
      name: 'John Doe',
      email: 'invalid-email',
      password: 'StrongPassword123!'
    };

    const response = await request(app)
      .post(baseUrl)
      .send(payload);

    expect(response.status).toBe(400);
    expect(response.body.error).toBeDefined();
  });
});
```

---

### Exemplo 2: Go (net/http/httptest)

```go
package integration_test

import (
"bytes"
"encoding/json"
"net/http"
"net/http/httptest"
"testing"

"github.com/stretchr/testify/assert"
)

func TestCreateUserEndpoint(t *testing.T) {
router := SetupRouter() // Inicializa suas rotas e dependências mockadas/test-db

t.Run("deve criar usuário com sucesso", func(t *testing.T) {
payload := map[string]string{
"name":  "Jane Doe",
"email": "jane@example.com",
}
jsonPayload, _ := json.Marshal(payload)

req, _ := http.NewRequest("POST", "/api/v1/users", bytes.NewBuffer(jsonPayload))
req.Header.Set("Content-Type", "application/json")

w := httptest.NewRecorder()
router.ServeHTTP(w, req)

// Assert Status Code
assert.Equal(t, http.StatusCreated, w.Code)

// Assert Response JSON
var response map[string]interface{}
err := json.Unmarshal(w.Body.Bytes(), &response)
assert.NoError(t, err)
assert.NotEmpty(t, response["id"])
assert.Equal(t, "jane@example.com", response["email"])
})
}
```

---

### Exemplo 3: PHP (PHPUnit + Laravel HTTP Client ou Symfony Client)

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class UserEndpointTest extends TestCase
{
    use RefreshDatabase; // Reseta o banco de dados de teste

    private string $baseUrl = '/api/v1/users';

    public function test_should_create_user_successfully(): void
    {
        $payload = [
            'name' => 'Alice Smith',
            'email' => 'alice@example.com',
        ];

        $response = $this->postJson($this->baseUrl, $payload);

        // Assert status
        $response->assertStatus(201);

        // Assert JSON structure and values
        $response->assertJsonStructure([
            'id',
            'name',
            'email',
            'created_at'
        ]);

        $response->assertJsonFragment([
            'email' => 'alice@example.com'
        ]);
    }
}
```

---

## 3. Guia de Casos de Teste Essenciais (Checklist)

Sempre garanta que os seguintes cenários sejam cobertos ao automatizar:

### Caso Feliz
- [ ] Cenário com todos os dados obrigatórios e opcionais válidos.
- [ ] Validação de formatos específicos de data, id (ex: UUID) e status.

### Casos de Erro de Validação (400 Bad Request)
- [ ] Envio de JSON malformado (ex: sem fechar chaves).
- [ ] Envio de payload vazio `{}`.
- [ ] Omissão de campos obrigatórios.
- [ ] Valores fora do range permitido (ex: strings vazias, números negativos, formato de e-mail inválido).

### Casos de Autenticação e Autorização
- [ ] Requisição sem token (esperado `401 Unauthorized`).
- [ ] Requisição com token expirado ou inválido (esperado `401 Unauthorized`).
- [ ] Requisição de usuário sem permissão suficiente para o recurso (esperado `403 Forbidden`).

### Casos de Estado do Recurso
- [ ] Buscar ID que não existe (esperado `404 Not Found`).
- [ ] Tentar criar recurso duplicado se houver restrição única (esperado `409 Conflict`).
