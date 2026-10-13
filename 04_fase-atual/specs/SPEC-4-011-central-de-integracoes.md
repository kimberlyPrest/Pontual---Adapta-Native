# SPEC-4-011 — Central de integrações e fila de exceção

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-09, RF-22, RN-14, RNF-05, RNF-10, G1, G6  
**Degrau da solução:** construção mínima — uma tela para ver e controlar o que entra e sai do sistema, e um lugar para o que falhou.

## Resultado observável

O administrador vê, em uma tela, cada integração do sistema, se está funcionando, quando foi o último evento e o que falhou. Cria e revoga as chaves de acesso. Um evento que falhou pode ser corrigido e reprocessado, e nenhuma falha de integração impede o trabalho manual.

O contrato de cada integração — endereços, campos, exemplos e o passo a passo de configuração em cada sistema externo — está em `integracoes.md`. Esta SPEC cobre o que o sistema precisa ter para sustentar aquele documento.

## Por que existe

- A regra RN-14 (falha de integração mantém fila de exceção) está nas SPECs desde a Fase 1 e não tem onde aparecer.
- A captura automática de leads (SPEC-4-002) precisa de chaves de acesso que possam ser criadas e revogadas sem código.
- O conector de Google Sheets e a agenda dependem de cada usuário conectar a própria conta; alguém precisa ver quem conectou.

## Limites e dependências

- **Inclui:** chaves de acesso, registro de eventos de integração, fila de exceção com reprocessamento, painel de situação de todas as integrações (inclusive WhatsApp e anúncios), tela de conectores do usuário, recebimento genérico de mensagens de conversa por chamada externa.
- **Fora de escopo desta SPEC:** o canal de WhatsApp pela Z-API (SPEC-4-013) e a sincronização das contas de anúncio (SPEC-4-009) — esta SPEC entrega a infraestrutura comum que as duas usam (registro de eventos, fila de exceção, painel) e não repete o que está nelas. Qualquer integração não listada em `integracoes.md` está fora.
- **Pré-condições:** SPEC-4-002 aceita (a rota de captura de leads usa as chaves e o registro de eventos desta SPEC; as duas são construídas na mesma leva).
- **Gate G6 (contrato técnico do canal):** resolvido pela escolha da Z-API, cujo contrato está em `integracoes.md`, seção 4. A rota genérica de recebimento de mensagens desta SPEC continua existindo para outros fornecedores (por exemplo, o RespondeChat durante a transição, se ele oferecer envio de eventos — `integracoes.md`, seção 8).

## Telas e comportamento

Configurações → **Integrações** (`admin`).

### Painel de situação

Um cartão por integração:

| Integração | Sentido | O cartão mostra |
|---|---|---|
| Captura de leads | entrada | chaves ativas, eventos em 24 h e 7 dias, último evento, falhas pendentes |
| WhatsApp (Z-API) | entrada e saída | um cartão por canal: conectado ou não, número, último evento recebido, mensagens em 24 h (recebidas e enviadas), envios na fila, falhas de envio |
| Meta Ads | entrada (sincronização) | conta conectada, última sincronização, dias cobertos, erro da última tentativa |
| Google Ads | entrada (sincronização ou relatório) | o mesmo; ou "por relatório importado" com a data do último |
| Mensagens de conversa (rota genérica) | entrada | chaves ativas, eventos, falhas |
| Google Sheets | entrada (importação) | quantos usuários conectaram, última importação por planilha |
| Agenda | saída (convite) | convites gerados em 7 dias; se o conector de agenda estiver habilitado, quantos usuários conectaram |
| Exportação de listas | saída (arquivo) | última exportação, formato configurado |
| Agentes de IA | interna | disponível ou não, agentes publicados, pedidos em 24 h, erros em 24 h, uso do mês |

Situação por cor: verde (evento com sucesso nas últimas 24 h ou sem uso esperado), âmbar (falhas pendentes), cinza (nunca usado), vermelho (todas as chamadas das últimas 24 h falharam).

Cada cartão tem o link **Como configurar**, que abre a seção correspondente de `integracoes.md` reproduzida em uma página de ajuda dentro do sistema.

### Chaves de acesso

Tabela: nome, tipo (captura de leads, mensagens), origem padrão, responsável padrão, criada por, criada em, último uso, situação.

- **Nova chave:** nome (ex.: "Formulário do site"), tipo, origem padrão (catálogo), responsável padrão (opcional). O valor da chave aparece **uma única vez**, com o botão Copiar e o aviso "Guarde agora. Não será mostrada de novo."
- **Revogar:** imediato; pede motivo.
- **Testar:** envia um evento de teste pela própria tela e mostra a resposta. O evento de teste é marcado e não cria lead.
- Ao lado, o exemplo de chamada pronto para copiar, já com o endereço do ambiente.

### Registro de eventos

Tabela de tudo o que entrou: data e hora, integração, chave, situação (`processado`, `duplicado`, `repetido`, `negado`, `falhou`, `descartado`), o que gerou (link para o lead), tempo de processamento.

- Filtros: integração, situação, período, chave.
- Abrir um evento mostra o conteúdo recebido com telefone, e-mail e documento mascarados, e a resposta devolvida.

### Fila de exceção

A mesma tabela, filtrada em `falhou`, com contador no menu.

Para cada evento com falha: o motivo em linguagem clara ("telefone sem DDD", "origem não existe no catálogo", "erro interno"), e as ações:

- **Corrigir e reprocessar:** mostra os campos do evento em um formulário, deixa corrigir e processa de novo. A correção fica registrada.
- **Reprocessar** como está (para erro interno já resolvido).
- **Descartar**, com motivo.

Ações em massa para eventos com o mesmo motivo. Evento reprocessado com sucesso sai da fila e mantém o registro da falha original.

### Meus conectores

No menu do usuário → **Minhas conexões**. Lista os conectores habilitados no ambiente (`listConnectors`, de `src/lib/skipConnectors.ts`) com a situação de cada um para aquele usuário — conectado, não conectado, expirado — e os botões Conectar e Desconectar. Cada usuário conecta a **própria** conta; o sistema age com ela só quando o próprio usuário pede (importar uma planilha, criar um evento).

## Recebimento genérico de mensagens

O WhatsApp pela Z-API **não** usa esta rota: ele tem o endereço próprio de cada canal (SPEC-4-013), porque o formato dos eventos é o da Z-API. Esta rota é o contrato neutro da Central para qualquer outro sistema que queira entregar mensagens.

### `POST /backend/v1/integrations/inbound/message`

Rota pública, autenticada por chave do tipo "mensagens" no cabeçalho `X-Integration-Token`. Contrato completo em `integracoes.md`, seção 3.2.

| Campo | Obrigatório | Observação |
|---|---|---|
| `idempotency_key` | sim | identificador da mensagem na origem |
| `telefone` | sim | do contato |
| `direcao` | sim | `entrada` (do contato) ou `saida` (da Pontual) |
| `conteudo` | sim | texto; mídia entra como "[imagem]", "[áudio]", "[arquivo]" |
| `timestamp` | sim | ISO 8601 |
| `canal` | não | padrão `whatsapp` |
| `autor` | não | nome do atendente ou "robô" |
| `conversa_id` | não | identificador da conversa na origem |

Comportamento:

1. Localiza o lead pelo telefone. Não achou: cria um lead em `novo` com origem padrão da chave (ou `desconhecida`) **se** a direção for `entrada`; mensagem de saída para número desconhecido vai para a fila de exceção.
2. Grava em `conversations` e `messages`, como a importação de conversas da Fase 3; a mesma `idempotency_key` não duplica (CA-3-01 continua valendo).
3. Mensagem de `entrada` atualiza `ultima_interacao_at`; se o lead estava em `novo`, cria a próxima ação "Responder mensagem" para agora.
4. Mensagem de `entrada` de contato suprimido não cria ação; abre transbordo "contato suprimido enviou mensagem".
5. Mensagem de `entrada` cujo texto, sem acento e em minúsculas, é exatamente uma das expressões de recusa configuradas (padrão: "sair", "parar", "não tenho interesse", "remover") abre transbordo com motivo `RECUSA` — a supressão continua sendo confirmada por uma pessoa.
6. Esta rota não envia resposta ao contato. Se a conversa tiver agente de IA ativo, vale a SPEC-4-008 — o que só é possível quando o canal de origem também tem envio, como o da Z-API.

## Dados

### `integration_tokens`

`nome`, `tipo` (`inbound_lead`, `inbound_message`), `token_hash` (o valor em claro nunca é guardado), `prefixo` (4 primeiros caracteres, para identificar na tela), `origem_padrao`, `responsavel_padrao`, `ativo`, `criado_por`, `revogado_por`, `revogado_em`, `revogado_motivo`, `ultimo_uso`.

### `integration_events`

`integracao`, `token_id`, `idempotency_key`, `status`, `payload` (json, com telefone, e-mail e documento mascarados; o conteúdo completo necessário para reprocessar fica em `payload_reprocesso`, apagado quando o evento é processado ou descartado), `erro_codigo`, `erro_mensagem`, `lead_id`, `tentativas`, `duracao_ms`, `teste` (bool), `resolvido_por`, `resolvido_em`, `resolucao` (`reprocessado`, `corrigido`, `descartado`), `created`.

Índice único em (`integracao`, `idempotency_key`). Eventos processados são apagados depois de 90 dias; os contadores diários ficam em `integration_stats`.

## API

| Rota | Papéis | Função |
|---|---|---|
| `GET /backend/v1/integrations/status` | admin | painel de situação. |
| `GET/POST /backend/v1/integrations/tokens` | admin | lista e cria (o valor só vem na resposta da criação). |
| `POST /backend/v1/integrations/tokens/{id}/revoke` | admin | revoga. |
| `POST /backend/v1/integrations/tokens/{id}/test` | admin | evento de teste. |
| `GET /backend/v1/integrations/events` · `GET .../events/{id}` | admin | registro e fila de exceção. |
| `POST /backend/v1/integrations/events/{id}/retry` | admin | reprocessa, com ou sem correção. |
| `POST /backend/v1/integrations/events/{id}/discard` | admin | descarta, com motivo. |
| `POST /backend/v1/integrations/inbound/lead` | chave | SPEC-4-002. |
| `POST /backend/v1/integrations/inbound/message` | chave | acima. |

## Regras

1. **Falha de integração nunca bloqueia a operação manual** (RN-14, RNF-10): cadastro, importação por arquivo, fila e agendamento funcionam com todas as integrações fora do ar. Os eventos da Z-API e as sincronizações de anúncio gravam em `integration_events` e usam a mesma fila de exceção desta SPEC.
2. **As rotas de entrada respondem rápido:** validam, gravam o evento e respondem em até 2 segundos. O processamento que falhar por erro interno deixa o evento como `falhou` e responde 202, para a origem não reenviar em laço.
3. **Limite:** 60 chamadas por minuto por chave; acima disso, 429.
4. **Tamanho:** corpo de até 64 KB; maior, 413.
5. **Comparação da chave** em tempo constante; chave inválida não revela se o problema é a chave ou o formato.
6. **Nenhuma chave, senha ou credencial no código** nem em migração. Segredos de ambiente usam o cofre de segredos da plataforma (`$secrets`), como a Fase 1 já faz para as fixtures.
7. **Dado pessoal no registro de eventos fica mascarado.** O conteúdo completo existe só enquanto o evento estiver pendente.
8. Criar, revogar, testar, reprocessar e descartar gravam auditoria.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Admin cria a chave "Formulário do site" e clica em Testar | Resposta 201 de teste na tela; evento de teste no registro; nenhum lead criado. |
| Principal | Mensagem de entrada de um telefone conhecido | Aparece na aba Conversa do lead; próxima ação criada se era lead novo. |
| Limite | Mesma mensagem enviada duas vezes | Uma só na conversa; segundo evento `repetido`. |
| Limite | Lead recebido com origem que não existe no catálogo | Lead criado com `desconhecida`; evento `processado` com aviso. |
| Limite | Chamada número 61 no mesmo minuto | 429. |
| Falha | Lead recebido sem telefone e sem e-mail | 422; evento `falhou` na fila de exceção com o motivo. |
| Falha | Chave revogada | 401; evento `negado`. |
| Recuperação | Corrigir o telefone de um evento com falha e reprocessar | Lead criado; evento sai da fila; a falha original continua registrada. |
| Recuperação | Todas as integrações desligadas | Cadastro manual, importação e fila funcionam. |

## Estados vazios, erro e permissão

- Sem chaves: "Nenhuma chave criada. Crie uma para receber leads do site ou de anúncios."
- Fila de exceção vazia: "Nenhum evento com falha."
- Conector não habilitado no ambiente: o cartão aparece em cinza com "Não habilitado neste ambiente" e a instrução de como pedir a habilitação.

## Critérios de aceite

- [ ] **CA-4-41:** A central de integrações mostra a situação de cada integração e o último evento; o administrador cria, testa e revoga chaves de acesso, e o valor da chave aparece uma única vez.
- [ ] **CA-4-42:** Um evento que falha vai para a fila de exceção com o motivo em linguagem clara e pode ser corrigido e reprocessado, ou descartado com motivo; com todas as integrações desligadas, cadastro, importação, fila e agendamento continuam funcionando.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Contrato | executar todos os exemplos de `integracoes.md` e de `fixtures/captura-fase4.json` contra o ambiente | tabela chamada × resposta |
| Idempotência | repetir 3 vezes a mesma chamada de lead e de mensagem | contagem de registros |
| Fila de exceção | provocar 3 falhas diferentes, corrigir uma, reprocessar uma, descartar uma | capturas |
| Operação sem integração | revogar todas as chaves e percorrer cadastro, importação e fila | gravação |
| Segredos | busca no repositório por chaves e senhas | resultado vazio |

**Dados de teste:** `fixtures/captura-fase4.json`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.24 | Criar as chaves de acesso (com o valor guardado só como resumo criptográfico), o registro de eventos de integração com mascaramento, o limite de chamadas e as rotas de criar, revogar e testar chave | Dev backend | SPEC-4-011 | CA-4-41 | RED/GREEN: chave revogada; limite: chamada acima do limite | respostas das rotas + registro de eventos mascarado | Fase 3 aceita | ☐ | D |
| T4.70 | Criar a central de integrações: painel de situação com um cartão por integração (captura, WhatsApp, Meta Ads, Google Ads, Google Sheets, agenda, IA), tela de chaves, registro de eventos e fila de exceção com corrigir, reprocessar e descartar | Dev full-stack | SPEC-4-011 | CA-4-41, CA-4-42 | Prova: três falhas diferentes — corrigir uma, reprocessar uma, descartar uma; operação com tudo desligado | capturas + gravação da operação sem integrações | T4.44 aceita; T4.66 aceita; T4.25 aceita; T4.56 aceita | ☐ | M |
| T4.71 | Implementar a rota genérica de recebimento de mensagens para outros sistemas, com idempotência e as regras de lead novo, supressão e recusa | Dev backend | SPEC-4-011 | CA-4-42 | Prova: executar os exemplos de integracoes.md contra o ambiente | tabela chamada × resposta | T4.24 aceita; T4.45 aceita | ☐ | M |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
