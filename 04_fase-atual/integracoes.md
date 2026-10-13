# Integrações — Fase 4

**Projeto:** Pontual Tecnologia — Central de Aquisição e Qualificação de Leads  
**Para quem é:** quem vai implementar as integrações no Skip e quem, na Pontual ou na agência, vai configurar os sistemas externos.  
**Como ler:** a seção 1 é o mapa. Cada seção seguinte traz, para uma integração, o que ela faz, o que é preciso ter em mãos, o contrato técnico e o passo a passo de ativação. A seção 11 é a lista de conferência para ligar tudo.

Onde este documento cita uma SPEC, é lá que está o comportamento do sistema; aqui está o contrato com o mundo de fora.

---

## 1. Mapa das integrações

| # | Sistema externo | Sentido | Para quê | Como | SPEC | Situação nesta fase |
|---|---|---|---|---|---|---|
| 1 | WhatsApp, pela **Z-API** | entrada e saída | conversar com leads dentro da Central; agentes de IA; envio de listas e de régua | chamadas HTTP do backend à Z-API; a Z-API chama a Central a cada evento | 4-013, 4-008 | construir e ativar |
| 2 | **Meta Ads** (Facebook e Instagram) | entrada | gasto, impressões, cliques e leads por anúncio | leitura diária pela API de marketing da Meta; alternativa: relatório importado | 4-009 | construir; ativação depende de acesso à conta |
| 3 | **Google Ads** | entrada | o mesmo | leitura diária pela API do Google Ads; alternativa: relatório importado | 4-009 | construir; a sincronização depende de credencial que o Google aprova — começar por relatório |
| 4 | **Site, formulários e páginas de anúncio** | entrada | lead entra sozinho, com origem e campanha | chamada HTTP à rota de captura da Central | 4-002, 4-011 | construir e ativar |
| 5 | **Google Sheets** | entrada | importar planilhas sem baixar arquivo | conector do Skip, com a conta de cada usuário | 4-003 | construir, se o conector estiver habilitado |
| 6 | **Agenda** (Outlook, Google Agenda, Teams) | saída | convite da apresentação | arquivo `.ics`; opcionalmente, criação do evento pelo conector de agenda | 4-007 | `.ics` sempre; conector opcional |
| 7 | **Modelo de IA** | interna | agentes e copiloto | recurso de IA do Skip, chamado pelo backend | 4-008 | construir e ativar, com autorização da Pontual |
| 8 | **RespondeChat** | saída (arquivo) | período de transição: disparo por lá, com lista montada na Central | exportação de arquivo de 4 colunas | 4-006 | disponível; deixa de ser necessário quando o item 1 estiver em uso |
| 9 | Planilhas da operação (SharePoint/Excel) | entrada, uma vez | carga inicial | importação assistida | 4-003, 4-014 | usar na carga e arquivar |
| 10 | Outros sistemas de mensagem | entrada | entregar mensagens de outro canal | rota genérica de mensagens | 4-011 | pronta, sem uso previsto |

**Ficam fora, de propósito:** Autentique (propostas e assinatura), Conexa (financeiro), portal Hiper (revenda), CRM Pontual e MoveDesk (suporte e chamados), e-mail. A ficha do lead tem lugar para anotar o resultado depois da apresentação, mas nenhum desses sistemas é integrado — o projeto termina no agendamento da apresentação.

---

## 2. Princípios que valem para todas

1. **Segredos só no servidor.** Tokens, chaves e senhas ficam no cofre de segredos do backend (`$secrets`) ou em coleção que nenhuma rota devolve. Nunca no código, em migração, no arquivo `.env`, em log ou no navegador. Toda chamada a sistema externo sai do backend (`pocketbase/hooks`).
2. **Nada entra duas vezes.** Todo evento de entrada tem um identificador próprio da origem; a Central guarda esse identificador e ignora a repetição.
3. **Responder rápido, processar depois.** As rotas de entrada validam, gravam o evento e respondem em até 2 segundos.
4. **Falha não trava a operação.** O que falha vai para a fila de exceção (Configurações → Integrações), com o motivo e os botões de corrigir, reprocessar e descartar. Cadastro manual, importação por arquivo, fila e agendamento funcionam com tudo desligado.
5. **Dado pessoal mascarado nos registros.** O registro de eventos guarda telefone, e-mail e documento mascarados.
6. **Recusa prevalece.** Nenhuma integração de saída envia para contato suprimido; a conferência é feita no servidor, na hora do envio.
7. **Formato de telefone:** só dígitos, com código do país e DDD — `5521999998888`. É o formato que a Central guarda e o que a Z-API usa.
8. **Datas e horas:** ISO 8601 com fuso (`2026-10-20T14:30:00-03:00`). A Central guarda em UTC e exibe em `America/Sao_Paulo`.
9. **Endereço base** das rotas da Central: o endereço do backend do aplicativo (o valor de `VITE_POCKETBASE_URL`), seguido do caminho. Nos exemplos: `https://SEU-BACKEND`.

---

## 3. Captura automática — contrato da Central

Para qualquer sistema que queira entregar um lead ou uma mensagem à Central.

### 3.1 Entregar um lead

```
POST https://SEU-BACKEND/backend/v1/integrations/inbound/lead
X-Integration-Token: <chave criada em Configurações → Integrações>
Content-Type: application/json
```

Corpo:

```json
{
  "idempotency_key": "form-site-2026-10-20-000123",
  "contato_nome": "João Souza",
  "empresa_nome": "Mega Loja Materiais de Construção",
  "telefone": "(21) 99999-8888",
  "email": "joao@megaloja.example",
  "origem": "google_ads",
  "campanha": "Automec - Pesquisa - RJ",
  "conjunto": "sistema para loja de material",
  "anuncio": "anuncio-07",
  "utm": {
    "source": "google",
    "medium": "cpc",
    "campaign": "automec-pesquisa-rj",
    "content": "anuncio-07",
    "term": "sistema loja material construcao"
  },
  "produto": "automec",
  "cidade": "Niterói",
  "uf": "RJ",
  "segmento": "material_construcao",
  "mensagem": "Quero conhecer o sistema. Tenho duas lojas.",
  "custom": { "quantidade_de_caixas": 3 },
  "consentimento": {
    "texto": "Aceito ser contatado pela Pontual sobre o sistema.",
    "data": "2026-10-20T14:02:11-03:00"
  }
}
```

| Campo | Obrigatório | Regra |
|---|---|---|
| `idempotency_key` | sim | qualquer texto único por lead na origem, até 128 caracteres |
| `telefone` | sim, se não houver `email` | qualquer formato brasileiro com DDD; a Central normaliza |
| `email` | sim, se não houver `telefone` | |
| `contato_nome`, `empresa_nome` | não | |
| `origem` | não | código do catálogo de origens (Configurações → Catálogos). Ausente ou inválido: usa a origem padrão da chave; sem ela, `desconhecida` |
| `campanha`, `conjunto`, `anuncio` | não | nomes ou identificadores, como estiverem na plataforma |
| `utm` | não | os cinco parâmetros de campanha do endereço da página |
| `produto`, `segmento` | não | códigos dos catálogos |
| `cidade`, `uf`, `cnpj` | não | |
| `mensagem` | não | vira a primeira nota do lead |
| `custom` | não | valores de campos personalizados, pelo **código** do campo (Configurações → Campos) |
| `consentimento` | não | guardado na timeline do lead |

Respostas:

| Código | Significado | Corpo |
|---|---|---|
| 201 | lead criado | `{"lead_id": "…", "duplicate": false}` |
| 200 | chamada repetida (mesma `idempotency_key`) | `{"lead_id": "…", "duplicate": false, "replayed": true}` |
| 200 | o contato já existia | `{"lead_id": "<o existente>", "duplicate": true}` — a mensagem entrou como nota no existente |
| 202 | recebido; o processamento falhou e foi para a fila de exceção | `{"queued": true}` — **não reenviar** |
| 401 | chave ausente, inválida ou revogada | |
| 413 | corpo maior que 64 KB | |
| 422 | corpo inválido | `{"errors": [{"field": "telefone", "message": "Informe o DDD."}]}` |
| 429 | mais de 60 chamadas por minuto na mesma chave | reenviar depois de 60 segundos |

Exemplo de chamada, para teste:

```bash
curl -X POST "https://SEU-BACKEND/backend/v1/integrations/inbound/lead" \
  -H "X-Integration-Token: SUA_CHAVE" \
  -H "Content-Type: application/json" \
  -d '{"idempotency_key":"teste-001","contato_nome":"Teste","telefone":"21999998888","origem":"site_organico","mensagem":"Teste de captura"}'
```

**Formulário do site.** O formulário deve enviar os dados para um trecho de código **no servidor do site**, e esse trecho chama a Central com a chave. A chave não pode ficar no código da página, onde qualquer visitante a veria. Os parâmetros de campanha do endereço (`utm_source` etc.) devem ser lidos pela página, guardados em campos ocultos do formulário e enviados junto.

**Formulários de anúncio (Meta e Google).** As plataformas não chamam endereços arbitrários diretamente com este formato. Dois caminhos: (a) uma ferramenta de automação intermediária (Make, Zapier, n8n) recebe o lead da plataforma e chama a rota acima — é o mais rápido; (b) anúncios que levam ao WhatsApp, que dispensam formulário e são atribuídos sozinhos (seção 4.6). Para a Pontual, cujo atendimento já acontece no WhatsApp, o caminho (b) é o recomendado.

### 3.2 Entregar uma mensagem (rota genérica)

O WhatsApp pela Z-API **não** usa esta rota (seção 4). Ela existe para outros sistemas.

```
POST https://SEU-BACKEND/backend/v1/integrations/inbound/message
X-Integration-Token: <chave do tipo "mensagens">
```

```json
{
  "idempotency_key": "msg-8f3a21",
  "telefone": "5521999998888",
  "direcao": "entrada",
  "conteudo": "Bom dia, quero saber do sistema",
  "timestamp": "2026-10-20T09:14:03-03:00",
  "canal": "whatsapp",
  "autor": "João",
  "conversa_id": "conv-771"
}
```

`direcao`: `entrada` (do contato) ou `saida` (da empresa). Mídia: enviar `conteudo` como `[imagem]`, `[áudio]` ou `[arquivo]`. Respostas como na seção 3.1.

---

## 4. WhatsApp pela Z-API

### 4.1 O que é e o que implica

A Z-API (https://z-api.io) conecta um número de WhatsApp a uma "instância" e permite enviar e receber por chamadas HTTP. A conexão é feita lendo um QR code com o WhatsApp do celular, como no WhatsApp Web: o número continua funcionando no aparelho.

Implicações que a Pontual precisa conhecer antes de decidir:

- **Não é a API oficial da Meta.** O WhatsApp pode restringir números que enviam em volume para quem não os conhece. Recomendações: usar um número **separado** para disparo frio; manter o ritmo e o limite diário da SPEC-4-013; nunca enviar para quem recusou.
- **O celular precisa continuar com o WhatsApp ativo e com internet.** Se o aparelho ficar desconectado por muito tempo, a instância desconecta e a Central avisa.
- **Um número por instância.** Dois números (Comercial e Comunicação) são duas instâncias.
- **Arquivos recebidos ficam disponíveis na Z-API por 30 dias**; a Central baixa cada arquivo ao receber.

### 4.2 O que ter em mãos

| Item | Onde obter | Onde vai |
|---|---|---|
| Conta na Z-API e uma instância por número | painel da Z-API | — |
| **ID da instância** | painel → Instâncias → editar | Configurações → Canais → Novo canal |
| **Token da instância** | o mesmo lugar | idem |
| **Token de segurança da conta** (`Client-Token`) | painel → Segurança → "Token de Segurança da Conta" → configurar e **ativar** | idem |

Depois de ativado o token de segurança, a Z-API recusa qualquer chamada sem o cabeçalho `Client-Token`. A Central envia esse cabeçalho em todas as chamadas.

### 4.3 Como a Central chama a Z-API

Endereço base de toda chamada:

```
https://api.z-api.io/instances/{ID_DA_INSTANCIA}/token/{TOKEN_DA_INSTANCIA}/{operação}
```

Cabeçalhos: `Client-Token: {token de segurança da conta}` e `Content-Type: application/json`.

**Enviar texto** — operação `send-text`, método POST.

```json
{ "phone": "5521999998888", "message": "Bom dia, João! Aqui é a Lidiane, da Pontual." }
```

| Campo | Regra |
|---|---|
| `phone` | obrigatório; código do país + DDD + número, só dígitos |
| `message` | obrigatório |
| `delayMessage` | opcional; segundos de espera antes de enviar, de 1 a 15 |
| `delayTyping` | opcional; segundos mostrando "digitando…", de 1 a 15 |

Resposta: `{"zaapId": "…", "messageId": "…", "id": "…"}`. A Central guarda `messageId` como `provider_message_id`; é por ele que os eventos de estado encontram a mensagem.

**Enviar imagem** — corpo com `phone`, `image` (endereço do arquivo ou conteúdo em Base64 com o prefixo `data:image/png;base64,`) e, opcionalmente, `caption`, `messageId` (para responder a uma mensagem) e `delayMessage`. Áudio, vídeo e documento seguem o mesmo padrão, cada um com a sua operação e o seu campo de arquivo.

**Operações usadas pela Central**

| Para quê | Operação |
|---|---|
| Enviar texto | `send-text` |
| Enviar imagem, áudio, vídeo, documento | `send-image`, `send-audio`, `send-video`, `send-document/{extensão}` |
| Obter o QR code para conectar | `qr-code/image` |
| Conectar por código, sem QR | `phone-code/{número}` |
| Situação da instância | `status` → `{"connected": true, "smartphoneConnected": true, "error": "…"}` |
| Desconectar; reiniciar | `disconnect`; `restart` |
| Conferir se o número tem WhatsApp | `phone-exists/{número}` |
| Marcar conversa como lida | `read-message` |
| Definir o endereço que recebe todos os eventos | `update-every-webhooks`, corpo `{"value": "<endereço>", "notifySentByMe": true}` |

> **Conferir antes de implementar.** O endereço base, o `send-text` (campos e resposta), o corpo de imagem, a resposta de `status`, o corpo de `update-every-webhooks`, o cabeçalho `Client-Token` e o formato dos eventos da seção 4.4 foram conferidos na documentação da Z-API em 13/10/2026. Os **nomes exatos** das demais operações e o método HTTP de cada uma devem ser conferidos na referência (https://developer.z-api.io) na hora de implementar, porque a página de cada operação é a fonte que vale. O adaptador da Z-API fica em um único arquivo do backend, para que uma diferença de nome seja corrigida em um lugar só.

**Conexão pelo QR code:** o WhatsApp invalida o QR code a cada 20 segundos. A Central pede um novo a cada 15 segundos e, depois de 3 pedidos sem leitura, para e mostra o botão "Gerar novo QR code" — é a recomendação da própria Z-API, para não fazer chamadas desnecessárias. Em alguns aparelhos o WhatsApp pede uma confirmação adicional depois da leitura; a tela orienta a concluir no celular.

### 4.4 Como a Z-API chama a Central

A Z-API envia cada evento por POST, com corpo JSON, para um endereço **HTTPS** (ela não aceita outro). A Central registra, para cada canal, o endereço:

```
https://SEU-BACKEND/backend/v1/channels/{ID_DO_CANAL}/webhook/{SEGREDO_DO_CANAL}
```

O segredo é um texto aleatório de 32 caracteres, gerado pela Central para cada canal. A documentação da Z-API não descreve um mecanismo de assinatura dos eventos; por isso a Central valida **o segredo do endereço** e **o `instanceId` do corpo** contra o canal, e recusa o que não bater.

A Central registra esse endereço com `update-every-webhooks` e `notifySentByMe: true`, para receber também as mensagens enviadas direto do celular.

**Mensagem recebida** — `type: "ReceivedCallback"`:

```json
{
  "type": "ReceivedCallback",
  "instanceId": "A20DA9C0183A2D35A260F53F5D2B9244",
  "messageId": "3EB0C767D097B7C7C2A1",
  "phone": "5521999998888",
  "fromMe": false,
  "momment": 1760962443000,
  "status": "RECEIVED",
  "chatName": "João Souza",
  "senderName": "João Souza",
  "isGroup": false,
  "isNewsletter": false,
  "broadcast": false,
  "isEdit": false,
  "connectedPhone": "5521988887777",
  "text": { "message": "Bom dia, quero saber do sistema" }
}
```

| Campo | Como a Central usa |
|---|---|
| `messageId` | identificador único; evento com `messageId` já gravado no canal é ignorado |
| `phone` | o contato; localiza o lead ou cria um |
| `fromMe` | `true` = enviada pelo número da Pontual (pelo celular ou por outra ferramenta); entra como mensagem de saída "pelo celular", a menos que o `messageId` já exista como enviado pela Central |
| `momment` | data e hora, em milissegundos (o nome do campo é escrito assim pela Z-API) |
| `senderName`, `chatName` | nome para o lead novo |
| `isGroup`, `isNewsletter`, `broadcast` | qualquer um `true` → evento ignorado |
| `isEdit` | marca a mensagem como editada, preservando o texto original |
| `text.message` | o texto |
| `image` | `imageUrl`, `caption`, `mimeType` |
| `audio` | `audioUrl`, `mimeType`, `seconds`, `ptt` |
| `document` | `documentUrl`, `fileName`, `title`, `mimeType` |
| `externalAdReply` | dados do anúncio de origem — seção 4.6 |

**Mudança de estado da mensagem** — `type: "MessageStatusCallback"`:

```json
{
  "type": "MessageStatusCallback",
  "instanceId": "A20DA9C0183A2D35A260F53F5D2B9244",
  "status": "READ",
  "ids": ["3EB0C767D097B7C7C2A1"],
  "momment": 1760962501000,
  "phone": "5521999998888",
  "isGroup": false
}
```

| `status` | Estado na Central |
|---|---|
| `SENT` | enviada |
| `RECEIVED` | entregue |
| `READ`, `PLAYED` | lida |
| `READ_BY_ME` | zera as não lidas da conversa (foi lida no celular) |

O estado só avança (enviada → entregue → lida); evento fora de ordem não volta o estado.

**Conectado e desconectado** — atualizam a situação do canal e, na desconexão, avisam o administrador e pausam envios de lista e de régua.

### 4.5 Regras de envio

| Situação | Regra |
|---|---|
| Pessoa respondendo em Conversas | sai na hora |
| Agente de IA | espera de 4 a 10 segundos (configurável) e usa `delayTyping` |
| Régua automática | um envio por lead por vez; só agenda o seguinte depois que o anterior saiu |
| Lista | intervalo aleatório entre envios (padrão 40 a 90 segundos); limite diário por canal (padrão 150); janela de horário; pausa automática por recusa, falha ou desconexão |
| Contato suprimido | nenhum envio, por nenhum caminho |
| Primeiro envio para um número | confere antes se o número tem WhatsApp |
| Resposta `429` ou erro de rede | até 3 tentativas, com intervalo crescente; depois, "falhou" com "Tentar de novo" |

Todo envio passa pela fila de envio da Central (`outbox`), que é o que garante que nada sai em dobro quando o servidor reinicia.

### 4.6 Anúncios que levam ao WhatsApp

Quando alguém clica em um anúncio do Meta com o botão de WhatsApp e envia a primeira mensagem, o evento `ReceivedCallback` traz o objeto `externalAdReply`, com `sourceType` igual a `"ad"`, `sourceId` (o identificador do anúncio), `sourceUrl` (o endereço do anúncio) e `ctwaClid` (o identificador do clique).

A Central usa `sourceId` para ligar o lead ao anúncio exato no dashboard de anúncios (SPEC-4-009), sem que ninguém informe a campanha. É a principal fonte de atribuição para a Pontual.

Para anúncios do Google e links do site que abrem o WhatsApp, não existe esse dado. Use o **código de campanha no texto pré-preenchido** (seção 7.4).

### 4.7 Ativar

1. A Pontual contrata a Z-API e cria uma instância por número.
2. No painel, em Segurança, configura e ativa o token de segurança da conta.
3. Na Central: Configurações → Canais → Novo canal; informa nome, ID da instância, token da instância e token de segurança.
4. **Conectar** e ler o QR code com o WhatsApp do número.
5. A Central registra o endereço de eventos e mostra "conectado".
6. Enviar uma mensagem de teste de outro celular para o número e conferir que aparece em Conversas; responder pela Central.
7. Definir quem atende o canal, a origem padrão e a etapa de entrada dos leads novos.
8. Só depois das decisões da Pontual: ligar, se for o caso, as chaves de envio automático do canal (régua, listas, agentes).

Fazer a conexão **depois** da carga inicial de dados (SPEC-4-014, Parte 3), para que as primeiras conversas já encontrem os leads cadastrados.

---

## 5. Google Sheets

**O que faz:** no assistente de importação, o usuário informa o endereço de uma planilha Google e a Central lê as células, sem baixar arquivo.

**Como:** conector do Skip. Cada usuário conecta a **própria** conta Google (menu do usuário → Minhas conexões). A Central lê apenas a planilha indicada, no momento em que o usuário pede; não há leitura automática nem escrita.

**Contrato no código:** `src/lib/skipConnectors.ts` já traz `listConnectors`, `connect`, `disconnect` e `execute`. A leitura usa `execute('GOOGLESHEETS_BATCH_GET', { spreadsheet_id, ranges: ['Aba!A1:Z20000'] })`. A resposta vem em `data`, no formato do Google; o código deve conferir o formato antes de usar (os auxiliares já tratam formato inesperado como erro).

**Ativar:** confirmar que o conector Google Sheets está habilitado para o aplicativo no Skip. Se não estiver, a aba "Google Sheets" do assistente fica oculta e o caminho é baixar a planilha como `.csv` (Arquivo → Fazer download → Valores separados por vírgula) ou copiar e colar as células.

**Planilhas no SharePoint/Excel Online** (como a "CRM - Acompanhamento"): não há conector nesta fase. Baixar como `.xlsx` ou `.csv`, ou copiar e colar.

---

## 6. Agenda

**Padrão — arquivo `.ics`.** Ao agendar, a Central gera um convite de calendário padrão, que abre no Outlook, no Google Agenda e no Teams. Conteúdo: título "Apresentação <produto> — <empresa>", início e fim com fuso `America/Sao_Paulo`, local (link ou endereço), descrição com os participantes e o contexto (sem telefone e sem dados de qualificação), e um identificador estável por apresentação — ao reagendar e importar o novo arquivo, a agenda **atualiza** o evento em vez de criar outro.

**Opcional — criar o evento direto na agenda Google.** Se o conector de agenda do Google estiver habilitado no Skip, o botão "Adicionar à minha agenda" cria o evento na agenda do **apresentador** (que precisa ter conectado a própria conta). O nome exato da ferramenta do conector deve ser consultado no catálogo de conectores do Skip na hora de implementar. Sem o conector, o botão não aparece e o `.ics` cobre o caso.

**Horários livres para os agentes.** A capacidade "propor horários de apresentação" (SPEC-4-008) usa os compromissos registrados na própria Central (tela Agenda) e o horário de atendimento. Não lê a agenda externa nesta fase; por isso é importante que as apresentações sejam registradas na Central.

---

## 7. Anúncios

O dashboard de anúncios (SPEC-4-009) precisa de duas coisas: **o gasto e o desempenho por anúncio e por dia**, e **saber de qual anúncio veio cada lead**. As seções 7.1 a 7.3 tratam da primeira; a 7.4, da segunda.

Todos os caminhos gravam na mesma tabela, `ad_insights_daily` — uma linha por anúncio e por dia, com investimento, impressões, cliques e leads informados pela plataforma.

### 7.1 Meta Ads — sincronização

**O que ter em mãos**

| Item | Como obter |
|---|---|
| Identificador da conta de anúncios (`act_…`) | Gerenciador de Anúncios → configurações da conta |
| Um **usuário do sistema** no Gerenciador de Negócios da Pontual, com acesso de leitura à conta de anúncios | Configurações do negócio → Usuários → Usuários do sistema |
| Um **token de acesso** desse usuário, com a permissão de leitura de anúncios (`ads_read`) | gerado no mesmo lugar; escolher token sem prazo de expiração curto |

O token vai em Configurações → Integrações → Meta Ads e é guardado no cofre de segredos.

**Chamada de leitura** (uma por conta, paginada):

```
GET https://graph.facebook.com/{versão vigente}/act_{ID_DA_CONTA}/insights
    ?level=ad
    &time_increment=1
    &time_range={"since":"2026-10-06","until":"2026-10-12"}
    &fields=campaign_id,campaign_name,adset_id,adset_name,ad_id,ad_name,spend,impressions,clicks,actions
    &limit=500
    &access_token={TOKEN}
```

| Campo da resposta | Vai para |
|---|---|
| `date_start` | `data` |
| `campaign_id`, `adset_id`, `ad_id` e os nomes | `ad_entities` e as chaves de `ad_insights_daily` |
| `spend` | `investimento` |
| `impressions`, `clicks` | `impressoes`, `cliques` |
| `actions` (itens de lead e de conversa iniciada) | `leads_plataforma` |

A resposta vem em páginas; seguir `paging.next` até acabar. Uma segunda chamada, por anúncio novo, busca o texto e o endereço de destino para o dashboard.

**Rotina:** todo dia, de madrugada, regravar os últimos 7 dias (a plataforma corrige números retroativamente). Na primeira vez, os últimos 90 dias, em blocos de 7.

**Erros esperados:** token expirado ou revogado (o cartão da integração mostra e pede novo token); limite de chamadas (esperar e continuar de onde parou); conta sem permissão.

> A versão da API da Meta muda algumas vezes por ano. Usar a versão vigente no momento da implementação e guardar o número em uma constante única.

### 7.2 Google Ads — sincronização

**O que ter em mãos**

| Item | Observação |
|---|---|
| Identificador do cliente do Google Ads (10 dígitos) | no topo do painel do Google Ads |
| **Token de desenvolvedor** | solicitado na conta de administrador do Google Ads; **o Google analisa e aprova, em prazo que não controlamos** |
| Credenciais OAuth (identificador e segredo do cliente) e um token de atualização de um usuário com acesso à conta | Google Cloud |

**Consulta** (linguagem de consulta do Google Ads), pelo método de busca em fluxo da API:

```
SELECT
  segments.date,
  campaign.id, campaign.name, campaign.status,
  ad_group.id, ad_group.name,
  ad_group_ad.ad.id, ad_group_ad.ad.name,
  metrics.cost_micros, metrics.impressions, metrics.clicks, metrics.conversions
FROM ad_group_ad
WHERE segments.date BETWEEN '2026-10-06' AND '2026-10-12'
```

`metrics.cost_micros` vem em milionésimos da moeda: dividir por 1.000.000. Campanha → `campanha`; grupo de anúncios → `conjunto`; anúncio → `anuncio`.

**Recomendação para esta fase:** começar o Google Ads por **relatório importado** (seção 7.3) e pedir o token de desenvolvedor em paralelo, na primeira semana. Quando o token for aprovado, liga-se a sincronização; os dados históricos importados continuam valendo. Se o conector de Google Ads estiver habilitado no Skip, ele substitui as credenciais acima — conferir no catálogo de conectores.

### 7.3 Relatório importado (Meta e Google)

Funciona sempre, sem credencial. Assistente de importação → cartão Mídia.

**Meta Ads:** Gerenciador de Anúncios → aba **Anúncios** → período desejado → Detalhamento → Por tempo → **Dia** → Exportar → `.csv`. Colunas necessárias: Dia, Nome da campanha, Nome do conjunto de anúncios, Nome do anúncio, Valor usado, Impressões, Cliques (todos), Resultados. Se disponível, incluir as colunas de identificação da campanha, do conjunto e do anúncio.

**Google Ads:** Campanhas → Anúncios → período → Segmentar → Tempo → **Dia** → Fazer download → `.csv`. Colunas: Dia, Campanha, Grupo de anúncios, Anúncio (ou título), Custo, Impressões, Cliques, Conversões.

Os perfis "Relatório Meta Ads" e "Relatório Google Ads" reconhecem essas colunas. Relatório sem a quebra por dia é aceito e distribuído no mês informado, com o aviso de que o gráfico diário ficará aproximado.

### 7.4 Saber de qual anúncio veio o lead

A Central atribui o lead na criação, pela primeira regra que servir (SPEC-4-009):

| # | Sinal | O que a agência ou o site precisa fazer |
|---|---|---|
| 1 | Anúncio do Meta com botão de WhatsApp | **nada** — a Z-API entrega o identificador do anúncio (seção 4.6) |
| 2 | Parâmetros de campanha no endereço | colocar em **todos** os anúncios que levam ao site: `utm_source` (`google`, `facebook`, `instagram`), `utm_medium` (`cpc`), `utm_campaign` (nome ou identificador da campanha), `utm_content` (identificador do anúncio), `utm_term` (palavra-chave, no Google). No Meta, usar os marcadores automáticos: `utm_campaign={{campaign.id}}` e `utm_content={{ad.id}}`. No Google, `utm_campaign={campaignid}` e `utm_content={creative}`. O formulário do site guarda esses valores e os envia na captura (seção 3.1). |
| 3 | Código no texto do WhatsApp | em links e botões que abrem o WhatsApp (site e anúncios do Google), usar o endereço `https://wa.me/55DDDNUMERO?text=Olá!%20Quero%20conhecer%20o%20sistema.%20[G07]`, em que `[G07]` é um código curto por campanha. Cadastrar a correspondência código → campanha em Anúncios → Códigos de campanha. |
| 4 | Informado por uma pessoa | campo Campanha, no cadastro |
| 5 | Nenhum | o lead aparece em "Sem anúncio identificado" |

Padrão de nomes recomendado para campanhas: `produto - objetivo - público/região`, por exemplo `Automec - WhatsApp - Lojas RJ`. Nomes estáveis facilitam a leitura; a ligação técnica é feita pelo identificador, não pelo nome.

---

## 8. RespondeChat (transição)

Enquanto o disparo ainda for feito pelo RespondeChat, a Central monta e higieniza a lista e gera o arquivo que ele aceita (SPEC-4-006).

**Formato do arquivo** — exatamente quatro colunas, nesta ordem:

| Coluna | Conteúdo |
|---|---|
| `Nome` | nome do contato; na falta, nome fantasia; na falta, razão social |
| `Número` | telefone com código do país e DDD, só dígitos |
| `E-mail` | pode ficar vazio |
| `Etiquetas` | a etiqueta de entrada do fluxo de conversa — precisa ser **idêntica** à configurada no fluxo do RespondeChat |

O RespondeChat recusa arquivo com colunas a mais ou em outra ordem. Os nomes das colunas e o formato do número (com ou sem `55`) são configuráveis em Configurações → Geral, para ajustar na primeira importação real.

**Respostas:** a Central não lê as respostas dadas dentro do RespondeChat. O retorno é registrado à mão, na aba Retornos. Perguntas a fazer ao suporte do RespondeChat, se a transição for longa: (1) existe envio de eventos (webhook) para mensagens recebidas e enviadas? (2) em que formato? (3) há custo? (4) há exportação de conversas em arquivo? Com resposta positiva para a primeira, a rota genérica da seção 3.2 recebe as mensagens.

Quando o WhatsApp estiver conectado na Central (seção 4), o envio de lista passa a ser feito por ela e esta seção deixa de ser necessária.

---

## 9. Modelo de IA

**Como:** o backend chama o recurso de IA do Skip (`$ai.chat`, no formato de conversa com ferramentas). O navegador nunca chama o modelo. Em `src/lib/skipAi.ts` há auxiliares para ler respostas transmitidas aos poucos; a área de teste dos agentes pode usá-los, sempre por uma rota do backend.

**O que é enviado ao modelo:** a conversa, o primeiro nome do contato, a empresa, os campos de qualificação, as notas e as etapas. **O que não é enviado:** telefone, e-mail, CNPJ, valores de proposta. Telefones, e-mails e documentos que apareçam dentro de textos são substituídos por marcadores antes do envio.

**Ativar:** depende da autorização da Pontual (decisão registrada em `decisoes-go-live.md`). Sem ela, os agentes e o copiloto ficam desligados e o resto do sistema não muda.

---

## 10. Sistemas que não são integrados

| Sistema | Uso na Pontual | O que a Central faz |
|---|---|---|
| Autentique | propostas e assinatura | nada; campo de anotação "valor da proposta" e "data de fechamento" |
| Conexa | financeiro | nada |
| Portal Hiper | clientes da revenda | nada; o relatório do parceiro é exportado em arquivo |
| CRM Pontual | suporte e chamados | nada; lead encaminhado pelo suporte entra pela origem "Encaminhado pelo Suporte" |
| MoveDesk | legado | importação única, se houver leads a aproveitar (perfil "Modelo padrão da Central") |
| Outlook | e-mail | nada; o convite `.ics` abre nele |

---

## 11. Lista de conferência para ligar tudo

Ordem recomendada. Cada linha tem um responsável e uma prova.

| # | Passo | Quem | Prova |
|---|---|---|---|
| 1 | Pedir o token de desenvolvedor do Google Ads | Marcelo, com a agência | protocolo do pedido |
| 2 | Criar o usuário do sistema e o token de leitura do Meta Ads | Marcelo, com a agência | token entregue ao administrador da Central, por canal seguro |
| 3 | Contratar a Z-API; criar as instâncias; ativar o token de segurança | Marcelo | três valores por instância em mãos |
| 4 | Conferir no Skip quais conectores estão habilitados (Google Sheets, agenda, Google Ads) | Dev | lista |
| 5 | Criar a chave de captura "Formulário do site" e testar pela tela | Admin | evento de teste no registro |
| 6 | Ajustar o formulário do site para chamar a captura, com os parâmetros de campanha | quem mantém o site | lead de teste na fila, com origem e campanha |
| 7 | Padronizar os parâmetros de campanha e os códigos nos anúncios (seção 7.4) | agência | três anúncios conferidos |
| 8 | Conectar a conta do Meta Ads e sincronizar 90 dias | Admin | dashboard com três meses; três dias conferidos contra o painel da Meta |
| 9 | Importar os relatórios do Google Ads dos últimos 3 meses | Marketing | idem |
| 10 | Fazer a carga inicial de leads (SPEC-4-014, Parte 3) | Lidiane | conciliação |
| 11 | Conectar os números de WhatsApp (seção 4.7) | Admin | mensagem de teste recebida e respondida |
| 12 | Publicar o primeiro agente em modo copiloto | Admin | sugestões aparecendo em Conversas |
| 13 | Depois das decisões: ligar as chaves de envio automático que a Pontual aprovar | Admin | registro na auditoria |
| 14 | Quando aprovado o token do Google: ligar a sincronização | Admin | cartão verde |

## 12. Fontes

- Documentação da Z-API: https://developer.z-api.io — identificação e token; token de segurança da conta; envio de texto e de imagem; eventos de mensagem recebida e de estado; situação da instância; QR code; atualização dos endereços de eventos. Conferida em 13/10/2026.
- Referência local do projeto: `src/lib/skipConnectors.ts` e `src/lib/skipAi.ts`, no repositório do sistema.
- Mapeamento de processos da Pontual (vídeos de 25 e 26/08/2026): formato de importação do RespondeChat e rotina de disparo.
- As APIs da Meta e do Google mudam de versão; os trechos das seções 7.1 e 7.2 mostram a forma da chamada e devem ser conferidos na documentação vigente de cada plataforma na implementação.
