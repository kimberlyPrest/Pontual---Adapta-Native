# SPEC-4-002 — Entrada de leads: cadastro manual, duplicidade e captura automática

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-01, RF-02, RF-03, RF-22, RN-02, RN-14, RN-15  
**Degrau da solução:** construção — hoje não existe nenhuma forma de colocar um lead no sistema pela interface.

## Resultado observável

A Lidiane cadastra um lead em menos de um minuto, o sistema avisa antes de salvar se aquele contato já existe, e leads vindos de formulário do site ou de anúncio entram sozinhos, já com origem e campanha.

## Por que existe

- A rota `POST /backend/v1/leads/import` existe, mas nenhuma tela a chama; não há formulário de cadastro.
- A tela de conflitos de duplicidade não existe, embora as rotas existam.
- Os leads de tráfego pago chegam hoje pelo WhatsApp e são digitados à mão na planilha; a origem se perde.

## Limites e dependências

- **Inclui:** botão "Novo lead", formulário rápido e completo, verificação de duplicidade em tempo real, tela de conflitos com mesclagem, rota de captura automática com token.
- **Fora de escopo:** importação em lote (SPEC-4-003), gestão dos tokens e fila de exceção (SPEC-4-011), mensagens de conversa (SPEC-4-011).
- **Pré-condições:** SPEC-4-001 aceita.
- **Risco e plano B:** se o site ou o anúncio não conseguirem chamar a rota, o cadastro manual e a importação cobrem a entrada; a captura automática não bloqueia o resto da fase.

## Telas e comportamento

### Botão "Novo lead"

Fica no cabeçalho do aplicativo, visível em todas as telas, para `admin`, `gestor` e `sdr`. Atalho de teclado: `N`.

Abre um painel lateral com dois níveis:

**Cadastro rápido (sempre visível)**

| Campo | Regra |
|---|---|
| Empresa | obrigatório se "Contato" estiver vazio |
| Contato | obrigatório se "Empresa" estiver vazia |
| Telefone / WhatsApp | obrigatório; máscara brasileira; normalizado ao sair do campo |
| Origem | obrigatório; lista do catálogo; padrão: a última usada pelo usuário |
| Produto | lista do catálogo; padrão "A definir" |
| Observação inicial | texto livre; vira a primeira nota do lead |

**Mais detalhes (recolhido)**

E-mail, CNPJ, cargo, segmento, cidade, UF, sistema atual, campanha, detalhe da origem (ex.: quem indicou), responsável (padrão: o próprio usuário).

Botões: **Salvar**, **Salvar e abrir ficha**, **Salvar e cadastrar outro**.

Ao salvar: o lead nasce em `novo`, com `responsavel_id` do usuário, próxima ação "Fazer primeiro contato" para o mesmo dia e `data_entrada` de hoje. Aparece no Kanban e na fila sem recarregar a página.

### Verificação de duplicidade em tempo real

Ao sair dos campos telefone, CNPJ ou e-mail, o formulário consulta a rota de verificação. Se houver correspondência, aparece um aviso dentro do formulário, antes de salvar:

> **Já existe um registro com este telefone.**  
> Mega Loja Materiais · João · Em contato · responsável: Lidiane · última interação há 3 dias  
> [Abrir registro existente] [Adicionar como nota ao existente] [Criar mesmo assim]

- **Abrir registro existente:** fecha o formulário e abre a ficha.
- **Adicionar como nota ao existente:** grava a observação digitada como nota no lead existente, com a origem nova registrada na timeline ("novo contato pela origem X"), e não cria lead.
- **Criar mesmo assim:** exige motivo (texto, mínimo de 10 caracteres) e grava na timeline dos dois registros.

Se o registro existente for um `prospect`, a ação oferecida é **Promover a lead** (SPEC-4-006), sem criar novo registro.

### Regra única de duplicidade

A mesma regra vale para formulário, importação e captura automática.

| Ordem | Chave | Força |
|---|---|---|
| 1 | `cnpj` igual | forte |
| 2 | `telefone` ou `telefone_secundario` igual a qualquer telefone do outro registro | forte |
| 3 | `email` igual | forte |
| 4 | `empresa_nome` igual (sem acento, minúsculas, sem "ltda", "me", "eireli") **e** mesma `cidade` | fraca: só avisa, nunca bloqueia nem mescla sozinha |

### Tela "Duplicidades"

Menu Leads → Duplicidades. Mostra o contador de pendentes no menu.

- Lista de pares pendentes, com os dois registros lado a lado, campo a campo, e as diferenças destacadas.
- Ações por par: **Manter os dois** (com motivo), **Mesclar**, **Descartar o novo**.
- **Mesclar:** o usuário escolhe o registro principal. Campos vazios do principal recebem o valor do outro; campos em conflito mostram os dois valores para escolha. Notas, tentativas, timeline, conversas e agendamentos do registro secundário passam para o principal. O secundário não é apagado: fica marcado como `mesclado_em` com o id do principal e some das telas.
- Seleção múltipla com ação em massa quando a decisão for a mesma.
- Usa as rotas existentes `GET /backend/v1/leads/conflicts` e `POST /backend/v1/leads/conflicts/{id}/decision`, estendidas com a decisão `merged`.

## Captura automática

### `POST /backend/v1/integrations/inbound/lead`

Rota pública, autenticada por token no cabeçalho `X-Integration-Token`. O contrato completo, com exemplos, está em `integracoes.md`, seção 3.

Corpo (JSON):

| Campo | Obrigatório | Observação |
|---|---|---|
| `idempotency_key` | sim | identificador único do evento na origem (id do formulário, id do lead no anúncio). |
| `telefone` | sim, se não houver `email` | normalizado pela mesma função do formulário. |
| `email` | sim, se não houver `telefone` | |
| `contato_nome`, `empresa_nome` | não | se ambos vierem vazios, `contato_nome` = "Contato sem nome". |
| `origem` | não | código do catálogo; ausente ou inválido vira `desconhecida`. Se o token tiver origem padrão, ela é usada. |
| `campanha`, `conjunto`, `anuncio` | não | |
| `utm` | não | objeto com `source`, `medium`, `campaign`, `content`, `term`. |
| `produto` | não | código do catálogo; padrão `a_definir`. |
| `mensagem` | não | vira a primeira nota. |
| `cidade`, `uf`, `segmento`, `cnpj` | não | |
| `consentimento` | não | objeto com `texto` e `data`; guardado na timeline. |

Respostas:

| Código | Quando | Corpo |
|---|---|---|
| 201 | lead criado | `{ "lead_id": "...", "duplicate": false }` |
| 200 | mesma `idempotency_key` já processada | `{ "lead_id": "...", "duplicate": false, "replayed": true }` |
| 200 | contato já existe (chave forte) | `{ "lead_id": "<existente>", "duplicate": true }` — a mensagem entra como nota no existente; a nova origem vai para a timeline; se o existente estava em `nutricao` ou `desqualificado` (exceto supressão), volta para `triagem` com o motivo "novo contato do lead". |
| 401 | token ausente, inválido ou revogado | sem detalhe |
| 422 | corpo inválido | lista de campos com erro |
| 429 | mais de 60 chamadas por minuto no mesmo token | |

Regras:

1. O lead criado nasce em `novo`, sem responsável se o token não definir um; aparece na fila como "novo sem primeiro contato".
2. Contato suprimido que entra de novo por vontade própria (formulário) **não** é reativado sozinho: gera transbordo "contato suprimido voltou a procurar", para decisão humana.
3. Toda chamada, aceita ou negada, grava um registro em `integration_events` (SPEC-4-011) sem telefone nem e-mail em claro.
4. Falha interna ao processar devolve 202 e deixa o evento na fila de exceção para reprocessar; a origem não precisa reenviar.

### `GET /backend/v1/leads/check-duplicate`

Parâmetros `telefone`, `cnpj`, `email`, `empresa`, `cidade`. Devolve até 5 correspondências, com `id`, `tipo`, nome, estágio, responsável, última interação e a chave que bateu. Papéis: `admin`, `gestor`, `sdr`.

### `POST /backend/v1/leads`

Cria um lead pelo formulário. Valida com os normalizadores da SPEC-4-001, aplica a regra de duplicidade (bloqueia chave forte sem `forcarMotivo`) e grava a criação na timeline.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Empresa, telefone e origem preenchidos | Lead criado em `novo`; aparece no Kanban e na fila na hora. |
| Principal | Formulário do site envia para a rota de captura | Lead criado com origem e UTM; aparece na fila em até 5 segundos. |
| Limite | Telefone sem DDD | Erro no campo: "Informe o DDD." Nada é salvo. |
| Limite | Mesmo telefone de um lead em nutrição entra pelo site | Não duplica; o lead existente volta para triagem com a mensagem como nota. |
| Limite | Reenvio da mesma `idempotency_key` | Mesma resposta; nenhum registro novo. |
| Falha | Token revogado | 401; evento registrado como negado. |
| Falha | Usuário sem papel tenta criar | Negado e auditado. |

## Estados vazios, erro e permissão

- Tela "Duplicidades" sem pendências: "Nenhuma duplicidade para revisar."
- Erro de rede ao salvar: o formulário mantém o que foi digitado e mostra "Não foi possível salvar. Tente de novo."
- Papel `marketing` não vê o botão "Novo lead".

## Critérios de aceite

- [ ] **CA-4-04:** Um usuário cria um lead pelo botão "Novo lead" em menos de 1 minuto, com três informações obrigatórias (empresa ou contato, telefone, origem), e o lead aparece no Kanban e na fila sem recarregar a página.
- [ ] **CA-4-05:** Ao informar telefone, CNPJ ou e-mail que já existe, a tela mostra o registro existente antes de salvar e oferece abrir, anotar no existente ou criar mesmo assim com motivo; a tela Duplicidades permite mesclar dois registros sem perder notas, tentativas nem histórico.
- [ ] **CA-4-06:** Uma chamada válida à rota de captura cria o lead com origem, campanha e parâmetros de campanha em até 5 segundos; a repetição com a mesma chave de idempotência não duplica; chave inválida é negada e registrada.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Cadastro em menos de 1 minuto | cronometrar o cadastro rápido | gravação de tela |
| Duplicidade | cadastrar o mesmo telefone em 3 formatos diferentes | capturas do aviso |
| Mesclagem | mesclar dois registros com notas e tentativas | ficha do principal com tudo reunido |
| Captura | chamadas válida, repetida, sem token e inválida | respostas e registros em `integration_events` |

**Dados de teste:** `fixtures/captura-fase4.json`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.22 | Criar o botão Novo lead e o formulário rápido e completo, com a rota de criação, os campos personalizados obrigatórios e a criação da primeira próxima ação | Dev full-stack | SPEC-4-002 | CA-4-04 | Principal: cadastro cronometrado; limite: telefone sem DDD | gravação com o tempo + lead no Kanban e na fila | T4.18 aceita | ☐ | D |
| T4.23 | Implementar a regra única de duplicidade (CNPJ, telefone, e-mail, nome com cidade), a verificação em tempo real no formulário, a tela Duplicidades e a mesclagem de registros | Dev full-stack | SPEC-4-002 | CA-4-05 | RED/GREEN: mesmo telefone em três formatos; principal: mesclar com notas e tentativas | capturas do aviso + ficha do registro principal depois da mesclagem | T4.22 aceita | ☐ | D |
| T4.25 | Implementar a rota de captura de lead: validação, idempotência, origem e parâmetros de campanha, campos personalizados, tratamento de contato existente e de contato suprimido, resposta rápida com fila de exceção | Dev backend | SPEC-4-002 | CA-4-06 | Principal, repetida, sem chave e inválida, com fixtures/captura-fase4.json | tabela chamada × resposta + eventos registrados | T4.24 aceita; T4.23 aceita | ☐ | D |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
