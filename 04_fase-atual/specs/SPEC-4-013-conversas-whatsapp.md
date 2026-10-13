# SPEC-4-013 — Conversas: atendimento pelo WhatsApp dentro da Central (Z-API)

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-09, RF-10, RF-13, RF-15, RN-04, RN-05, RN-07, RN-14, RNF-05, RNF-10, G5, G6; reunião de alinhamento ("clone do WhatsApp com transbordo")  
**Degrau da solução:** construção — a conversa deixa de ser um histórico importado e passa a acontecer dentro do sistema, com o número de WhatsApp da Pontual conectado pela Z-API.

## Resultado observável

A Lidiane abre **Conversas** e vê a lista de conversas do WhatsApp comercial, como no WhatsApp Web: quem escreveu, o que falta responder, quem está atendendo. Ela responde ali mesmo, anexa arquivo, usa um modelo de mensagem e, ao lado, vê e edita a ficha do lead, move a etapa e agenda a apresentação — sem trocar de sistema. Tudo o que é dito fica no histórico do lead e conta na produtividade sem ninguém digitar.

## Por que existe

- A equipe vive no RespondeChat e na planilha ao mesmo tempo: conversa em um, registro no outro.
- O "Kanban" do RespondeChat não mostra o histórico do contato, e as etiquetas se perderam em uma migração.
- Leads do site chegam na linha "Comunicação", usada pelo Financeiro, e precisam ser repassados à mão para o Comercial.
- O motor de cadência do RespondeChat envia de uma vez as mensagens de vários dias.
- As rotas de conversa, transbordo e opt-out da Fase 3 foram construídas sobre arquivo importado; a tela de conversa é somente leitura e não tem link.

## Limites e dependências

- **Inclui:** conexão de um ou mais números de WhatsApp por QR code; recebimento e envio de texto, imagem, áudio, vídeo e documento; lista de conversas com situação, dono e não lidas; painel do lead ao lado da conversa; notas internas; modelos e respostas rápidas; atribuição, transferência e encerramento; criação automática de lead a partir de número desconhecido; identificação do anúncio de origem; envio de lista (disparo) com ritmo controlado; envio automático da régua; convivência com agente de IA (SPEC-4-008).
- **Fora de escopo:** grupos e canais do WhatsApp (ignorados); ligações de voz e vídeo; status/stories; botões, listas e enquetes interativas; outros canais (Instagram, e-mail, chat do site) — o desenho do canal é genérico para recebê-los depois; API oficial da Meta (ver risco).
- **Pré-condições:** SPEC-4-001, SPEC-4-004, SPEC-4-011 aceitas. Conta e instância na Z-API contratadas pela Pontual, com o identificador da instância, o token da instância e o token de segurança da conta em mãos (`integracoes.md`, seção 4).
- **Decisões da Pontual na T4.2:** quais números conectar (Comercial, Comunicação); se o número atual migra do RespondeChat ou se entra um número novo; quem atende cada número; se o envio automático (régua e agente) é liberado; texto para "onde conseguiu meu contato".
- **Risco, dito com clareza:** a Z-API conecta o número como um aparelho adicional do WhatsApp, não pela API oficial da Meta. Isso permite usar o número que a Pontual já tem, sem aprovação de modelos de mensagem, mas o WhatsApp pode restringir números que enviam em volume para quem não os conhece. Mitigações desta SPEC: ritmo e limite diário nos envios em lista, parada automática por recusas, supressão obrigatória, e a recomendação de usar um número separado para disparo frio. A camada de canal é escrita com um adaptador por fornecedor, de modo que trocar para a API oficial depois não mexe nas telas.
- **Plano B:** com o canal desconectado, as telas continuam funcionando: a conversa mostra o histórico, o botão de envio vira "Abrir no WhatsApp" (`wa.me`) e as tentativas voltam a ser registradas à mão (SPEC-4-005). A exportação de lista em arquivo (SPEC-4-006) continua disponível.

## Telas e comportamento

### Conversas

Item **Conversas** no menu, logo abaixo de "Minha fila", com o contador de não lidas. Papéis: `admin`, `gestor`, `sdr`. Três colunas.

**Coluna 1 — lista de conversas**

- Abas: **Minhas** · **Sem dono** · **Todas** (a última só para `gestor` e `admin`, ou para todos se a Pontual assim decidir).
- Situação, em seletor: **Abertas** · **Aguardando cliente** · **Resolvidas** — os mesmos três estados que a equipe já usa no RespondeChat (Atendendo, Aguardando, Resolvido).
- Filtros: canal (número), etapa do funil, etiqueta, não lidas, com agente de IA ativo, com transbordo aberto.
- Busca por nome, empresa, telefone e texto de mensagem.
- Cada item: nome do contato e empresa, trecho da última mensagem, hora, contador de não lidas, etiqueta do canal, iniciais do dono, selos (transbordo, agente de IA, suprimido) e a etapa do lead em uma tarja de cor.
- Ordenação: não lidas primeiro, depois a mais recente. Atualiza em tempo real.

**Coluna 2 — a conversa**

- Cabeçalho: contato, empresa, telefone, canal, situação, dono; botões **Assumir**, **Transferir** (para outro usuário, com nota), **Resolver**, **Reabrir**; e, quando houver agente de IA na conversa, **Pausar IA** / **Devolver à IA**.
- Mensagens em balões, com data, hora e, nas enviadas, o estado: enviando, enviada, entregue, lida, falhou (com **Tentar de novo**). Mostra quem enviou: nome do usuário, "Agente <nome>" ou "pelo celular" (mensagem enviada direto do aparelho).
- Imagem e vídeo com miniatura; áudio com tocador; documento com nome e botão de baixar.
- Mensagem editada ou apagada pelo contato aparece marcada como tal; o texto original que o sistema recebeu é preservado.
- **Notas internas** aparecem no meio da conversa, em balão amarelo, visíveis só para a equipe. São as mesmas notas da ficha (`lead_notes`).
- Eventos do lead aparecem como linhas discretas no meio da conversa: mudou de etapa, apresentação agendada, transbordo aberto, opt-out.
- **Campo de escrever:** texto com várias linhas (Enter envia, Shift+Enter quebra linha); anexar arquivo (imagem, vídeo, áudio, PDF e documentos; até 16 MB); emoji; alternar para **Nota interna**; botão **Modelos** (ou digitar `/`), que lista os modelos de mensagem da SPEC-4-010 com as variáveis já preenchidas; botão **Sugerir com IA** (SPEC-4-008), que escreve a sugestão no campo para a pessoa revisar.
- Contato suprimido: o campo fica bloqueado com "Este contato pediu para não ser procurado." Se o contato **escrever** depois da supressão, a conversa abre com o aviso e o botão "Remover supressão e responder", restrito a `admin` e `gestor`, com motivo.

**Coluna 3 — o lead**

A ficha resumida (SPEC-4-004), no mesmo componente: etapa com o seletor para mover, responsável, próxima ação, campos principais e personalizados editáveis, etiquetas, notas, botões **Agendar apresentação**, **Registrar opt-out**, **Abrir transbordo**, **Abrir ficha completa**. O painel pode ser recolhido.

Se a conversa for de um número que ainda não é lead (ver regra 3), a coluna mostra o lead recém-criado, com a origem a confirmar.

### Começar uma conversa

- Botão **Conversar** na ficha, no card da fila e no Kanban: abre a conversa do lead, criando-a se não existir. Substitui o "Abrir no WhatsApp" quando há canal conectado.
- Antes do primeiro envio para um número, o sistema confere com a Z-API se o número tem WhatsApp; se não tiver, avisa e oferece marcar o telefone como inválido.
- Com mais de um canal conectado, o usuário escolhe por qual número falar; o padrão é o canal da última conversa com o lead ou o canal padrão do usuário.

### Canais

Configurações → **Canais** (`admin`).

- Lista: nome (ex.: "Comercial", "Comunicação"), número, situação (conectado, desconectado, aguardando leitura do QR code), fornecedor (Z-API), quem atende, origem padrão para leads novos, funil e etapa de entrada, agente de IA padrão (opcional).
- **Novo canal:** nome; identificador da instância, token da instância e token de segurança da conta da Z-API. Os três valores são gravados no servidor e **nunca** voltam para a tela nem para o navegador.
- **Conectar:** mostra o QR code para ler com o WhatsApp do celular. O QR code é renovado a cada 15 segundos; depois de 3 renovações sem leitura, a tela para e mostra o botão "Gerar novo QR code". Alternativa: conexão por código, informando o número.
- Ao conectar, o sistema registra na Z-API o endereço de recebimento de eventos do canal (regra 1) e marca a opção de receber também as mensagens enviadas pelo próprio número.
- **Situação:** conferida a cada 5 minutos e ao abrir a tela. Canal desconectado gera aviso no sino para `admin` e faixa vermelha em Conversas: "O número Comercial está desconectado. As mensagens não estão chegando."
- **Desconectar**, **Reiniciar** e **Remover** (remover não apaga o histórico de conversas).
- **Horário de atendimento** do canal e **mensagem fora do horário** (opcional, enviada no máximo uma vez a cada 12 horas por contato).
- **Envio automático:** três chaves, todas desligadas por padrão — "régua de cadência pode enviar", "agentes de IA podem responder", "envio de listas permitido". Ligam-se depois da T4.2.

### Envio de lista pela Central

Complementa a SPEC-4-006. Na tela da lista, além de **Exportar**, aparece **Enviar pela Central**, quando o canal permitir.

- Escolhas: canal, modelo de mensagem (com as variáveis), data e hora de início, **ritmo** (intervalo aleatório entre envios; padrão de 40 a 90 segundos; mínimo de 20), **limite diário** do canal (padrão 150 envios de lista por dia), janela de horário.
- Pré-visualização com 3 contatos reais da lista.
- **Aprovação:** lista com mais de 50 contatos, ou qualquer lista de base fria, exige aprovação de `gestor` ou `admin` antes de começar. Quem aprova vê o texto, o tamanho e as exclusões.
- Andamento em tempo real: enviadas, entregues, lidas, respondidas, falhas, recusas. Botões **Pausar**, **Retomar**, **Cancelar**.
- **Parada automática:** o envio pausa sozinho se 5% dos contatos responderem com recusa, se 10% dos envios falharem, ou se o canal desconectar. Avisa quem iniciou.
- Quem responde entra em Conversas como conversa aberta; o retorno da lista (SPEC-4-006) é preenchido automaticamente: respondeu, sem interesse (pelas expressões de recusa, com confirmação humana da supressão), número inválido.
- A fila de envio é do próprio sistema, processada pelo agendador do backend a cada minuto; cada envio é um registro com estado, de modo que reiniciar o servidor não duplica nem perde envio.

### Envio automático da régua

Complementa a SPEC-4-005. Em Configurações → Cadência, cada tentativa da régua ganha o modo: **Manual** (padrão; aparece na fila para a pessoa fazer) ou **Automática** (o sistema envia o modelo escolhido, pelo canal do lead, na data e hora calculadas).

- Só funciona com a chave "régua de cadência pode enviar" ligada no canal.
- Antes de cada envio automático, o sistema confere de novo: lead não suprimido, sem transbordo aberto, sem resposta do contato desde a última tentativa, dentro do horário, etapa ainda ativa. Qualquer "não" cancela o envio e devolve o lead para a fila, com o motivo.
- O envio é registrado como tentativa (SPEC-4-005), com autor "automático".
- **Uma tentativa por vez:** o próximo envio só é agendado depois que o anterior saiu. É exatamente o defeito do motor atual que esta regra impede.
- Resposta do contato cancela os envios automáticos pendentes daquele lead e abre a conversa para uma pessoa (ou para o agente, se houver).

## Dados

### `channels`

`nome`, `fornecedor` (`zapi`), `numero`, `status` (`conectado`, `desconectado`, `aguardando_qr`), `credenciais` (json cifrado ou referência ao cofre de segredos; nunca devolvido por rota), `webhook_segredo` (texto aleatório de 32 caracteres, usado no endereço de recebimento), `origem_padrao`, `pipeline_id`, `stage_entrada`, `agente_padrao_id`, `atendentes` (json), `horario` (json), `msg_fora_horario`, `auto_regua`, `auto_agente`, `auto_lista` (bools), `limite_diario_lista`, `ativo`, `conectado_em`, `ultimo_evento_em`.

### `conversations` (evolução da coleção da Fase 3)

Campos novos: `channel_id`, `telefone`, `situacao` (`aberta`, `aguardando`, `resolvida`), `dono_id`, `nao_lidas`, `ultima_mensagem_em`, `ultima_mensagem_trecho`, `ultima_direcao`, `agente_id`, `agente_ativo` (bool), `anuncio` (json com os dados do anúncio de origem, quando houver). Uma conversa por par (canal, telefone).

### `messages` (evolução)

Campos novos: `channel_id`, `provider_message_id` (único por canal — garante que o mesmo evento não duplique), `tipo` (`texto`, `imagem`, `audio`, `video`, `documento`, `outro`), `midia_url`, `midia_mime`, `midia_nome`, `legenda`, `status` (`na_fila`, `enviando`, `enviada`, `entregue`, `lida`, `falhou`, `recebida`), `erro`, `enviado_por` (`usuario`, `agente`, `automatico`, `celular`, `contato`), `autor_id`, `agente_id`, `resposta_a`, `editada`, `apagada`, `enviada_em`, `entregue_em`, `lida_em`.

As mensagens importadas na Fase 3 continuam válidas, com `enviado_por` vazio.

### `outbox`

Fila de envio: `channel_id`, `conversation_id`, `lead_id`, `payload` (json), `origem` (`conversa`, `lista`, `regua`, `agente`, `automacao`), `origem_id`, `agendada_para`, `status` (`pendente`, `processando`, `enviada`, `falhou`, `cancelada`), `tentativas`, `erro`, `message_id`. Índice único em (`origem`, `origem_id`, `lead_id`) para envios de lista e de régua — a reexecução não duplica.

### `list_sends`

`list_id`, `channel_id`, `template_id`, `texto_final` (o texto aprovado, congelado), `ritmo_min`, `ritmo_max`, `inicio`, `status` (`aguardando_aprovacao`, `aprovada`, `enviando`, `pausada`, `concluida`, `cancelada`), `aprovada_por`, `aprovada_em`, `pausa_motivo`, contadores.

## API

| Rota | Papéis | Função |
|---|---|---|
| `GET /backend/v1/inbox?tab=&status=&channel=&q=` | admin, gestor, sdr | lista de conversas, paginada. |
| `GET /backend/v1/inbox/{conversationId}/messages?before=` | admin, gestor, sdr | mensagens, 50 por vez, das mais novas para as mais antigas. |
| `POST /backend/v1/inbox/{conversationId}/messages` | admin, gestor, sdr | envia texto ou mídia; devolve a mensagem em `na_fila`. |
| `POST /backend/v1/inbox/{conversationId}/read` | admin, gestor, sdr | zera as não lidas e marca como lidas no WhatsApp. |
| `POST /backend/v1/inbox/{conversationId}/assign` · `/transfer` · `/resolve` · `/reopen` | admin, gestor, sdr | dono e situação. |
| `POST /backend/v1/inbox/{conversationId}/agent` | admin, gestor, sdr | `pausar` ou `devolver`. |
| `POST /backend/v1/leads/{id}/conversation` | admin, gestor, sdr | abre ou cria a conversa do lead em um canal. |
| `POST /backend/v1/uploads` | admin, gestor, sdr | recebe o anexo e devolve o endereço para o envio. |
| `GET/POST/PATCH /backend/v1/admin/channels` | admin | canais; as credenciais só entram, nunca saem. |
| `POST /backend/v1/admin/channels/{id}/connect` · `GET .../qr` · `GET .../status` · `POST .../disconnect` · `POST .../restart` | admin | conexão. |
| `POST /backend/v1/channels/{id}/webhook/{segredo}` | público, validado pelo segredo | recebe os eventos da Z-API (regra 1). |
| `POST /backend/v1/lists/{id}/send` · `/approve` · `/pause` · `/resume` · `/cancel` · `GET .../progress` | conforme a SPEC-4-006; aprovar: admin, gestor | envio de lista. |

As rotas da Fase 3 (`/conversations/{leadId}`, `/handoff`, `/opt-out`, `/suggestion`, `/timeline`) continuam respondendo. `POST /backend/v1/canal/ativar` passa a consultar as chaves de envio automático do canal: nega e audita quando a chave estiver desligada (o comportamento da CA-3-07 é preservado até a Pontual ligar).

## Regras

1. **Recebimento.** A Z-API chama o endereço do canal a cada evento. O sistema valida o segredo do endereço e o identificador da instância no corpo; responde 200 em menos de 2 segundos; grava o evento em `integration_events` (SPEC-4-011) e processa. Evento repetido (mesmo identificador de mensagem) não duplica. Tipos tratados: mensagem recebida (inclusive as enviadas pelo próprio número, direto do celular), mudança de estado da mensagem, conectado, desconectado. Os demais são registrados e ignorados.
2. **Grupos, canais e listas de transmissão do WhatsApp são ignorados** (o evento informa quando a mensagem é de grupo).
3. **Número desconhecido que escreve** vira lead: tipo `lead`, na etapa de entrada do funil do canal, com a origem padrão do canal, nome vindo do perfil do WhatsApp, e próxima ação "Responder mensagem" para agora. Se o número for de um `prospect`, ele é promovido. Se houver dado de anúncio na mensagem (regra 4), a origem é a do anúncio.
4. **Anúncio de origem.** Quando a primeira mensagem vem de um anúncio "clique para o WhatsApp", o evento da Z-API traz os dados do anúncio (identificador, endereço, tipo de origem e o identificador do clique). O sistema grava esses dados na conversa e no lead (`anuncio`, `campanha`, `origem = meta_ads`) e os usa no dashboard de anúncios (SPEC-4-009). É a principal forma de saber de qual anúncio veio um lead que chega pelo WhatsApp.
5. **Supressão prevalece.** Nenhum envio — manual, de lista, de régua, de agente ou de automação — sai para contato suprimido. A conferência é feita no servidor, no momento do envio.
6. **Toda mensagem enviada conta como tentativa** (SPEC-4-005): a primeira mensagem de saída do dia para um lead, por pessoa, régua ou lista, cria uma tentativa de canal WhatsApp. As seguintes do mesmo dia não criam outra. A resposta do contato atualiza o resultado da tentativa para "respondeu". A produtividade (SPEC-4-009) passa a ser calculada sem digitação.
7. **Situação da conversa.** Mensagem recebida reabre conversa resolvida e a põe em `aberta`. Mensagem enviada por pessoa põe em `aguardando`. `resolvida` é sempre ação humana ou do agente autorizado.
8. **Dono.** A primeira pessoa que responde uma conversa sem dono vira a dona. Conversa de lead com responsável nasce com esse responsável como dono.
9. **Humano tem prioridade sobre a IA.** Se uma pessoa envia mensagem em uma conversa com agente ativo, o agente é pausado naquela conversa até alguém clicar em "Devolver à IA".
10. **Mídia.** Arquivos recebidos são baixados da Z-API para o armazenamento do sistema assim que chegam (o endereço fornecido por ela expira). Limite de 16 MB por arquivo; acima disso, fica o registro "arquivo grande demais" com o nome.
11. **Envio.** Passa sempre pela `outbox`. Falha de rede: até 3 tentativas com intervalo crescente; depois, `falhou`, com o botão "Tentar de novo". Canal desconectado: a mensagem fica `na_fila` por até 30 minutos e a tela avisa.
12. **Ritmo.** Mensagens de conversa (pessoa respondendo) saem na hora. Envios de lista, régua e agente respeitam o ritmo configurado e nunca saem dois ao mesmo tempo no mesmo canal.
13. **Privacidade.** O conteúdo das conversas é dado pessoal: só `admin`, `gestor` e `sdr` leem; `marketing` não acessa; exportação de conversa não existe nesta fase; o registro de eventos guarda o conteúdo mascarado.
14. **Credenciais** da Z-API ficam só no servidor. Todas as chamadas à Z-API saem do backend, nunca do navegador.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Contato conhecido escreve | Mensagem aparece em Conversas em até 3 segundos, sem recarregar; contador de não lidas sobe. |
| Principal | Lidiane responde com texto e um PDF | Balões aparecem como "enviando", depois "enviada", "entregue", "lida"; tentativa do dia registrada. |
| Principal | Número desconhecido escreve a partir de um anúncio | Lead criado na etapa de entrada, com origem Meta Ads e o anúncio identificado; conversa em "Sem dono". |
| Principal | Mover a etapa pelo painel ao lado da conversa | Kanban atualiza; linha "mudou para Qualificado" aparece na conversa. |
| Principal | Enviar lista de 80 contatos, aprovada | Envios saem um a um, com intervalo aleatório; andamento em tempo real. |
| Limite | A mesma mensagem chega duas vezes da Z-API | Uma só na conversa. |
| Limite | Mensagem enviada direto do celular | Aparece na conversa como "pelo celular". |
| Limite | Mensagem de grupo | Ignorada. |
| Limite | Contato responde durante uma régua automática | Envios pendentes cancelados; conversa aberta para a pessoa. |
| Limite | 5% de recusas em um envio de lista | Envio pausa sozinho e avisa. |
| Falha | Canal desconecta | Faixa vermelha em Conversas; aviso ao admin; mensagens ficam na fila; listas pausam. |
| Falha | Enviar para contato suprimido | Negado no servidor e auditado. |
| Falha | Chamada ao endereço de recebimento com segredo errado | 401, registrada. |
| Falha | Régua configurada como automática com a chave do canal desligada | Nada é enviado; a tentativa aparece na fila como manual. |
| Recuperação | Reconectar o canal pelo QR code | Mensagens na fila saem; eventos recebidos voltam a chegar. |

## Estados vazios, erro e permissão

- Sem canal conectado: Conversas mostra "Nenhum número de WhatsApp conectado." e, para `admin`, o botão **Conectar número**.
- Sem conversas: "Nenhuma conversa ainda."
- Conversa sem dono aberta por `sdr`: pode responder; ao responder, vira dono.

## Critérios de aceite

- [ ] **CA-4-47:** O administrador conecta um número de WhatsApp lendo o QR code na tela da Central; as credenciais da Z-API nunca aparecem em tela nem trafegam para o navegador; a desconexão do número é avisada na tela de conversas e ao administrador.
- [ ] **CA-4-48:** Uma mensagem recebida aparece em Conversas em até 3 segundos, sem recarregar; texto, imagem, áudio e documento são enviados e mostram os estados (enviada, entregue, lida, falhou); o mesmo evento recebido duas vezes não duplica; grupos são ignorados.
- [ ] **CA-4-49:** Um número desconhecido que escreve vira lead na etapa de entrada do funil, com a origem do canal ou do anúncio; ao lado da conversa, o painel do lead permite editar campos, mover a etapa, anotar e agendar sem sair da tela.
- [ ] **CA-4-50:** Nenhuma mensagem — de pessoa, lista, régua, agente ou automação — é enviada a contato suprimido, e a tentativa é negada no servidor e auditada; toda primeira mensagem do dia para um lead é registrada como tentativa, sem digitação.
- [ ] **CA-4-51:** O envio de lista pela Central exige aprovação quando previsto, sai com intervalo aleatório dentro do ritmo configurado, respeita o limite diário e pausa sozinho por recusas, falhas ou desconexão; a régua automática envia uma tentativa por vez e cancela os envios pendentes quando o contato responde; com as chaves do canal desligadas, nada automático é enviado.
- [ ] **CA-4-52:** A conversa tem dono e situação (aberta, aguardando, resolvida); assumir, transferir e resolver ficam registrados; notas internas aparecem só para a equipe; os modelos de mensagem são inseridos com as variáveis preenchidas.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Ida e volta | com um número de teste, trocar 10 mensagens de cada tipo (texto, imagem, áudio, documento) | gravação e conferência dos estados |
| Idempotência | reenviar 3 vezes o mesmo evento de `fixtures/zapi-eventos-fase4.json` | contagem de mensagens |
| Número novo e anúncio | evento de exemplo com dados de anúncio | lead criado com a origem e o anúncio |
| Supressão | tentar os 5 caminhos de envio para um contato suprimido | 5 negações auditadas |
| Envio de lista | lista de 20 números de teste, com ritmo de 20 a 30 s | horários de envio no registro, todos com o intervalo; pausa e retomada |
| Régua automática | régua de 3 tentativas em modo automático com intervalos de minutos, em ambiente de teste | uma tentativa por vez; cancelamento ao responder |
| Desconexão | desconectar o canal no meio de um envio | faixa, pausa e retomada |
| Credenciais | inspecionar as respostas das rotas de canal e o tráfego do navegador | nenhum token presente |

**Dados de teste:** `fixtures/zapi-eventos-fase4.json`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.3 | Contratar a Z-API e criar uma instância por número; ativar o token de segurança da conta; criar o usuário do sistema e o token de leitura do Meta Ads; pedir o token de desenvolvedor do Google Ads; conferir os conectores habilitados no Skip | Pontual (Marcelo) | SPEC-4-013 | CA-4-47 | Manual: itens 1 a 4 da lista de conferência de integracoes.md | credenciais entregues por canal seguro + protocolo do pedido ao Google + lista de conectores | Fase 3 aceita | ☐ | A |
| T4.44 | Criar os canais e o adaptador da Z-API em um único arquivo do backend: guardar credenciais só no servidor, conectar por QR code com renovação, situação, desconectar, reiniciar, registrar o endereço de eventos; e a tela Canais | Dev backend | SPEC-4-013 | CA-4-47 | Principal: conectar um número de teste; falha: credenciais não aparecem em rota nem no navegador; desconexão avisada | gravação da conexão + inspeção das respostas e do tráfego | T4.3 aceita; T4.24 aceita; T4.40 aceita | ☐ | I |
| T4.45 | Implementar o recebimento dos eventos da Z-API: validação do segredo e da instância, idempotência por identificador de mensagem, mensagens de texto e mídia (baixando os arquivos), mudança de estado, mensagens enviadas pelo celular, grupos ignorados, lead novo para número desconhecido e dados do anúncio de origem | Dev backend | SPEC-4-013 | CA-4-48, CA-4-49 | RED/GREEN: mesmo evento três vezes; principal e limite com fixtures/zapi-eventos-fase4.json | contagem de mensagens + lead criado com origem e anúncio | T4.44 aceita; T4.23 aceita | ☐ | I |
| T4.46 | Implementar a fila de envio e o envio de texto e mídia pela Z-API, com estados, novas tentativas, conferência de supressão no servidor, conferência de número com WhatsApp e mensagens em espera quando o canal estiver desconectado | Dev backend | SPEC-4-013 | CA-4-48, CA-4-50 | RED/GREEN: envio a contato suprimido; falha: canal desconectado; limite: servidor reiniciado no meio | registros da fila + negação auditada | T4.45 aceita; T4.14 aceita | ☐ | I |
| T4.47 | Criar a tela Conversas em três colunas com atualização em tempo real: lista com abas, situação, filtros, busca e não lidas; conversa com balões, estados, mídia e eventos do lead; campo de escrever com anexo; e usar o mesmo componente na aba Conversa da ficha | Dev frontend | SPEC-4-013 | CA-4-48 | Principal: trocar dez mensagens de cada tipo com um número de teste | gravação com os estados das mensagens | T4.46 aceita | ☐ | I |
| T4.48 | Completar o atendimento: painel do lead ao lado da conversa (editar, mover etapa, anotar, agendar), assumir, transferir, resolver e reabrir, notas internas na conversa, modelos de mensagem pelo campo de escrever, horário de atendimento e mensagem fora do horário | Dev full-stack | SPEC-4-013 | CA-4-49, CA-4-52 | Principal: atender uma conversa do início ao agendamento sem sair da tela | gravação + registros de dono e situação | T4.47 aceita; T4.41 aceita; T4.36 aceita | ☐ | I |
| T4.49 | Registrar automaticamente a tentativa na primeira mensagem do dia para um lead e atualizar o resultado quando o contato responde; mostrar a conversa não respondida como pendência na fila; trocar os botões de WhatsApp da ficha, da fila e do Kanban por Conversar | Dev backend | SPEC-4-013 | CA-4-50 | Principal: enviar e receber; limite: várias mensagens no mesmo dia geram uma tentativa | produtividade do dia sem digitação + fila com a pendência | T4.48 aceita; T4.34 aceita | ☐ | I |
| T4.54 | Implementar o envio de lista pela Central: escolha de canal, modelo, início, ritmo e limite diário; aprovação; processamento pelo agendador com intervalo aleatório; andamento em tempo real; pausar, retomar, cancelar; pausa automática; preenchimento automático dos retornos | Dev backend | SPEC-4-013 | CA-4-51 | Prova com 20 números de teste: intervalos no registro, pausa e retomada, pausa automática por recusa | horários de envio + gravação do andamento | T4.53 aceita; T4.46 aceita; T4.41 aceita | ☐ | J |
| T4.55 | Implementar o modo automático da régua: tentativa marcada como automática envia o modelo na data calculada, com as conferências antes de cada envio, uma tentativa por vez e cancelamento quando o contato responde; respeitar a chave do canal | Dev backend | SPEC-4-013 | CA-4-51 | RED/GREEN: chave do canal desligada; prova com régua de minutos em ambiente de teste | registro das tentativas, uma por vez + cancelamento ao responder | T4.49 aceita; T4.41 aceita | ☐ | J |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
