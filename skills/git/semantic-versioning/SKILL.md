---
name: semantic-versioning
description: >
  Aplica o versionamento semântico (SemVer) de forma correta e consistente.
  Use esta skill ao criar uma nova release, determinar qual versão incrementar,
  gerenciar CHANGELOG, tags Git e manifests de versão (package.json, pyproject.toml etc.).
---

# Semantic Versioning

Esta skill define como aplicar o versionamento semântico (SemVer 2.0.0) em projetos,
incluindo quando incrementar cada segmento, como gerar CHANGELOG e como criar releases.

---

## Formato da Versão

```
MAJOR.MINOR.PATCH[-pre-release][+build-metadata]
```

Exemplos:
```
1.0.0
2.3.1
1.0.0-alpha.1
1.0.0-beta.2
1.0.0-rc.1
2.0.0+20240101.sha.abc1234
```

---

## Regras de Incremento

### PATCH — `x.x.N`

Incremente quando houver **correções de bug retrocompatíveis**.

Disparadores:
- Correção de bug que não altera a API pública
- Correção de typo em mensagens de erro
- Correção de comportamento inesperado sem breaking change
- Melhoria de performance interna sem mudança de interface

```
1.2.3 → 1.2.4
```

### MINOR — `x.N.0`

Incremente quando houver **nova funcionalidade retrocompatível**. Reseta PATCH para 0.

Disparadores:
- Nova função, endpoint ou opção de configuração
- Deprecação de funcionalidade (aviso, não remoção)
- Melhoria substancial de funcionalidade existente sem quebrar compatibilidade
- Nova dependência opcional

```
1.2.3 → 1.3.0
```

### MAJOR — `N.0.0`

Incremente quando houver **mudança incompatível com versões anteriores**. Reseta MINOR e PATCH para 0.

Disparadores (Breaking Changes):
- Remoção de função, endpoint ou campo público
- Mudança de assinatura de função (parâmetros obrigatórios, tipo de retorno)
- Mudança de comportamento existente que quebra contratos
- Remoção de suporte a versão de runtime/plataforma
- Renomeação de identificadores públicos sem alias de compatibilidade

```
1.2.3 → 2.0.0
```

---

## Versões Pré-release

Use para versões instáveis antes de uma release oficial:

| Sufixo         | Significado                               | Exemplo          |
| -------------- | ----------------------------------------- | ---------------- |
| `-alpha.N`     | Funcionalidade incompleta, API pode mudar | `2.0.0-alpha.1`  |
| `-beta.N`      | Funcionalidade completa, em teste         | `2.0.0-beta.3`   |
| `-rc.N`        | Release Candidate — pronto p/ release     | `2.0.0-rc.1`     |

Ordem de precedência: `alpha < beta < rc < release`

```
2.0.0-alpha.1 < 2.0.0-alpha.2 < 2.0.0-beta.1 < 2.0.0-rc.1 < 2.0.0
```

---

## Versão 0.x.x — Desenvolvimento Inicial

Enquanto `MAJOR = 0`, a API é considerada instável:
- `0.x.y`: API pública não está estável.
- `0.1.0`: primeira release funcional.
- `0.x.0`: qualquer mudança em MINOR pode ser breaking change.
- `1.0.0`: primeira versão estável e pronta para produção.

---

## Mapeamento: Conventional Commits → SemVer

| Tipo de commit          | Versão a incrementar |
| ----------------------- | -------------------- |
| `fix:`                  | PATCH                |
| `feat:`                 | MINOR                |
| `BREAKING CHANGE:` no footer ou `!` no tipo | MAJOR |
| `docs:`, `style:`, `chore:`, `test:`, `ci:` | Nenhuma (não gera release) |
| `perf:`                 | PATCH                |
| `refactor:`             | PATCH (sem breaking) ou MAJOR (com breaking) |

---

## CHANGELOG

### Formato (Keep a Changelog)

```markdown
# Changelog

Todas as mudanças notáveis deste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/),
e este projeto adere ao [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
-

### Changed
-

### Deprecated
-

### Removed
-

### Fixed
-

### Security
-

---

## [1.2.0] — 2024-03-15

### Added
- Suporte a autenticação via OAuth 2.0 (#42)
- Endpoint `GET /users/:id/permissions` para listar permissões por usuário (#45)

### Fixed
- Correção de race condition no refresh de tokens (#48)
- Resposta incorreta ao passar `null` como `userId` (#49)

### Security
- Atualização do `jsonwebtoken` para 9.0.2 (CVE-2022-23529)

---

## [1.1.0] — 2024-02-01

### Added
- Cache de sessão com Redis (#30)

### Changed
- Tempo de expiração do token aumentado de 1h para 8h (#32)

### Deprecated
- `GET /session/info` — use `GET /auth/session` a partir da v1.2.0

---

## [1.0.0] — 2024-01-10

### Added
- Autenticação JWT
- Endpoints de login e logout
- Middleware de autorização por roles

[Unreleased]: https://github.com/org/repo/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/org/repo/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/org/repo/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/org/repo/releases/tag/v1.0.0
```

### Seções do CHANGELOG

| Seção        | O que incluir                                           |
| ------------ | ------------------------------------------------------- |
| `Added`      | Novas funcionalidades                                   |
| `Changed`    | Mudanças em funcionalidades existentes (retrocompat.)   |
| `Deprecated` | Funcionalidades que serão removidas em versões futuras  |
| `Removed`    | Funcionalidades removidas nesta versão                  |
| `Fixed`      | Correções de bugs                                       |
| `Security`   | Correções de vulnerabilidades (mencionar CVE se houver) |

---

## Workflow de Release

### Passo 1 — Determinar a nova versão

1. Liste todos os commits desde a última tag:
   ```bash
   git log v$(cat VERSION)..HEAD --oneline
   ```
2. Identifique o tipo de mudança mais significativa:
   - Há `BREAKING CHANGE` ou `!`? → MAJOR
   - Há `feat:`? → MINOR
   - Só há `fix:` e outros? → PATCH

### Passo 2 — Atualizar manifests de versão

Atualize a versão em todos os arquivos relevantes:

```bash
# Node.js
npm version <major|minor|patch> --no-git-tag-version

# Python (pyproject.toml)
# Edite manualmente a campo version = "x.y.z"

# Outros: VERSION file, pom.xml, build.gradle, Cargo.toml
```

### Passo 3 — Atualizar o CHANGELOG

1. Mova as entradas de `[Unreleased]` para a nova versão com a data de hoje.
2. Adicione os links de comparação no rodapé.
3. Crie uma seção `[Unreleased]` vazia para o próximo ciclo.

### Passo 4 — Commit de release

```bash
git add CHANGELOG.md package.json  # e demais manifests
git commit -m "chore(release): bump version to v2.1.0"
```

### Passo 5 — Tag Git

```bash
# Tag anotada (recomendado — inclui mensagem e assinatura)
git tag -a v2.1.0 -m "Release v2.1.0"

# Enviar tag ao remoto
git push origin v2.1.0
```

### Passo 6 — Criar a release na plataforma

No GitHub/GitLab, crie uma Release a partir da tag com:
- **Título**: `v2.1.0`
- **Descrição**: copiar o conteúdo do CHANGELOG para esta versão.
- **Assets**: binários, arquivos compilados se aplicável.

---

## Verificação de Versão Atual

```bash
# Ver a última tag de release
git describe --tags --abbrev=0

# Ver todas as tags ordenadas
git tag --sort=-v:refname | head -20

# Ver commits desde a última release
git log $(git describe --tags --abbrev=0)..HEAD --oneline
```

---

## Arquivos de Versão por Ecossistema

| Ecossistema | Arquivo(s)                          | Campo            |
| ----------- | ----------------------------------- | ---------------- |
| Node.js     | `package.json`                      | `"version"`      |
| Python      | `pyproject.toml`, `setup.cfg`       | `version`        |
| Rust        | `Cargo.toml`                        | `version`        |
| Java/Maven  | `pom.xml`                           | `<version>`      |
| Go          | `go.mod` + tag Git                  | tag Git          |
| .NET        | `.csproj`                           | `<Version>`      |
| Ruby        | `*.gemspec`                         | `spec.version`   |
| Genérico    | `VERSION` (arquivo plano)           | conteúdo bruto   |

---

## Checklist de Release

- [ ] Todos os commits desde a última release foram revisados?
- [ ] O incremento de versão está correto (MAJOR/MINOR/PATCH)?
- [ ] Todos os manifests de versão foram atualizados?
- [ ] O CHANGELOG foi atualizado com data e links?
- [ ] O commit de release foi criado com a mensagem correta?
- [ ] A tag Git foi criada como tag anotada (`-a`)?
- [ ] A tag foi enviada ao remoto (`git push origin vX.Y.Z`)?
- [ ] A Release foi criada na plataforma (GitHub/GitLab)?
- [ ] Breaking changes foram comunicados aos consumidores?
- [ ] Dependências foram notificadas se aplicável?

---

## Referências

- [Semantic Versioning 2.0.0](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/)
- [Conventional Commits + SemVer](https://www.conventionalcommits.org/#how-does-this-relate-to-semver)
