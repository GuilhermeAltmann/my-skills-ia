---
name: pr-documentation
description: >
  Gera e valida documentação de Pull Requests de forma estruturada.
  Use esta skill ao abrir uma PR, revisar sua descrição ou padronizar
  o template de PR de um projeto. Cobre título, descrição, tipo de mudança,
  checklist, screenshots e instruções de teste.
---

# PR Documentation

Esta skill define o padrão para criar descrições de Pull Requests completas,
claras e rastreáveis, facilitando revisão, aprovação e rastreabilidade.

---

## Princípios

1. **Toda PR conta uma história** — o revisor não tem contexto do que você estava pensando.
2. **O "por quê" é mais importante que o "o quê"** — o diff mostra o que mudou; a PR explica a razão.
3. **PRs pequenas são melhores** — prefira PRs focadas com objetivo único.
4. **Inclua evidências** — screenshots, logs ou resultados de testes quando visual ou comportamental.

---

## Template Padrão de PR

```markdown
## Descrição

<!-- Explique o que esta PR faz e POR QUÊ essa mudança é necessária.
     Inclua contexto de negócio, técnico ou de UX. -->

## Tipo de mudança

<!-- Marque com [x] os tipos aplicáveis -->
- [ ] 🆕 Nova funcionalidade (`feat`)
- [ ] �� Correção de bug (`fix`)
- [ ] 🔥 Hotfix em produção (`hotfix`)
- [ ] ♻️ Refatoração sem mudança de comportamento (`refactor`)
- [ ] 📝 Documentação (`docs`)
- [ ] 🧪 Testes (`test`)
- [ ] 🔧 Configuração / build / CI (`chore`, `ci`, `build`)
- [ ] ⚡ Performance (`perf`)
- [ ] 💥 Breaking change

## Issue relacionada

<!-- Use "Closes #123" para fechar automaticamente a issue no merge -->
Closes #

## O que foi feito

<!-- Liste as principais mudanças implementadas (não o diff, mas as decisões) -->
-
-
-

## Como testar

<!-- Passo a passo para o revisor verificar a mudança localmente -->
1.
2.
3.

**Resultado esperado:**

## Screenshots / Vídeos

<!-- Se a mudança afeta a UI, inclua antes/depois. Remova se não aplicável. -->

| Antes | Depois |
|-------|--------|
| -     | -      |

## Checklist

- [ ] Código segue os padrões do projeto (linting, formatação)
- [ ] Testes foram adicionados/atualizados para cobrir as mudanças
- [ ] Todos os testes passam localmente (`npm test` / `go test ./...` etc.)
- [ ] Documentação atualizada (README, docstrings, comentários)
- [ ] Breaking changes estão documentados no footer do commit e nesta PR
- [ ] Variáveis de ambiente novas estão documentadas
- [ ] Migrações de banco de dados são reversíveis
- [ ] PR não contém arquivos de debug, `console.log` ou credenciais

## Notas para o revisor

<!-- Destaque partes específicas que precisam de atenção especial,
     decisões de design polêmicas ou áreas de incerteza. -->
```

---

## Regras do Título da PR

O título deve seguir o mesmo padrão do Conventional Commits:

```
<type>(<scope>): <descrição curta>
```

### Regras

- Máximo de **72 caracteres**.
- Usar o **mesmo tipo** do commit principal da PR.
- Sem ponto final.
- Imperativo: "add feature", não "added feature".
- Deve ser compreensível sem ler a descrição.

### Exemplos válidos

```
feat(auth): add JWT refresh token rotation
fix(api): handle null response from payment gateway
chore: upgrade all dependencies to latest versions
docs(readme): add Docker setup instructions
refactor(db)!: migrate from raw queries to Prisma ORM
```

### O `!` para breaking changes

Adicione `!` após o type/scope para indicar breaking change no título:

```
refactor(db)!: replace knex with Prisma ORM
feat(api)!: require API key for all public endpoints
```

---

## Tamanho Ideal de uma PR

| Linhas alteradas | Classificação | Recomendação                                           |
| ---------------- | ------------- | ------------------------------------------------------ |
| 1 — 100          | Pequena       | Ideal, revisão rápida e focada                         |
| 101 — 300        | Média         | Aceitável para features completas                      |
| 301 — 500        | Grande        | Justifique o tamanho ou divida                         |
| 500+             | Muito grande  | Divida em PRs menores sempre que possível              |

> **Exceções aceitas**: geração automática de código (migrations, snapshots, mocks), refatorações mecânicas de rename em massa.

---

## Workflow de Criação de PR

### Passo 1 — Antes de abrir

1. Rode os testes localmente e certifique-se que passam.
2. Verifique o diff com `git diff main...HEAD` ou `git diff develop...HEAD`.
3. Confirme que não há arquivos de debug, credenciais ou logs esquecidos.
4. Verifique se a branch segue o padrão da skill `git-conventions`.

### Passo 2 — Preencher a descrição

1. **Título**: seguir Conventional Commits (ver regras acima).
2. **Descrição**: explicar o contexto e a motivação, não o diff.
3. **Tipo de mudança**: marcar todos os aplicáveis.
4. **Issue**: linkar com `Closes #N` ou `Refs #N`.
5. **Como testar**: passo a passo reproduzível por qualquer pessoa do time.
6. **Screenshots**: obrigatório para mudanças de UI.
7. **Checklist**: revisar e marcar cada item honestamente.

### Passo 3 — Configurar a PR na plataforma

- **Assignee**: você mesmo.
- **Reviewers**: pelo menos 1 pessoa obrigatória; 2 para mudanças críticas.
- **Labels**: adicionar label correspondente ao tipo (`feature`, `bug`, `docs`...).
- **Milestone**: vincular à sprint ou versão correspondente.
- **Draft**: abra como Draft se ainda não está pronta para revisão.

### Passo 4 — Revisão dos revisores

- Aguarde aprovação antes de fazer merge.
- Responda **todos** os comentários antes de solicitar re-review.
- Use `Resolve conversation` apenas quando a sugestão foi implementada ou discutida.
- Prefira squash merge para manter o histórico limpo.

---

## Tipos de Merge

| Estratégia    | Quando usar                                                            |
| ------------- | ---------------------------------------------------------------------- |
| Squash merge  | Features e fixes — mantém 1 commit limpo no `develop`/`main`          |
| Merge commit  | Releases — preserva o histórico completo da release branch             |
| Rebase merge  | Hotfixes simples — aplica commits diretamente sem commit de merge      |

---

## Validação de uma PR existente

Ao revisar a documentação de uma PR, verifique:

1. **Título** segue Conventional Commits?
2. **Descrição** explica o "por quê" (não só o "o quê")?
3. **Issue** está referenciada?
4. **Como testar** está claro e reproduzível?
5. **Screenshots** incluídas se há mudança visual?
6. **Checklist** completamente preenchida?
7. **Tamanho** é adequado (< 500 linhas)?
8. **Breaking changes** estão documentadas?

---

## Checklist de Abertura

- [ ] Título segue Conventional Commits?
- [ ] Descrição explica o "por quê"?
- [ ] Issue está vinculada (`Closes #N`)?
- [ ] Tipo de mudança está marcado?
- [ ] Passo a passo de como testar está claro?
- [ ] Screenshots incluídas para mudanças de UI?
- [ ] Todos os itens do checklist foram revisados?
- [ ] Reviewer(s) foram adicionados?
- [ ] Labels e milestone configurados?
- [ ] Testes passam localmente?

---

## Referências

- [GitHub Pull Request Best Practices](https://docs.github.com/en/pull-requests)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [How to Write a Great PR Description](https://www.pullrequest.com/blog/writing-a-great-pull-request-description/)
