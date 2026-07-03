---
name: brainstorming
description: >
  Use esta skill obrigatoriamente antes de iniciar qualquer trabalho criativo — como criar
  funcionalidades, construir componentes ou alterar comportamentos complexos. Esta skill guia
  a exploração do escopo, esclarecimento de requisitos e design técnico através de um diálogo
  colaborativo e socrático com o usuário antes da escrita de qualquer código.
---

# Brainstorming — Concepção de Funcionalidades e Design Técnico

Esta skill define o fluxo de diálogo e planejamento que o agente deve seguir antes de iniciar qualquer tarefa de implementação, evitando o "vibe coding" (programar por intuição sem planejar) e garantindo alinhamento de requisitos.

---

## Regra de Ouro (Hard Gate)

> [!IMPORTANT]
> **NÃO escreva código, não inicialize projetos, nem tome ações de implementação até que o design técnico tenha sido apresentado e o usuário o tenha aprovado explicitamente.**

Isso se aplica a todo e qualquer projeto ou alteração, por mais simples que pareça. Alterações simples que parecem não precisar de design são as que mais causam retrabalho por assunções incorretas.

---

## Checklist de Etapas

Você deve criar tarefas para cada um destes itens e completá-los em ordem:

1. **Explorar contexto do projeto**: Analisar a estrutura atual de pastas, banco de dados, dependências e commits recentes antes de fazer perguntas óbvias.
2. **Esclarecer escopo e requisitos**: Fazer perguntas para refinar a ideia do usuário, **uma por vez** para não sobrecarregar a conversa.
3. **Avaliar decomposição**: Se a ideia englobar múltiplos subsistemas, alertar o usuário para decompô-la em partes menores e focar em apenas uma por ciclo.
4. **Propor 2 a 3 abordagens**: Apresentar alternativas de implementação com seus respectivos prós, contras e recomendação técnica.
5. **Apresentar o design**: Detalhar a arquitetura proposta (estruturas de dados, contratos de API, novos componentes e lógica).
6. **Escrever documento de design**: Gravar o documento aprovado em `docs/plans/YYYY-MM-DD-<topic>-design.md` e commitar na branch apropriada.
7. **Revisão e aprovação final**: Pedir a revisão final do arquivo markdown gerado antes de iniciar as tarefas de execução.

---

## Fluxo de Processo

```
[Explorar Contexto]
        │
        ▼
[Fazer Perguntas Clarificadoras] (Uma de cada vez)
        │
        ▼
[Propor 2-3 Abordagens com Trade-offs]
        │
        ▼
[Apresentar Seções do Design] ◀──────┐ (Se recusado, revisa)
        │                            │
        ▼                            │
[Usuário Aprovou o Design?] ──(Não)──┘
        │
      (Sim)
        │
        ▼
[Gravar specs em docs/plans/*.md]
        │
        ▼
[Solicitar Validação do Arquivo]
        │
        ▼
[Iniciar Planejamento de Execução (Task.md)]
```

---

## Detalhes de Cada Etapa

### 1. Entendendo a Ideia
- Antes de fazer perguntas, verifique o código existente para não perguntar coisas que já estão óbvias na estrutura de arquivos.
- Se o projeto for muito complexo (ex: "criar um e-commerce com chat e recomendação de produtos"), ajude o usuário a quebrar em partes menores. Crie uma especificação focada apenas no primeiro passo viável (MVP).

### 2. Perguntas Clarificadoras
- **Regra de ouro**: Faça apenas **uma pergunta por mensagem** ou, no máximo, duas que estejam intimamente conectadas. Enviar listas longas de perguntas desencoraja respostas detalhadas.
- Busque entender:
  - Qual o objetivo principal de negócio?
  - Quais as restrições técnicas (linguagem, performance, dependências)?
  - Quais os limites de escopo (o que NÃO deve ser feito)?

### 3. Proposta de Abordagens
- Nunca apresente apenas uma opção de design. Apresente no mínimo duas com trade-offs de tempo, complexidade de código e escalabilidade.
- Indique claramente qual você recomenda e os motivos.

### 4. O Documento de Design (`docs/plans/`)
O documento final deve ser salvo em `docs/plans/YYYY-MM-DD-<nome-da-feature>-design.md` utilizando o seguinte formato:

```markdown
# Design Spec: [Nome da Funcionalidade]

- **Autor**: Agente de IA / Usuário
- **Data**: AAAA-MM-DD
- **Status**: [Em Revisão / Aprovado]

## 1. Contexto e Motivação
[Breve resumo do problema e por que estamos desenvolvendo esta funcionalidade]

## 2. Requisitos de Negócio e Sucesso
[Lista objetiva dos requisitos que determinam o sucesso da entrega]

## 3. Abordagem Proposta e Arquitetura
[Explicação detalhada da arquitetura técnica adotada, diagramas de fluxo se necessário]

## 4. Estrutura de Arquivos e Novos Componentes
[Lista de arquivos a serem criados ou alterados]

## 5. Contratos e Modelo de Dados
[Assinaturas de métodos, JSON Schemas de endpoints ou schemas de banco de dados]

## 6. Plano de Validação e Testes
[Como será verificado que a funcionalidade está pronta]
```

---

## Anti-Padrões a Evitar

* **Começar a programar sem design**: Começar a codificar logo após o primeiro prompt do usuário sem esclarecer edge cases.
* **Sobrecarga de perguntas**: Enviar questionários de 10 perguntas na primeira interação.
* **Monopólio de design**: Tomar todas as decisões de arquitetura e apenas comunicar ao usuário, sem apresentar opções e trade-offs.
* **Ignorar o contexto**: Propor pacotes ou bibliotecas que entram em conflito com o que já está instalado e ativo no projeto.

---

## Checklist Rápido de Início de Tarefa

- [ ] Analisei a estrutura do repositório antes de propor soluções?
- [ ] Limitei minhas perguntas clarificadoras em blocos pequenos (uma de cada vez)?
- [ ] Apresentei alternativas com trade-offs e a minha recomendação técnica?
- [ ] O usuário validou a arquitetura proposta antes de eu tocar em arquivos de código?
- [ ] Salvei o design spec em `docs/plans/`?
- [ ] A nova branch (ex: `feature/design-spec`) já foi criada para salvar as especificações sem sujar a branch principal?
