---
name: git-conventions
description: >
  Padroniza mensagens de commit e nomes de branch seguindo convenções estabelecidas
  (Conventional Commits e GitFlow). Use esta skill sempre que precisar criar um commit,
  nomear uma branch ou revisar se mensagens/branches estão em conformidade com o padrão.
---

# Git Conventions — Commit Messages & Branch Names

Esta skill define e aplica os padrões de mensagens de commit e nomenclatura de branches
para garantir um histórico Git limpo, legível e rastreável.

---

## Padrão de Mensagem de Commit

Seguimos a especificação [Conventional Commits](https://www.conventionalcommits.org/).

### Formato

```
<type>(<scope>): <subject>

[body opcional]

[footer(s) opcional(is)]
```

### Tipos (`type`) permitidos

| Tipo       | Uso                                                               |
| ---------- | ----------------------------------------------------------------- |
| `feat`     | Nova funcionalidade para o usuário                                |
| `fix`      | Correção de bug para o usuário                                    |
| `docs`     | Alterações apenas em documentação                                 |
| `style`    | Formatação, ponto e vírgula faltando etc. (sem mudança de lógica) |
| `refactor` | Refatoração de código de produção (sem feat nem fix)              |
| `test`     | Adição ou correção de testes (sem mudança de código de produção)  |
| `chore`    | Atualização de build, configs, dependências etc.                  |
| `perf`     | Melhoria de performance                                           |
| `ci`       | Mudanças em arquivos e scripts de CI/CD                           |
| `revert`   | Reverte um commit anterior                                        |
| `build`    | Mudanças que afetam o sistema de build ou dependências externas   |

### Regras do `<scope>` (opcional)

- Identifica **qual módulo/componente** foi alterado (ex: `auth`, `api`, `ui`, `db`).
- Sempre em **minúsculas** e **sem espaços** (use `-` se necessário).
- Omita o scope quando a mudança for transversal ou difícil de categorizar.

### Regras do `<subject>`

- Máximo de **72 caracteres**.
- Escrito no **modo imperativo** (ex: "add feature" e não "added feature").
- **Primeira letra minúscula** (a menos que seja um nome próprio).
- **Sem ponto final**.
- Em **inglês** (padrão internacional) ou **português** — escolha um e mantenha consistência no projeto.

### Body (opcional)

- Separado do subject por **uma linha em branco**.
- Explica o **"por quê"** da mudança, não o "o quê" (o diff já mostra isso).
- Máximo de **72 caracteres por linha**.

### Footer (opcional)

- Para referências a issues: `Closes #123`, `Fixes #456`, `Refs #789`.
- Para breaking changes: `BREAKING CHANGE: <descrição>`.

### Exemplos válidos

```
feat(auth): add JWT refresh token support

Implementa renovação automática de tokens expirados para melhorar
a experiência do usuário sem exigir novo login a cada hora.

Closes #42
```

```
fix(api): handle null response from payment gateway
```

```
chore: upgrade dependencies to latest versions
```

```
docs(readme): update installation instructions
```

```
refactor(db): extract query builder into separate module

BREAKING CHANGE: QueryBuilder now requires explicit connection parameter
```

### Exemplos inválidos

```
# Sem tipo
updated login page

# Tipo inválido
update(auth): fix login bug

# Subject muito longo (> 72 chars)
feat(ui): add a very long description that exceeds the maximum allowed character limit per line

# Modo imperativo errado
fix(api): fixed the null pointer exception

# Com ponto final
feat(auth): add refresh token.
```

---

## Padrão de Nomenclatura de Branch

Seguimos o modelo **GitFlow** adaptado.

### Formato

```
<categoria>/<identificador-curto>
```

ou com rastreamento de issue:

```
<categoria>/<numero-issue>-<identificador-curto>
```

### Categorias permitidas

| Categoria    | Uso                                              | Exemplo                           |
| ------------ | ------------------------------------------------ | --------------------------------- |
| `feature`    | Nova funcionalidade                              | `feature/user-authentication`     |
| `fix`        | Correção de bug (em desenvolvimento)             | `fix/login-redirect-loop`         |
| `hotfix`     | Correção urgente em produção                     | `hotfix/payment-gateway-timeout`  |
| `release`    | Preparação de versão                             | `release/2.1.0`                   |
| `chore`      | Tarefas de manutenção, dependências, configs     | `chore/update-eslint-config`      |
| `docs`       | Apenas documentação                              | `docs/api-endpoints`              |
| `refactor`   | Refatoração sem mudança de comportamento         | `refactor/auth-service`           |
| `test`       | Adição ou melhoria de testes                     | `test/payment-unit-tests`         |
| `ci`         | Mudanças em pipelines de CI/CD                   | `ci/add-github-actions`           |
| `experiment` | Experimentos ou provas de conceito               | `experiment/new-caching-strategy` |

### Branches protegidas (não criar nem deletar)

| Branch    | Propósito                                  |
| --------- | ------------------------------------------ |
| `main`    | Código de produção estável                 |
| `develop` | Branch de integração (próxima release)     |

### Regras do identificador curto

- Máximo de **50 caracteres** no total (incluindo `categoria/`).
- Apenas **letras minúsculas**, **números** e **hífens** (`-`).
- **Sem espaços**, underscores (`_`), pontos (`.`) ou caracteres especiais.
- **Sem acentos** ou caracteres não-ASCII.
- Descritivo: deve deixar claro o propósito da branch ao ler o nome.

### Exemplos válidos

```
feature/user-authentication
feature/42-user-authentication
fix/login-redirect-loop
hotfix/payment-timeout
release/2.1.0
chore/upgrade-react-18
docs/contributing-guide
refactor/extract-auth-service
test/add-payment-integration-tests
ci/setup-github-actions
```

### Exemplos inválidos

```
# Sem categoria
user-authentication

# Underscore em vez de hífen
feature/user_authentication

# Letras maiúsculas
Feature/UserAuthentication

# Muito vago
fix/bug

# Acentos
feature/autenticacao-usuario
```

---

## Workflow de Aplicação desta Skill

### Ao criar um commit

1. Analise as mudanças com `git diff --staged` para entender o escopo.
2. Determine o tipo correto baseado nas alterações.
3. Identifique o scope (opcional) pelo módulo afetado.
4. Escreva o subject no imperativo, em minúsculas, sem ponto final.
5. Adicione body/footer se a mudança for complexa ou quebrar compatibilidade.
6. Valide contra as regras antes de confirmar.

### Ao criar uma branch

1. Identifique a categoria da tarefa que será realizada.
2. Verifique se há número de issue para incluir no nome.
3. Crie um identificador curto descritivo, em kebab-case.
4. Valide o nome completo: sem maiúsculas, sem underscores, sem acentos.
5. Crie a branch a partir da base correta (`develop` para features/fixes, `main` para hotfixes).

### Ao revisar commits/branches existentes

1. Liste os commits ou branches para análise.
2. Verifique cada item contra os padrões desta skill.
3. Reporte os que não estão em conformidade com sugestão de correção.
4. Não altere histórico de commits sem confirmação explícita do usuário.

---

## Validação Rápida (Checklist)

### Commit

- [ ] Tipo é um dos permitidos?
- [ ] Scope em minúsculas e sem espaços?
- [ ] Subject menor ou igual a 72 chars, imperativo, sem ponto final?
- [ ] Body separado por linha em branco?
- [ ] Breaking changes documentados no footer?
- [ ] Issue referenciada no footer?

### Branch

- [ ] Começa com categoria válida?
- [ ] Usa apenas letras minúsculas, números e hífens?
- [ ] Nome total menor ou igual a 50 chars?
- [ ] Descritivo o suficiente?
- [ ] Criada a partir da base correta?

---

## Referências

- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [GitFlow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [Semantic Versioning](https://semver.org/)
