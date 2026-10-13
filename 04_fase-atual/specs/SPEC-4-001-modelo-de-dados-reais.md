# SPEC-4-001 — Modelo de dados reais do lead, catálogos e funil até o agendamento

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-02, RF-04, RF-06, RF-07, RF-16, RN-01, RN-02, RN-05, RN-08  
**Degrau da solução:** fundação — nada novo aparece para o usuário nesta SPEC além das colunas do Kanban; ela existe para que todas as outras telas da fase trabalhem com dado real e para que funil e campos possam ser configurados pela interface (SPEC-4-012).

## Resultado observável

O lead passa a ter os campos que a operação comercial da Pontual realmente usa (os mesmos da planilha "CRM - Funil de Vendas"), as listas de origem, produto, segmento e motivo vêm de catálogos editáveis, e o funil padrão vai até **Agendado**. As etapas deixam de estar escritas no código: passam a ser dados, que o administrador edita pela tela (SPEC-4-012). Os leads que já existem continuam lá, com o mesmo histórico.

## Por que existe

- O lead hoje só tem `nome_ficticio`, `email_ficticio`, `telefone_ficticio`, origem, campanha, produto e status. A planilha real tem decisor, segmento, cidade, sistema atual, fonte, tentativas, última interação, etapa, produto, valor e observações.
- O Kanban termina em "Qualificado", mas o limite funcional do projeto é o agendamento da apresentação (RN-01).
- O funil atual só anda em linha reta (`novo → triagem → em_contato`). Na prática, indicação pula a triagem e lead de anúncio errado é descartado na hora.

## Limites e dependências

- **Inclui:** migração aditiva da coleção `leads`, catálogos com dados iniciais, funis e etapas como dados (com o funil padrão), categorias de etapa e suas regras, área de campos personalizados no lead, normalizadores compartilhados (telefone, CNPJ, e-mail, data, texto), compatibilidade das rotas existentes.
- **Fora de escopo:** telas de edição do lead (SPEC-4-004), editor de funil e de campos (SPEC-4-012), tela de catálogos (SPEC-4-010), importação (SPEC-4-003).
- **Pré-condições:** Fase 3 aceita.
- **Risco e plano B:** se a renomeação de campo quebrar alguma rota antiga, os campos `*_ficticio` ficam no banco como cópia somente leitura até a regressão da SPEC-4-014; nenhuma rota nova lê deles.
- **Rollback:** a migração é aditiva. Reverter é remover os campos novos; nada antigo é apagado nesta SPEC.

## Dados

### Coleção `leads` — campos novos

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `tipo` | select `lead`, `prospect` | sim | padrão `lead`. `prospect` é contato de base fria (SPEC-4-006) e não entra no Kanban nem na fila. |
| `empresa_nome` | texto 255 | condicional | recebe o valor de `nome_ficticio`. Obrigatório `empresa_nome` **ou** `contato_nome`. |
| `nome_fantasia` | texto 255 | não | |
| `cnpj` | texto 14 | não | só dígitos; valida dígito verificador; único quando preenchido. |
| `contato_nome` | texto 255 | condicional | decisor ou pessoa de contato. |
| `contato_cargo` | texto 120 | não | |
| `telefone` | texto 13 | sim para `lead` | só dígitos com DDI: `55` + DDD + número (12 ou 13 dígitos). Recebe `telefone_ficticio` normalizado. |
| `telefone_secundario` | texto 13 | não | mesma regra. |
| `email` | email | não | recebe `email_ficticio`. Deixa de ser obrigatório: boa parte dos leads reais não tem e-mail. |
| `segmento` | texto (código do catálogo) | não | |
| `cidade`, `bairro` | texto 120 | não | |
| `uf` | texto 2 | não | sigla válida. |
| `sistema_atual` | texto 255 | não | ex.: "nenhum", "planilha", nome do concorrente. |
| `regime_tributario` | select `simples`, `presumido`, `real`, `mei`, `nao_informado` | não | padrão `nao_informado`. |
| `situacao_loja` | select `aberta`, `em_abertura`, `nao_informado` | não | |
| `usuarios_qtd`, `lojas_qtd` | número inteiro ≥ 0 | não | |
| `origem` | texto (código do catálogo) | sim | já existe; passa a aceitar só códigos do catálogo. |
| `origem_detalhe` | texto 255 | não | ex.: "indicado por Fulano", nome do parceiro. |
| `campanha`, `conjunto`, `anuncio` | texto 255 | não | `campanha` já existe. |
| `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term` | texto 255 | não | preenchidos pela captura automática (SPEC-4-002). |
| `produto` | texto (código do catálogo) | sim | já existe; padrão `a_definir`. |
| `responsavel_id` | relação com `users` | não | lead ativo sem responsável aparece como exceção na fila (RN-05). |
| `data_entrada` | data | sim | padrão: data de criação. A importação preserva a data original. |
| `next_action_tipo` | select `whatsapp`, `ligacao`, `email`, `reuniao`, `outro` | não | complementa `next_action` e `next_action_at`, que já existem. |
| `tentativas_count` | número | sim | padrão 0; mantido pelo backend (SPEC-4-005). |
| `ultima_interacao_at` | data | não | mantido pelo backend. |
| `ultima_interacao_canal` | texto | não | mantido pelo backend. |
| `nutricao_retorno_em` | data | condicional | obrigatório quando `stage = nutricao`. |
| `motivo_codigo` | texto (código do catálogo) | condicional | obrigatório em `desqualificado` e `nutricao`. |
| `motivo_detalhe` | texto 500 | não | |
| `resultado_pos` | select `nenhum`, `apresentacao_realizada`, `no_show`, `proposta_enviada`, `ganho`, `perdido` | sim | padrão `nenhum`. Informativo: não cria fluxo (SPEC-4-007). |
| `valor_proposta` | texto 120 | não | texto livre, ex.: "R$ 500,00 + R$ 275,00/mês". |
| `data_fechamento` | data | não | |
| `suprimido` | bool | sim | padrão falso; espelha a existência de registro em `contact_suppressions`. |
| `historico` | bool | sim | padrão falso; verdadeiro para lead importado já encerrado. Não gera item de fila. |
| `ultima_nota` | texto 280 | não | trecho da última nota, para o card (SPEC-4-004). |
| `pipeline_id` | relação com `pipelines` | sim | funil em que o lead está; padrão: o funil padrão. |
| `stage` | texto | sim | deixa de ser lista fixa: guarda o `codigo` de uma etapa de `pipeline_stages` do funil do lead. |
| `stage_entrou_em` | data | sim | quando o lead entrou na etapa atual; usado para o alerta de lead parado. |
| `custom` | json | não | valores dos campos personalizados, por código do campo (SPEC-4-012). |
| `ia_pausada` | bool | sim | padrão falso; verdadeiro quando uma pessoa assumiu a conversa (SPEC-4-008, SPEC-4-013). |

Índices: `telefone`, `cnpj`, `email`, `stage`, `responsavel_id`, `next_action_at`, `tipo`, `origem`.

Os campos `nome_ficticio`, `email_ficticio` e `telefone_ficticio` continuam no banco, sem uso por rotas novas, até a T4.74.

### Coleção nova `catalog_items`

| Campo | Tipo | Regra |
|---|---|---|
| `tipo` | select `origem`, `produto`, `segmento`, `motivo`, `etiqueta` | |
| `codigo` | texto 64 | minúsculas, sem acento, `_` no lugar de espaço; único por `tipo`; nunca muda depois de criado. |
| `rotulo` | texto 120 | o que aparece na tela. |
| `grupo` | texto 64 | para `motivo`: `desqualificacao`, `nutricao`, `perda`; para `origem`: `pago`, `organico`, `parceiro`, `ativo`, `outro`. |
| `ativo` | bool | item inativo some dos formulários, mas continua valendo para o histórico. |
| `ordem` | número | |
| `meta` | json | ex.: para motivo, `{"gera_supressao": true, "nutricao_dias": 45}`. |

### Dados iniciais dos catálogos

| Tipo | Itens (código → rótulo) |
|---|---|
| origem | `google_ads` → Google Ads · `meta_ads` → Meta Ads · `youtube` → YouTube · `site_organico` → Site / orgânico · `indicacao` → Indicação · `parceiro_hiper` → Parceiro Hiper · `disparo_massivo` → Disparo massivo · `contato_antigo` → Contato antigo · `suporte_interno` → Encaminhado pelo Suporte · `desconhecida` → Desconhecida |
| produto | `automec` → Automec · `climb` → Climb · `hiper_gestao` → Hiper Gestão · `hiper_mini` → Hiper Mini · `certificado` → Certificado digital · `a_definir` → A definir |
| segmento | `material_construcao` → Material de construção · `ferragens` → Ferragens · `madeireira` → Madeireira · `varejo_geral` → Comércio varejista · `outro` → Outro · `a_classificar` → A classificar |
| motivo (desqualificacao) | `fora_do_segmento` → Fora do segmento atendido · `buscava_produto` → Buscava comprar material, não o sistema · `ja_tem_sistema` → Já tem sistema consolidado · `sem_interesse` → Sem interesse (gera supressão) · `sem_resposta` → Sem resposta após a cadência · `contato_invalido` → Telefone ou contato inválido · `duplicado` → Duplicado |
| motivo (nutricao) | `optou_gratuito` → Optou por sistema gratuito (retorno em 45 dias) · `sem_orcamento_agora` → Sem orçamento agora · `pediu_retorno` → Pediu contato mais adiante · `loja_em_abertura` → Loja ainda em abertura |
| motivo (perda) | `perdido_concorrente` → Fechou com concorrente · `perdido_preco` → Preço |

Os itens `desconhecida`, `a_definir` e `a_classificar` são protegidos: não podem ser desativados.

### Funis e etapas como dados

As etapas saem de `src/lib/leadStages.js` e do código do backend e passam a morar em duas coleções. O arquivo `leadStages.js` deixa de ter a lista fixa: a tela busca as etapas pela API.

**Coleção nova `pipelines`**

| Campo | Tipo | Regra |
|---|---|---|
| `nome` | texto 80 | ex.: "Comercial" |
| `codigo` | texto 64 | imutável |
| `padrao` | bool | exatamente um funil é o padrão |
| `ativo` | bool | |
| `ordem` | número | |

**Coleção nova `pipeline_stages`**

| Campo | Tipo | Regra |
|---|---|---|
| `pipeline_id` | relação | |
| `codigo` | texto 64 | imutável; único dentro do funil |
| `rotulo` | texto 60 | o que aparece na coluna |
| `categoria` | select (lista abaixo) | define o comportamento; é o que o sistema usa nas regras e nos indicadores |
| `ordem` | número | posição da coluna |
| `cor` | texto 7 | cor da coluna |
| `ativo` | bool | etapa inativa não recebe leads novos e some do Kanban quando vazia |
| `dias_alerta` | número | lead parado na etapa há mais dias que isso ganha o selo "parado"; vazio = sem alerta |
| `campos_obrigatorios` | json | códigos de campos (de sistema ou personalizados) que precisam estar preenchidos para o lead entrar na etapa |
| `ao_entrar` | json | ações automáticas ao entrar (SPEC-4-012) |
| `recolhida` | bool | coluna começa recolhida no Kanban |

**Categorias de etapa (fixas, do sistema)**

A categoria é o que dá significado à etapa. O administrador cria quantas etapas quiser e dá o nome que quiser; cada uma pertence a uma categoria.

| Categoria | Significado | Regra ao entrar |
|---|---|---|
| `entrada` | lead que acabou de chegar | nenhuma |
| `andamento` | sendo trabalhado, ainda não qualificado | nenhuma |
| `qualificado` | aderente ao ICP, decisão registrada | exige qualificação registrada: critério, evidência e autor (SPEC-4-004) |
| `agendamento` | apresentação pedida, ainda não confirmada | exige agendamento em aberto (SPEC-4-007) |
| `agendado` | apresentação confirmada | exige agendamento confirmado; só acontece pela confirmação |
| `pos_apresentacao` | depois da apresentação (proposta, negociação) | exige apresentação realizada |
| `ganho` | fechou | grava `resultado_pos = ganho` |
| `nutricao` | pausa com data de retorno | exige `motivo_codigo` e `nutricao_retorno_em` |
| `perdido` | desqualificado ou perdido | exige `motivo_codigo`; motivo com `gera_supressao` cria a supressão |

Regras gerais de movimentação:

1. O lead pode ir de qualquer etapa para qualquer outra do mesmo funil, desde que cumpra a regra da categoria de destino e os `campos_obrigatorios` da etapa. Não existe mais lista de "próximas etapas permitidas".
2. Sair de `perdido` (reabrir) só para `admin` e `gestor`, com motivo.
3. Sair de `agendado` para trás acontece ao reagendar, cancelar ou registrar falta (SPEC-4-007).
4. Mudar o lead de funil é permitido para `admin`, `gestor` e `sdr`; o lead entra na primeira etapa de categoria `entrada` do novo funil, salvo escolha de outra, e a mudança vai para a timeline.
5. Toda movimentação continua gravando `lead_stage_transitions` e `lead_critical_timeline`, agora com `pipeline_id` e o código da etapa. As transições antigas permanecem como estão.
6. **Exatamente uma** etapa de cada uma das categorias `entrada`, `agendamento`, `agendado` e pelo menos uma de `perdido` e `nutricao` precisam existir e estar ativas em cada funil; o editor não deixa desativar a última.

**Funil padrão (dados iniciais)** — funil "Comercial":

| Ordem | Código | Rótulo | Categoria | Observação |
|---|---|---|---|---|
| 1 | `novo` | Novo | `entrada` | |
| 2 | `triagem` | Triagem | `andamento` | |
| 3 | `em_contato` | Em contato | `andamento` | |
| 4 | `qualificado` | Qualificado | `qualificado` | |
| 5 | `agendamento_solicitado` | Agendamento solicitado | `agendamento` | |
| 6 | `agendado` | Agendado | `agendado` | |
| 7 | `proposta_enviada` | Proposta enviada | `pos_apresentacao` | criada inativa; a Pontual decide na T4.2 se o funil continua depois do agendamento |
| 8 | `negociacao` | Negociação | `pos_apresentacao` | criada inativa |
| 9 | `ganho` | Fechado – ganho | `ganho` | criada inativa |
| 10 | `nutricao` | Nutrição | `nutricao` | recolhida |
| 11 | `desqualificado` | Desqualificado | `perdido` | recolhida |

Os leads existentes mantêm o código de etapa que já tinham (`novo`, `triagem`, `em_contato`, `qualificado`, `desqualificado`, `nutricao`), que coincide com o do funil padrão.

**Leitura das outras SPECs:** onde as demais SPECs desta fase citam uma etapa pelo código do funil padrão, a regra vale para a **categoria** correspondente — `novo` → `entrada`; `triagem` e `em_contato` → `andamento`; `qualificado` → `qualificado`; `agendamento_solicitado` → `agendamento`; `agendado` → `agendado`; `nutricao` → `nutricao`; `desqualificado` → `perdido`. "Estágios ativos" são as categorias `entrada`, `andamento`, `qualificado` e `agendamento`. O código nunca deve comparar o rótulo nem o código da etapa para decidir comportamento: compara a categoria.

## Normalizadores compartilhados

Funções puras, usadas por formulário, importação e webhook. Uma implementação só, com teste unitário.

| Função | Entrada → saída | Exemplos |
|---|---|---|
| `normalizarTelefone` | texto → dígitos com DDI ou erro | `(21) 99999-8888` → `5521999998888` · `021 3333-4444` → `552133334444` · `+55 21 99999 8888` → `5521999998888` · `99999-8888` → erro "sem DDD" |
| `formatarTelefone` | dígitos → exibição | `5521999998888` → `(21) 99999-8888` |
| `normalizarCnpj` | texto → 14 dígitos ou erro | pontuação removida; dígito verificador conferido |
| `normalizarEmail` | texto → minúsculas sem espaços ou erro | |
| `interpretarData` | texto → data ISO ou erro | `10/07/2026`, `10/07/26`, `2026-07-10`, número serial do Excel |
| `limparTexto` | texto → texto seguro | remove espaços duplos e neutraliza prefixos `=`, `+`, `-`, `@`, tab e CR, como a exportação já faz |

## API

Nenhuma rota nova. Ajustes de compatibilidade nas rotas existentes:

| Rota | Ajuste |
|---|---|
| `GET /backend/v1/pipelines` | **nova.** Devolve os funis ativos com as etapas (código, rótulo, categoria, ordem, cor, alerta, campos obrigatórios). Papéis: todos. |
| `GET /backend/v1/leads/kanban?pipeline=` | devolve as colunas do funil pedido (padrão: o funil padrão), na ordem configurada; cada item ganha `companyName`, `contactName`, `phone`, `ownerId`, `ownerName`, `nextAction`, `nextActionAt`, `nextActionType`, `attemptsCount`, `lastInteractionAt`, `lastNote`, `suppressed`. O campo `name` continua existindo (igual a `companyName` ou, se vazio, `contactName`). Só `tipo = lead`. |
| `POST /backend/v1/leads/{id}/stage` | aceita qualquer etapa ativa do funil do lead, valida a regra da categoria e os campos obrigatórios, e aceita `motivoCodigo`, `motivoDetalhe`, `nutricaoRetornoEm`, `pipelineId` (para mudar de funil). Em caso de recusa, devolve a lista do que falta. |
| `GET /backend/v1/leads/metrics` | passa a contar por categoria de etapa; ignora `tipo = prospect`. |
| `GET /backend/v1/leads/queue` | ignora `tipo = prospect`, `historico = true` e `suprimido = true`. |
| `GET /backend/v1/leads/export` | exporta os campos novos pela lista de campos permitidos. |

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Base com os leads das fases 1 a 3 | Após a migração, todos aparecem no Kanban com nome, telefone e origem; a contagem por estágio é igual à de antes. |
| Limite | Lead antigo com `origem` fora do catálogo | Vira `desconhecida`, com o valor original guardado em `origem_detalhe`. |
| Limite | Telefone antigo que não normaliza | Fica guardado como está, e o lead aparece na fila de exceção "telefone a corrigir". |
| Falha | Mover lead para etapa de categoria `agendado` sem agendamento confirmado | Negado, com a mensagem "Confirme o agendamento para mover o lead para Agendado." |
| Limite | Administrador renomeia "Triagem" para "Primeiro filtro" | A coluna muda de nome; nenhum lead muda; indicadores e fila continuam iguais, porque usam a categoria. |
| Limite | Etapa com `campos_obrigatorios = [segmento]` e lead sem segmento | Movimento negado, com "Preencha: Segmento". |
| Falha | Mover para `nutricao` sem data de retorno | Negado, com o campo que falta indicado. |

## Critérios de aceite

- [ ] **CA-4-01:** Todo lead que já existia continua visível depois da migração, com nome, telefone e e-mail nos campos novos; nenhuma tela nem resposta de rota mostra a palavra "fictício"; o telefone é guardado normalizado e exibido formatado.
- [ ] **CA-4-02:** Origem, produto, segmento e motivo vêm de catálogos: valor fora do catálogo não é aceito em formulário e, em importação ou integração, cai em "Desconhecida", "A definir" ou "A classificar", de forma visível.
- [ ] **CA-4-03:** As colunas do Kanban vêm dos dados do funil, não do código; mover um lead sem cumprir a regra da categoria ou um campo obrigatório da etapa é negado com a lista do que falta; o histórico anterior permanece intacto.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Contagem antes e depois | contar leads por estágio antes e depois da migração | tabela com os dois números lado a lado |
| Normalizadores | testes unitários com os exemplos da tabela e 10 casos de erro | saída dos testes |
| Movimentação | uma tentativa válida e uma inválida por categoria de etapa | respostas das rotas |
| Etapas como dados | busca no código por comparação com código ou rótulo de etapa fora dos dados iniciais | resultado vazio |
| Regressão | critérios de aceite das fases 1 a 3 que usam Kanban, fila, métricas e exportação | lista com passou ou falhou |

**Dados de teste:** `fixtures/leads-planilha-fase4.csv`.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.4 | Criar a migração aditiva da coleção leads com os campos reais, copiar os valores dos campos *_ficticio, normalizar telefones e criar os índices | Dev backend | SPEC-4-001 | CA-4-01 | RED/GREEN: contagem por etapa antes e depois | tabela antes × depois + lista de telefones a corrigir | Fase 3 aceita | ☐ | A |
| T4.5 | Implementar os normalizadores compartilhados (telefone, CNPJ, e-mail, data, moeda, texto seguro), usados por formulário, importação e integração | Dev full-stack | SPEC-4-001 | CA-4-01 | RED/GREEN: exemplos da SPEC e 10 casos de erro | saída dos testes unitários | Fase 3 aceita | ☐ | A |
| T4.6 | Criar a coleção de catálogos com os dados iniciais (origens, produtos, segmentos, motivos, etiquetas) e a rota de leitura; converter origens e produtos antigos para códigos do catálogo | Dev backend | SPEC-4-001 | CA-4-02 | Principal/limite: valor fora do catálogo | catálogos carregados + leads antigos convertidos | T4.4 aceita | ☐ | A |
| T4.7 | Criar as coleções de funis e etapas com as categorias do sistema e o funil padrão; trocar a lista fixa de estágios pela leitura dos dados; reescrever a rota de mudança de etapa para validar por categoria e por campos obrigatórios | Dev backend | SPEC-4-001 | CA-4-03 | RED/GREEN: uma movimentação válida e uma inválida por categoria | respostas das rotas + busca no código sem etapas fixas | T4.4 aceita | ☐ | A |
| T4.8 | Ajustar as rotas existentes de Kanban, fila, métricas e exportação aos campos novos e às categorias de etapa, mantendo os campos de resposta antigos | Dev backend | SPEC-4-001 | CA-4-03 | Regressão: critérios das fases 1 a 3 que usam essas rotas | lista de critérios com passou ou falhou | T4.7 aceita; T4.6 aceita | ☐ | A |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
