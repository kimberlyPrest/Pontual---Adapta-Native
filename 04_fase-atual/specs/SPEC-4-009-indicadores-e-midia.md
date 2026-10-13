# SPEC-4-009 — Indicadores, produtividade, dashboard de anúncios e relatório do parceiro

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-12 (indicadores), RF-13 (feedback), RF-18, RF-19, RF-20, RF-21, RN-02, RN-11, RN-12, G3  
**Degrau da solução:** evolução — os painéis das fases 1 e 2 passam a cobrir o funil até o agendamento, a ler dado real, e o painel de mídia vira um dashboard de anúncios que desce até o anúncio e cruza o gasto com o que aconteceu com cada lead.

## Resultado observável

O Marcelo abre **Indicadores** e vê, por período e por origem, quantos leads entraram, quantos foram qualificados, quantos viraram apresentação e quanto custou cada um. Clicar em qualquer número mostra os leads que o compõem. A Lidiane deixa de preencher as abas "Rotina Diária" e "Resumo Semanal": o sistema calcula.

## Por que existe

- A dor que originou o projeto é não saber quais canais trazem lead qualificado: R$ 3.000 a R$ 4.000 por mês em mídia e 3 leads qualificados em três meses.
- O dashboard atual para em "qualificado" e mede só fixtures.
- A importação de mídia (`POST /backend/v1/media/import`) não tem tela; o investimento não tem como entrar.
- A planilha tem abas preenchidas à mão todo dia (até as 18h) e toda sexta-feira com contagem de mensagens, ligações, follow-ups, respostas e reuniões.
- A aba "Leads Hiper - Tráfego Pago" é compartilhada com a gerência do parceiro como prestação de contas.

## Limites e dependências

- **Inclui:** painel de funil, painel de produtividade, dashboard de anúncios em três níveis (campanha, conjunto, anúncio), entrada do gasto por sincronização com as contas de anúncio, por relatório importado ou por lançamento manual, atribuição do lead ao anúncio, CPLQ e custo por agendamento com dado real, recomendações explicáveis, feedback para o Marketing, relatório do parceiro, cadastro de metas, exportações.
- **Fora de escopo:** atribuição multitoque (vale o primeiro anúncio que trouxe o lead); alteração de campanha, orçamento ou anúncio pelo sistema — o dashboard lê, não escreve; criação de anúncios.
- **Risco e plano B:** a sincronização com o Google Ads exige uma credencial de desenvolvedor que o Google aprova em prazo que não controlamos; a do Meta exige um usuário de sistema com permissão de leitura na conta de anúncios. Se qualquer uma atrasar, aquele canal entra por **relatório importado** (o arquivo que a própria plataforma exporta), que alimenta as mesmas tabelas e as mesmas telas. Nenhuma tela depende de a sincronização existir.
- **Pré-condições:** SPEC-4-001, SPEC-4-005 e SPEC-4-007 aceitas (as tentativas e os agendamentos são a fonte dos números); SPEC-4-013 para a atribuição pelo WhatsApp; SPEC-4-011 para o registro das sincronizações.
- **Gate G3 (baseline e metas):** as metas aparecem nos painéis **somente** depois de cadastradas pela Pontual (RN-12). Sem meta cadastrada, o painel mostra só o realizado. As referências do escopo (20 qualificados, 10 apresentações, CPLQ de até R$ 200; ou 25, 8 e R$ 160) são levadas para decisão na T4.2, não são pré-cadastradas.

## Definições (dicionário de indicadores)

Estas definições aparecem na própria tela, no ícone de ajuda de cada número.

| Indicador | Definição | Data de referência |
|---|---|---|
| Entradas | leads (`tipo = lead`) criados no período | `data_entrada` |
| Primeiro contato | leads com pelo menos uma tentativa | data da 1ª tentativa |
| Tempo até o primeiro contato | horas úteis entre a entrada e a 1ª tentativa; mediana | entrada |
| Dentro do prazo | % de leads cujo primeiro contato ocorreu no prazo da régua | entrada |
| Qualificados | leads que chegaram a `qualificado` | data da transição |
| Taxa de qualificação | qualificados ÷ entradas do mesmo grupo de entrada | entrada |
| Agendamentos | séries de agendamento que chegaram a `confirmado` | data da confirmação |
| Apresentações realizadas | séries com `realizado` | data da apresentação |
| Taxa de comparecimento | realizadas ÷ (realizadas + faltas) | data da apresentação |
| Desqualificados por motivo | contagem por `motivo_codigo` | data da transição |
| Em nutrição | leads em `nutricao` no fim do período | — |
| Investimento | soma do investimento lançado | mês de competência |
| CPL | investimento ÷ entradas | mês |
| CPLQ | investimento ÷ qualificados | mês |
| Custo por agendamento | investimento ÷ agendamentos | mês |
| Origem desconhecida | leads com `origem = desconhecida`; sempre em linha separada (RN-02) | — |

Dois modos de leitura, com seletor na tela: **Por atividade** (o que aconteceu no período, pela data de cada evento) e **Por safra** (o que aconteceu com os leads que entraram no período). O padrão é "Por atividade". Leads com `historico = true` entram nos totais por safra e ficam fora da produtividade.

## Telas e comportamento

### Indicadores → Funil

- Filtros no topo: período (este mês, mês passado, últimos 90 dias, personalizado), origem, campanha, produto, responsável. Ficam no endereço da página.
- Faixa de números: entradas, primeiro contato, qualificados, agendamentos, realizadas — cada um com a taxa de passagem para o seguinte e, se houver meta cadastrada, a barra de progresso.
- Funil visual com as mesmas cinco etapas.
- **Tabela por origem:** uma linha por origem, com entradas, qualificados, taxa, agendamentos, realizadas, investimento, CPL, CPLQ e custo por agendamento. Linha "Desconhecida" sempre visível. Ordenável.
- **Tabela por campanha** (ao expandir uma origem paga).
- **Motivos de desqualificação:** barras por motivo, com filtro por origem — é a visão que mostra, por exemplo, quantos leads de um anúncio "buscavam comprar material".
- **Tempo:** mediana até o primeiro contato e % dentro do prazo, por responsável.
- **Detalhamento:** clicar em qualquer número abre a lista dos leads que o compõem (a tela Leads, já filtrada). O número da lista é igual ao do painel.
- Rodapé: data e hora da última atualização dos dados e o aviso de lacunas: "12 leads sem origem identificada · investimento de outubro ainda não lançado."

### Indicadores → Produtividade

Substitui as abas "Rotina Diária" e "Resumo Semanal".

- Visão **Dia** e **Semana**, por usuário (o `sdr` vê a própria; `gestor` e `admin` veem todos).
- Colunas, calculadas automaticamente:

| Coluna | Fonte |
|---|---|
| WhatsApp enviados | tentativas com canal `whatsapp` (as de `origem = lista` em coluna separada, "disparos") |
| Ligações | tentativas com canal `ligacao` |
| Follow-ups | tentativas de número 2 em diante |
| Respostas | tentativas com resultado `respondeu`, `pediu_retorno` ou `agendou` |
| Leads novos atendidos | primeiros contatos |
| Reuniões agendadas | agendamentos confirmados |
| Reuniões realizadas | apresentações realizadas |
| Qualificações | decisões de qualificar, desqualificar e nutrir |

- Na visão Semana, a linha de total e, se houver meta semanal cadastrada, o percentual atingido.
- Botão **Exportar** (`.csv`).

### Anúncios

A tela `MediaPanel` existente é substituída pelo **dashboard de anúncios**, no item de menu "Anúncios" (rota `/midia` mantida). Papéis: `admin`, `gestor`, `marketing`.

**Topo**

- Filtros: período (com comparação com o período anterior), plataforma (Meta, Google, outras), conta, campanha, produto.
- Faixa de números, cada um com a variação contra o período anterior: **investimento**, **impressões**, **cliques**, **CTR**, **CPC**, **leads no CRM**, **CPL**, **qualificados**, **CPLQ**, **agendamentos**, **custo por agendamento**, **apresentações realizadas**.
- Data e hora da última sincronização de cada plataforma e o aviso de lacuna: "Google Ads: sem dados depois de 28/09."

**Gráfico de evolução** — por dia ou por semana: investimento (barras) e leads, qualificados e agendamentos (linhas). O usuário liga e desliga cada série.

**Tabela em três níveis** — campanha → conjunto de anúncios (grupo de anúncios, no Google) → anúncio. Cada linha expande a seguinte.

| Coluna | De onde vem |
|---|---|
| Nome, situação (ativo, pausado) | plataforma |
| Investimento, impressões, cliques, CTR, CPC, CPM | plataforma |
| Leads informados pela plataforma | plataforma (formulários e conversas iniciadas) |
| **Leads no CRM** | leads atribuídos àquela linha (regra de atribuição abaixo) |
| **Qualificados**, **taxa de qualificação** | funil |
| **Agendamentos**, **realizadas** | agendamentos |
| **CPL**, **CPLQ**, **custo por agendamento** | investimento ÷ cada contagem |
| **Motivo de desqualificação mais comum** | funil, com o percentual |
| Primeira resposta dentro do prazo | fila |

- Ordenável por qualquer coluna; o usuário escolhe as colunas; exporta em `.csv`.
- Clicar em **Leads no CRM**, **Qualificados** ou **Agendamentos** abre a lista dos leads daquela linha (para `marketing`, a lista vem sem nome, telefone e e-mail: só etapa, motivo e datas).
- No nível do anúncio: o nome, o texto principal e o endereço de destino, quando a plataforma informar, e o link "Ver na plataforma".
- Linha fixa **"Sem anúncio identificado"**: leads de origem paga que não puderam ser ligados a campanha nenhuma. Linha fixa **"Origem desconhecida"** (RN-02).
- Diferença entre "leads informados pela plataforma" e "leads no CRM" maior que 20% em uma campanha aparece destacada, com a explicação provável (lead não respondeu, número repetido, captura não configurada).

**Qualidade por anúncio** — visão dedicada à dor original do projeto: para cada campanha ou anúncio, a barra dos leads dividida em qualificados, em andamento e desqualificados por motivo. É onde aparece, por exemplo, que 70% dos leads de um anúncio "buscavam comprar material".

**Como o gasto entra** — três caminhos, que alimentam as mesmas tabelas:

| Caminho | Como | Quando usar |
|---|---|---|
| **Sincronização** | Configurações → Integrações → conectar a conta do Meta Ads e a do Google Ads. O sistema busca, todo dia de madrugada e ao clicar em **Atualizar agora**, os números por anúncio e por dia dos últimos 7 dias (para pegar correções da plataforma) e, na primeira vez, dos últimos 90 dias. Contrato em `integracoes.md`, seção 7. | caminho principal |
| **Relatório importado** | assistente da SPEC-4-003, cartão Mídia, com os perfis "Relatório Meta Ads" e "Relatório Google Ads" | quando a sincronização não estiver disponível, ou para preencher histórico |
| **Lançamento manual** | formulário: mês, plataforma, campanha, valor | canais sem relatório (ex.: patrocínio, indicação paga) |

Quando houver mais de um caminho para o mesmo dia e a mesma campanha, vale a sincronização; a tela avisa a sobreposição em vez de somar.

**Atribuição do lead ao anúncio** — regra única, aplicada na criação do lead e gravada nele (não muda depois):

| Ordem | Sinal | Resultado |
|---|---|---|
| 1 | A primeira mensagem do WhatsApp veio de um anúncio "clique para o WhatsApp": a Z-API entrega o identificador do anúncio junto com a mensagem (SPEC-4-013, regra 4) | lead ligado ao **anúncio** exato |
| 2 | O lead veio por formulário ou pela captura automática com os parâmetros de campanha no endereço (UTM) — `utm_campaign`, `utm_content` | lead ligado à campanha e, se `utm_content` trouxer o identificador, ao anúncio |
| 3 | A mensagem inicial do WhatsApp contém um código de campanha combinado (ex.: "[A12]" no texto pré-preenchido do link do anúncio ou do botão do site) | lead ligado à campanha do código |
| 4 | A pessoa informou a campanha no cadastro ou na importação | lead ligado à campanha pelo nome (com a tabela de correspondência de nomes) |
| 5 | Nada disso | "Sem anúncio identificado" |

O padrão de parâmetros de campanha e de códigos, para a agência seguir, está em `integracoes.md`, seção 7.4. A tela **Campanhas sem correspondência** lista nomes que vieram dos leads e não existem na plataforma (e o contrário), e deixa o usuário ligar um ao outro uma vez.

**Recomendações** — as recomendações da Fase 2 continuam, agora com dado real e no nível do anúncio, todas explicáveis (RN-11): cada uma mostra o dado, o período, a hipótese e a confiança.

| Regra | Condição | Texto |
|---|---|---|
| CPLQ desigual (existente) | CPLQ de uma campanha é mais de 1,5 vez o de outra na mesma plataforma | como hoje |
| Anúncio sem qualificado | 10 ou mais leads no CRM e nenhum qualificado em 30 dias | "O anúncio X trouxe N leads e nenhum foi qualificado. Motivo mais comum: Y." |
| Motivo dominante | mais de 50% dos desqualificados de um anúncio com o mesmo motivo | "N de M leads do anúncio X foram desqualificados por Y. O texto do anúncio pode estar atraindo quem quer comprar material." |
| Gasto sem lead | investimento acima de R$ 200 em 14 dias e nenhum lead no CRM | "O anúncio X gastou R$ V e não gerou lead no CRM. Confira o destino do anúncio." |
| Bom desempenho | CPLQ pelo menos 30% abaixo da média da conta, com 3 ou mais qualificados | "O anúncio X tem o melhor custo por qualificado da conta." |

Nenhuma recomendação altera campanha. A decisão registrada (aceitar, rejeitar, com motivo) continua sendo de `gestor` ou `admin`, e o histórico das decisões fica na tela.

### Feedback Marketing–Comercial

A tela `MediaFeedback` existente passa a ler dado real e ganha o filtro de período e o botão **Gerar devolutiva do mês**: uma página com, por campanha, entradas, qualificados, agendamentos, CPLQ, os três motivos de desqualificação mais comuns e até cinco exemplos de observação (sem nome, telefone ou e-mail). Exporta em `.csv` e em página para imprimir. É o que o papel `marketing` vê.

### Relatório do parceiro

Indicadores → Parceiros. Para cada origem do grupo `parceiro` (inicialmente `parceiro_hiper`):

- tabela dos leads daquela origem: data de entrada, empresa, cidade, produto, estágio, última interação, resultado, motivo quando desqualificado;
- totais do período;
- botão **Exportar para o parceiro** (`.xlsx` ou `.csv`), sem telefone, e-mail, notas nem dados de outras origens.

Substitui a aba "Leads Hiper - Tráfego Pago". O parceiro não ganha acesso ao sistema nesta fase; recebe o arquivo.

### Metas

Configurações → Metas (`admin`, `gestor`): por mês, e opcionalmente por origem — qualificados, agendamentos, realizadas, CPLQ máximo; e metas semanais de produtividade por usuário. Cada meta guarda quem cadastrou e quando. Meta alterada não muda o histórico dos meses fechados.

## Dados

- `ad_accounts`: `plataforma` (`meta`, `google`), `conta_id`, `nome`, `status`, `credencial_ref` (referência ao cofre de segredos; nunca devolvida por rota), `ultima_sync`, `ultimo_erro`, `moeda`, `fuso`.
- `ad_entities`: a árvore de anúncios — `plataforma`, `conta_id`, `nivel` (`campanha`, `conjunto`, `anuncio`), `externo_id`, `pai_id`, `nome`, `status`, `texto`, `destino_url`, `atualizado_em`. Único por (`plataforma`, `nivel`, `externo_id`).
- `ad_insights_daily`: uma linha por anúncio e por dia — `data`, `plataforma`, `conta_id`, `campanha_id`, `conjunto_id`, `anuncio_id`, `investimento`, `impressoes`, `cliques`, `leads_plataforma`, `fonte` (`sync`, `importado`). Único por (`plataforma`, `anuncio_id`, `data`); a sincronização regrava os últimos 7 dias. É a tabela de onde todos os números de gasto saem.
- `media_spend_manual`: `mes` (AAAA-MM), `origem`, `campanha`, `valor`, `actor_id`, `created`, `updated`.
- `leads`: campos novos de atribuição — `ad_plataforma`, `ad_campanha_id`, `ad_conjunto_id`, `ad_anuncio_id`, `ad_sinal` (qual das cinco regras atribuiu), `ad_click_id`.
- As coleções `media_imports` e as rotas da Fase 2 continuam existindo; os relatórios importados passam a gravar também em `ad_insights_daily`.
- `campaign_aliases`: `nome_midia`, `nome_lead`, `origem`, `actor_id` — ligações manuais entre nomes de campanha.
- `targets`: `mes`, `escopo` (`geral`, `origem`, `usuario`), `chave`, `indicador`, `valor`, `actor_id`, `created`.

## API

| Rota | Papéis | Função |
|---|---|---|
| `GET /backend/v1/metrics/funnel?from=&to=&mode=&origin=&campaign=&product=&owner=` | admin, gestor, sdr | números do funil, tabelas por origem e campanha, motivos, tempos, lacunas. |
| `GET /backend/v1/metrics/funnel/leads?...&metric=` | admin, gestor, sdr | ids dos leads que compõem um número (para o detalhamento). |
| `GET /backend/v1/metrics/productivity?from=&to=&user=&grain=` | admin, gestor; sdr só o próprio | produtividade. |
| `GET /backend/v1/ads/overview?from=&to=&compare=&platform=&account=&campaign=&product=` | admin, gestor, marketing | números do topo e série do gráfico. |
| `GET /backend/v1/ads/table?level=&parent=&...` | admin, gestor, marketing | linhas de um nível, com todas as colunas. |
| `GET /backend/v1/ads/leads?level=&id=&metric=` | admin, gestor (marketing: sem dado pessoal) | leads de uma linha. |
| `GET /backend/v1/ads/quality?...` | admin, gestor, marketing | divisão por resultado e motivo. |
| `GET/POST /backend/v1/admin/ad-accounts` · `POST .../{id}/sync` · `DELETE .../{id}` | admin | contas de anúncio e sincronização sob demanda. |
| `GET/POST/PATCH/DELETE /backend/v1/media/spend` | admin, gestor, marketing | lançamentos manuais (o lançamento é o único dado desta fase que pode ser apagado, e a exclusão é auditada). |
| `GET/POST /backend/v1/media/aliases` | admin, gestor, marketing | ligações de campanha. |
| `GET /backend/v1/metrics/partner?origin=&from=&to=` | admin, gestor | relatório do parceiro. |
| `GET/PUT /backend/v1/targets` | admin, gestor | metas. |

A rota `GET /backend/v1/leads/metrics` continua respondendo como hoje, para não quebrar a regressão da Fase 1; a tela nova usa as rotas acima. `GET /backend/v1/media/cross`, `/media/recommendations` e `/media/feedback` passam a considerar os lançamentos manuais e as ligações de campanha.

## Regras

1. **Reconciliação:** todo número do painel é igual à contagem da lista que ele abre. É a prova central desta SPEC.
2. **Origem desconhecida nunca é distribuída** entre as outras origens (RN-02).
3. **Divisão por zero:** indicador de custo sem qualificado ou sem agendamento mostra "—", não zero nem infinito.
4. **Mês sem investimento lançado** mostra "não lançado", não R$ 0,00.
5. **Meta só aparece se cadastrada** (RN-12).
6. A sincronização roda pelo agendador do backend, uma vez por dia, e pode ser pedida na tela (no máximo uma vez a cada 15 minutos por conta). Cada execução grava um evento em `integration_events` (SPEC-4-011); falha vai para a fila de exceção e aparece no cartão da integração. A sincronização só **lê** das plataformas.
6a. As credenciais das contas de anúncio ficam no cofre de segredos do backend e nunca chegam ao navegador.
6b. A atribuição é gravada no lead quando ele é criado e não é recalculada; mudar o nome de uma campanha na plataforma não quebra nada, porque a ligação é pelo identificador.
6c. Os painéis calculam sobre os dados no momento da consulta; não há tabela de consolidação além de `ad_insights_daily`. Se a consulta do funil passar de 3 segundos com a base real, a T4.63 acrescenta cache de 5 minutos.
7. `marketing` vê Anúncios e Feedback, sem nome, telefone ou e-mail de lead em nenhuma tela.
8. Exportações seguem a lista de campos permitidos e a neutralização de fórmulas das fases anteriores.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Mês com 40 entradas, 6 qualificados, 3 agendamentos, R$ 3.600 lançados em Google Ads | CPLQ R$ 600, custo por agendamento R$ 1.200 na linha do Google Ads; os números abrem as listas correspondentes. |
| Principal | Lidiane registra 12 tentativas e confirma 1 agendamento no dia | Produtividade do dia mostra os 12 divididos por canal e 1 reunião agendada, sem digitação. |
| Principal | Lead chega pelo WhatsApp a partir de um anúncio do Meta | O lead aparece na linha daquele anúncio, em "Leads no CRM", sem ninguém informar a campanha. |
| Principal | Abrir a campanha, o conjunto e o anúncio | Três níveis com gasto, leads, qualificados e CPLQ; clicar no número abre os leads. |
| Limite | Campanha com investimento e nenhum qualificado | CPLQ "—"; recomendação "anúncio sem qualificado" quando passar de 10 leads. |
| Limite | Sincronização do Google indisponível | O cartão mostra o erro; o canal segue por relatório importado; o dashboard funciona. |
| Limite | Mesmo dia e campanha por sincronização e por relatório | Vale a sincronização; aviso de sobreposição. |
| Limite | Nome da campanha no relatório diferente do nome no lead | Aparece em "Campanhas sem correspondência" até ser ligado. |
| Limite | Sem meta cadastrada | Nenhuma barra de meta. |
| Falha | Lançamento manual e importação para o mesmo mês e campanha | Aviso de possível contagem dupla, com os dois valores. |
| Falha | `marketing` tenta abrir a lista de leads pelo detalhamento | Negado. |

## Estados vazios, erro e permissão

- Sem dados no período: "Nenhum lead neste período." e a sugestão de ampliar o período.
- Anúncios sem dados: "Nenhuma conta de anúncio conectada e nenhum investimento lançado." com os botões **Conectar conta**, **Importar relatório** e **Lançar investimento**.

## Critérios de aceite

- [ ] **CA-4-33:** O painel de funil mostra, por período, origem, produto e responsável, entradas, primeiro contato, qualificados, agendamentos e apresentações realizadas, com taxas e tempos; clicar em qualquer número abre a lista dos leads que o compõem, e a contagem da lista é igual ao número.
- [ ] **CA-4-34:** A produtividade diária e semanal por usuário (mensagens, ligações, follow-ups, respostas, reuniões agendadas e realizadas) é calculada automaticamente a partir das tentativas, mensagens e agendamentos, sem digitação.
- [ ] **CA-4-35:** O dashboard de anúncios mostra, em três níveis (campanha, conjunto, anúncio), investimento, impressões, cliques, leads no CRM, qualificados, agendamentos, CPL, CPLQ e o motivo de desqualificação mais comum; a soma dos filhos é igual ao pai; "Sem anúncio identificado" e "Origem desconhecida" ficam em linhas próprias; metas só aparecem se cadastradas.
- [ ] **CA-4-36:** Um lead que chega por anúncio com botão de WhatsApp, ou por página com parâmetros de campanha, é ligado ao anúncio sem ninguém informar; o gasto entra por sincronização, relatório importado ou lançamento manual, sem contagem em dobro; falha de sincronização não derruba o dashboard.
- [ ] **CA-4-37:** A devolutiva para o Marketing e o relatório do parceiro são gerados e exportados sem dado pessoal de lead e sem dados de outras origens; o papel marketing não vê nome, telefone nem e-mail em nenhuma tela.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Reconciliação | para 5 números do funil, abrir a lista e contar | tabela número do painel × contagem da lista |
| Produtividade | registrar um conjunto conhecido de tentativas e conferir as colunas | captura |
| CPLQ | `fixtures/midia-fase4.csv` com valores conhecidos | cálculo à mão × tela |
| Atribuição | um lead por cada um dos 5 sinais, com `fixtures/zapi-eventos-fase4.json` e `fixtures/captura-fase4.json` | tabela lead × sinal × linha do dashboard |
| Sincronização | conectar uma conta real (ou de teste) e comparar 3 dias com o painel da plataforma | tabela plataforma × Central |
| Três níveis | somar os anúncios de um conjunto e os conjuntos de uma campanha | somas iguais às linhas de cima |
| Privacidade | entrar como `marketing` e percorrer Mídia e Feedback | capturas sem dado pessoal |
| Parceiro | exportar o relatório e conferir as colunas | arquivo |

**Dados de teste:** `fixtures/midia-fase4.csv`, `fixtures/leads-planilha-fase4.csv`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.63 | Criar o painel de funil por categoria e por etapa, com filtros, modos por atividade e por safra, tabelas por origem e campanha, motivos, tempos, lacunas e o detalhamento que abre a lista de leads | Dev full-stack | SPEC-4-009 | CA-4-33 | Prova de reconciliação: cinco números × contagem da lista | tabela com os cinco pares | T4.37 aceita; T4.35 aceita; T4.16 aceita | ☐ | L |
| T4.64 | Criar o painel de produtividade diária e semanal por usuário, calculado das tentativas, mensagens e agendamentos, com exportação | Dev full-stack | SPEC-4-009 | CA-4-34 | Prova: conjunto conhecido de tentativas e mensagens | captura conferida com o esperado | T4.49 aceita; T4.37 aceita | ☐ | L |
| T4.65 | Criar o modelo de dados de anúncios (contas, árvore campanha–conjunto–anúncio, números diários) e a atribuição do lead ao anúncio pelos cinco sinais, gravada na criação; códigos de campanha e correspondência de nomes | Dev backend | SPEC-4-009 | CA-4-36 | RED/GREEN: um lead por sinal, com as fixtures da Z-API e da captura | tabela lead × sinal × linha | T4.45 aceita; T4.25 aceita; T4.31 aceita | ☐ | L |
| T4.66 | Implementar a sincronização do Meta Ads: conta e credencial no cofre, leitura diária dos últimos 7 dias e inicial de 90, paginação, eventos e erros na central de integrações, atualização sob demanda | Dev backend | SPEC-4-009 | CA-4-36 | Prova: três dias comparados com o painel da Meta; falha: token inválido | tabela plataforma × Central + cartão com o erro | T4.65 aceita; T4.3 aceita | ☐ | L |
| T4.67 | Implementar a entrada do Google Ads: perfil de relatório importado gravando nos números diários e, com o token aprovado ou o conector habilitado, a sincronização diária | Dev backend | SPEC-4-009 | CA-4-36 | Prova: relatório de um mês conferido com o painel do Google; limite: mesmo dia por sincronização e por relatório | tabela plataforma × Central + aviso de sobreposição | T4.65 aceita | ☐ | L |
| T4.68 | Criar o dashboard de anúncios: números do topo com comparação, gráfico de evolução, tabela em três níveis com colunas escolhidas, qualidade por anúncio, linhas de sem anúncio e origem desconhecida, lançamento manual, recomendações explicáveis com decisão registrada | Dev full-stack | SPEC-4-009 | CA-4-35 | Prova dos três níveis: soma dos filhos igual ao pai; CPLQ com fixtures/midia-fase4.csv | cálculo à mão × tela | T4.66 aceita; T4.67 aceita; T4.63 aceita | ☐ | L |
| T4.69 | Atualizar o feedback para o Marketing com dado real e a devolutiva do mês, criar o relatório do parceiro com exportação e o cadastro de metas | Dev full-stack | SPEC-4-009 | CA-4-37 | Prova de privacidade: percorrer como marketing; relatório do parceiro sem dados de outras origens | capturas + arquivo exportado | T4.68 aceita; T4.40 aceita | ☐ | L |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
