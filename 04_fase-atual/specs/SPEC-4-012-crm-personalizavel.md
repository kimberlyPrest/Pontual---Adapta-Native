# SPEC-4-012 — CRM personalizável: funis, etapas, campos, card e automações pela interface

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-12, RF-14 (administração: etapas, campos, regras), RF-03, RF-06, RN-13; reunião de alinhamento ("configurar etapas e automação pela interface", "Kanban muda conforme a conversa")  
**Degrau da solução:** construção — o que hoje está escrito no código (etapas, campos do lead, conteúdo do card) vira configuração que o administrador muda sozinho.

## Resultado observável

O Marcelo abre Configurações e, sem pedir nada a ninguém: cria um segundo funil para o Hiper, acrescenta a etapa "Demonstração", renomeia "Triagem", cria o campo "Quantidade de caixas" na ficha do lead, escolhe o que aparece no card do Kanban e define que todo lead que entra em "Qualificado" ganha a tarefa "Propor horário de apresentação". Tudo passa a valer na hora, para todos os usuários.

## Por que existe

- O Playbook comercial da Pontual descreve um funil diferente do que está no sistema (Lead → Primeiro contato → Qualificação → Demonstração → Proposta → Negociação → Fechamento) e prevê funis distintos por produto (certificado, Automec, Hiper).
- O RespondeChat, que a equipe usa hoje, tem mais de um quadro Kanban ("Atendimento comercial/SDR" e "Hiper").
- A planilha tem colunas que o sistema não tem e que mudam com o tempo; cada coluna nova não pode depender de programação.
- O escopo definitivo exige que o administrador configure etapas, campos e regras pela interface.

## Limites e dependências

- **Inclui:** editor de funis e etapas; campos personalizados do lead; organização da ficha; conteúdo do card do Kanban; visões salvas da lista de leads; ações automáticas por etapa; registro de versões e auditoria.
- **Fora de escopo:** fórmulas e campos calculados; campos personalizados em outras entidades que não o lead; construtor visual de fluxos com condições encadeadas (as automações desta fase são "ao entrar na etapa, faça isto"); permissões por campo.
- **Pré-condições:** SPEC-4-001 (etapas e campos como dados) e SPEC-4-004 (ficha e Kanban) aceitas.
- **Risco e plano B:** se o arrastar-e-soltar do editor der problema, a ordenação por setas (subir, descer) é suficiente para o aceite.

## Telas e comportamento

Configurações → **Funis e etapas**, **Campos**, **Ficha e card**. Papéis: `admin` e `gestor`.

### Funis e etapas

**Lista de funis** — nome, quantas etapas, quantos leads, padrão (sim ou não), ativo. Botões: **Novo funil**, **Duplicar** (copia etapas e automações, sem leads), **Tornar padrão**, **Desativar**.

Funil só pode ser desativado quando não tiver lead ativo; a tela oferece mover os leads para outro funil, escolhendo a etapa de destino para cada etapa de origem.

**Editor do funil** — as etapas aparecem como colunas, na ordem do Kanban, arrastáveis.

Cada etapa abre um painel com:

| Campo | Regra |
|---|---|
| Nome | o que aparece na coluna; pode mudar a qualquer momento |
| Categoria | uma das categorias do sistema (SPEC-4-001). A tela explica cada uma em uma frase. **Não pode ser alterada depois que a etapa tiver leads** — para mudar, cria-se outra etapa e movem-se os leads. |
| Cor | paleta de 10 cores |
| Alerta de lead parado | número de dias; vazio = sem alerta |
| Campos obrigatórios para entrar | escolhidos na lista de campos (de sistema e personalizados) |
| Começa recolhida | sim ou não |
| Ações automáticas ao entrar | lista (abaixo) |
| Ativa | etapa inativa não recebe leads |

**Nova etapa:** nome e categoria; entra à direita da última etapa da mesma categoria.

**Excluir etapa:** só etapa sem leads e sem histórico. Com leads, a tela pede a etapa de destino e move todos, gravando a movimentação na timeline de cada um com o motivo "etapa removida pelo administrador". Etapa com histórico é **desativada**, não apagada, para o histórico continuar legível.

**Proteções** (o editor impede e explica):

- cada funil precisa de exatamente uma etapa ativa de categoria `entrada`, uma de `agendamento` e uma de `agendado`, e de pelo menos uma de `perdido` e uma de `nutricao`;
- a ordem das categorias no funil é livre, mas a tela avisa (sem impedir) se uma etapa `qualificado` vier antes de uma `entrada`;
- máximo de 20 etapas ativas por funil e 10 funis ativos.

**Publicação:** as alterações ficam em rascunho, visíveis só no editor, até o clique em **Publicar**. Antes de publicar, a tela mostra o resumo: "2 etapas novas · 1 renomeada · 14 leads serão movidos de 'Triagem' para 'Primeiro contato'". Publicar grava uma versão; **Restaurar versão anterior** está disponível enquanto as etapas daquela versão ainda existirem.

### Ações automáticas ao entrar na etapa

Cada etapa pode ter até 5 ações, executadas na ordem, quando um lead entra nela — por uma pessoa, por importação, por integração ou por agente de IA.

| Ação | Parâmetros |
|---|---|
| Definir próxima ação | texto, tipo, prazo ("em N horas úteis" ou "em N dias úteis") |
| Atribuir responsável | um usuário fixo · rodízio entre usuários escolhidos · manter o atual |
| Adicionar etiqueta | etiqueta |
| Remover etiqueta | etiqueta |
| Preencher campo | campo e valor (só campos de lista, sim/não e texto curto) |
| Enviar modelo de mensagem pelo WhatsApp | modelo e canal; **só se** houver canal conectado e o envio automático estiver liberado (SPEC-4-013); respeita supressão e horário |
| Acionar agente de IA | agente (SPEC-4-008); o agente passa a atender a conversa do lead |
| Pausar agente de IA | — |
| Abrir transbordo | motivo e contexto |
| Notificar | usuário ou papel; aparece no sino |

Regras:

1. Ação automática nunca muda o lead de etapa (para não criar laço).
2. Ação que falha não impede a movimentação: fica registrada na timeline como "ação automática não executada: <motivo>" e aparece para o administrador em Configurações → Funis → Falhas de automação.
3. Toda ação executada vai para a timeline com a marca "automático", o nome da etapa e a versão do funil.
4. Em importação em massa, as ações "Enviar modelo" e "Acionar agente" **não** disparam; as demais, sim. Isso evita mandar mensagem para uma base inteira ao importar.

### Campos

Configurações → **Campos**. Duas abas: **Campos do sistema** e **Campos personalizados**.

**Campos do sistema** (os da SPEC-4-001): o administrador pode mudar o **rótulo**, a **seção da ficha**, a **ordem**, se aparece ou fica **oculto** e o **texto de ajuda**. Não pode excluir nem mudar o tipo. Telefone, origem e etapa não podem ser ocultados.

**Campos personalizados:**

| Propriedade | Regra |
|---|---|
| Rótulo | o que aparece na tela |
| Código | gerado do rótulo; imutável; é a chave em `leads.custom`, na importação, na captura e nas variáveis de mensagem |
| Tipo | Texto curto · Texto longo · Número · Moeda (R$) · Data · Sim/Não · Lista (uma opção) · Lista (várias opções) · Telefone · E-mail · Link |
| Opções | para os tipos de lista; cada opção tem rótulo e código; opção pode ser desativada, não excluída |
| Seção da ficha | uma das seções (abaixo) |
| Obrigatório no cadastro | sim ou não |
| Funis em que aparece | todos ou alguns |
| Texto de ajuda | |
| Usar como filtro | aparece nos filtros da lista de leads e do Kanban |
| Disponível para agentes de IA | o agente pode ler e, se permitido, preencher (SPEC-4-008) |

- **Mudar o tipo** de um campo que já tem valores só é permitido de Texto curto para Texto longo. Qualquer outra mudança pede criar um campo novo.
- **Excluir campo:** some das telas; os valores ficam guardados por 30 dias, com **Restaurar**; depois são apagados. Campo usado em etapa (obrigatório), em automação, em ICP ou em agente não pode ser excluído enquanto estiver em uso — a tela lista onde.
- Limite de 60 campos personalizados ativos.
- Validação por tipo no servidor, com os mesmos normalizadores da SPEC-4-001 (telefone, e-mail, data, moeda).

Onde um campo personalizado passa a valer, automaticamente, depois de criado:

| Lugar | Como |
|---|---|
| Ficha do lead | na seção escolhida, editável no lugar, com auditoria |
| Cadastro de lead | em "Mais detalhes"; no cadastro rápido se for obrigatório |
| Importação | como destino no mapeamento de colunas (SPEC-4-003) |
| Captura automática | dentro do objeto `custom` do corpo (SPEC-4-002) |
| Lista de leads | coluna opcional e filtro |
| Kanban | filtro; no card, se escolhido |
| Critério de ICP | pode ser o campo que responde um critério (SPEC-4-010) |
| Modelos de mensagem | variável `{custom.<codigo>}` |
| Agentes de IA | leitura e preenchimento, se marcado |
| Exportação | coluna, se incluída na lista de campos permitidos pelo administrador |
| Briefing da apresentação | se a seção do campo estiver marcada para aparecer no briefing |

### Ficha e card

**Seções da ficha:** lista ordenável das seções da aba Resumo (Empresa, Contato, Aquisição, Qualificação rápida, Resultado — e as que o administrador criar). Para cada seção: nome, ordem, aberta ou recolhida por padrão, e "aparece no briefing da apresentação".

**Card do Kanban:** pré-visualização do card à direita; à esquerda, a escolha do que mostrar:

- linha de título: empresa ou contato;
- até 4 campos adicionais, escolhidos entre campos de sistema e personalizados;
- o que é fixo e não sai: próxima ação com atraso, responsável, selos (suprimido, transbordo, parado, agente de IA ativo);
- mostrar ou não a última nota (padrão: sim).

A configuração do card é por funil.

### Visões salvas da lista de leads

Na tela Leads, o usuário salva a combinação de filtros, colunas e ordenação como uma **visão**, com nome. Visão pessoal por padrão; `admin` e `gestor` podem marcar como **compartilhada com todos**. As visões aparecem como abas no topo da lista. Dados iniciais: "Meus leads ativos", "Sem próxima ação", "Parados", "Entraram esta semana".

## Dados

- `pipelines`, `pipeline_stages`: definidos na SPEC-4-001. Acrescenta-se `pipeline_versions` (`pipeline_id`, `versao`, `conteudo` json com as etapas e automações, `resumo`, `actor_id`, `created`).
- `custom_fields`: `codigo`, `rotulo`, `tipo`, `opcoes` (json), `secao`, `obrigatorio`, `pipelines` (json), `ajuda`, `filtravel`, `ia_ler` (bool), `ia_preencher` (bool), `ordem`, `ativo`, `excluido_em`.
- `field_settings`: uma linha por campo de sistema: `campo`, `rotulo`, `secao`, `ordem`, `oculto`, `ajuda`.
- `form_sections`: `codigo`, `nome`, `ordem`, `recolhida`, `no_briefing`.
- `card_layouts`: `pipeline_id`, `titulo`, `campos` (json), `mostrar_nota`.
- `lead_views`: `nome`, `dono_id`, `compartilhada`, `filtros` (json), `colunas` (json), `ordenacao` (json).
- `automation_runs`: `lead_id`, `pipeline_stage`, `acao`, `resultado` (`ok`, `falhou`, `ignorada`), `detalhe`, `created` — sem atualização nem exclusão.

## API

| Rota | Papéis | Função |
|---|---|---|
| `GET /backend/v1/pipelines` | todos | funis e etapas publicados (SPEC-4-001). |
| `GET/POST/PATCH /backend/v1/admin/pipelines` e `.../{id}/stages` | admin, gestor | edição em rascunho. |
| `POST /backend/v1/admin/pipelines/{id}/preview` | admin, gestor | resumo do que a publicação muda, com contagem de leads afetados. |
| `POST /backend/v1/admin/pipelines/{id}/publish` | admin, gestor | publica; move leads conforme o mapeamento informado; grava versão. |
| `POST /backend/v1/admin/pipelines/{id}/restore` | admin, gestor | restaura versão. |
| `GET /backend/v1/fields` | todos | campos de sistema (com as configurações) e personalizados ativos — é o que as telas usam para se desenhar. |
| `POST/PATCH/DELETE /backend/v1/admin/fields` | admin, gestor | campos personalizados e configurações dos campos de sistema. |
| `GET/PUT /backend/v1/admin/layout` | admin, gestor | seções e card. |
| `GET/POST/PATCH/DELETE /backend/v1/lead-views` | admin, gestor, sdr | visões. |
| `GET /backend/v1/admin/automation-runs?result=falhou` | admin, gestor | falhas de automação. |

`PATCH /backend/v1/leads/{id}` (SPEC-4-004) passa a aceitar `custom: { <codigo>: <valor> }`, validando cada valor pelo tipo do campo.

## Regras

1. **Nenhuma tela tem etapa ou campo escrito no código.** Kanban, ficha, cadastro, filtros, importação e indicadores desenham a partir de `GET /pipelines` e `GET /fields`.
2. **Comportamento depende da categoria, nunca do nome** (SPEC-4-001).
3. **Configuração nova vale para ações novas.** Renomear etapa ou campo muda o rótulo em todo lugar, inclusive no histórico exibido; o histórico guarda o código, que não muda (RN-13).
4. **Publicar é atômico:** ou todas as mudanças do rascunho entram, ou nenhuma.
5. **Edição concorrente do funil:** dois administradores no mesmo rascunho — o segundo a publicar é avisado e precisa recarregar.
6. Toda mudança de configuração grava auditoria com quem, quando, antes e depois.
7. Os indicadores (SPEC-4-009) agrupam por categoria e mostram o detalhe por etapa do funil escolhido.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Criar a etapa "Demonstração" (categoria `pos_apresentacao`) e publicar | Coluna nova no Kanban de todos os usuários, sem recarregar o sistema. |
| Principal | Criar o campo "Quantidade de caixas" (Número, seção Qualificação rápida) | Aparece na ficha, no cadastro, no mapeamento da importação e nos filtros. |
| Principal | Etapa "Qualificado" com a ação "Definir próxima ação: propor horário, em 4 horas úteis" | Lead movido para lá ganha a próxima ação; timeline mostra "automático". |
| Principal | Criar o funil "Hiper" duplicando o "Comercial" | Seletor de funil no Kanban mostra os dois; cada um com suas etapas. |
| Limite | Renomear "Triagem" para "Primeiro filtro" | Só o nome muda; leads, fila e indicadores iguais. |
| Limite | Excluir etapa com 14 leads | Pede a etapa de destino; move os 14 com registro na timeline. |
| Limite | Importar 300 leads direto para uma etapa com "Enviar modelo" | Nenhuma mensagem enviada; as outras ações executadas. |
| Falha | Desativar a única etapa de categoria `entrada` | Impedido, com a explicação. |
| Falha | Mudar o tipo de um campo de Texto para Número com valores gravados | Impedido; orienta criar outro campo. |
| Falha | Excluir campo usado como obrigatório em uma etapa | Impedido; lista onde é usado. |
| Recuperação | Funil publicado com erro | Restaurar a versão anterior. |
| Recuperação | Campo excluído por engano | Restaurar em até 30 dias, com os valores. |

## Estados vazios, erro e permissão

- Sem campos personalizados: "Nenhum campo personalizado. Crie o primeiro para registrar o que a sua operação precisa."
- `sdr` e `marketing` não acessam os editores; veem o resultado.

## Critérios de aceite

- [ ] **CA-4-43:** O administrador cria um funil, acrescenta, renomeia, reordena e desativa etapas e publica, tudo pela tela; o Kanban de todos os usuários reflete a mudança; as proteções impedem deixar o funil sem etapa de entrada, de agendamento ou de encerramento; a versão anterior pode ser restaurada.
- [ ] **CA-4-44:** Um campo personalizado, de qualquer um dos tipos, criado pela tela, passa a valer sem programação na ficha, no cadastro, no mapeamento da importação, na captura, nos filtros, no card, nos modelos de mensagem e nos agentes; excluído por engano, pode ser restaurado com os valores em até 30 dias.
- [ ] **CA-4-45:** As ações automáticas configuradas em uma etapa são executadas quando um lead entra nela e ficam na timeline marcadas como automáticas; uma ação que falha não impede a movimentação e aparece para o administrador; importação em massa não dispara envio de mensagem nem agente.
- [ ] **CA-4-46:** O administrador define as seções da ficha e o conteúdo do card do Kanban por funil; os usuários salvam visões da lista de leads com filtros, colunas e ordenação.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Funil sem código | criar funil, etapa, renomear, reordenar, publicar, restaurar | gravação |
| Campo ponta a ponta | criar um campo de cada tipo e conferir os 10 lugares da tabela "onde passa a valer" | tabela lugar × funcionou |
| Automação | uma etapa com 3 ações; mover lead à mão, por importação e por integração | timeline dos três leads |
| Proteções | as 4 tentativas inválidas dos cenários de falha | capturas das mensagens |
| Sem etapa no código | busca no código-fonte por códigos e rótulos de etapa fora dos dados iniciais | resultado vazio |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.17 | Criar os campos personalizados (definições, valores no lead, validação por tipo), as configurações dos campos de sistema e a rota que descreve os campos para as telas | Dev backend | SPEC-4-012 | CA-4-44 | RED/GREEN: valor inválido por tipo | testes por tipo + resposta da rota de campos | T4.10 aceita | ☐ | C |
| T4.18 | Criar a tela Campos: criar, editar, ordenar, ocultar, excluir e restaurar; e fazer a ficha, o cadastro, os filtros e a exportação se desenharem a partir da rota de campos | Dev full-stack | SPEC-4-012 | CA-4-44 | Principal: um campo de cada tipo nos lugares previstos; falha: mudar tipo com valores, excluir campo em uso | tabela lugar × funcionou + capturas das recusas | T4.17 aceita | ☐ | C |
| T4.19 | Criar o editor de funis e etapas: rascunho, criação, renomeação, reordenação, cor, alerta de lead parado, campos obrigatórios, duplicar funil, prévia do impacto, publicação atômica com movimentação de leads, versões e restauração, com as proteções | Dev full-stack | SPEC-4-012 | CA-4-43 | Principal: criar etapa e funil; falha: as proteções; recuperação: restaurar versão | gravação + capturas das mensagens de proteção | T4.7 aceita; T4.17 aceita; T4.15 aceita | ☐ | C |
| T4.20 | Implementar as ações automáticas ao entrar na etapa (próxima ação, responsável fixo ou em rodízio, etiqueta, preencher campo, transbordo, notificar), o registro de execução e a tela de falhas; deixar previstas as ações de enviar modelo e acionar agente, que passam a funcionar nas levas I e K | Dev backend | SPEC-4-012 | CA-4-45 | Principal: três ações em uma etapa; falha: ação que falha não impede a movimentação; limite: importação em massa | timeline de três leads (movido à mão, importado, por integração) | T4.19 aceita | ☐ | C |
| T4.21 | Criar a configuração das seções da ficha e do card do Kanban por funil, com pré-visualização, e as visões salvas da lista de leads (pessoais e compartilhadas) | Dev full-stack | SPEC-4-012 | CA-4-46 | Principal: mudar o card e salvar uma visão | capturas do card antes e depois + visão compartilhada vista por outro usuário | T4.18 aceita; T4.15 aceita; T4.16 aceita | ☐ | C |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
