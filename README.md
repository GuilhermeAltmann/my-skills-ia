# My AI Agent Skills

Coleção de skills personalizadas para agentes de IA (compatível com Antigravity IDE).

## Estrutura

```
my-skills/
└── skills/
    ├── git-devops/                 # DevOps e Controle de Versão
    │   ├── git-conventions/        # Padronização de commits e branches
    │   ├── pr-documentation/       # Documentação de Pull Requests
    │   └── semantic-versioning/    # Versionamento semântico (SemVer)
    ├── architecture/               # Padrões Arquiteturais e Design Patterns
    │   ├── js-ts-clean-architecture/
    │   ├── js-ts-dependency-injection/
    │   ├── go-architecture-patterns/
    │   └── php-architecture-patterns/
    ├── testing/                    # Testes e Qualidade de Código
    │   ├── code-review/
    │   └── endpoint-test-automation/
    └── agent-workflows/            # Otimização e Fluxos de Agentes de IA
        ├── brainstorming/
        ├── writing-plans/
        ├── token-reduction/
        └── task-finalization/      # Limpeza, documentação e entrega de tarefas estruturadas
```

## Skills disponíveis

### 📁 Git & DevOps (`git-devops/`)

| Skill | Gatilho | Descrição resumida |
| :--- | :--- | :--- |
| `git-conventions` | Criar commit, nomear branch, revisar convenções Git | Conventional Commits + GitFlow |
| `pr-documentation` | Abrir PR, escrever descrição, validar template de PR | Template completo com checklist, tipo, evidências e como testar |
| `semantic-versioning` | Criar release, definir versão, atualizar CHANGELOG | SemVer 2.0.0 com CHANGELOG, tags Git e manifests |

### 📁 Arquitetura & Design (`architecture/`)

| Skill | Gatilho | Descrição resumida |
| :--- | :--- | :--- |
| `js-ts-clean-architecture` | Estruturar projeto, decidir onde uma classe vive, revisar camadas | Domain / Application / Infrastructure / Presentation + regras |
| `js-ts-dependency-injection` | Criar classes com dependências, configurar container, escrever mocks | DI manual, Factories, TSyringe, InversifyJS, testes unitários |
| `go-architecture-patterns` | Estruturar projeto Go, criar repositórios, configurar structs | Hexagonal, Repository SQL, DI manual/Wire, Functional Options |
| `php-architecture-patterns` | Revisar PSRs, estruturar projeto PHP, aplicar SOLID e clean code | PSRs 1/3/4/7/12/14/15, SOLID, Repository, Value Objects, Enums |

### 📁 Qualidade & Testes (`testing/`)

| Skill | Gatilho | Descrição resumida |
| :--- | :--- | :--- |
| `code-review` | Revisar PR, diff ou arquivo; avaliar qualidade de código | Revisão por severidade: BLOCKER, MAJOR, MINOR, NIT |
| `endpoint-test-automation` | Criar, validar ou estruturar testes de endpoint (integração/e2e) | Métodos, URL base, Request JSON, status/response esperado e script independente |

### 📁 Fluxos & Otimizações do Agente (`agent-workflows/`)

| Skill | Gatilho | Descrição resumida |
| :--- | :--- | :--- |
| `brainstorming` | Antes de iniciar qualquer trabalho criativo ou mudança de lógica | Exploração de escopo, perguntas clarificadoras, propor 2-3 designs |
| `writing-plans` | Ter requisitos/specs definidos de uma tarefa de múltiplos passos | Planos de implementação estruturados em micro-tarefas (TDD/commits) |
| `token-reduction` | Ao trabalhar com arquivos muito grandes ou logs extensos de terminal | Diretrizes para ler trechos específicos, evitar rewrites e filtrar logs |
| `task-finalization` | Após realizar commits finais, push e solicitar abertura de PR | Elaboração de walkthrough, limpeza de workspace, pull e cleanup de branch |

---

## Como usar

As skills desta pasta são descobertas automaticamente pelo agente quando o workspace
`my-skills` está aberto. Para utilizá-las em outros projetos, adicione um `skills.json`
no diretório `.agents/` do projeto apontando para este diretório.

### Exemplo de `skills.json`

```json
{
  "entries": [
    { "path": "/home/galtmann/Documents/my-skills/skills" }
  ]
}
```

## Instalação Global

Para que as skills fiquem disponíveis em **qualquer projeto**, sem precisar configurar
cada workspace individualmente, instale-as via symlink nas pastas globais de cada ferramenta.

### 🖥️ Antigravity IDE

O IDE lê skills globais de `~/.gemini/config/skills/`:

```bash
mkdir -p ~/.gemini/config/skills

# git-devops
ln -sfn ~/Documents/my-skills/skills/git-devops/git-conventions            ~/.gemini/config/skills/git-conventions
ln -sfn ~/Documents/my-skills/skills/git-devops/pr-documentation           ~/.gemini/config/skills/pr-documentation
ln -sfn ~/Documents/my-skills/skills/git-devops/semantic-versioning        ~/.gemini/config/skills/semantic-versioning

# architecture
ln -sfn ~/Documents/my-skills/skills/architecture/js-ts-clean-architecture   ~/.gemini/config/skills/js-ts-clean-architecture
ln -sfn ~/Documents/my-skills/skills/architecture/js-ts-dependency-injection ~/.gemini/config/skills/js-ts-dependency-injection
ln -sfn ~/Documents/my-skills/skills/architecture/go-architecture-patterns   ~/.gemini/config/skills/go-architecture-patterns
ln -sfn ~/Documents/my-skills/skills/architecture/php-architecture-patterns  ~/.gemini/config/skills/php-architecture-patterns

# testing
ln -sfn ~/Documents/my-skills/skills/testing/code-review                ~/.gemini/config/skills/code-review
ln -sfn ~/Documents/my-skills/skills/testing/endpoint-test-automation   ~/.gemini/config/skills/endpoint-test-automation

# agent-workflows
ln -sfn ~/Documents/my-skills/skills/agent-workflows/brainstorming              ~/.gemini/config/skills/brainstorming
ln -sfn ~/Documents/my-skills/skills/agent-workflows/writing-plans              ~/.gemini/config/skills/writing-plans
ln -sfn ~/Documents/my-skills/skills/agent-workflows/token-reduction            ~/.gemini/config/skills/token-reduction
ln -sfn ~/Documents/my-skills/skills/agent-workflows/task-finalization          ~/.gemini/config/skills/task-finalization
```

### 💻 Gemini CLI

O CLI lê skills globais de `~/.gemini/skills/`:

```bash
mkdir -p ~/.gemini/skills

# git-devops
ln -sfn ~/Documents/my-skills/skills/git-devops/git-conventions            ~/.gemini/skills/git-conventions
ln -sfn ~/Documents/my-skills/skills/git-devops/pr-documentation           ~/.gemini/skills/pr-documentation
ln -sfn ~/Documents/my-skills/skills/git-devops/semantic-versioning        ~/.gemini/skills/semantic-versioning

# architecture
ln -sfn ~/Documents/my-skills/skills/architecture/js-ts-clean-architecture   ~/.gemini/skills/js-ts-clean-architecture
ln -sfn ~/Documents/my-skills/skills/architecture/js-ts-dependency-injection ~/.gemini/skills/js-ts-dependency-injection
ln -sfn ~/Documents/my-skills/skills/architecture/go-architecture-patterns   ~/.gemini/skills/go-architecture-patterns
ln -sfn ~/Documents/my-skills/skills/architecture/php-architecture-patterns  ~/.gemini/skills/php-architecture-patterns

# testing
ln -sfn ~/Documents/my-skills/skills/testing/code-review                ~/.gemini/skills/code-review
ln -sfn ~/Documents/my-skills/skills/testing/endpoint-test-automation   ~/.gemini/skills/endpoint-test-automation

# agent-workflows
ln -sfn ~/Documents/my-skills/skills/agent-workflows/brainstorming              ~/.gemini/skills/brainstorming
ln -sfn ~/Documents/my-skills/skills/agent-workflows/writing-plans              ~/.gemini/skills/writing-plans
ln -sfn ~/Documents/my-skills/skills/agent-workflows/token-reduction            ~/.gemini/skills/token-reduction
ln -sfn ~/Documents/my-skills/skills/agent-workflows/task-finalization          ~/.gemini/skills/task-finalization
```

> **Dica:** como são symlinks, qualquer edição nos arquivos de `~/Documents/my-skills/skills/`
> é refletida automaticamente nas duas ferramentas — um único lugar para manter tudo.

### 🤖 Claude Code e Claude CI

Para utilizar estas skills com o **Claude Code** no terminal ou em pipelines do **Claude CI**, você deve disponibilizá-las no diretório de contexto ou de prompts (`.claude/` ou equivalente) do seu projeto.

**Claude Code (Local):**
Crie links simbólicos das skills que deseja aplicar diretamente no diretório do seu projeto:

```bash
mkdir -p .claude/prompts

# Exemplo de link local para o projeto
ln -sfn ~/Documents/my-skills/skills/git-devops/git-conventions .claude/prompts/git-conventions
```

**Claude CI (Pipeline):**
Em ambientes de CI/CD, clone este repositório como um step antes da execução do Claude CI e copie as skills para a pasta de contexto que o Claude analisa:

```yaml
- name: Preparar Skills para Claude CI
  run: |
    git clone https://github.com/GuilhermeAltmann/my-skills-ia.git /tmp/my-skills
    mkdir -p .claude/prompts
    cp -r /tmp/my-skills/skills/* .claude/prompts/
```

### Caminhos de descoberta

| Escopo | Ferramenta | Caminho |
|--------|-----------|---------|
| Global | Antigravity IDE | `~/.gemini/config/skills/` |
| Global | Gemini CLI | `~/.gemini/skills/` |
| Por projeto | Ambos | `.gemini/skills/` (raiz do projeto) |
| Por workspace | Antigravity IDE | `.agents/skills/` (raiz do workspace) |

---

## Adicionando novas skills

1. Crie uma pasta em `skills/<nome-da-skill>/`
2. Adicione o arquivo `SKILL.md` com o frontmatter obrigatório:
   ```yaml
   ---
   name: nome-da-skill
   description: >
     Descrição clara de quando e como usar esta skill.
   ---
   ```
3. O agente passará a usar a skill automaticamente quando o contexto for relevante.
