# SPEC-4-003 — Importação assistida de dados

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-03, RF-10 (bases), RF-19, RF-22, RN-02, RN-14, RN-15, RNF-05, RNF-09  
**Degrau da solução:** construção — substitui o importador de 8 colunas fixas por um assistente que aceita as planilhas como elas são.

## Resultado observável

A Lidiane pega a planilha que já usa, sem mexer nela, e coloca tudo no sistema em uma sessão: escolhe o arquivo, confere o mapeamento de colunas, vê a prévia, decide as duplicidades e confirma. O que deu errado fica em um arquivo com o motivo por linha, e ela pode desfazer o lote.

O passo a passo para o usuário está em `importacao-de-dados.md`.

## Por que existe

- O importador atual exige cabeçalho idêntico a `external_id, nome_ficticio, email_ficticio, telefone_ficticio, origem, campanha, produto, status`, exige e-mail e rejeita o lote inteiro por uma linha inválida. É a mesma dor do TABELAMODELO do RespondeChat.
- Não existe tela de envio de arquivo para leads, mídia nem conversas.
- A operação tem quatro fontes de dados com formatos diferentes: a planilha "CRM - Funil de Vendas", as bases de prospecção enviadas pela parceira, as listas do RespondeChat e os relatórios de mídia.

## Limites e dependências

- **Inclui:** assistente em 5 passos, três formas de entrada (arquivo, colar, Google Sheets), mapeamento de colunas com perfis salvos, conversões de valor, validação por linha, importação parcial, duplicidades, relatório, desfazer lote, histórico de importações; telas de importação de mídia e de conversas.
- **Fora de escopo:** sincronização contínua com planilha (a planilha é aposentada depois da carga); leitura de cor de célula (ver regra 9).
- **Pré-condições:** SPEC-4-001 e a regra de duplicidade da SPEC-4-002 aceitas.
- **Risco e plano B:** se a plataforma não permitir a biblioteca de leitura de `.xlsx`, o assistente aceita `.csv` e o modo "colar do Excel", que cobrem o mesmo conteúdo. Se o conector do Google Sheets não estiver habilitado, a planilha Google é baixada como `.csv`.
- **Rollback:** todo lote pode ser desfeito (regra 8).

## Telas e comportamento

Menu **Importar**. A tela inicial tem quatro cartões — **Leads**, **Base de prospecção**, **Mídia**, **Conversas** — e, abaixo, o **Histórico de importações**.

### Passo 1 — Origem dos dados

Três abas:

| Aba | Como funciona |
|---|---|
| **Arquivo** | arrastar ou escolher `.csv` ou `.xlsx`, até 10 MB e 20.000 linhas. Em `.xlsx` com mais de uma aba, o usuário escolhe a aba. Em `.csv`, o separador (`,` `;` tab) e a codificação (UTF-8 ou Windows-1252) são detectados e podem ser trocados. |
| **Colar** | caixa de texto: o usuário copia as células no Excel ou no Google Sheets e cola. Separador tab. |
| **Google Sheets** | botão "Conectar Google" (conector do Skip, com a conta do próprio usuário); depois, colar o endereço da planilha e escolher a aba. Ver `integracoes.md`, seção 5. |

O usuário indica em qual linha está o cabeçalho (padrão: a primeira linha não vazia). Planilhas com cabeçalho em duas linhas (como a "CRM - Funil de Vendas", que tem grupo e subcoluna) usam a linha de baixo.

### Passo 2 — Mapeamento de colunas

Tabela com uma linha por coluna do arquivo:

| Coluna do arquivo | 3 primeiros valores | Vai para | Conversão |
|---|---|---|---|
| Tel / WhatsApp | (21) 99999-8888 … | Telefone | Telefone brasileiro |

- O sistema sugere o destino pelo nome da coluna, usando a tabela de sinônimos abaixo. O usuário pode trocar ou escolher "Ignorar".
- No topo, o seletor **Perfil de mapeamento**. Escolher um perfil aplica todo o mapeamento de uma vez. O sistema escolhe sozinho o perfil cujas colunas batem com as do arquivo.
- Botão **Salvar como perfil**, para reutilizar.
- **Valores fixos para o lote:** o usuário define um valor para todas as linhas quando a coluna não existe — funil, etapa, origem, produto, responsável, etiqueta, tipo (lead ou prospect).
- Para colunas de catálogo (origem, produto, segmento, etapa, motivo), o passo mostra os **valores distintos** encontrados e pede a correspondência de cada um com um item do catálogo. Valor sem correspondência vai para `desconhecida`, `a_definir` ou `a_classificar`, e o texto original é preservado no detalhe.

Destinos disponíveis para leads e prospects: todos os campos de sistema da SPEC-4-001, **todos os campos personalizados ativos** (SPEC-4-012) — um campo criado pelo administrador aparece aqui sem programação —, mais os destinos especiais:

| Destino especial | O que faz |
|---|---|
| Etapa do funil | define `stage` (e, quando for o caso, `resultado_pos` e `motivo_codigo`) pela tabela de conversão do perfil. |
| Tentativa 1, 2, 3… | cria um registro em `lead_attempts` para cada célula preenchida. Lê data e canal do texto: `17/08/2026 Ligação` → data 17/08/2026, canal `ligacao`. Sem data legível: a tentativa é criada com a data de entrada e o texto original na nota. |
| Última interação | preenche `ultima_interacao_at` e canal. |
| Observações | vira uma nota do lead, com autor "importação" e a data do lote. |
| Etiquetas | cria ou associa etiquetas (catálogo `etiqueta`), separadas por vírgula. |
| Identificador externo | `external_id`; opcional. |

### Passo 3 — Prévia

- As 20 primeiras linhas já convertidas, com as células alteradas pela normalização destacadas (valor original ao passar o mouse).
- Resumo: total de linhas, válidas, com aviso, rejeitadas, possíveis duplicidades.
- Lista dos erros agrupados por tipo, com a contagem e a primeira linha de exemplo.
- Botão **Voltar ao mapeamento**.

### Passo 4 — Duplicidades

Só aparece se houver. Três grupos:

| Grupo | Padrão | Opções |
|---|---|---|
| Linha repetida dentro do próprio arquivo | manter a primeira | manter a primeira · manter a mais completa |
| Linha que bate com registro existente (chave forte) | atualizar campos vazios do existente | pular · atualizar só campos vazios · sobrescrever campos do arquivo · criar mesmo assim |
| Linha que bate por nome e cidade (chave fraca) | criar e marcar para revisão | criar e marcar · pular |

A decisão vale para o grupo, com possibilidade de exceção por linha. "Atualizar" nunca rebaixa estágio, nunca remove supressão e sempre grava a mudança na timeline do lead existente, com o lote como origem.

### Passo 5 — Confirmar e relatório

Antes de confirmar, o resumo final: quantos serão criados, atualizados, pulados e rejeitados. Botão **Importar N registros**.

Depois, o relatório do lote:

- contagens finais;
- botão **Baixar linhas rejeitadas** (`.csv` com todas as colunas originais mais `linha`, `motivo`), pronto para corrigir e importar de novo;
- botão **Ver registros importados** (abre a lista filtrada pelo lote);
- botão **Desfazer importação**.

### Histórico de importações

Tabela: data, tipo, nome do arquivo, perfil, quem importou, criados, atualizados, rejeitados, situação (`concluído`, `desfeito`, `em andamento`). Cada linha abre o relatório do lote.

## Perfis de mapeamento prontos

Entregues como dados iniciais. O mapeamento completo coluna a coluna está em `importacao-de-dados.md`, seção 3.

| Perfil | Fonte | Tipo | Observação |
|---|---|---|---|
| **Planilha CRM – Funil de Vendas** | aba "CRM - Funil de Vendas" do arquivo "CRM - Acompanhamento - ok" | lead | converte etapa, fonte, tentativas e observações. |
| **Base de prospecção (cadastro CNPJ)** | planilha "Base de contatos" enviada pela parceira: `cnpj`, `razaosocial`, `nomefantasia`, `dataAbertura`, `situacaoCadastral`, endereço, telefone, e-mail | prospect | ignora linhas com situação cadastral diferente de ativa. |
| **Lista RespondeChat** | `Nome`, `Número`, `E-mail`, `Etiquetas` | prospect | para trazer listas já disparadas. |
| **Leads Hiper – tráfego pago** | aba "Leads Hiper - Trafego Pago" | lead | origem fixa `parceiro_hiper`. |
| **Modelo padrão da Central** | `modelos/modelo-leads.csv` | lead | para qualquer outra fonte. |

Conversão de etapa do perfil "Planilha CRM – Funil de Vendas":

| Valor na planilha | Estágio | Complemento |
|---|---|---|
| Novo Lead | `novo` | |
| Contatado | `em_contato` | |
| Qualificado | `qualificado` | |
| Proposta Enviada | `agendado` | `resultado_pos = proposta_enviada`, `historico = true` |
| Fechado - Ganho | `agendado` | `resultado_pos = ganho`, `historico = true` |
| Fechado - Perdido, Perdido - Fechido | `desqualificado` | `motivo_codigo = perdido_concorrente`, `resultado_pos = perdido`, `historico = true` |
| Sem Interesse | `desqualificado` | `motivo_codigo = sem_interesse`, cria supressão, `historico = true` |
| vazio | `novo` | |

Regra adicional do perfil: linha cuja coluna Observações começa com a palavra `NUTRIR` vai para a etapa de nutrição, com o motivo `optou_gratuito` e retorno em 45 dias, qualquer que seja o valor da coluna de etapa. É como as linhas laranja da planilha (optou por sistema gratuito) entram, já que a cor não é importada e a planilha não tem etapa para elas; o guia orienta a marcar essas linhas antes de exportar.

A conversão acima aponta para as etapas do funil padrão. Como o funil é configurável (SPEC-4-012), o passo 2 sempre mostra a correspondência "valor da planilha → etapa" para o funil escolhido e deixa o usuário ajustar; valor sem etapa correspondente cai na etapa de entrada. As regras de categoria continuam valendo: linha que iria para uma etapa de categoria `agendado` sem data de apresentação entra como registro histórico.

Conversão de fonte: Tráfego Pago → `google_ads` ou `meta_ads` se a campanha identificar, senão `desconhecida` com detalhe "Tráfego pago (canal não informado)"; Indicação → `indicacao`; Disparo Massivo → `disparo_massivo`; Contato antigo → `contato_antigo`.

## Dados

### `import_batches` (evolução de `lead_import_batches`)

Campos novos: `destino` (`lead`, `prospect`, `midia`, `conversa`), `entrada` (`arquivo`, `colar`, `sheets`), `arquivo_nome`, `perfil_id`, `mapeamento` (json), `valores_fixos` (json), `decisoes_duplicidade` (json), `content_hash`, `total`, `criados`, `atualizados`, `pulados`, `rejeitados`, `status` (`rascunho`, `validado`, `concluido`, `desfeito`), `actor_id`, `desfeito_por`, `desfeito_em`.

### `import_rows`

Uma linha por linha do arquivo: `batch_id`, `linha`, `original` (json), `convertido` (json), `resultado` (`criado`, `atualizado`, `pulado`, `rejeitado`), `erros` (json), `lead_id`, `antes` (json com os campos alterados, para desfazer). Guardada por 90 dias e depois apagada, mantendo só as contagens do lote.

### `import_profiles`

`nome`, `destino`, `mapeamento` (json: coluna do arquivo → destino), `conversoes` (json: valor do arquivo → código do catálogo), `sinonimos` (json), `protegido` (bool, verdadeiro para os perfis prontos).

## API

| Rota | Papéis | Função |
|---|---|---|
| `POST /backend/v1/imports` | admin, gestor, sdr | cria o lote em rascunho a partir das linhas já lidas no navegador (cabeçalho + linhas como matriz de texto). Recusa `content_hash` igual ao de um lote concluído, com a opção `forcar`. |
| `POST /backend/v1/imports/{id}/validate` | os mesmos | recebe mapeamento, conversões e valores fixos; devolve prévia, contagens, erros agrupados e duplicidades. Não grava leads. |
| `POST /backend/v1/imports/{id}/commit` | os mesmos | recebe as decisões de duplicidade; grava. Processa em blocos de 200 linhas e devolve o progresso. Repetir a chamada não duplica. |
| `GET /backend/v1/imports` e `GET /backend/v1/imports/{id}` | os mesmos | histórico e relatório. |
| `GET /backend/v1/imports/{id}/rejected.csv` | os mesmos | linhas rejeitadas com motivo. |
| `POST /backend/v1/imports/{id}/undo` | admin, gestor; sdr só o próprio lote | desfaz (regra 8). |
| `GET/POST/PATCH /backend/v1/import-profiles` | admin, gestor, sdr (ler); admin, gestor (gravar) | perfis. |

Mídia e conversas continuam usando `POST /backend/v1/media/import` e `POST /backend/v1/conversations/import`, que já existem; esta SPEC acrescenta a tela, com o mesmo assistente e mapeamento, convertendo para o formato que essas rotas esperam. `lead_id` da importação de conversas passa a aceitar também telefone, para localizar o lead.

## Regras

1. **A leitura do arquivo acontece no navegador.** O servidor recebe só texto. Nenhum arquivo é armazenado.
2. **Importação parcial é o padrão.** Linhas válidas entram; inválidas ficam no relatório. O usuário pode marcar "só importar se todas forem válidas".
3. **Motivos de rejeição**, sempre em linguagem do usuário: sem telefone e sem e-mail; telefone sem DDD; telefone com dígitos demais ou de menos; CNPJ inválido; sem nome de empresa e de contato; data ilegível em campo obrigatório; linha vazia (ignorada, sem contar como erro).
4. **Avisos não rejeitam:** e-mail inválido (o campo fica vazio), valor de catálogo desconhecido, data de tentativa ilegível, texto cortado em 255 caracteres.
5. **Segurança do conteúdo:** todo valor passa por `limparTexto`; nenhuma célula é interpretada como fórmula.
6. **Rastro:** todo lead criado guarda o lote; a timeline registra "criado pela importação <arquivo>, linha N".
7. **Lead importado como `prospect`** nunca entra no Kanban nem na fila. Lead importado em estágio ativo entra **sem próxima ação**, e o relatório oferece "Definir primeiro contato para todos em <data>", para não lotar a fila do dia.
8. **Desfazer:** remove os registros criados pelo lote e restaura os campos alterados em registros existentes. Um registro criado pelo lote que já recebeu nota, tentativa, mudança de estágio ou agendamento **não** é removido; o relatório lista quantos ficaram e por quê.
9. **Cores da planilha não são importadas.** A cor não vai no `.csv` nem é confiável no `.xlsx`. O guia `importacao-de-dados.md` orienta a conferir a coluna de etapa antes de exportar, e o perfil converte o que está na coluna. Linhas laranja (optou por sistema gratuito) precisam estar com a etapa ou a observação indicando isso; o guia traz o passo.
10. **Dados reais:** a importação de dados reais em produção só é liberada depois da T4.2 (termo de carga). Até lá, o ambiente mostra a faixa "Ambiente de teste — use apenas dados fictícios".

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Planilha "CRM - Funil de Vendas" exportada como está | Perfil reconhecido sozinho; etapas, fontes, tentativas e observações convertidas; relatório com contagens. |
| Principal | 300 células copiadas do Excel e coladas | Mesmo resultado do arquivo. |
| Limite | Arquivo com 5 linhas sem telefone entre 200 | 195 entram; as 5 ficam no arquivo de rejeitadas com o motivo. |
| Limite | Mesmo arquivo enviado duas vezes | Aviso "Este arquivo já foi importado em <data>"; continuar exige confirmação e não duplica (cai em duplicidade). |
| Limite | Coluna "Fonte do Lead" com valor "Trafego pago " (com espaço, sem acento) | Reconhecido como o mesmo valor de "Tráfego Pago". |
| Falha | Arquivo sem cabeçalho reconhecível | Mensagem pedindo para indicar a linha do cabeçalho; nada é gravado. |
| Falha | Queda de conexão no meio da gravação | O lote fica "em andamento"; ao reabrir, o botão "Continuar" retoma de onde parou, sem duplicar. |
| Recuperação | Lote importado com mapeamento errado | "Desfazer importação" devolve a base ao estado anterior. |

## Estados vazios, erro e permissão

- Histórico vazio: "Nenhuma importação ainda. Comece por um dos cartões acima.", com link para os modelos.
- Cada cartão tem o link **Baixar modelo** (`modelos/`).
- Papel `marketing` só vê o cartão Mídia.

## Critérios de aceite

- [ ] **CA-4-07:** O usuário importa um arquivo CSV ou XLSX, um texto colado do Excel ou uma planilha Google, com colunas em qualquer ordem e com qualquer nome, mapeando as colunas na tela (inclusive para campos personalizados), e vê a prévia já normalizada antes de confirmar.
- [ ] **CA-4-08:** A planilha real "CRM - Funil de Vendas" entra com o perfil pronto, sem edição manual do arquivo: etapas, fontes, tentativas (data e canal) e observações viram etapa, origem, tentativas e nota.
- [ ] **CA-4-09:** Linhas inválidas não bloqueiam as válidas: o usuário importa as válidas, baixa as rejeitadas com o motivo por linha, decide as duplicidades em grupo e pode desfazer o lote inteiro enquanto os leads não tiverem sido trabalhados; mídia e conversas têm tela de importação.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Planilha real | importar `fixtures/leads-planilha-fase4.csv`, que reproduz o formato e os defeitos da planilha real | relatório do lote e 3 fichas conferidas |
| Colar | copiar a mesma fixture de uma planilha e colar | relatório idêntico |
| Parcial | conferir o arquivo de rejeitadas contra as linhas defeituosas conhecidas da fixture | arquivo baixado |
| Desfazer | desfazer o lote e contar os leads | contagem antes e depois |
| Retomada | interromper a gravação e continuar | contagem final sem duplicatas |

**Dados de teste:** `fixtures/leads-planilha-fase4.csv`, `fixtures/base-prospeccao-fase4.csv`, `fixtures/midia-fase4.csv`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.26 | Criar o motor de importação: lote em rascunho, validação por linha com os normalizadores, prévia, gravação em blocos com retomada, importação parcial, detecção de arquivo repetido | Dev backend | SPEC-4-003 | CA-4-07 | RED/GREEN: linhas defeituosas conhecidas da fixture; falha: queda no meio da gravação | relatório do lote + contagem sem duplicatas depois da retomada | T4.23 aceita; T4.17 aceita | ☐ | E |
| T4.27 | Criar o assistente de importação em cinco passos: arquivo CSV ou XLSX e texto colado, linha do cabeçalho, mapeamento de colunas com sugestão e valores fixos, correspondência dos valores de catálogo e de etapa, prévia e relatório; histórico de importações | Dev frontend | SPEC-4-003 | CA-4-07 | Principal: arquivo e colar com o mesmo resultado; falha: arquivo sem cabeçalho | gravação do assistente + relatórios idênticos | T4.26 aceita | ☐ | E |
| T4.28 | Criar os perfis de mapeamento prontos (planilha de funil, base de prospecção, lista RespondeChat, leads Hiper, modelo padrão) com as conversões de etapa, fonte, tentativas com data e canal, observações e a regra NUTRIR; permitir salvar perfis | Dev full-stack | SPEC-4-003 | CA-4-08 | Principal: fixtures/leads-planilha-fase4.csv sem editar o arquivo | relatório do lote + três fichas conferidas | T4.27 aceita | ☐ | E |
| T4.29 | Implementar o passo de duplicidades em grupo, o arquivo de linhas rejeitadas com motivo, a definição do primeiro contato em lote e o desfazer do lote | Dev full-stack | SPEC-4-003 | CA-4-09 | Recuperação: desfazer lote com e sem leads já trabalhados | arquivo de rejeitadas + contagem antes e depois de desfazer | T4.28 aceita | ☐ | E |
| T4.30 | Acrescentar a entrada por Google Sheets com o conector do Skip (conta do próprio usuário) e a tela Minhas conexões; ocultar a opção quando o conector não estiver habilitado | Dev full-stack | SPEC-4-003 | CA-4-07 | Principal: planilha Google; limite: conector não habilitado | captura da importação + captura do estado sem conector | T4.27 aceita | ☐ | E |
| T4.31 | Criar as telas de importação de mídia e de conversas sobre as rotas existentes, com o mesmo assistente e os perfis de relatório do Meta Ads e do Google Ads; aceitar telefone para localizar o lead nas conversas | Dev full-stack | SPEC-4-003 | CA-4-09 | Principal: fixtures/midia-fase4.csv; limite: relatório com linhas de título e de total | relatórios dos lotes | T4.27 aceita | ☐ | E |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
