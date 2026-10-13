# SPEC-4-007 — Agendamento da apresentação, briefing e resultado

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-11 (agendamento), RF-16, RF-17, RN-01, RN-09, RN-10, G7  
**Degrau da solução:** construção — o agendamento é registrado na Central e sai como convite de calendário padrão; não há sincronização com agenda externa.

## Resultado observável

O lead qualificado é agendado com data, hora, apresentador e confirmação. O apresentador recebe um briefing de uma página com tudo o que precisa saber. Reagendar, cancelar ou marcar falta não apaga nada, e o que aconteceu depois da apresentação fica anotado para medir resultado.

## Por que existe

- O limite funcional do projeto é a apresentação agendada (RN-01), e o sistema não tem onde registrá-la.
- A meta de referência do projeto inclui apresentações **realizadas** por mês; sem registrar se a apresentação aconteceu, não há como medir.
- A Lidiane usa a planilha também para proposta enviada, valor e fechamento. Se esses três dados não tiverem lugar na Central, a planilha continua viva.

## Limites e dependências

- **Inclui:** registro de agendamento, estados e confirmação, reagendamento, cancelamento, falta e realização, tela Agenda, briefing do apresentador, convite `.ics`, resultado pós-apresentação informativo.
- **Fora de escopo:** proposta, contrato, assinatura (Autentique), faturamento (Conexa), implantação — continuam fora do projeto. Os campos de resultado são anotações, sem fluxo, sem alerta e sem integração.
- **Pré-condições:** SPEC-4-001 e SPEC-4-004 aceitas.
- **Gate G7 (agenda externa):** não bloqueia. O convite `.ics` funciona em Outlook, Google Agenda e Teams sem integração. A criação direta do evento na agenda Google fica como opcional, se o conector estiver habilitado (`integracoes.md`, seção 6).
- **Agentes de IA:** um agente pode propor horários e registrar o pedido de apresentação como **solicitado**, se o administrador ligar essas capacidades (SPEC-4-008). A confirmação continua sendo de uma pessoa.
- **Decisões da Pontual na T4.2:** quem apresenta cada produto, duração padrão, se a apresentação é online ou presencial por padrão e quem confirma.

## Telas e comportamento

### Agendar apresentação

Botão na ficha, disponível para lead em `qualificado` ou `agendamento_solicitado`. Também abre ao arrastar o card para "Agendamento solicitado" ou "Agendado", e ao registrar tentativa com resultado "Agendou".

| Campo | Regra |
|---|---|
| Data e hora | obrigatório; não pode ser no passado; fuso `America/Sao_Paulo` |
| Duração | padrão 60 minutos |
| Apresentador | obrigatório; lista de usuários ativos; padrão conforme o produto (configurável) |
| Modalidade | Online · Presencial · Telefone |
| Link ou local | obrigatório para online (link) e presencial (endereço) |
| Participantes do cliente | nome e papel de cada um; pelo menos um |
| Produto | vem do lead; editável |
| Contexto para a apresentação | texto; o que o cliente quer ver, dores, objeções |
| Já está confirmado com o cliente? | sim ou não |

Salvar com "não confirmado": o agendamento fica `solicitado`, o lead vai para `agendamento_solicitado` e aparece na fila em "Agendamentos a confirmar".

Salvar com "confirmado", ou clicar depois em **Confirmar**: pede "como foi confirmado" (WhatsApp, ligação, e-mail) e grava quem confirmou e quando. O agendamento fica `confirmado` e o lead vai para `agendado`.

Se o lead estiver em `em_contato` ou antes, o botão avisa: "Qualifique o lead antes de agendar." e leva à aba Qualificação.

### Cartão do agendamento na ficha

No cabeçalho da ficha, quando há agendamento vigente: data, hora, apresentador, modalidade, link, situação, e os botões **Confirmar**, **Reagendar**, **Cancelar**, **Baixar convite**, **Ver briefing**, e — depois da hora marcada — **Aconteceu** e **Cliente faltou**.

| Ação | Pede | Efeito |
|---|---|---|
| Reagendar | nova data e hora, motivo | novo evento ligado ao anterior; o anterior fica `reagendado`; o novo nasce `solicitado` (ou `confirmado`, se marcado) |
| Cancelar | motivo, e o destino do lead: voltar para qualificado · nutrição (com data) · desqualificar (com motivo) | evento fica `cancelado`; lead muda conforme a escolha |
| Aconteceu | observação opcional | evento fica `realizado`; `resultado_pos = apresentacao_realizada` |
| Cliente faltou | observação opcional | evento fica `no_show`; lead volta para `agendamento_solicitado` com próxima ação "Reagendar apresentação" para hoje |

### Resultado pós-apresentação

Bloco "Resultado" da ficha (SPEC-4-004), habilitado depois de uma apresentação realizada: **Proposta enviada**, **Ganho**, **Perdido** (com motivo do catálogo, grupo `perda`), valor da proposta (texto), data de fechamento. É só registro. Não muda o estágio, não cria tarefa, não notifica ninguém.

### Tela Agenda

Rota `/agenda`.

- Duas visões: **Semana** (grade de segunda a sexta, 8h às 19h) e **Lista**.
- Filtros: apresentador, situação, produto.
- Cada compromisso mostra hora, empresa, apresentador e situação por cor: solicitado (âmbar), confirmado (verde), realizado (cinza), faltou (vermelho), cancelado (riscado).
- Clique abre a ficha no cartão do agendamento.
- Topo da tela: "a confirmar", "esta semana", "realizadas no mês", "faltas no mês".
- Aviso de conflito: ao agendar no mesmo horário de outro compromisso do mesmo apresentador, a tela avisa e deixa prosseguir.

### Briefing do apresentador

Botão **Ver briefing** no cartão do agendamento. Página de uma folha, para ler antes da reunião, imprimir ou copiar.

| Bloco | Conteúdo | Fonte |
|---|---|---|
| Compromisso | data, hora, duração, modalidade, link ou local, participantes | agendamento |
| Empresa | nome, segmento, cidade, UF, lojas, usuários, situação da loja | lead |
| Situação atual | sistema atual, regime tributário | lead |
| Produto e interesse | produto, contexto para a apresentação | agendamento |
| Qualificação | critérios do ICP atendidos e não atendidos, com as respostas; pendências | última ficha de qualificação |
| Como chegou | origem, campanha, data de entrada, quem atendeu | lead |
| Histórico resumido | número de tentativas, as 5 últimas notas, pontos de atenção | notas e tentativas; com o botão "Resumir com IA" (SPEC-4-008), opcional |

O briefing **não** mostra: telefone e e-mail de pessoas que não participarão, campos de UTM, histórico de auditoria, dados de outros leads. Usuário que não é o apresentador nem o responsável vê o briefing sem telefone e e-mail (RNF-04).

Botões: **Copiar texto**, **Imprimir**, **Baixar convite**.

Abrir o briefing grava na auditoria quem abriu e quando.

### Convite `.ics`

Arquivo de calendário padrão, gerado no navegador:

- título: "Apresentação <produto> — <empresa>";
- início e fim com fuso `America/Sao_Paulo`;
- local: o link ou o endereço;
- descrição: participantes e o contexto (sem telefone, sem dados de qualificação);
- identificador estável por agendamento, para que o reagendamento **atualize** o evento na agenda de quem importar de novo, em vez de criar outro.

Ao lado do botão: **Enviar na conversa** (leva o texto do convite, com data, hora e link em português, ao campo de escrever da conversa do lead — SPEC-4-013) e **Copiar texto do convite**.

## Dados

### Coleção nova `appointments`

Cada linha é um **evento** do agendamento. Sem atualização e sem exclusão, exceto o campo `status` do evento anterior quando um novo o substitui (regra 2).

| Campo | Tipo | Regra |
|---|---|---|
| `lead_id` | texto | |
| `serie_id` | texto | o mesmo para todos os eventos de uma apresentação (do primeiro pedido ao desfecho) |
| `anterior_id` | texto | evento que este substitui |
| `inicio` | data e hora | |
| `duracao_min` | número | 15 a 480 |
| `apresentador_id` | relação `users` | |
| `modalidade` | select `online`, `presencial`, `telefone` | |
| `local` | texto 500 | link ou endereço |
| `participantes` | json | lista de `{nome, papel}` |
| `produto` | código do catálogo | |
| `contexto` | texto 3.000 | |
| `status` | select `solicitado`, `confirmado`, `reagendado`, `cancelado`, `realizado`, `no_show` | |
| `confirmado_por`, `confirmado_em`, `confirmado_como` | texto, data, select | preenchidos na confirmação |
| `motivo` | texto 500 | reagendamento ou cancelamento |
| `observacao` | texto 2.000 | desfecho |
| `actor_id`, `actor_role`, `created` | | |

## API

| Rota | Papéis | Função |
|---|---|---|
| `POST /backend/v1/leads/{id}/appointments` | admin, gestor, sdr | cria; valida os campos obrigatórios no servidor (RN-09); ajusta o estágio do lead. |
| `POST /backend/v1/appointments/{id}/confirm` | admin, gestor, sdr | confirma. |
| `POST /backend/v1/appointments/{id}/reschedule` | admin, gestor, sdr | cria o novo evento e marca o anterior. |
| `POST /backend/v1/appointments/{id}/cancel` | admin, gestor, sdr | cancela e move o lead. |
| `POST /backend/v1/appointments/{id}/outcome` | admin, gestor, sdr | `realizado` ou `no_show`; só depois do horário de início. |
| `GET /backend/v1/appointments?from=&to=&presenter=&status=` | admin, gestor, sdr | agenda. |
| `GET /backend/v1/leads/{id}/appointments` | admin, gestor, sdr | todos os eventos do lead. |
| `GET /backend/v1/appointments/{id}/briefing` | admin, gestor, sdr | conteúdo do briefing, já sem o que o papel não pode ver; grava o acesso. |
| `PATCH /backend/v1/leads/{id}/outcome` | admin, gestor, sdr | resultado pós-apresentação. |

## Regras

1. **"Agendado" só conta com data, hora, apresentador e confirmação registrada** (RN-09). Agendamento `solicitado` não entra no indicador de agendados.
2. **Nada é apagado** (RN-10). Reagendar e cancelar criam registro novo ou mudam só o `status` do evento que deixou de valer.
3. **Um agendamento vigente por lead.** Criar outro com um vigente em aberto é recusado: "Este lead já tem apresentação marcada. Reagende ou cancele."
4. **Indicadores** (usados na SPEC-4-009): *agendamentos* = séries que chegaram a `confirmado`, contadas na data da confirmação; *realizadas* = séries com `realizado`, na data do evento; *faltas* = eventos `no_show`; *taxa de comparecimento* = realizadas ÷ (realizadas + faltas).
5. O funil do projeto termina no primeiro agendamento confirmado (RN-01). O que vem depois é anotação.
6. Lead suprimido não pode ser agendado; se o lead for suprimido com agendamento em aberto, a ficha pede o cancelamento.
7. Todos os horários são gravados em UTC e exibidos em `America/Sao_Paulo`.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Lead qualificado, agendamento completo e confirmado | Lead em "Agendado"; aparece na Agenda em verde; conta no indicador. |
| Principal | Reagendar | Dois eventos na ficha, o antigo como reagendado; a Agenda mostra só o novo; o indicador conta uma apresentação, não duas. |
| Principal | Marcar "Aconteceu" e depois "Proposta enviada" com valor | Ficha mostra o desfecho; estágio continua "Agendado". |
| Limite | Agendamento salvo sem confirmação | Lead em "Agendamento solicitado"; item na fila; não conta como agendado. |
| Limite | Mesmo horário de outro compromisso do apresentador | Aviso de conflito; permite seguir. |
| Limite | Marcar "Aconteceu" antes do horário | Negado. |
| Falha | Agendar sem apresentador ou sem data | Erro no campo; nada é salvo. |
| Falha | Chamar a rota de exclusão de um evento | Negado e auditado. |
| Recuperação | Cliente faltou | Lead volta para "Agendamento solicitado" com ação para hoje. |

## Estados vazios, erro e permissão

- Agenda sem compromissos na semana: "Nenhuma apresentação nesta semana."
- `marketing` não acessa a Agenda nem o briefing.

## Critérios de aceite

- [ ] **CA-4-24:** Um lead só conta como agendado com data, hora, apresentador e confirmação registrada; agendamento sem confirmação aparece como pendência na fila e não entra no indicador; a tela Agenda mostra a semana por apresentador.
- [ ] **CA-4-25:** Reagendar, cancelar, registrar falta e registrar que aconteceu criam eventos novos sem apagar os anteriores; os indicadores contam uma apresentação por série; o resultado depois da apresentação é anotação e não cria fluxo.
- [ ] **CA-4-26:** O briefing da apresentação cabe em uma página, com produto, contexto, critérios do ICP atendidos, pendências e histórico resumido, sem dado desnecessário; o convite .ics abre no Outlook e no Google Agenda com a hora correta e é atualizado ao reagendar.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Regras de contagem | testes com `fixtures/agendamentos-fase4.csv` (completo, sem confirmação, reagendado, cancelado, faltou, realizado) | saída dos testes e indicador |
| Convite | abrir o `.ics` no Outlook e no Google Agenda | duas capturas com a hora correta |
| Reagendamento no calendário | importar o convite, reagendar, importar de novo | um único evento, atualizado |
| Briefing | abrir como apresentador e como outro usuário | duas capturas, a segunda sem telefone e e-mail |
| Histórico | reagendar e cancelar, depois listar os eventos | todos presentes |

**Dados de teste:** `fixtures/agendamentos-fase4.csv`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.36 | Criar o agendamento: coleção de eventos, formulário, validação dos campos obrigatórios no servidor, estados solicitado e confirmado, ajuste da etapa do lead e cartão do agendamento na ficha | Dev full-stack | SPEC-4-007 | CA-4-24 | RED/GREEN: agendamento sem confirmação não conta; falha: sem apresentador ou sem data | testes com fixtures/agendamentos-fase4.csv + capturas | T4.13 aceita; T4.7 aceita | ☐ | G |
| T4.37 | Implementar reagendar, cancelar, registrar falta e registrar que aconteceu, sem apagar eventos, com a contagem por série; e o bloco de resultado depois da apresentação | Dev full-stack | SPEC-4-007 | CA-4-25 | RED/GREEN: tentativa de apagar evento; principal: reagendar conta uma apresentação | lista de eventos do lead + indicador | T4.36 aceita | ☐ | G |
| T4.38 | Criar o briefing da apresentação (com o que cada papel pode ver e o registro de acesso) e o convite .ics com fuso e identificador estável | Dev full-stack | SPEC-4-007 | CA-4-26 | Principal: abrir como apresentador e como outro usuário; convite no Outlook e no Google Agenda; reagendar e reimportar | capturas dos dois briefings + dos dois calendários | T4.37 aceita | ☐ | G |
| T4.39 | Criar a tela Agenda nas visões semana e lista, com filtros, cores por situação, contadores e aviso de conflito de horário | Dev frontend | SPEC-4-007 | CA-4-24 | Principal: semana com compromissos em todas as situações; limite: conflito | capturas das duas visões | T4.37 aceita | ☐ | G |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
