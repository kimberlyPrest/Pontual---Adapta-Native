# SPEC-4-004 — Ficha do lead e Kanban de trabalho

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-04, RF-05, RF-06, RF-07, RF-08, RF-09, RF-10, RF-15, RF-23, RN-03, RN-05, RN-06, RN-07  
**Degrau da solução:** construção de interface sobre rotas que já existem — quase todo o backend desta SPEC foi entregue nas fases 1 e 3 e nunca ganhou tela.

## Resultado observável

Clicar em qualquer lead abre a ficha completa: dados, notas, tentativas, conversa, qualificação, agendamento e histórico, tudo editável no lugar. O Kanban mostra no card o que a Lidiane precisa para decidir o que fazer, inclusive a última anotação.

## Por que existe

- A Lidiane abandonou o Kanban do RespondeChat porque o card não mostrava anotações; ela usa o campo "Observações" da planilha como histórico. O sistema atual tem o mesmo defeito.
- Timeline, conversa, ficha de qualificação com ICP, opt-out e transbordo têm rota e não têm tela. A tela de conversa existe, mas nenhum link leva a ela.
- No Kanban atual o lead só anda por botões, um estágio por vez, e não há busca nem filtro.

## Limites e dependências

- **Inclui:** navegação do aplicativo, busca global, ficha do lead com abas, notas, edição com auditoria, Kanban versão 2.
- **Fora de escopo:** registrar tentativa e régua (SPEC-4-005), agendamento (SPEC-4-007), botões de IA (SPEC-4-008) — a ficha reserva o lugar e essas SPECs preenchem.
- **Pré-condições:** SPEC-4-001 aceita.

## Navegação do aplicativo

Menu lateral, nesta ordem. Itens que o papel não pode ver não aparecem.

| Item | Rota | Papéis | SPEC |
|---|---|---|---|
| Minha fila | `/fila` | admin, gestor, sdr | 4-005 |
| Conversas | `/conversas` | admin, gestor, sdr | 4-013 |
| Kanban | `/kanban` (redireciona de `/painel`) | admin, gestor, sdr | esta |
| Leads | `/leads` | admin, gestor, sdr | esta |
| Agenda | `/agenda` | admin, gestor, sdr | 4-007 |
| Prospecção | `/prospeccao` | admin, gestor, sdr | 4-006 |
| Importar | `/importar` | admin, gestor, sdr (marketing: só mídia) | 4-003 |
| Indicadores | `/indicadores` (redireciona de `/dashboard`) | admin, gestor, sdr | 4-009 |
| Anúncios | `/midia` | admin, gestor, marketing | 4-009 |
| Feedback | `/feedback` | admin, gestor, marketing | 4-009 |
| Configurações | `/configuracoes` | admin (gestor: partes) | 4-008, 4-010, 4-011, 4-012, 4-013 |

A página inicial depois do login é **Minha fila** para `sdr`, **Indicadores** para `gestor` e `admin`, **Anúncios** para `marketing`.

Cabeçalho fixo: busca global, botão "Novo lead" (SPEC-4-002), sino de pendências (transbordos abertos, agendamentos a confirmar, duplicidades), menu do usuário.

**Busca global** (atalho `/`): procura por empresa, contato, telefone (em qualquer formato), e-mail e CNPJ, em leads e prospects. Resultados a partir de 3 caracteres, com estágio e responsável. Enter abre a ficha.

A tela de login deixa de anunciar recursos que não existem: os textos de vitrine passam a descrever o que o sistema faz. O nome exibido passa a ser o definido na T4.2.

## Ficha do lead

Rota `/leads/{id}`. Abre como página; a partir do Kanban e da fila, abre como painel lateral largo, com o botão "Abrir em página".

### Cabeçalho (sempre visível)

- Empresa e contato; selos: estágio, origem, produto, **Suprimido** (vermelho), **Transbordo aberto** (âmbar), **Histórico**.
- Telefone formatado com dois botões: **Copiar** e **Conversar**, que abre a conversa do lead dentro da Central (SPEC-4-013). Sem canal de WhatsApp conectado, o botão vira **Abrir no WhatsApp** (`https://wa.me/<telefone>` em nova aba).
- Responsável (trocável), próxima ação com data e atraso em dias, número de tentativas, última interação.
- Ações principais: **Registrar tentativa** (SPEC-4-005), **Mover estágio**, **Agendar apresentação** (SPEC-4-007), **Nova nota**, menu "Mais" (opt-out, transbordo, mesclar, exportar ficha).

### Aba Resumo

Duas colunas.

Esquerda — **Dados**, em seções, todos editáveis no lugar (clicar, editar, Enter salva, Esc cancela). As seções, a ordem, os rótulos e os campos exibidos vêm da configuração (SPEC-4-012): a ficha é desenhada a partir de `GET /backend/v1/fields`, e os campos personalizados aparecem na seção escolhida pelo administrador, com o controle adequado ao tipo. A tabela abaixo é a configuração inicial:

| Bloco | Campos |
|---|---|
| Empresa | empresa, nome fantasia, CNPJ, segmento, cidade, bairro, UF |
| Contato | nome, cargo, telefone, telefone secundário, e-mail |
| Aquisição | origem, detalhe da origem, campanha, conjunto, anúncio, UTM (somente leitura), data de entrada |
| Qualificação rápida | produto, sistema atual, regime tributário, situação da loja, usuários, lojas |
| Resultado | resultado pós-apresentação, valor da proposta, data de fechamento |

Direita — **Notas** e **Próximos passos**:

- Caixa "Escreva uma nota…" sempre aberta no topo. `Ctrl+Enter` salva.
- Lista das notas, da mais nova para a mais antiga, com autor e data. Uma nota pode ser **fixada** (fica no topo).
- Nota é imutável depois de 10 minutos; antes disso o autor pode corrigir. Não há exclusão: há "marcar como incorreta", que risca o texto e mantém o registro.
- Bloco "Próxima ação": texto, tipo, data e hora, com atalhos "hoje", "amanhã", "em 3 dias", "próxima segunda".

### Aba Atividades

Timeline unificada, da mais nova para a mais antiga, juntando:

| Tipo | Fonte |
|---|---|
| Mudança de estágio e de campo | `lead_stage_transitions`, `lead_critical_timeline` |
| Nota | `lead_notes` |
| Tentativa de contato | `lead_attempts` (SPEC-4-005) |
| Mensagem de conversa | `messages` |
| Transbordo, retomada, opt-out, sugestão | `conversation_timeline` |
| Agendamento e seus eventos | `appointments` (SPEC-4-007) |
| Uso de IA | `ai_runs` (SPEC-4-008) |
| Importação, captura, exportação em lista | lote ou evento de origem |

Filtro por tipo (caixas de seleção) e busca por texto. Cada item mostra quem, quando e o quê; mudança de campo mostra o valor de antes e o de depois.

### Aba Conversa

A conversa do lead, no mesmo componente da tela Conversas (SPEC-4-013): com canal conectado, a pessoa lê e responde daqui mesmo; sem canal, a aba mostra o histórico (importado ou recebido antes) em modo leitura. A tela antiga `ConversationView` é substituída por esta aba. Se não houver mensagens: "Nenhuma conversa com este lead." e o botão **Conversar**.

### Aba Qualificação

- Mostra o **ICP vigente** do produto do lead (`GET /backend/v1/icp`), com o aviso de versão.
- Uma linha por critério do ICP: pergunta, campo de resposta, situação (`atende`, `não atende`, `não informado`). Respostas que já existem nos dados do lead (segmento, regime, sistema atual, usuários) vêm preenchidas.
- Campo "Pendências" e "Observação da qualificação".
- Grava com `POST /backend/v1/leads/{leadId}/qualification`.
- Três botões de decisão, todos humanos (RN-03):
  - **Qualificar** — exige todos os critérios obrigatórios respondidos e uma evidência; move para `qualificado`.
  - **Desqualificar** — exige motivo do catálogo; se o motivo gera supressão, avisa antes.
  - **Nutrir** — exige motivo e data de retorno (sugerida pelo motivo).
- Se faltar critério obrigatório, o botão Qualificar fica desabilitado e a tela lista o que falta.
- Histórico das fichas anteriores do lead, com a versão do ICP usada em cada uma. Mudar o ICP não reclassifica nada.

### Supressão e transbordo

- **Registrar opt-out** (menu Mais): origem (`recusa_conversa`, `pedido_contato`, `cancelamento`, `manual`) e evidência (texto obrigatório). Usa `POST /backend/v1/conversations/{leadId}/opt-out`. Depois disso: selo "Suprimido" em toda tela, lead fora da fila e de qualquer lista, próxima ação apagada, botão "Registrar tentativa" desabilitado com a explicação.
- Remover supressão: só `admin`, com motivo, registrado na auditoria.
- **Transbordo:** se houver transbordo aberto, faixa âmbar no topo da ficha com motivo e contexto, e os botões **Assumir** e **Resolver** (com o que foi feito). "Abrir transbordo" manual no menu Mais. Usa `POST /backend/v1/conversations/{leadId}/handoff` e `GET /backend/v1/handoffs`.

## Tela Leads (lista)

Tabela de todos os leads, para busca e trabalho em massa.

- Colunas: empresa, contato, telefone, origem, produto, estágio, responsável, próxima ação, tentativas, última interação, entrada. O usuário escolhe quais ver.
- Filtros: estágio, origem, produto, responsável, período de entrada, "sem próxima ação", "atrasados", "suprimidos", "lote de importação", etiqueta.
- Ordenação por qualquer coluna. Paginação de 50.
- Seleção múltipla com ações: atribuir responsável, definir próxima ação, adicionar etiqueta, exportar selecionados.
- Filtros ficam no endereço da página, para poder salvar e compartilhar o link.

## Kanban versão 2

- **Seletor de funil** no topo, quando houver mais de um; o Kanban lembra o último funil usado pelo usuário.
- As colunas são as etapas ativas do funil, na ordem e nas cores configuradas (SPEC-4-012); nada é fixo no código. Etapas marcadas como recolhidas (no funil padrão, "Desqualificado" e "Nutrição") mostram só o contador; clicar expande. Etapas de categoria `agendado`, `ganho` e `perdido` mostram apenas os últimos 60 dias.
- Barra superior: busca, filtros (responsável — padrão "meus" para `sdr`, "todos" para `gestor` —, origem, produto, período), botão "limpar".
- **Arrastar e soltar** entre colunas. Soltar em etapa que exige dados — pela categoria (qualificar, desqualificar, nutrir, agendar) ou pelos campos obrigatórios da etapa — abre o diálogo correspondente, pedindo só o que falta. Os botões de mover continuam existindo, para teclado e leitor de tela.
- **Card:** o conteúdo é configurável por funil (SPEC-4-012). Configuração inicial: empresa (ou contato), contato, origem, produto, responsável (iniciais), próxima ação com data — em vermelho se atrasada, âmbar se for hoje —, número de tentativas, dias desde a última interação e **a última nota em até 2 linhas**. Selos: suprimido, transbordo, **parado** (passou do alerta de dias da etapa), **agente de IA ativo** e mensagens não lidas.
- Clique no card abre a ficha em painel lateral.
- Cada coluna carrega 30 cards e traz mais ao rolar. O contador da coluna mostra o total real.
- Atualização em tempo real quando outro usuário move um lead (o projeto já tem `use-realtime`).

## Dados

### Coleção nova `lead_notes`

`lead_id`, `texto` (até 5.000 caracteres), `autor_id`, `autor_role`, `fixada` (bool), `incorreta` (bool), `origem` (`manual`, `importacao`, `captura`, `ia`), `created`. Sem atualização depois de 10 minutos, exceto `fixada` e `incorreta`. Sem exclusão.

Ao gravar uma nota, o backend atualiza `leads.ultima_nota` com os primeiros 280 caracteres.

## API

| Rota | Papéis | Função |
|---|---|---|
| `GET /backend/v1/leads/{id}` | admin, gestor, sdr | ficha completa: dados, responsável, supressão, transbordo aberto, agendamento vigente, contadores. |
| `PATCH /backend/v1/leads/{id}` | admin, gestor, sdr | edita campos permitidos; valida com os normalizadores; exige `expectedVersion`; grava antes e depois na timeline. Estágio não é editável por esta rota. |
| `GET /backend/v1/leads` | admin, gestor, sdr | lista com filtros, ordenação e paginação. |
| `POST /backend/v1/leads/bulk` | admin, gestor; sdr só nos próprios | ações em massa, até 500 por chamada. |
| `GET /backend/v1/leads/search?q=` | admin, gestor, sdr | busca global, até 10 resultados. |
| `GET/POST /backend/v1/leads/{id}/notes` | admin, gestor, sdr | notas. |
| `PATCH /backend/v1/leads/{id}/notes/{noteId}` | autor (10 min) ou qualquer um para `fixada`/`incorreta` | |
| `GET /backend/v1/leads/{id}/activities` | admin, gestor, sdr | timeline unificada, paginada, com filtro por tipo. |

Rotas existentes que passam a ter tela: `GET /backend/v1/leads/{id}/timeline`, `GET /backend/v1/conversations/{leadId}`, `GET /backend/v1/conversations/{leadId}/timeline`, `GET/POST /backend/v1/leads/{leadId}/qualification`, `GET /backend/v1/icp`, `GET /backend/v1/leads/{leadId}/suppression`, `POST /backend/v1/conversations/{leadId}/opt-out`, `POST /backend/v1/conversations/{leadId}/handoff`, `GET /backend/v1/handoffs`.

## Regras

1. Edição concorrente: se dois usuários editarem o mesmo lead, o segundo recebe "Este lead foi alterado por <nome>. Recarregue para ver a versão atual." e nada é sobrescrito.
2. Telefone e e-mail aparecem inteiros para `admin`, `gestor` e `sdr` na ficha. Em exportações e capturas de tela de evidência, seguem mascarados como nas fases anteriores.
3. Mudar o telefone, o CNPJ ou o e-mail roda a verificação de duplicidade da SPEC-4-002.
4. `sdr` vê todos os leads, mas as ações em massa só valem para os que são dele. `gestor` e `admin` agem em todos.
5. Nenhum dado é apagado pela ficha.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Clicar em um card do Kanban | Ficha abre em painel lateral em menos de 1 segundo, com dados, notas e próxima ação. |
| Principal | Escrever uma nota e salvar | A nota aparece na ficha, no card do Kanban e na fila, sem recarregar. |
| Principal | Arrastar de "Em contato" para "Qualificado" | Abre a aba de qualificação; só move depois de salvar com critérios e evidência. |
| Limite | Buscar por `21 99999-8888`, `21999998888` ou `+5521999998888` | O mesmo lead aparece nas três. |
| Limite | Arrastar para coluna inválida | O card volta, com a mensagem do motivo. |
| Limite | Lead suprimido | Sem botão de tentativa; selo visível; não aparece na fila. |
| Falha | Qualificar com critério obrigatório do ICP sem resposta | Botão desabilitado e a pendência listada. |
| Falha | Dois usuários editando o mesmo campo | O segundo é avisado; nada é perdido. |

## Estados vazios, erro e permissão

- Ficha sem notas: "Nenhuma nota ainda. Registre o que foi conversado."
- Kanban sem leads: "Nenhum lead ainda." com os botões **Novo lead** e **Importar**.
- Coluna vazia com filtro ativo: "Nenhum lead com estes filtros."
- Produto sem ICP publicado: a aba Qualificação avisa e permite qualificar só com critério e evidência livres, como na Fase 1.

## Critérios de aceite

- [ ] **CA-4-10:** A ficha do lead abre a partir do Kanban, da fila, da busca, da lista e das conversas; todo campo é editável no lugar e cada alteração fica na aba Atividades com autor, data, valor anterior e novo; a busca global encontra o lead por nome, telefone em qualquer formato, e-mail ou CNPJ.
- [ ] **CA-4-11:** Uma nota escrita na ficha aparece na ficha, no card do Kanban e na fila, sem recarregar; notas não são apagadas.
- [ ] **CA-4-12:** A aba Atividades reúne, em ordem cronológica e com filtro por tipo, mudanças de etapa e de campo, notas, tentativas, mensagens, transbordos, agendamentos, ações de IA e de automação.
- [ ] **CA-4-13:** Qualificar, desqualificar e nutrir são feitos na ficha, com o ICP vigente do produto e motivo do catálogo; faltando critério obrigatório, a tela mostra a pendência e não avança; a versão do ICP usada fica registrada e o histórico não é reclassificado.
- [ ] **CA-4-14:** O opt-out é registrado em dois cliques, com origem e evidência; o lead suprimido mostra o selo em toda tela e sai das filas, listas e envios; transbordo aberto aparece na ficha e na fila, com assumir e resolver.
- [ ] **CA-4-15:** O Kanban tem seletor de funil, busca, filtros e arrastar entre colunas; o card mostra o conteúdo configurado, a próxima ação com atraso e a última nota; soltar em etapa que exige dados abre o diálogo pedindo só o que falta.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Roteiro da ficha | abrir, editar 3 campos, escrever nota, fixar, mover estágio | gravação de tela |
| Auditoria | conferir as 3 edições na aba Atividades, com antes e depois | captura |
| Busca | os três formatos de telefone | capturas |
| Supressão | registrar opt-out e conferir fila, Kanban e lista | capturas |
| Acessibilidade | percorrer o Kanban e mover um lead só com o teclado | gravação |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.9 | Reorganizar a navegação: menu lateral por papel, rotas novas com redirecionamento das antigas, cabeçalho com busca global e sino de pendências, página inicial por papel; retirar da tela de login os recursos que não existem | Dev frontend | SPEC-4-004 | CA-4-10 | Principal: menu de cada papel; limite: endereço proibido | capturas do menu por papel + busca pelos três formatos de telefone | T4.8 aceita | ☐ | B |
| T4.10 | Criar as rotas de leitura e edição do lead e a ficha (cabeçalho e aba Resumo) com edição no lugar, controle de edição concorrente e registro de antes e depois | Dev full-stack | SPEC-4-004 | CA-4-10 | RED/GREEN: edição concorrente; principal: editar 3 campos | gravação + aba Atividades com as 3 edições | T4.9 aceita | ☐ | B |
| T4.11 | Criar as notas do lead (coleção, rotas, caixa de nota na ficha, fixar, marcar como incorreta) e mostrar a última nota no card do Kanban e na fila | Dev full-stack | SPEC-4-004 | CA-4-11 | Principal: nota aparece nos três lugares; falha: tentativa de apagar | capturas da ficha, do card e da fila | T4.10 aceita | ☐ | B |
| T4.12 | Criar a rota e a aba Atividades, que reúnem em ordem cronológica mudanças, notas, tentativas, mensagens, transbordos, agendamentos, IA e automações, com filtro por tipo | Dev full-stack | SPEC-4-004 | CA-4-12 | Principal: um item de cada tipo | captura da aba com os filtros | T4.10 aceita | ☐ | B |
| T4.13 | Criar a aba Qualificação: ICP vigente do produto, respostas por critério, pendências e as decisões Qualificar, Desqualificar e Nutrir com motivo do catálogo, usando as rotas de qualificação e de ICP já existentes | Dev full-stack | SPEC-4-004 | CA-4-13 | RED/GREEN: critério obrigatório faltando; principal: as três decisões | capturas + registros das fichas com a versão do ICP | T4.10 aceita; T4.6 aceita | ☐ | B |
| T4.14 | Levar para a ficha o registro de opt-out e a remoção por administrador, o selo de suprimido em todas as telas e a faixa de transbordo com abrir, assumir e resolver, usando as rotas já existentes | Dev full-stack | SPEC-4-004 | CA-4-14 | Falha: ação em lead suprimido; principal: opt-out em dois cliques | capturas da ficha, do Kanban e da fila + auditoria | T4.10 aceita | ☐ | B |
| T4.15 | Refazer o Kanban: seletor de funil, colunas a partir dos dados, busca, filtros, arrastar e soltar com diálogo do que falta, card com próxima ação, atraso e última nota, carregamento por coluna e atualização em tempo real | Dev frontend | SPEC-4-004 | CA-4-15 | Principal: arrastar; limite: soltar em etapa que exige dados; acessibilidade por teclado | gravação com o mouse e só com o teclado | T4.10 aceita; T4.11 aceita | ☐ | B |
| T4.16 | Criar a tela Leads: tabela com colunas escolhidas, filtros no endereço, ordenação, paginação, seleção múltipla e ações em massa (responsável, próxima ação, etiqueta, exportar) | Dev full-stack | SPEC-4-004 | CA-4-10 | Principal: filtro e ação em massa; limite: ação em massa de sdr em lead de outro | capturas + registro das ações na auditoria | T4.10 aceita | ☐ | B |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
