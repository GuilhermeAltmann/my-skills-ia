---
name: token-reduction
description: >
  Diretrizes mandatórias para otimizar o consumo de tokens na interação do agente com o workspace.
  Use esta skill para orientar a leitura de arquivos, buscas, execuções de comandos e respostas
  concisas, evitando estourar a janela de contexto do LLM.
---

# Token Reduction & Context Optimization

Esta skill define as melhores práticas para otimizar o uso da janela de contexto (context window) do modelo, reduzindo o tráfego de dados (tokens de input/output) e tornando a execução de tarefas mais rápida e barata.

---

## 1. Leitura de Arquivos (View File)

- **Regra de Ouro**: **NUNCA** leia um arquivo inteiro se você precisa apenas de uma seção dele.
- **Uso correto**: Sempre forneça `StartLine` e `EndLine` no `view_file` para buscar trechos específicos.
- **Filtro Prévio**: Se não souber a linha, execute uma busca direcionada com `grep_search` para achar o termo e a linha exata antes de abrir o arquivo.

---

## 2. Escrita e Edição de Arquivos (Write/Replace)

- **Evite Overwrites**: Evite usar `write_to_file` com `Overwrite: true` para modificar arquivos existentes. Isso força o envio do arquivo inteiro no payload de retorno.
- **Use Edições Contíguas**: Sempre prefira `replace_file_content` para alterar trechos isolados (chunks) de código.
- **Edições Multi-Chunk**: Utilize `multi_replace_file_content` para alterações em linhas não consecutivas dentro do mesmo arquivo, evitando múltiplas chamadas consecutivas de edição.
- **Sem Repetições**: Não gere códigos duplicados, comentários redundantes ou imports desnecessários.

---

## 3. Busca Eficiente (Grep e Listagem)

- **Restrinja o escopo**: Ao rodar `grep_search` ou `list_dir`, nunca aponte para a raiz se você sabe em qual módulo/pasta o arquivo está.
- **Ignore dependências**: Sempre ignore pastas como `node_modules`, `vendor`, `dist`, `.git` ou `.cache`. Use os filtros de `Includes` do ripgrep para focar apenas nos arquivos da sua tecnologia (ex: `*.go`, `*.ts`).
- **Limite de Resultados**: Não peça listagens infinitas. Filtre termos específicos em vez de fazer buscas genéricas.

---

## 4. Execução de Comandos e Logs de Terminal

- **Limite de Saída (Output Truncation)**: Evite rodar testes ou builds que imprimem milhares de linhas de log no terminal.
- **Direcionamento de Saída**: Quando executar comandos muito verbosos, redirecione a saída ou filtre os logs importantes (ex: `npm test 2>&1 | head -n 50` ou `go test ./... | grep FAIL`).
- **Não faça pooling de status**: Evite verificar o status de comandos em loops infinitos. Utilize a ferramenta `schedule` para esperar no background ou aguarde a notificação automática de conclusão do sistema.

---

## 5. Estilo de Comunicação do Agente

- **Respostas Concisas**: Mantenha as explicações textuais o mais curtas possíveis. Evite resumir linhas de código que o usuário já consegue ver no editor ou no próprio diff da ferramenta.
- **Evite Desculpas**: Em caso de erros de ferramentas ou lints, corrija-os imediatamente na próxima chamada de ferramenta em vez de enviar parágrafos pedindo desculpas.
- **Links Diretos**: Ao referenciar arquivos, use o formato de link markdown com esquema `file://` apontando para a linha específica (ex: `[User.go:L42](file:///path/to/User.go#L42)`). Isso ajuda a focar o contexto no arquivo e linha exatos.

---

## Checklist Rápido de Otimização de Tokens

- [ ] Especifiquei `StartLine` e `EndLine` ao usar a ferramenta de leitura de arquivos?
- [ ] Usei `replace_file_content` em vez de reescrever o arquivo inteiro com `write_to_file`?
- [ ] Filtrei o output de comandos longos ou verbosos utilizando redirecionamento ou pipes (`grep`, `head`)?
- [ ] Limitei o escopo de busca às pastas relevantes, ignorando `node_modules` / `vendor`?
- [ ] Escrevi respostas diretas e concisas, sem redundâncias textuais?
