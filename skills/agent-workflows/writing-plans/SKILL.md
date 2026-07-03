---
name: writing-plans
description: >
  Use esta skill quando tiver especificações ou requisitos para uma tarefa de múltiplos passos,
  antes de alterar qualquer código de produção. Ela define a criação de planos de implementação
  detalhados com base em micro-tarefas (TDD, commits frequentes, validações passo a passo).
---

# Writing Plans — Elaboração de Planos de Implementação

Esta skill orienta a criação de planos de implementação técnicos extremamente detalhados, estruturados em micro-tarefas, assumindo que quem vai executar o plano pode não ter contexto completo sobre o codebase ou melhores práticas de testes.

---

## Visão Geral e Boas Práticas

Antes de encostar no código de produção para iniciar uma funcionalidade:
1. **Anuncie no chat**: "Estou utilizando a skill de `writing-plans` para criar o plano de implementação."
2. **Separação de escopo**: Cada plano deve cobrir apenas um subsistema independente de cada vez, resultando em software funcional e testável.
3. **Caminho padrão de gravação**: Salve os planos em `docs/plans/YYYY-MM-DD-<nome-da-feature>-plan.md`.

---

## Estrutura do Documento de Plano

Todo plano de implementação deve conter o cabeçalho padrão e seguir esta estrutura:

```markdown
# Plano de Implementação: [Nome da Feature]

> **Para Executores Autônomos**: Execute este plano passo a passo. Marque cada item como concluído com [x] somente após rodar a verificação descrita. Realize commits frequentes e evite alterar arquivos fora do planejado.

## 1. Arquivos Envolvidos
[Mapeie os arquivos que serão criados ou alterados e suas responsabilidades]
- `[NOVO]` [Caminho do arquivo] - [Responsabilidade]
- `[ALTERAR]` [Caminho do arquivo] - [Mudanças previstas]

## 2. Tarefas e Micro-passos
[Divida a implementação em blocos de tarefas testáveis de forma independente]
```

---

## Granularidade e Micro-passos (TDD/BDD)

Cada tarefa deve ser dividida em ações atômicas e muito curtas (de 2 a 5 minutos). O fluxo preferencial segue os princípios de TDD (Test-Driven Development):

```markdown
### Tarefa 1: [Nome da Tarefa Atômica]
- [ ] 🧪 Escrever o teste (unitário/integração) que descreve o comportamento esperado e que deve falhar inicialmente.
- [ ] 🏃 Rodar a suíte de testes para garantir que ele falha pelo motivo correto.
- [ ] 💻 Implementar a quantidade mínima de código de produção para fazer o teste passar.
- [ ] 🚀 Rodar os testes novamente para garantir que passaram com sucesso.
- [ ] 📝 Adicionar documentação ou comentários necessários.
- [ ] �� Realizar o commit das alterações seguindo a skill de `git-conventions`.
```

---

## Dimensionamento de Tarefas (Task Right-Sizing)

- **Unidade Mínima de Entrega**: Uma tarefa é a menor unidade que carrega seu próprio ciclo de testes e pode ser revisada/aprovada de forma independente.
- **Evite tarefas órfãs**: Passos de configuração, scaffolding e setup de banco de dados devem ser integrados à primeira tarefa que realmente depende deles, e não criados como tarefas separadas sem validação comportamental.
- **Revisabilidade**: Se uma tarefa for muito grande para ser revisada em um único commit ou PR, divida-a.

---

## Checklist de Criação de Planos

- [ ] Mapeei todos os arquivos que serão criados ou alterados antes de listar os passos?
- [ ] O plano está salvo em `docs/plans/` com a convenção de data?
- [ ] Cada tarefa possui validação imediata (geralmente testes)?
- [ ] Os passos descrevem micro-ações atômicas de 2 a 5 minutos?
- [ ] Incluí os lembretes de commits frequentes no plano?
