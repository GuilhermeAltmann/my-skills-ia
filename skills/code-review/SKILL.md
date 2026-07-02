---
name: code-review
description: >
  Realiza revisão de código estruturada e objetiva. Use esta skill ao revisar
  Pull Requests, diffs ou arquivos individuais. Avalia qualidade, segurança,
  performance, legibilidade e aderência a boas práticas, emitindo feedback
  categorizando por severidade (blocker, major, minor, nit).
---

# Code Review

Esta skill define como conduzir revisões de código de forma consistente,
imparcial e construtiva, cobrindo desde segurança até legibilidade.

---

## Princípios de Revisão

1. **Revise o código, não a pessoa** — feedback sobre a implementação, nunca sobre o autor.
2. **Seja específico** — indique arquivo, linha e o motivo exato do comentário.
3. **Proponha soluções** — sempre que apontar um problema, sugira uma alternativa.
4. **Reconheça o bom** — elogie boas decisões de design e soluções criativas.
5. **Priorize blockers** — não misture problemas críticos com preferências estéticas.

---

## Categorias de Feedback

| Categoria  | Símbolo | Significado                                                         | Bloqueia merge? |
| ---------- | ------- | ------------------------------------------------------------------- | --------------- |
| `BLOCKER`  | 🔴      | Bug crítico, vulnerabilidade de segurança ou quebra de contrato     | Sim             |
| `MAJOR`    | 🟠      | Problema de design, performance ou manutenibilidade significativos  | Sim             |
| `MINOR`    | 🟡      | Melhoria importante mas não crítica                                 | Recomendado     |
| `NIT`      | 🔵      | Preferência estilística, formatação, nomenclatura                   | Não             |
| `PRAISE`   | ��      | Reconhecimento de boa prática ou solução elegante                   | —               |
| `QUESTION` | ❓      | Dúvida genuína sobre a intenção do código                           | Não             |

---

## Áreas de Avaliação

### 1. Corretude e Lógica

- O código faz o que a tarefa/issue pede?
- Existem edge cases não tratados (null, vazio, overflow, concorrência)?
- Há condições de corrida (race conditions) em código assíncrono?
- Os retornos de erro são verificados e tratados adequadamente?

### 2. Segurança

- Há inputs do usuário sendo usados sem sanitização/validação?
- Existem segredos, tokens ou credenciais hardcoded?
- Queries ao banco de dados estão usando prepared statements / ORM seguro?
- Dados sensíveis estão sendo logados ou expostos?
- A autenticação/autorização está sendo verificada corretamente?
- Há possibilidade de path traversal, XSS, SSRF ou injeção?

### 3. Performance

- Há consultas N+1 ao banco de dados?
- Loops desnecessários ou complexidade algorítmica elevada?
- Recursos (conexões, arquivos, streams) são fechados corretamente?
- Caching está sendo usado onde apropriado?
- Operações pesadas estão bloqueando a thread principal?

### 4. Design e Arquitetura

- O código segue os padrões estabelecidos no projeto?
- Há violação de SOLID, DRY ou outros princípios aplicáveis?
- A responsabilidade dos módulos/funções está clara e delimitada?
- Existe acoplamento desnecessário entre componentes?
- A mudança introduz débito técnico injustificado?

### 5. Legibilidade e Manutenibilidade

- Nomes de variáveis, funções e classes são descritivos e consistentes?
- Funções com mais de 30 linhas que poderiam ser divididas?
- Comentários explicam o "por quê", não o "o quê"?
- Há código morto (dead code), comentários desatualizados ou TODOs antigos?
- A indentação e formatação estão consistentes com o restante do projeto?

### 6. Testes

- A mudança possui cobertura de testes adequada?
- Testes cobrem casos felizes E casos de erro?
- Os testes são determinísticos (sem dependência de tempo, ordem ou rede)?
- Há mocks/stubs onde deveriam haver integrações reais (ou vice-versa)?

### 7. Documentação

- Funções públicas / APIs possuem docstrings/JSDoc atualizados?
- O README ou docs foram atualizados se comportamento externo mudou?
- Migrations, variáveis de ambiente ou configurações novas estão documentadas?

---

## Formato do Feedback

### Estrutura de um comentário

```
[CATEGORIA] arquivo.ext:linha — Descrição do problema

Contexto: por que isso é um problema.

Sugestão:
```suggestion
// código sugerido aqui
```
```

### Exemplo de revisão completa

```markdown
## Code Review — PR #42: Add user authentication

**Resumo**: Implementação sólida no geral. Dois blockers de segurança precisam
ser resolvidos antes do merge. Alguns nitpicks de legibilidade.

---

🔴 **BLOCKER** `src/auth/login.js:47` — Senha comparada com `==` em vez de comparação em tempo constante

Comparações diretas de string são vulneráveis a timing attacks.

Sugestão: use `crypto.timingSafeEqual()` ou a função utilitária `safeCompare()` já existente em `src/utils/crypto.js`.

---

🔴 **BLOCKER** `src/auth/login.js:82` — Token JWT hardcoded no fallback

`const secret = process.env.JWT_SECRET || 'dev-secret-123'` expõe um
segredo previsível em produção se a variável de ambiente não estiver definida.

Sugestão: lance um erro na inicialização se `JWT_SECRET` não estiver definido.

---

🟠 **MAJOR** `src/auth/middleware.js:23` — Consulta N+1 ao buscar permissões

Para cada request autenticado, há uma query por permissão do usuário.
Com 10 permissões, são 10 queries por request.

Sugestão: carregue todas as permissões em uma única query com `WHERE user_id = $1`.

---

🟡 **MINOR** `src/auth/login.js:15` — Erro genérico expõe informação sobre existência do usuário

Retornar "Usuário não encontrado" vs "Senha incorreta" permite enumerar usuários.

Sugestão: retorne sempre "Credenciais inválidas" independente do motivo.

---

🔵 **NIT** `src/auth/login.js:5` — Importação não utilizada

`import { deprecated } from './legacy'` não é usado neste arquivo.

---

🟢 **PRAISE** `src/auth/logout.js:12` — Excelente uso de refresh token rotation

Invalidar o refresh token anterior ao emitir um novo previne reutilização
em caso de vazamento. Ótima escolha de segurança.

---

❓ **QUESTION** `src/auth/middleware.js:41` — Por que o cache de sessão tem TTL de 5 minutos?

Há um requisito específico de negócio para esse valor? Pergunto pois pode
causar sessões "zumbi" por até 5 minutos após logout.

---

**Resultado**: 🔴 Não aprovado — 2 blockers a resolver.
```

---

## Workflow de Revisão

### Passo 1 — Entender o contexto

1. Leia a descrição da PR/issue vinculada.
2. Identifique o objetivo da mudança.
3. Verifique o tamanho: PRs com +500 linhas devem ser divididas.

### Passo 2 — Revisão por área

Percorra o diff avaliando cada área (Corretude → Segurança → Performance → Design → Legibilidade → Testes → Docs).

### Passo 3 — Consolidar e priorizar

1. Agrupe feedbacks por categoria.
2. Inicie sempre pelos blockers.
3. Inclua um resumo executivo no início da revisão.

### Passo 4 — Emitir resultado

| Resultado           | Condição                                     |
| ------------------- | -------------------------------------------- |
| ✅ Aprovado         | Zero blockers e majors                       |
| 🔄 Aprovado c/ nits | Zero blockers, majors menores pontuados      |
| 🟠 Alterações necessárias | Há majors a resolver                   |
| 🔴 Não aprovado     | Há um ou mais blockers                       |

---

## Checklist Rápido

- [ ] Li a descrição da PR e entendi o objetivo?
- [ ] Verifiquei corretude e edge cases?
- [ ] Revisei segurança (inputs, segredos, auth, queries)?
- [ ] Avaliei performance (N+1, complexidade, recursos)?
- [ ] Verifiquei design e aderência aos padrões do projeto?
- [ ] Avaliei legibilidade e nomenclatura?
- [ ] Verifiquei cobertura e qualidade dos testes?
- [ ] Documentação foi atualizada se necessário?
- [ ] Todos os comentários têm sugestão ou pergunta clara?
- [ ] Emiti resultado com resumo executivo?
