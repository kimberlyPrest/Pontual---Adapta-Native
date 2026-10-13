# SPEC-4-010 — Administração: usuários, catálogos, régua, mensagens, ICP e auditoria

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-12, RF-14 (administração), RF-15 (acesso e auditoria), RF-23, RF-24, RN-13, RNF-01, RNF-02, RNF-08  
**Degrau da solução:** construção de interface — as rotas de ICP, base aprovada, rascunho, publicação e restauração existem desde a Fase 3 e não têm tela; usuários hoje só são criados por migração.

## Resultado observável

O Marcelo cria o acesso de um vendedor novo, ajusta a lista de origens, muda a régua de follow-up e publica uma resposta aprovada, tudo pela tela, sem pedir a ninguém e sem código.

## Por que existe

- Não há tela de usuários. O único usuário real foi criado por uma migração, com a senha escrita no código.
- O escopo exige que o administrador configure etapas, campos, regras e mensagens pela interface.
- ICP por produto e base aprovada só podem ser alterados chamando rotas à mão.
- A auditoria é gravada desde a Fase 1 e ninguém consegue lê-la.

## Limites e dependências

- **Inclui:** usuários e papéis, papel `marketing`, catálogos, régua de cadência, modelos de mensagem, ICP por produto, base aprovada, preferências gerais, tela de auditoria.
- **Fora de escopo desta SPEC:** o editor de funis, etapas, campos, ficha e card (SPEC-4-012); os canais de WhatsApp (SPEC-4-013); o editor de agentes de IA (SPEC-4-008) — todos ficam dentro do mesmo menu Configurações, mas são especificados nas SPECs indicadas. Acesso de parceiro externo ao sistema continua fora.
- **Pré-condições:** SPEC-4-001 aceita. A régua depende da SPEC-4-005; os modelos de mensagem são usados pela SPEC-4-008.

## Telas e comportamento

Menu **Configurações**, com as seções abaixo. `admin` vê todas; `gestor` vê as marcadas.

| Seção | admin | gestor |
|---|---|---|
| Usuários | sim | não |
| Funis e etapas · Campos · Ficha e card (SPEC-4-012) | sim | sim |
| Canais de WhatsApp (SPEC-4-013) | sim | não |
| Agentes de IA (SPEC-4-008) | sim | ver e testar |
| Catálogos | sim | sim |
| Cadência | sim | sim |
| Modelos de mensagem | sim | sim |
| ICP por produto | sim | sim |
| Base aprovada | sim | sim |
| Metas (SPEC-4-009) | sim | sim |
| Integrações (SPEC-4-011) | sim | não |
| Geral | sim | não |
| Auditoria | sim | só leitura |

### Usuários

Tabela: nome, e-mail, papel, situação (ativo, inativo, convite pendente), último acesso.

- **Novo usuário:** nome, e-mail, papel. O sistema cria o usuário com senha temporária gerada, mostrada **uma única vez** na tela para o administrador repassar, e marca "trocar a senha no primeiro acesso". Se o envio de e-mail do ambiente estiver configurado, envia também o convite por e-mail.
- **Editar:** nome e papel.
- **Desativar / reativar:** usuário inativo não entra e some das listas de responsável e apresentador; o que ele fez continua no histórico com o nome dele. Ao desativar um usuário com leads ativos, a tela pede para quem transferir.
- **Redefinir senha:** gera nova senha temporária, mostrada uma vez.
- Não é possível desativar a si mesmo, nem desativar ou rebaixar o último `admin` ativo.
- Não há exclusão de usuário.

**Papéis:**

| Papel | Pode |
|---|---|
| `admin` | tudo |
| `gestor` | tudo da operação e dos indicadores; configurações marcadas acima; vê todos os leads |
| `sdr` | operar leads, fila, agenda, prospecção, importação; ver indicadores próprios |
| `marketing` (novo) | ver Anúncios e Feedback, lançar e importar investimento; nenhum dado pessoal de lead |

O papel `marketing` entra na coleção `users`, na `authorization_matrix` e nas verificações das rotas.

**Primeiro acesso:** usuário marcado para trocar a senha é levado à tela de troca antes de qualquer outra. Senha nova: mínimo de 10 caracteres, diferente da temporária.

### Catálogos

Quatro abas: Origens, Produtos, Segmentos, Motivos (e Etiquetas, só para renomear e desativar).

- Lista ordenável por arrastar. Colunas: rótulo, código (somente leitura), grupo, ativo, quantos leads usam.
- **Novo item:** rótulo e grupo; o código é gerado do rótulo e não muda mais.
- **Editar:** rótulo, grupo, ordem; em Motivos, também "gera supressão" e "dias de nutrição sugeridos".
- **Desativar:** o item some dos formulários e continua valendo para os leads que já o usam e para os indicadores.
- Itens protegidos (`desconhecida`, `a_definir`, `a_classificar`) não podem ser desativados.
- Em Produtos: apresentador padrão (usado pela SPEC-4-007).

### Cadência

Uma régua por linha: a padrão e as por origem.

Editor da régua: número de tentativas; para cada tentativa, o intervalo em dias úteis e o canal; janela de horário; dias da semana; prazo do primeiro contato; o que fazer ao esgotar; dias padrão de nutrição.

Ao lado do editor, a **simulação**: "Lead que entra na segunda, 9h: 1ª tentativa segunda 9h (WhatsApp) · 2ª quarta 9h (ligação) · 3ª terça seguinte 9h (WhatsApp)". Atualiza enquanto o usuário edita.

Salvar cria uma nova versão; a tela mostra as versões anteriores e permite **restaurar**. Vale para as próximas propostas de ação; ações já marcadas não mudam.

### Modelos de mensagem

Lista de modelos: nome, objetivo, produto, canal, situação.

- Campos: nome, objetivo (os mesmos da SPEC-4-008: primeiro contato, follow-up sem resposta, retomar depois de nutrição, confirmar apresentação, reagendar), produto (ou "todos"), canal, texto.
- Variáveis disponíveis, inseridas por botão: `{primeiro_nome}`, `{empresa}`, `{produto}`, `{cidade}`, `{meu_nome}`, `{data_apresentacao}`, `{hora_apresentacao}`, `{link_apresentacao}` e qualquer campo personalizado, como `{custom.<codigo>}` (SPEC-4-012).
- Pré-visualização com um lead de exemplo.
- Variável sem valor no lead: o trecho é marcado em amarelo na hora de usar, para o usuário completar.
- Na ficha, na fila e no campo de escrever da conversa (digitando `/`), o botão **Modelos** lista os modelos do objetivo, com as variáveis preenchidas. Com canal conectado, **Usar na conversa**; sem canal, **Copiar**. Funciona sem IA. Os mesmos modelos são usados pela régua automática, pelo envio de lista e pelas automações de etapa.
- Dados iniciais: um modelo por objetivo, com texto neutro, para a Pontual reescrever na T4.2.

### ICP por produto

Uma aba por produto do catálogo.

- Mostra a versão publicada: critérios (pergunta, tipo de resposta, obrigatório ou não, campo do lead que responde — de sistema ou personalizado —, se houver) e o aviso. Os critérios do ICP são o que o botão "Carregar do ICP" leva para o roteiro de um agente de qualificação (SPEC-4-008).
- **Editar** abre um rascunho. Tipos de resposta: sim/não, escolha em lista, número, texto.
- **Validar** confere: pelo menos um critério; sem pergunta repetida; campo do lead existente.
- **Publicar** pede confirmação: "Esta versão passa a valer para as próximas qualificações. Qualificações já feitas não mudam."
- **Versões anteriores**, com data e autor, e **Restaurar** (pede motivo).
- Usa as rotas existentes `GET/POST /backend/v1/icp`, `GET /backend/v1/icp/versions`, `GET/POST /backend/v1/config/drafts`, `.../validate`, `.../publish`, `POST /backend/v1/config/rollback`.
- Dados iniciais: um ICP rascunho por produto, com os critérios citados no Playbook comercial (segmento, regime tributário, situação da loja, número de usuários, sistema atual), para a Pontual revisar e publicar na T4.2 (gate G2).

### Base aprovada

Perguntas e respostas que a IA pode usar (SPEC-4-008).

- Lista da versão publicada, com busca.
- **Editar** abre rascunho: adicionar, alterar, remover e reordenar entradas. Cada entrada: pergunta, resposta, e a marca "contém preço, prazo ou condição comercial".
- **Importar entradas** por colar (duas colunas: pergunta, resposta).
- **Publicar** e **Restaurar**, como no ICP.
- Usa as rotas existentes `GET /backend/v1/approved-base`, `POST /backend/v1/approved-base/draft`, `POST /backend/v1/approved-base/{id}/publish` e as de rascunho e restauração.

### Geral

Nome da empresa e nome do sistema exibidos no cabeçalho e no login; fuso horário (fixo em `America/Sao_Paulo`, exibido); dias de retenção das linhas de importação (padrão 90); formato de exportação para o RespondeChat (nomes e ordem das colunas, número com ou sem `55`); limite padrão de contatos por lista; intervalo mínimo entre disparos para o mesmo contato.

### Auditoria

Tabela de eventos, do mais novo para o mais antigo, reunindo `security_events`, `lead_critical_timeline`, `conversation_timeline` e os registros de configuração, importação, exportação e integração.

- Colunas: data e hora, usuário, papel, ação, recurso, resultado (permitido ou negado), detalhe.
- Filtros: período, usuário, tipo de ação, resultado, lead.
- Telefone, e-mail e identificadores aparecem mascarados no detalhe.
- Exportar o período filtrado em `.csv`.
- Somente leitura; nenhum evento pode ser alterado ou apagado, nem por `admin`.

## Dados

- `users`: campos novos `ativo` (bool, padrão verdadeiro), `trocar_senha` (bool), `ultimo_acesso` (data); `role` aceita `marketing`.
- `message_templates`: `nome`, `objetivo`, `produto`, `canal`, `texto`, `ativo`, `versao`, `actor_id`.
- `app_settings`: chave e valor (json), uma linha por preferência, com `actor_id` e `updated`.
- `cadence_rules` e `catalog_items`: definidos nas SPEC-4-005 e SPEC-4-001.

## API

| Rota | Papéis | Função |
|---|---|---|
| `GET/POST /backend/v1/users` · `PATCH /backend/v1/users/{id}` | admin | lista, cria, edita, ativa e desativa. |
| `POST /backend/v1/users/{id}/reset-password` | admin | devolve a senha temporária uma vez. |
| `POST /backend/v1/users/me/password` | qualquer usuário | troca a própria senha. |
| `GET/POST/PATCH /backend/v1/catalogs/{tipo}` | ler: todos os papéis; gravar: admin, gestor | catálogos. |
| `GET/POST/PATCH /backend/v1/message-templates` | ler: admin, gestor, sdr; gravar: admin, gestor | modelos. |
| `POST /backend/v1/message-templates/{id}/render?lead=` | admin, gestor, sdr | devolve o texto com as variáveis do lead. |
| `GET/PUT /backend/v1/settings` | ler: admin, gestor; gravar: admin | preferências. |
| `GET /backend/v1/audit?from=&to=&user=&action=&result=&lead=` | admin, gestor | auditoria. |

## Regras

1. Toda alteração de configuração grava auditoria com quem, quando, o valor de antes e o de depois.
2. Configuração nova vale para ações novas; nada do histórico é recalculado (RN-13).
3. Usuário inativo tem a sessão encerrada na próxima chamada.
4. A senha temporária nunca é gravada em log nem devolvida por rota de leitura.
5. O bloqueio por tentativas de login da Fase 1 continua valendo.
6. Código de catálogo é imutável, para que importações, integrações e histórico não quebrem quando o rótulo muda.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Admin cria usuário `sdr` | Senha temporária mostrada uma vez; no primeiro acesso, o usuário é obrigado a trocar. |
| Principal | Admin acrescenta a origem "Feira do setor" | Aparece no formulário de novo lead e no mapeamento da importação na hora. |
| Principal | Gestor muda a régua padrão para 5 tentativas | A simulação muda; a próxima proposta de ação usa a régua nova; ações já marcadas não mudam. |
| Principal | Gestor publica nova versão do ICP do Automec | Próximas qualificações usam a nova; as antigas mostram a versão em que foram feitas. |
| Limite | Desativar usuário com 14 leads ativos | Pede o novo responsável antes de concluir. |
| Limite | Desativar a origem "YouTube" com leads | Some do formulário; os leads continuam com a origem e aparecem nos indicadores. |
| Falha | Rebaixar o último admin | Negado. |
| Falha | `gestor` abre Usuários pelo endereço | Negado e auditado. |
| Falha | `marketing` abre `/leads` | Negado e auditado. |
| Recuperação | Régua publicada com erro | Restaurar a versão anterior. |

## Estados vazios, erro e permissão

- Produto sem ICP publicado: "Nenhum ICP publicado para este produto. A qualificação usa critério e evidência livres."
- Base aprovada vazia: aviso de que a sugestão de mensagem não tratará de assuntos de produto.

## Critérios de aceite

- [ ] **CA-4-38:** O administrador cria, desativa, redefine a senha e muda o papel de um usuário pela tela; a senha temporária aparece uma única vez e é trocada no primeiro acesso; usuário desativado não entra; cada papel só vê e faz o que lhe cabe, em teste positivo e negativo.
- [ ] **CA-4-39:** O administrador altera catálogos, régua, modelos de mensagem, ICP por produto e base aprovada pela interface; a alteração vale para ações novas, sem reclassificar o histórico; a versão anterior de régua, ICP e base aprovada pode ser restaurada.
- [ ] **CA-4-40:** A tela de auditoria lista os eventos críticos (acessos negados, alterações de configuração, importações, exportações, envios, decisões) com filtro por usuário, período e tipo; nenhum evento pode ser alterado ou apagado.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Ciclo do usuário | criar, entrar, trocar senha, desativar, tentar entrar | gravação |
| Papéis | para cada papel, percorrer o menu e tentar 3 endereços proibidos | tabela papel × rota × resultado |
| Configuração sem código | alterar origem, régua, modelo, ICP e base aprovada pela tela e usar na operação | gravação |
| Auditoria | localizar na tela cada alteração feita acima | capturas |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.40 | Criar a tela e as rotas de usuários: criar com senha temporária mostrada uma vez, troca obrigatória no primeiro acesso, editar, desativar com transferência de leads, redefinir senha; incluir o papel marketing na matriz de autorização e nas rotas | Dev full-stack | SPEC-4-010 | CA-4-38 | RED/GREEN: desativar o último administrador; prova por papel: menu e três endereços proibidos | tabela papel × rota × resultado + gravação do ciclo do usuário | T4.9 aceita | ☐ | H |
| T4.41 | Criar as telas de Catálogos, Cadência (com simulação e versões) e Modelos de mensagem (com variáveis, inclusive de campos personalizados, e pré-visualização), e as preferências gerais | Dev full-stack | SPEC-4-010 | CA-4-39 | Principal: alterar origem, régua e modelo pela tela e usar na operação; recuperação: restaurar régua | gravação + registros na auditoria | T4.33 aceita; T4.18 aceita | ☐ | H |
| T4.42 | Criar as telas de ICP por produto e de Base aprovada (rascunho, validar, publicar, versões, restaurar) sobre as rotas existentes, com os rascunhos iniciais de ICP | Dev frontend | SPEC-4-010 | CA-4-39 | Principal: publicar nova versão do ICP; limite: histórico não reclassificado | capturas + ficha antiga com a versão em que foi feita | T4.13 aceita | ☐ | H |
| T4.43 | Criar a rota e a tela de auditoria, reunindo os registros de segurança, de lead, de conversa, de configuração, de importação, de exportação e de integração, com filtros, mascaramento e exportação | Dev full-stack | SPEC-4-010 | CA-4-40 | Falha: tentativa de alterar ou apagar evento; principal: localizar alterações feitas nas tasks anteriores | capturas com os filtros + resposta negada | T4.40 aceita | ☐ | H |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
