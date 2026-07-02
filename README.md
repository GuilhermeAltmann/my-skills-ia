# My AI Agent Skills

Coleção de skills personalizadas para agentes de IA (compatível com Antigravity IDE).

## Estrutura

```
my-skills/
└── skills/
    ├── git-conventions/            # Padronização de commits e branches
    │   └── SKILL.md
    ├── code-review/                # Revisão de código estruturada
    │   └── SKILL.md
    ├── pr-documentation/           # Documentação de Pull Requests
    │   └── SKILL.md
    ├── semantic-versioning/        # Versionamento semântico (SemVer)
    │   └── SKILL.md
    ├── js-ts-clean-architecture/   # Arquitetura em Camadas (Clean Architecture)
    │   └── SKILL.md
    ├── js-ts-dependency-injection/ # Injeção de Dependência (DI)
    │   └── SKILL.md
    └── go-architecture-patterns/   # Hexagonal, Repository, DI e Functional Options (Go)
        └── SKILL.md
```

## Skills disponíveis

| Skill                        | Gatilho                                                              | Descrição resumida                                              |
| ---------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------- |
| `git-conventions`            | Criar commit, nomear branch, revisar convenções Git                  | Conventional Commits + GitFlow                                  |
| `code-review`                | Revisar PR, diff ou arquivo; avaliar qualidade de código             | Revisão por severidade: BLOCKER, MAJOR, MINOR, NIT              |
| `pr-documentation`           | Abrir PR, escrever descrição, validar template de PR                 | Template completo com checklist, tipo, evidências e como testar |
| `semantic-versioning`        | Criar release, definir versão, atualizar CHANGELOG                   | SemVer 2.0.0 com CHANGELOG, tags Git e manifests                |
| `js-ts-clean-architecture`   | Estruturar projeto, decidir onde uma classe vive, revisar camadas    | Domain / Application / Infrastructure / Presentation + regras  |
| `js-ts-dependency-injection` | Criar classes com dependências, configurar container, escrever mocks | DI manual, Factories, TSyringe, InversifyJS, testes unitários   |
| `go-architecture-patterns`   | Estruturar projeto Go, criar repositórios, configurar structs        | Hexagonal, Repository SQL, DI manual/Wire, Functional Options   |

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

ln -sfn ~/Documents/my-skills/skills/git-conventions            ~/.gemini/config/skills/git-conventions
ln -sfn ~/Documents/my-skills/skills/code-review                ~/.gemini/config/skills/code-review
ln -sfn ~/Documents/my-skills/skills/pr-documentation           ~/.gemini/config/skills/pr-documentation
ln -sfn ~/Documents/my-skills/skills/semantic-versioning        ~/.gemini/config/skills/semantic-versioning
ln -sfn ~/Documents/my-skills/skills/js-ts-clean-architecture   ~/.gemini/config/skills/js-ts-clean-architecture
ln -sfn ~/Documents/my-skills/skills/js-ts-dependency-injection ~/.gemini/config/skills/js-ts-dependency-injection
ln -sfn ~/Documents/my-skills/skills/go-architecture-patterns   ~/.gemini/config/skills/go-architecture-patterns
```

### 💻 Gemini CLI

O CLI lê skills globais de `~/.gemini/skills/`:

```bash
mkdir -p ~/.gemini/skills

ln -sfn ~/Documents/my-skills/skills/git-conventions            ~/.gemini/skills/git-conventions
ln -sfn ~/Documents/my-skills/skills/code-review                ~/.gemini/skills/code-review
ln -sfn ~/Documents/my-skills/skills/pr-documentation           ~/.gemini/skills/pr-documentation
ln -sfn ~/Documents/my-skills/skills/semantic-versioning        ~/.gemini/skills/semantic-versioning
ln -sfn ~/Documents/my-skills/skills/js-ts-clean-architecture   ~/.gemini/skills/js-ts-clean-architecture
ln -sfn ~/Documents/my-skills/skills/js-ts-dependency-injection ~/.gemini/skills/js-ts-dependency-injection
ln -sfn ~/Documents/my-skills/skills/go-architecture-patterns   ~/.gemini/skills/go-architecture-patterns
```

> **Dica:** como são symlinks, qualquer edição nos arquivos de `~/Documents/my-skills/skills/`
> é refletida automaticamente nas duas ferramentas — um único lugar para manter tudo.

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
