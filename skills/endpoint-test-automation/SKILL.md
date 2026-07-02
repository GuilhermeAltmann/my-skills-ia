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
```

---

## 3. Script de Execução Independente (Alternativa ao Postman)

Para realizar testes rápidos de ponta a ponta sem frameworks de teste pesados (como Postman/Newman), utilize o template abaixo. Ele é um script Node.js puro (sem dependências externas) que consome uma lista de endpoints, faz as requisições HTTP reais e valida as respostas.

### Template: `test-endpoints.js`

```javascript
#!/usr/bin/env node

/**
 * Script independente para testar endpoints de API.
 * Execução: node test-endpoints.js [base_url]
 */

const BASE_URL = process.argv[2] || 'http://localhost:3000';
let AUTH_TOKEN = ''; // Será populado dinamicamente caso haja um passo de login

const testCases = [
  {
    name: '1. Autenticação (Login)',
    path: '/api/v1/auth/login',
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: {
      email: 'admin@example.com',
      password: 'password123'
    },
    expectedStatus: 200,
    // Callback para processar o retorno e guardar dados (ex: token) para passos seguintes
    after: (resBody) => {
      if (resBody.token) {
        AUTH_TOKEN = resBody.token;
        console.log('   🔑 Token de autorização capturado com sucesso.');
      }
    }
  },
  {
    name: '2. Criar Usuário (Sucesso)',
    path: '/api/v1/users',
    method: 'POST',
    // Função dinâmica para injetar headers (ex: token obtido no passo anterior)
    headers: () => ({
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${AUTH_TOKEN}`
    }),
    body: {
      name: 'Novo Agente',
      email: 'agente.ia@example.com'
    },
    expectedStatus: 201,
    validate: (resBody) => {
      if (!resBody.id) throw new Error("Resposta não contém o campo 'id'");
    }
  },
  {
    name: '3. Buscar Usuários (Lista)',
    path: '/api/v1/users',
    method: 'GET',
    headers: () => ({
      'Authorization': `Bearer ${AUTH_TOKEN}`
    }),
    expectedStatus: 200
  }
];

async function runTests() {
  console.log(`🚀 Iniciando testes de endpoints em: ${BASE_URL}\n`);
  let passed = 0;
  let failed = 0;

  for (const tc of testCases) {
    console.log(`👉 Rodando: ${tc.name}`);
    const url = `${BASE_URL}${tc.path}`;
    
    // Resolve headers dinâmicos se forem definidos como função
    const resolvedHeaders = typeof tc.headers === 'function' ? tc.headers() : (tc.headers || {});
    
    const options = {
      method: tc.method,
      headers: resolvedHeaders
    };

    if (tc.body) {
      options.body = JSON.stringify(tc.body);
    }

    try {
      const response = await fetch(url, options);
      const rawText = await response.text();
      let body = {};
      
      if (rawText) {
        try {
          body = JSON.parse(rawText);
        } catch {
          body = rawText;
        }
      }

      if (response.status !== tc.expectedStatus) {
        throw new Error(`Status code inválido. Esperado: ${tc.expectedStatus}, Recebido: ${response.status}. Resposta: ${JSON.stringify(body)}`);
      }

      if (tc.validate) {
        tc.validate(body);
      }

      if (tc.after) {
        tc.after(body);
      }

      console.log(`   ✅ PASS (Status ${response.status})\n`);
      passed++;
    } catch (error) {
      console.error(`   ❌ FAIL: ${error.message}\n`);
      failed++;
    }
  }

  console.log(`=================================`);
  console.log(`📊 RESULTADO FINAL:`);
  console.log(`   Sucessos: ${passed}`);
  console.log(`   Falhas:   ${failed}`);
  console.log(`=================================`);

  if (failed > 0) {
    process.exit(1);
  }
}

runTests();
```

---

## 4. Guia de Casos de Teste Essenciais (Checklist)

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
