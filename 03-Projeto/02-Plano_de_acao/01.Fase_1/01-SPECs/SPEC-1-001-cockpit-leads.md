# SPEC-1-001 — Cockpit inicial de leads e Kanban

**Fase:** 1  
**Status:** planejada  
**Dono:** Produto/Desenvolvimento, com validação do champion  
**Origem no escopo:** RF-001, RF-002, RF-003, RF-006; Fase 1  
**Degrau da solução:** construção mínima — entregar o fluxo visual demonstrável sem substituir o CRM atual.

## Contexto e decisões fechadas

- **Estado atual:** informações dispersas entre CRM Pontual/MoveDesk, RespondeChat e planilhas; o estágio é controlado parcialmente por cores e observações livres.
- **Estado desejado:** usuário autorizado cria/consulta um lead, identifica origem, movimenta o card no Kanban e visualiza responsável, etapa e próxima ação.
- **Decisões já fechadas:** o limite do fluxo é aquisição até agendamento; a qualificação final permanece humana; o sistema deve preservar histórico.
- **Bloqueios:** nenhum para o recorte de demonstração; fonte oficial, stack e identificadores de produção ficam fora desta fase.

## Resultado observável

Uma tela utilizável de login + cockpit em que o SDR consegue cadastrar um lead de teste, visualizá-lo em um Kanban e consultar origem, etapa, responsável, próxima ação e histórico de alterações.

## Limites e dependências

- **Inclui:** autenticação básica de teste; papéis SDR, gestão e administrador; cadastro manual; Kanban; origem; responsável; próxima ação; histórico.
- **Fora de escopo:** integração definitiva, migração integral, envio de mensagens, IA autônoma, dashboard avançado e substituição do CRM.
- **Entradas e pré-condições:** ambiente de desenvolvimento; contas de teste; catálogo provisório de origens aprovado para a demonstração.
- **Saídas/artefatos:** tela demonstrável, registro de teste, log de mudanças e roteiro de validação.
- **Dependências e responsáveis:** cliente fornece usuários e origens; desenvolvimento implementa; champion valida fluxo.
- **Atores e permissões mínimas:** SDR cria/edita seus leads; gestão consulta e edita; administrador configura; visitante sem autenticação não acessa dados.
- **Superfícies afetadas:** aplicação do sistema e armazenamento de leads; os caminhos exatos serão definidos no repositório do produto, sem alterar sistemas legados.
- **Risco e plano B:** usar armazenamento isolado de demonstração, sem conexão de produção.
- **Rollback ou reversão:** apagar apenas dados de teste ou restaurar snapshot do ambiente de desenvolvimento; não alterar dados legados.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Cadastro manual → cockpit | ambiente de teste | nome, empresa, telefone/e-mail, produto, origem, etapa, responsável, próxima ação | usuário autenticado por papel | criação idempotente por identificador de teste | validar campos e mostrar erro sem salvar parcialmente |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1 | lead novo cadastrado | etapa inicial `Novo` e histórico de criação | origem desconhecida deve ser explícita | escopo-base/RF-006 |
| RN-2 | card movimentado | registrar etapa anterior, nova etapa, autor e data | edição sem permissão é recusada | escopo-base/RF-015 |
| RN-3 | contato recusa | marcar supressão, sem disparo | política completa fica pendente | escopo-base/RF-008 |

## Fluxo e regras

1. Usuário acessa login e informa conta de teste.
2. Sistema valida papel e apresenta o cockpit.
3. Usuário cria lead com campos obrigatórios.
4. Sistema cria card em `Novo`, registra origem e cria evento de histórico.
5. Usuário movimenta o card; sistema registra autor, data e transição.
6. Usuário consulta o histórico e a próxima ação.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | cadastro válido | card aparece em `Novo` | — |
| Limite | origem desconhecida | registro é aceito com origem `Desconhecida` | não inventar canal |
| Falha | campo obrigatório ausente ou papel insuficiente | não grava alteração e informa erro | corrigir entrada/reautenticar |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC e o escopo definitivo, seções 3, 4, 5 e Fase 1.
2. **Alterar somente:** cockpit, modelo de lead, Kanban, autenticação de teste e histórico no ambiente do produto.
3. **Não alterar:** CRM legado, contas de anúncios, políticas de WhatsApp ou dados de produção.
4. **Executar nesta ordem:** RED dos cenários; implementação mínima; GREEN; regressão; demonstração.
5. **Parar e pedir validação quando:** a execução tentar sair do ambiente de demonstração e tocar fonte, integração ou dados de produção.
6. **Estado válido ao parar:** ambiente de teste acessível e sem mutação de produção.

## Checklist de execução

- [ ] login e autorização de teste conferidos;
- [ ] cadastro, edição e visualização do lead concluídos;
- [ ] Kanban e transição de etapa concluídos;
- [ ] histórico e origem desconhecida exercitados;
- [ ] erro de validação e permissão exercitados;
- [ ] evidência anexada e demonstração do champion realizada.

## Critérios de aceite

- [ ] **CA-1-001:** usuário autenticado cria um lead de teste e o visualiza na etapa `Novo`.
- [ ] **CA-1-002:** movimentação de card registra etapa anterior, nova etapa, autor e data.
- [ ] **CA-1-003:** usuário sem permissão não cria nem altera configuração protegida.
- [ ] **CA-1-004:** entrada inválida não cria registro parcial e apresenta erro compreensível.
- [ ] **CA-1-005:** origem desconhecida permanece explícita e não é substituída por suposição.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | cadastro e transição ainda inexistentes | executar teste/roteiro `F1-COCKPIT-01` | falha nos critérios CA-1-001/002 | log do teste |
| GREEN | fluxo mínimo de cadastro e Kanban | executar `F1-COCKPIT-01` | todos os passos principais passam | relatório/captura |
| REFACTOR/REGRESSÃO | validação, permissão e origem desconhecida | executar `F1-COCKPIT-02` | CA-1-003/004/005 passam sem regressão | log/captura |

**Dados/fixtures:** três leads de teste anonimizados: origem conhecida, origem desconhecida e entrada inválida.  
**Caminhos de erro obrigatórios:** campo inválido, sessão expirada, permissão insuficiente e duplicação do identificador de teste.  
**Evidência exigida:** captura do fluxo, log de testes e aceite humano do champion.

## Handoff e operação

- **Como demonstrar:** login → criar lead → movimentar card → abrir histórico.
- **Como operar depois:** SDR opera leads de teste; administrador mantém contas de teste.
- **Como monitorar:** erros de validação, falhas de autorização e eventos sem autor.
- **Pendência conhecida:** fonte de verdade e arquitetura de produção.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T01 | Implementar login, papéis e shell do cockpit | Desenvolvimento | SPEC-1-001 | CA-1-001/003 | F1-COCKPIT-01 | captura + log | ambiente de teste | ☐ |
| F1-T02 | Implementar cadastro, Kanban e histórico | Desenvolvimento | SPEC-1-001 | CA-1-001/002/005 | F1-COCKPIT-01/02 | registro + captura | F1-T01 aprovada | ☐ |
| F1-T03 | Validar erros, duplicidade e demonstração | Champion/QA | SPEC-1-001 | CA-1-003/004/005 | F1-COCKPIT-02 | roteiro assinado | F1-T02 aprovada | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
