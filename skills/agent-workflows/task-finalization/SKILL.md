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
     [Passo 3: Verificar Git/Branch]
                 │
                 ▼
  [Passo 4: Emitir Relatório de Entrega]
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

## 3. Retorno para a Branch Principal e Cleanup de Branches

Após o envio da branch remota e solicitação da PR, o agente deve voltar à branch principal e atualizar o repositório local para garantir que seu workspace esteja limpo e pronto para a próxima tarefa:

1. **Retornar à branch principal** (`main` ou `develop`):
   ```bash
   git checkout main
   ```
2. **Atualizar a branch principal** com o servidor remoto:
   ```bash
   git pull origin main
   ```
3. **Excluir a branch temporária local** que acabou de ser enviada (para evitar acúmulo de branches locais órfãs):
   ```bash
   git branch -D <nome-da-branch-local>
   ```
   *(Nota: Use `-D` maiúsculo se a branch ainda não foi integrada remotamente, mas o push já foi realizado com sucesso).*

---

## 4. Emitir Relatório de Entrega

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
* **Branch Atual**: `main` (atualizada via `git pull`)
* **Branch Local Excluída**: `[nome-da-branch]`

### Ações Pendentes / Próximos Passos:
1. Abra o link abaixo para criar ou visualizar a Pull Request no GitHub:
   👉 [Criar Pull Request](https://github.com/usuario/repositorio/pull/new/nome-da-branch)
2. Aguarde a validação da pipeline de CI/CD do projeto.
3. Solicite a revisão dos revisores indicados.
```

---

## Checklist de Finalização

- [ ] Criei ou atualizei o arquivo `walkthrough.md` documentando o que foi testado?
- [ ] Removi todos os logs de debug (`console.log`, `var_dump`, etc.) do código de produção?
- [ ] Excluí arquivos de testes temporários e scratch scripts locais?
- [ ] Confirmei que a branch remota está atualizada com o último commit local?
- [ ] Voltei para a branch `main` e executei `git pull`?
- [ ] Excluí a branch temporária localmente (`git branch -D`)?
- [ ] Forneci o link direto do GitHub/GitLab para abertura/visualização da PR?
- [ ] Respondi de forma concisa e direta, sem redundâncias textuais?

