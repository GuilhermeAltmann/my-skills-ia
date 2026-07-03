---
name: task-finalization
description: >
  Realiza o encerramento estruturado de uma tarefa. Deve ser acionada imediatamente após a criação do
  commit final, o envio do push e a solicitação/criação da Pull Request (PR). Cobre a criação do
  walkthrough, limpeza do workspace, verificação de branch/status e relatório de entrega.
---

# Task Finalization — Encerramento de Tarefas

Esta skill define o fluxo de limpeza, documentação e entrega que o agente deve seguir após fazer o push da branch de implementação e solicitar a abertura de uma Pull Request (PR), garantindo que a entrega esteja completa, limpa e rastreável.

---

## Workflow de Finalização

O processo de finalização de tarefa deve seguir estes 4 passos obrigatórios:

```
[Abertura da PR / Push concluído]
                 │
                 ▼
     [Passo 1: Criar Walkthrough]
                 │
                 ▼
       [Passo 2: Limpar Workspace]
                 │
                 ▼
    [Passo 3: Sugerir Descrição de PR]
                 │
                 ▼
    [Passo 4: Retorno à Main (Cleanup)]
                 │
                 ▼
  [Passo 5: Emitir Relatório de Entrega]
```

---

## 1. Criar ou Atualizar o Walkthrough (`walkthrough.md`)

O agente deve registrar o histórico da entrega no artefato `walkthrough.md` localizado em `<appDataDir>/brain/<conversation-id>/walkthrough.md` para documentar o que foi alterado e como foi verificado.

O arquivo deve conter:
- **Resumo das Alterações**: Lista objetiva dos arquivos modificados/criados e suas respectivas mudanças.
- **Validação e Testes**: Comando dos testes automatizados executados e saída do resultado (ex. PHPUnit, Jest, Go tests).
- **Evidências Visuais** (se aplicável): Imagens ou GIFs mostrando alterações de UI.

---

## 2. Limpeza do Workspace

Antes de encerrar o turno, o agente deve garantir que não deixou "lixo" no workspace do usuário:
- **Remover scripts temporários**: Deletar arquivos gerados para testes manuais rápidos no workspace (como arquivos temporários na pasta `scratch/` ou scripts soltos na raiz).
- **Remover logs e dumps**: Verificar com `git diff` se não foram deixados `console.log()`, `var_dump()`, `print()` ou `fmt.Println()` de debug no código de produção.
- **Remover banco de dados de teste local**: Se foi criado um container temporário ou banco sqlite de teste local, garantir que foi finalizado e os arquivos de cache/banco temporário limpos.

---

## 3. Sugerir Descrição de PR/MR

Em vez de apenas fornecer o link de criação de PR vazio, o agente deve propor o preenchimento da descrição:

1. **Crie um arquivo temporário** (ex: `pr_suggestion.md`) na raiz do projeto contendo a sugestão.
2. **Estrutura obrigatória da sugestão**:
   - Um primeiro parágrafo explicando detalhadamente **o que** foi alterado.
   - Um segundo parágrafo explicando o **motivo** pelo qual a mudança foi realizada daquela maneira (decisões técnicas, trade-offs, resolução de problemas).
3. **Solicite a aprovação do usuário**: "Criei uma sugestão de descrição para o seu PR em `pr_suggestion.md`. Você aceita esta sugestão para a Pull Request?"
4. **Se o usuário aceitar**, o agente pode utilizar o GitHub/GitLab CLI (ex: `gh pr create -F pr_suggestion.md`) se estiver disponível, ou simplesmente instruir o usuário a copiar o conteúdo para o link do PR.
5. **Apague o arquivo temporário** após a submissão.

---

## 4. Retorno para a Branch Principal e Cleanup de Branches

Após a conclusão e envio do push para o remoto, o agente **não deve** apenas recomendar os comandos de git checkout/pull/delete no relatório final. Em vez disso, ele deve perguntar ativamente ao usuário se deseja que o agente execute a limpeza automaticamente.

### Fluxo de Ação:
1. **Pergunte ao usuário**: "Deseja que eu execute a limpeza local automática (voltar para a branch main, fazer pull e excluir a branch local <nome-da-branch>)?"
2. **Se o usuário aceitar**: Execute os seguintes comandos em sequência:
   ```bash
   git checkout main
   git pull origin main
   git branch -D <nome-da-branch-local>
   ```
3. **Se o usuário recusar**: Deixe as branches intactas e continue de onde parou.

---

## 5. Emitir Relatório de Entrega

No final do turno, apresente um sumário estruturado contendo:

```markdown
## ✅ Tarefa Concluída e Sincronizada

A branch com as implementações foi enviada com sucesso para o repositório remoto.

### Detalhes da Entrega:
* **Branch original**: `[nome-da-branch]`
* **Último Commit**: `[SHA-curto-do-commit] - [Mensagem do commit]`
* **Target Branch (Destino)**: `main` | `develop`
* **Artefato Walkthrough**: [walkthrough.md](file:///caminho/para/o/walkthrough.md)

### Estado do Repositório Local:
* **Limpeza Automática**: [Executada com sucesso (branch local removida e main atualizada) / Mantida a pedido do usuário]

### Ações Pendentes / Próximos Passos:
1. Abra o link abaixo para visualizar a Pull Request no GitHub:
   👉 [Acessar Pull Request](https://github.com/usuario/repositorio/pull/...)
2. Aguarde a validação da pipeline de CI/CD do projeto.
3. Solicite a revisão dos revisores indicados.
```

---

## Checklist de Finalização

- [ ] Criei ou atualizei o arquivo `walkthrough.md` documentando o que foi testado?
- [ ] Removi todos os logs de debug (`console.log`, `var_dump`, etc.) do código de produção?
- [ ] Excluí arquivos de testes temporários e scratch scripts locais?
- [ ] Confirmei que a branch remota está atualizada com o último commit local?
- [ ] Criei o arquivo de sugestão de PR (`pr_suggestion.md`) com 2 parágrafos (O quê e o Porquê)?
- [ ] Solicitei a aprovação do usuário para o texto do PR?
- [ ] Perguntei ao usuário se ele deseja a execução da limpeza da branch automática?
- [ ] Voltei para a branch `main`, realizei o `git pull` e apaguei a branch local temporária (se aprovado pelo usuário)?
- [ ] Forneci o link direto do GitHub/GitLab para abertura/visualização da PR?
- [ ] Apaguei o `pr_suggestion.md` no final do processo?
- [ ] Respondi de forma concisa e direta, sem redundâncias textuais?
