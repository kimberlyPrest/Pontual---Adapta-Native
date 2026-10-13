# SPEC-4-006 — Bases de prospecção, listas de disparo e reativação

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-10 (bases), RF-13, RF-14, RF-15, RN-03, RN-07, G5, G6  
**Degrau da solução:** construção — o sistema monta e higieniza a lista. A partir dela há dois caminhos: **enviar pela Central**, com ritmo controlado e aprovação (SPEC-4-013), ou **exportar o arquivo** no formato que o RespondeChat aceita, para o período de transição.

## Resultado observável

Em vez de copiar colunas de uma planilha para o TABELAMODELO, a Lidiane filtra os contatos na tela, confere quem ficou de fora e por quê, e baixa o arquivo pronto. Quem disse "sem interesse" nunca mais entra em lista nenhuma, sem ela precisar lembrar.

## Por que existe

- Montar a lista de disparo hoje é copiar nome, telefone e e-mail da base bruta para um modelo de 4 colunas, ajustar etiquetas, salvar e baixar — manualmente, toda semana.
- A exclusão de quem recusou é feita pintando a linha de vermelho na planilha e lembrando de tirar na próxima vez.
- Contatos de base fria (linhas brancas) estão misturados com leads em negociação na mesma planilha.
- A versão anterior desta fase previa um motor de disparo em simulação, que não resolvia a dor. Esta SPEC entrega a lista limpa e auditável; o envio de verdade está na SPEC-4-013 e só é ligado depois das decisões da Pontual sobre a política de contato (G5, G6).

## Limites e dependências

- **Inclui:** tela de bases de prospecção, promoção de prospect a lead, construtor de lista com prévia e exclusões, exportação no formato do RespondeChat, registro de cada exportação, registro do retorno, supressão automática, listas de reativação a partir de leads em nutrição ou desqualificados.
- **Fora de escopo desta SPEC:** o mecanismo de envio (SPEC-4-013); leitura automática das respostas quando o disparo é feito pelo RespondeChat (depende daquele fornecedor — ver `integracoes.md`, seção 8).
- **Pré-condições:** SPEC-4-001, SPEC-4-003 (para importar as bases) e SPEC-4-004 aceitas.
- **Gates G5 e G6:** não bloqueiam a montagem nem a exportação da lista. Condicionam o botão **Enviar pela Central**, que só aparece com a chave "envio de listas permitido" ligada no canal. A política de contato (quem pode receber disparo, com que frequência) é decisão da Pontual, registrada na T4.2 e refletida nas exclusões configuráveis.
- **Risco e plano B:** se o RespondeChat mudar o formato de importação, o formato de exportação é configurável (ordem e nome das colunas) em Configurações.

## Telas e comportamento

Menu **Prospecção**, com três abas: **Bases**, **Listas**, **Retornos**.

### Aba Bases

Tabela dos registros com `tipo = prospect`.

- Colunas: empresa, nome fantasia, CNPJ, contato, telefone, e-mail, cidade, UF, segmento, etiquetas, lote de importação, último disparo, retorno.
- Filtros: lote, etiqueta, cidade, UF, segmento, "nunca disparado", "disparado há mais de N dias", "com retorno", "suprimido".
- Botão **Importar base** (abre o assistente da SPEC-4-003 já em "Base de prospecção").
- Ação por linha e em massa: **Promover a lead**, **Adicionar etiqueta**, **Registrar opt-out**.
- Contador no topo: total, nunca disparados, disparados, com resposta, suprimidos.

**Promover a lead:** muda `tipo` para `lead`, mantém origem, etiquetas e histórico, coloca em `novo` (ou `em_contato` se já houver resposta registrada), pede responsável e próxima ação. Um clique por linha; em massa, um diálogo só.

### Aba Listas — construtor

Botão **Nova lista**. Três passos em uma página.

**1. De onde vêm os contatos**

| Fonte | Para quê |
|---|---|
| Base de prospecção | prospecção fria |
| Leads em nutrição | reativação de quem pediu retorno ou optou por gratuito |
| Leads desqualificados | reativação; nunca inclui motivo com supressão |
| Leads com apresentação e sem fechamento | `resultado_pos` em `apresentacao_realizada`, `proposta_enviada` |

**2. Critérios**

Filtros combináveis com "e": lote, etiqueta, segmento, cidade, UF, produto, origem, motivo, dias desde a última interação (mínimo), dias desde o último disparo (mínimo), data de entrada. Cada critério mostra ao lado quantos contatos passam.

**3. Prévia e exclusões**

Painel com três números grandes: **encontrados**, **excluídos**, **na lista**.

Tabela de exclusões, com o contador por motivo e o link para ver os contatos:

| Exclusão | Obrigatória | Padrão |
|---|---|---|
| Suprimido (opt-out, sem interesse) | sim, não desativável | — |
| Telefone inválido ou ausente | sim | — |
| Já é cliente (`resultado_pos = ganho`) | sim | — |
| Em negociação ativa (estágio `em_contato`, `qualificado`, `agendamento_solicitado`, `agendado` recente) | sim | — |
| Recebeu disparo nos últimos N dias | configurável | 30 dias |
| Teve interação nos últimos N dias | configurável | 15 dias |
| Telefone repetido dentro da lista | sim | mantém o primeiro |
| Nutrição com data de retorno ainda no futuro | configurável | ligado |

Abaixo, a prévia dos 20 primeiros contatos **como vão sair no arquivo**. O usuário pode remover contatos à mão (com motivo "removido manualmente").

Campos finais: **nome da lista**, **etiqueta** (vai na coluna Etiquetas; campo de texto com as etiquetas usadas antes como sugestão — precisa ser igual à etiqueta de entrada do fluxo no RespondeChat), **limite de contatos** (padrão 100, conforme a prática atual de listas de até 100), **observação**.

Botões: **Salvar rascunho**, **Enviar pela Central** (SPEC-4-013: escolhe canal, modelo de mensagem, ritmo e horário; pede aprovação quando exigida) e **Exportar** (arquivo).

### Exportação

Gera o arquivo e registra o lote.

- **Formato padrão "RespondeChat":** exatamente quatro colunas, nesta ordem e com estes nomes: `Nome`, `Número`, `E-mail`, `Etiquetas`.
  - `Nome`: contato; se vazio, nome fantasia; se vazio, empresa.
  - `Número`: dígitos com DDI e DDD, sem símbolos (ex.: `5521999998888`). O formato do número é configurável (com ou sem `55`), para ser ajustado na primeira importação real.
  - `E-mail`: vazio quando não houver.
  - `Etiquetas`: a etiqueta da lista.
- Arquivo `.xlsx` ou `.csv` (o usuário escolhe; o padrão é o último usado).
- Depois de exportar, a tela mostra o passo a passo curto: "1. No RespondeChat, crie a Lista de Contatos e importe este arquivo. 2. Crie a Transmissão e escolha o fluxo com a etiqueta **<etiqueta>**. 3. Quando o disparo terminar, volte aqui e registre os retornos."
- Para cada contato exportado: uma tentativa em `lead_attempts` com `origem = lista`, canal `whatsapp`, resultado `sem_resposta`, datada na exportação, e um item na timeline "Incluído na lista <nome>".
- A lista passa a `exportada`. Exportar de novo a mesma lista baixa o mesmo arquivo e **não** cria novas tentativas.

### Aba Retornos

Lista das listas exportadas: nome, data, tamanho, etiqueta, quem exportou, e os contadores de retorno (sem resposta, respondeu, sem interesse, número inválido).

Abrir uma lista mostra os contatos, com seleção múltipla e os botões:

| Ação | Efeito |
|---|---|
| **Respondeu** | registra o retorno; oferece **Promover a lead** (prospect) ou reabrir para `em_contato` (lead em nutrição ou desqualificado). |
| **Sem interesse** | registra o retorno e cria a supressão (origem `recusa_conversa`, evidência "resposta ao disparo <lista>"). O contato sai de todas as listas em rascunho. |
| **Número inválido** | marca o telefone como inválido. |
| **Sem resposta** | mantém; o contato volta a ser elegível depois do intervalo configurado. |

Também é possível **colar uma lista de telefones** e aplicar um retorno a todos de uma vez (útil para marcar os que responderam olhando o RespondeChat).

Botão **Concluir lista**: os que ficaram sem marcação são contados como "sem resposta".

## Dados

### `prospect_lists`

`nome`, `fonte` (`prospect`, `nutricao`, `desqualificado`, `pos_apresentacao`), `criterios` (json), `exclusoes_config` (json), `etiqueta`, `limite`, `observacao`, `encontrados`, `excluidos`, `tamanho`, `status` (`rascunho`, `exportada`, `concluida`, `cancelada`), `actor_id`, `exportada_em`, `concluida_em`.

### `prospect_list_members`

`list_id`, `lead_id`, `incluido` (bool), `exclusao_motivo`, `nome_exportado`, `numero_exportado`, `retorno` (`pendente`, `sem_resposta`, `respondeu`, `sem_interesse`, `numero_invalido`), `retorno_por`, `retorno_em`.

A composição é **congelada** na exportação: o que foi exportado fica registrado, mesmo que o lead mude depois.

### `list_exports`

`list_id`, `formato`, `arquivo_tipo`, `linhas`, `etiqueta`, `content_hash`, `actor_id`, `created`. Sem atualização nem exclusão.

## API

| Rota | Papéis | Função |
|---|---|---|
| `GET /backend/v1/prospects` | admin, gestor, sdr | bases, com filtros e paginação. |
| `POST /backend/v1/prospects/promote` | admin, gestor, sdr | promove um ou vários (até 200). |
| `POST /backend/v1/lists/preview` | admin, gestor, sdr | recebe fonte, critérios e exclusões; devolve contagens, exclusões por motivo e os 20 primeiros. Não grava. |
| `POST /backend/v1/lists` · `PATCH /backend/v1/lists/{id}` | admin, gestor, sdr | salva o rascunho. |
| `POST /backend/v1/lists/{id}/export` | admin, gestor, sdr | recalcula as exclusões obrigatórias no momento da exportação, congela os membros, registra e devolve as linhas do arquivo. Repetir devolve o mesmo conteúdo. |
| `GET /backend/v1/lists` · `GET /backend/v1/lists/{id}` | admin, gestor, sdr | listas e membros. |
| `POST /backend/v1/lists/{id}/returns` | admin, gestor, sdr | registra retornos em massa, por id ou por telefone. |
| `POST /backend/v1/lists/{id}/close` | admin, gestor, sdr | conclui. |

## Regras

1. As exclusões obrigatórias são aplicadas no **servidor**, de novo, no momento da exportação. Uma supressão registrada entre a prévia e a exportação tira o contato do arquivo.
2. A supressão vale para o **telefone**, não só para o registro: se o mesmo número existir em outro registro, ele também fica fora.
3. Nenhuma rota desta SPEC envia mensagem. O envio é da SPEC-4-013, que consome a lista já congelada e reaplica as exclusões obrigatórias no momento de cada envio. Quando o envio é feito pela Central, a aba Retornos é preenchida sozinha (respondeu, número inválido) e a recusa fica aguardando a confirmação de uma pessoa.
4. O arquivo é gerado no navegador a partir das linhas devolvidas; nada é armazenado em disco no servidor.
5. Exportação é registrada na auditoria com quem, quando, quantos contatos e a etiqueta — sem os telefones.
6. Lista em rascunho é recalculada a cada abertura; a contagem pode mudar.
7. O limite de contatos corta a lista pela ordem "nunca disparado primeiro, depois o disparo mais antigo".

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Base de 500 prospects, 40 suprimidos, 60 disparados há 10 dias, limite 100 | Encontrados 500, excluídos 100 (40 + 60), na lista 100 (os nunca disparados primeiro). |
| Principal | Exportar | Arquivo com 4 colunas e 100 linhas; 100 tentativas registradas. |
| Principal | Marcar 5 como "sem interesse" | 5 supressões; os 5 somem de qualquer lista em rascunho. |
| Limite | Contato marcado sem interesse entre a prévia e a exportação | Não sai no arquivo; a tela avisa "1 contato removido desde a prévia". |
| Limite | Exportar a mesma lista duas vezes | Mesmo arquivo; nenhuma tentativa nova. |
| Limite | Nome com vírgula ou aspas | Arquivo íntegro, com a célula entre aspas no `.csv`. |
| Falha | Lista sem nenhum contato elegível | Botão Exportar desabilitado, com a explicação das exclusões. |
| Falha | Etiqueta vazia | Erro no campo: a etiqueta é obrigatória para o RespondeChat aceitar o fluxo. |

## Estados vazios, erro e permissão

- Bases vazias: "Nenhuma base importada." com o botão **Importar base** e o link para o modelo.
- Listas vazias: "Nenhuma lista ainda."
- `marketing` não acessa Prospecção.

## Critérios de aceite

- [ ] **CA-4-20:** Bases de prospecção ficam em tela própria, fora do Kanban e da fila, até haver resposta; "Promover a lead" leva um clique e preserva origem, etiquetas e histórico.
- [ ] **CA-4-21:** A lista mostra, antes de ser usada, os critérios, o tamanho, os excluídos por motivo e a prévia dos contatos; as exclusões obrigatórias (suprimido, telefone inválido, já cliente, em negociação) não podem ser desligadas e são reaplicadas no servidor na hora do uso.
- [ ] **CA-4-22:** A exportação gera um arquivo com exatamente quatro colunas — Nome, Número, E-mail, Etiquetas — aceito pelo RespondeChat sem edição; cada exportação fica registrada (quem, quando, quantos, etiqueta) e marcada na timeline dos contatos; exportar de novo não duplica registros.
- [ ] **CA-4-23:** O retorno é registrado em grupo (respondeu, sem interesse, número inválido); "sem interesse" cria a supressão e tira o contato de toda lista futura; "respondeu" oferece promover a lead.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Exclusões | montar lista sobre `fixtures/base-prospeccao-fase4.csv`, que tem suprimidos, inválidos e repetidos conhecidos | tabela de exclusões conferida com o esperado da fixture |
| Formato | abrir o arquivo exportado e comparar com `modelos/modelo-respondechat.csv` | os dois arquivos |
| Importação real | na T4.73, importar um arquivo exportado no RespondeChat de verdade | captura da importação aceita |
| Supressão | marcar sem interesse e tentar incluir o contato em nova lista | prévia sem o contato |

**Dados de teste:** `fixtures/base-prospeccao-fase4.csv`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.50 | Criar a tela de bases de prospecção (registros do tipo prospect), com filtros, etiquetas, contadores e a promoção a lead, individual e em massa | Dev full-stack | SPEC-4-006 | CA-4-20 | Principal: promover; limite: prospect não aparece no Kanban nem na fila | capturas das três telas | T4.28 aceita | ☐ | J |
| T4.51 | Criar o construtor de listas: fontes, critérios com contagem, prévia e exclusões obrigatórias e configuráveis calculadas no servidor, remoção manual e limite de contatos | Dev full-stack | SPEC-4-006 | CA-4-21 | Prova com fixtures/base-prospeccao-fase4.csv: exclusões conhecidas | tabela de exclusões conferida com o esperado | T4.50 aceita; T4.32 aceita | ☐ | J |
| T4.52 | Implementar a exportação no formato de quatro colunas (configurável), o congelamento dos membros, o registro da exportação e as tentativas de origem lista, sem duplicar ao exportar de novo | Dev full-stack | SPEC-4-006 | CA-4-22 | Limite: exportar duas vezes; contato suprimido entre a prévia e a exportação | arquivo comparado com modelos/modelo-respondechat.csv + registros | T4.51 aceita | ☐ | J |
| T4.53 | Criar a aba Retornos: marcar em massa respondeu, sem interesse e número inválido, colar lista de telefones, supressão automática e conclusão da lista | Dev full-stack | SPEC-4-006 | CA-4-23 | Principal: sem interesse gera supressão e tira o contato de listas em rascunho | prévia de nova lista sem o contato | T4.52 aceita | ☐ | J |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
