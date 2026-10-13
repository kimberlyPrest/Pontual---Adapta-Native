# SPEC-4-005 — Minha fila, tentativas de contato e cadência de follow-up

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-05, RF-07 (cadência), RF-08, RF-13, RN-05, RN-07, RN-08, RN-13, G4  
**Degrau da solução:** construção — a régua é uma configuração simples, editável pelo administrador. Cada tentativa pode ser manual (padrão) ou enviada automaticamente pelo WhatsApp, quando houver canal conectado e liberado (SPEC-4-013).

## Resultado observável

A Lidiane abre o sistema de manhã e vê, em uma tela, tudo o que precisa fazer hoje, em ordem de prioridade. Ao registrar que tentou falar com alguém, o sistema já propõe quando e por qual canal tentar de novo. Nenhum lead ativo fica esquecido.

## Por que existe

- A cadência hoje é controlada nas colunas "1ª Tentativa", "2ª Tentativa", "3ª Tentativa" e "Última interação" da planilha, preenchidas à mão.
- O motor de cadência do RespondeChat dispara as mensagens de vários dias de uma vez; a operação precisa de um controle confiável de **quando** falar com cada lead, mesmo que o envio continue manual.
- As abas "Rotina Diária" e "Planejamento" da planilha existem para responder "o que eu faço hoje".
- A rota `GET /backend/v1/leads/queue` existe, mas só lista leads sem próxima ação.

## Limites e dependências

- **Inclui:** registro de tentativa, régua de cadência por origem, proposta de próxima ação, tela Minha fila, encerramento por fim de cadência, nutrição com data de retorno, prazo de primeiro contato.
- **Fora de escopo desta SPEC:** o mecanismo de envio pelo WhatsApp e o modo automático da régua (SPEC-4-013); modelos de mensagem (SPEC-4-010); sugestão de texto e agentes (SPEC-4-008). Esta SPEC entrega a fila, o registro de tentativas e o cálculo da régua, que funcionam com ou sem canal conectado.
- **Pré-condições:** SPEC-4-001 e SPEC-4-004 aceitas.
- **Gate G4 (cadência homologada):** os valores iniciais da régua abaixo são provisórios. A Pontual confirma ou altera na T4.2, pela tela, sem código.
- **Risco e plano B:** se a régua proposta não servir, o administrador desliga a proposta automática; o registro de tentativa e a fila continuam funcionando com data manual.

## Telas e comportamento

### Registrar tentativa

Botão na ficha, no card da fila e no menu do card do Kanban. Abre um diálogo pequeno:

| Campo | Opções | Padrão |
|---|---|---|
| Canal | WhatsApp · Ligação · E-mail · Presencial · Outro | o canal previsto na régua para esta tentativa |
| Resultado | Sem resposta · Respondeu · Pediu retorno · Número inválido · Recusou · Agendou | Sem resposta |
| Nota | texto livre | vazio |
| Quando | data e hora | agora |

Um clique em **Salvar** fecha o registro. Depois de salvar, o diálogo mostra a etapa seguinte conforme o resultado:

| Resultado | O que o sistema faz |
|---|---|
| Sem resposta | propõe a próxima ação pela régua: "Tentar de novo por **Ligação** em **qui., 15/10, 10h**". Botões: **Aceitar**, **Ajustar**, **Sem próxima ação**. |
| Respondeu | move para `em_contato` se estava antes disso; pede a próxima ação (padrão: "Continuar conversa", amanhã). |
| Pediu retorno | pede a data; cria a próxima ação nessa data. |
| Número inválido | marca o telefone como inválido; se houver telefone secundário ou e-mail, propõe tentar por lá; se não, propõe desqualificar com o motivo `contato_invalido`. |
| Recusou | abre o registro de opt-out (SPEC-4-004) já preenchido com origem `recusa_conversa` e a nota como evidência. |
| Agendou | abre o formulário de agendamento (SPEC-4-007). |

Se a tentativa for a última da régua e o resultado for "Sem resposta", o diálogo mostra: "Esta foi a 3ª tentativa sem resposta. O que fazer?" — **Encerrar como sem resposta**, **Mover para nutrição** (com data), **Tentar mais uma vez**. Nada é encerrado sem o clique (RN-03).

### Tela Minha fila

Rota `/fila`. Página inicial do `sdr`.

Topo: contadores do dia — a fazer, feitos hoje, atrasados — e o seletor "Minha fila / Fila de <usuário> / Todos" (o seletor só para `gestor` e `admin`).

Blocos, nesta ordem de prioridade. Bloco vazio não aparece.

| # | Bloco | Entra quando |
|---|---|---|
| 1 | **Transbordos** | transbordo aberto atribuído ao usuário ou sem dono |
| 2 | **Novos sem primeiro contato** | `stage = novo`, `tentativas_count = 0`. Mostra há quanto tempo o lead entrou e o prazo (ver régua); em vermelho quando estoura. |
| 3 | **Agendamentos a confirmar** | agendamento em `solicitado` (SPEC-4-007) |
| 4 | **Atrasados** | `next_action_at` antes de hoje |
| 5 | **Para hoje** | `next_action_at` hoje, em ordem de hora |
| 6 | **Retorno de nutrição** | `stage = nutricao` e `nutricao_retorno_em` até hoje |
| 7 | **Sem próxima ação** | lead ativo sem `next_action_at` (RN-05) |
| 8 | **A corrigir** | lead ativo sem responsável, ou com telefone inválido |
| 9 | **Próximos 7 dias** | recolhido; só para consulta |

Estágios ativos: `novo`, `triagem`, `em_contato`, `qualificado`, `agendamento_solicitado`. Ficam fora da fila: `prospect`, `historico`, `suprimido`.

Cada item mostra: empresa e contato, origem, estágio, a ação a fazer, a tentativa em que está ("2 de 3"), a última nota, e os botões **Conversar** (abre a conversa na Central; sem canal conectado, abre `wa.me`), **Copiar telefone**, **Registrar tentativa**, **Adiar** (amanhã, em 3 dias, escolher data) e **Abrir ficha**.

Depois de registrar a tentativa, o item sai da lista com animação e o contador "feitos hoje" sobe. A fila não recarrega inteira.

### Configuração da régua

Em Configurações → Cadência (tela na SPEC-4-010). Uma régua padrão e, opcionalmente, uma por origem.

## Dados

### Coleção nova `lead_attempts`

| Campo | Tipo | Regra |
|---|---|---|
| `lead_id` | texto | |
| `numero` | número | sequência dentro do ciclo atual de cadência |
| `ciclo` | número | começa em 1; sobe quando o lead volta de nutrição ou é reaberto |
| `canal` | select `whatsapp`, `ligacao`, `email`, `presencial`, `outro` | |
| `resultado` | select `sem_resposta`, `respondeu`, `pediu_retorno`, `numero_invalido`, `recusou`, `agendou` | |
| `nota` | texto 2.000 | |
| `ocorreu_em` | data | não pode ser no futuro |
| `actor_id`, `actor_role` | texto | |
| `origem` | select `manual`, `importacao`, `lista` | `lista` = exportação de disparo (SPEC-4-006) |
| `created` | autodate | |

Sem atualização e sem exclusão. Erro de registro se corrige com nova tentativa e nota.

Ao gravar, o backend atualiza no lead: `tentativas_count` (do ciclo atual), `ultima_interacao_at`, `ultima_interacao_canal`.

### Coleção nova `cadence_rules`

| Campo | Tipo | Valor inicial (provisório, G4) |
|---|---|---|
| `escopo` | select `padrao`, `origem` | `padrao` |
| `origem` | código do catálogo | vazio |
| `ativo` | bool | verdadeiro |
| `max_tentativas` | número 1–10 | 3 |
| `intervalos_dias` | json, lista | `[1, 2, 4]` — dias úteis entre a entrada e a 1ª, a 1ª e a 2ª, a 2ª e a 3ª |
| `canais` | json, lista | `["whatsapp", "ligacao", "whatsapp"]` |
| `janela_inicio`, `janela_fim` | hora | 09:00, 18:00 |
| `dias_uteis` | json | segunda a sexta |
| `prazo_primeiro_contato_horas` | número | 2 (horas úteis) |
| `ao_esgotar` | select `perguntar`, `sugerir_encerrar`, `sugerir_nutrir` | `perguntar` |
| `nutricao_padrao_dias` | número | 45 |
| `modos` | json, lista | `["manual", "manual", "manual"]` — cada posição é `manual` ou `automatica` (SPEC-4-013) |
| `modelos` | json, lista | modelo de mensagem de cada tentativa automática |
| `versao` | número | sobe a cada alteração; a anterior fica guardada |

Dados iniciais: a régua padrão acima e uma régua para `indicacao` com prazo de primeiro contato de 1 hora e 5 tentativas (`[0, 1, 2, 3, 5]`, canais alternados começando por WhatsApp).

## API

| Rota | Papéis | Função |
|---|---|---|
| `POST /backend/v1/leads/{id}/attempts` | admin, gestor, sdr | registra a tentativa; devolve a tentativa, os contadores e a `proposta` de próxima ação (data, hora, canal, texto, `ultima: bool`). |
| `GET /backend/v1/leads/{id}/attempts` | admin, gestor, sdr | lista. |
| `GET /backend/v1/queue?owner=` | admin, gestor, sdr | a fila em blocos, com contadores. Substitui o uso de `/leads/queue` pela tela; a rota antiga continua respondendo. |
| `POST /backend/v1/leads/{id}/snooze` | admin, gestor, sdr | adia a próxima ação; grava na timeline. |
| `GET /backend/v1/cadence` | admin, gestor, sdr | réguas vigentes. |
| `PUT /backend/v1/cadence/{id}` | admin, gestor | altera; cria nova versão. |

A rota existente `POST /backend/v1/leads/{id}/next-action` continua sendo a que grava a próxima ação aceita ou ajustada.

## Regras

1. **Cálculo da proposta:** pega a régua da origem do lead ou, na falta, a padrão. A data é a da tentativa mais o intervalo da próxima posição, contando só dias úteis configurados. A hora é a mesma da tentativa; se cair antes do início da janela, vai para o início da janela; se cair depois do fim, vai para o início da janela do próximo dia útil. O canal é o da posição na régua.
2. **A proposta nunca é gravada sozinha.** Vira próxima ação só quando o usuário aceita ou ajusta.
3. **Supressão prevalece (RN-07):** lead suprimido não recebe proposta, não entra na fila e a rota de tentativa o recusa.
4. **Transbordo pausa:** lead com transbordo aberto aparece só no bloco 1 até ser resolvido.
5. **Mudança de régua** vale para as próximas propostas. Próximas ações já gravadas não mudam (RN-13).
6. **Prazo de primeiro contato** conta em horas úteis a partir da entrada. O estouro é só visual na fila e vira indicador (SPEC-4-009); não bloqueia nada.
7. **Nutrição:** na data de retorno, o lead entra no bloco 6. A ação "Retomar" move para `em_contato`, abre o ciclo 2 e zera a contagem de tentativas do ciclo; "Adiar" pede nova data; "Encerrar" pede motivo.
8. **Feriados** não são tratados nesta fase; o usuário ajusta a data proposta quando cair em feriado.
9. A fila de um usuário só mostra leads em que ele é o responsável, mais os blocos 1 e 8, que mostram também os sem dono.
10. **Tentativa registrada sozinha:** com canal de WhatsApp conectado, a primeira mensagem enviada a um lead no dia cria a tentativa automaticamente, e a resposta do contato muda o resultado para "respondeu" (SPEC-4-013, regra 6). O registro manual continua existindo para ligação, presencial e para quando não houver canal.
11. **Conversa não lida conta como pendência:** lead com mensagem do contato sem resposta aparece no topo do bloco "Para hoje", com o selo "respondeu", e some quando alguém responde.
12. **Etapas e categorias:** a fila usa a categoria da etapa (SPEC-4-001), de modo que continua correta quando o administrador muda o funil.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | 1ª tentativa por WhatsApp, sem resposta, numa terça às 10h, régua padrão | Proposta: ligação na quinta às 10h. |
| Principal | Aceitar a proposta | Próxima ação gravada; item sai de "Para hoje"; "feitos hoje" sobe 1. |
| Limite | Tentativa registrada na sexta às 17h50, intervalo de 2 dias úteis | Proposta para terça às 17h50. |
| Limite | 3ª tentativa sem resposta | Pergunta o que fazer; nenhuma mudança de estágio sem clique. |
| Limite | Lead em nutrição com retorno hoje | Aparece em "Retorno de nutrição". |
| Limite | Lead novo há 3 horas úteis sem tentativa | Em "Novos sem primeiro contato", marcado como fora do prazo. |
| Falha | Registrar tentativa em lead suprimido | Negado: "Este contato pediu para não ser procurado." |
| Falha | Data da tentativa no futuro | Erro no campo. |
| Recuperação | Administrador desliga a régua | Diálogo passa a pedir a data manualmente; fila continua. |

## Estados vazios, erro e permissão

- Fila vazia: "Tudo em dia. Nenhuma ação pendente para hoje.", com o atalho para "Próximos 7 dias".
- `marketing` não acessa `/fila`.

## Critérios de aceite

- [ ] **CA-4-16:** Registrar uma tentativa de contato leva no máximo três cliques (canal, resultado, salvar); contadores e última interação atualizam na hora.
- [ ] **CA-4-17:** Depois de uma tentativa sem resposta, o sistema propõe a próxima ação pela régua (dia, hora e canal), dentro da janela e dos dias úteis; o usuário aceita ou ajusta; a régua é alterada pelo administrador na tela, sem código.
- [ ] **CA-4-18:** A tela Minha fila mostra, em ordem de prioridade, transbordos, novos sem primeiro contato com o prazo, agendamentos a confirmar, atrasados, de hoje, retornos de nutrição, sem próxima ação e a corrigir; cada item é resolvido na própria fila; nenhum lead ativo fica fora dela.
- [ ] **CA-4-19:** Ao esgotar as tentativas, o sistema pergunta o que fazer (encerrar, nutrir com data, tentar de novo); nada é encerrado sem confirmação humana; lead em nutrição reaparece na fila na data de retorno.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Cálculo da régua | testes unitários com `fixtures/cadencias-fase4.csv` (dia útil, janela, fim de semana, última tentativa) | saída dos testes |
| Três cliques | registrar tentativa a partir da fila | gravação |
| Cobertura da fila | somar os leads ativos de um usuário e comparar com a soma dos blocos | tabela com os dois números |
| Supressão | tentativa em lead suprimido | resposta negada e auditoria |

**Dados de teste:** `fixtures/cadencias-fase4.csv`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.32 | Criar as tentativas de contato: coleção sem alteração nem exclusão, rotas, diálogo de registro com os desfechos por resultado e os contadores no lead | Dev full-stack | SPEC-4-005 | CA-4-16 | RED/GREEN: data futura e lead suprimido; principal: três cliques | gravação + contadores atualizados | T4.10 aceita; T4.14 aceita | ☐ | F |
| T4.33 | Implementar a régua de cadência (padrão e por origem, com versões) e o cálculo da proposta de próxima ação com dias úteis e janela de horário | Dev backend | SPEC-4-005 | CA-4-17 | RED/GREEN: fixtures/cadencias-fase4.csv (dia útil, janela, fim de semana, última tentativa) | saída dos testes | T4.32 aceita | ☐ | F |
| T4.34 | Criar a tela Minha fila com os blocos em ordem de prioridade, o prazo do primeiro contato, as ações no próprio item (conversar, registrar tentativa, adiar) e o seletor de fila para gestor e administrador | Dev full-stack | SPEC-4-005 | CA-4-18 | Principal: resolver um item de cada bloco; prova de cobertura: leads ativos × soma dos blocos | tabela com os dois números + gravação | T4.33 aceita; T4.11 aceita | ☐ | F |
| T4.35 | Implementar o fim da cadência (perguntar, encerrar, nutrir, tentar de novo), a nutrição com data de retorno e o retorno à fila com novo ciclo de tentativas | Dev full-stack | SPEC-4-005 | CA-4-19 | Limite: última tentativa; principal: retorno de nutrição na data | capturas do diálogo + lead de volta à fila | T4.34 aceita | ☐ | F |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
