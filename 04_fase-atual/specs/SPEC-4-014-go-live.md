# SPEC-4-014 — Entrada em operação: segurança, carga real, validação ponta a ponta e aceite

**Fase:** 4  
**Status:** planejada  
**Dono:** Consultora + Pontual (execução técnica: Dev full-stack e QA)  
**Origem no escopo:** validação transversal das fases 1 a 5 do Escopo Definitivo (seção 6, Fase 5), RF-15, RF-24, RNF-01–04, G1–G7  
**Degrau da solução:** encerramento — esta é a última fase do projeto. O que seria a Fase 5 (consolidação, validação ponta a ponta e decisão de entrada em operação) está aqui.

## Resultado observável

A Lidiane trabalha um dia inteiro no sistema, com os leads reais dela, sem abrir a planilha. O Marcelo vê os números do mês. A planilha "CRM - Acompanhamento" fica arquivada, somente leitura. O projeto é encerrado com aceite assinado e uma lista do que ficou para depois.

## Por que existe

- Três fases foram concluídas sem nenhum dado real no sistema.
- A senha inicial do usuário do Marcelo está escrita em uma migração no repositório.
- As decisões D1 a D7 e os gates G1 a G7 estão abertos desde a análise crítica; sem resolvê-los, o sistema não pode receber dado real nem ser medido.
- O escopo exige roteiro ponta a ponta com o champion e decisão humana de entrada em operação.

## Limites e dependências

- **Inclui:** correções de segurança, decisões do cliente, limpeza dos dados de teste, cópia de segurança, carga real, regressão das fases 1 a 3, roteiro ponta a ponta, material de uso, acompanhamento da primeira semana, aceite e lista de evoluções.
- **Fora de escopo:** qualquer funcionalidade nova. O que for descoberto aqui e não for defeito vira item da lista de evoluções.
- **Pré-condições:** SPEC-4-001 a SPEC-4-013 aceitas. A T4.1 (segurança) e a T4.2 (decisões) não dependem das outras e devem começar na **primeira semana** da fase.

## Parte 1 — Segurança antes do dado real

| # | Item | Como fica |
|---|---|---|
| 1 | Senha inicial em `pocketbase/migrations/0001_seed_initial_user.js` | Nova migração remove a senha fixa: o usuário inicial passa a ser criado com senha vinda do cofre de segredos (`$secrets`), como a migração `0002` já faz, e marcado para trocar no primeiro acesso. A senha atual do usuário do Marcelo é trocada **imediatamente**, antes de qualquer outra task. O repositório é conferido para estar privado. |
| 2 | Histórico do repositório | A senha antiga continua no histórico do Git. Como ela terá sido trocada, não dá mais acesso; registrar isso no relatório. Não reescrever o histórico. |
| 3 | Usuários de fixture (`Fixture Administrador`, `Fixture Gestor`, `Fixture SDR`) | Desativados em produção. |
| 4 | Regras de acesso direto às coleções | Conferir, coleção por coleção, que nenhuma coleção nova ou antiga pode ser lida ou gravada pela API padrão sem passar pelas rotas `/backend/v1`. Tabela coleção × regra no relatório. |
| 5 | Papéis nas rotas | Para cada rota nova da fase, uma chamada sem login, uma com papel insuficiente e uma com papel correto. |
| 6 | Arquivo `.env` no repositório | Contém só o endereço público do backend. Confirmar que nenhum segredo entra nele. |
| 7 | Sessão | Usuário inativo perde o acesso; troca de senha encerra as outras sessões. |
| 8 | Faixa de ambiente | A faixa "Ambiente de teste" (SPEC-4-003, regra 10) só sai quando a T4.2 for concluída. |

## Parte 2 — Decisões da Pontual

Uma reunião de até 90 minutos com o Marcelo e a Lidiane, conduzida pela consultora. O resultado é o arquivo `decisoes-go-live.md`, preenchido e datado, na pasta da fase. Cada decisão é aplicada no sistema **pela tela**, na própria reunião quando possível. O formulário completo, com o valor provisório de cada decisão, é o `decisoes-go-live.md`; a tabela abaixo é o resumo do que precisa sair da reunião.

| # | Decisão | Opções ou formato | Onde se aplica | Gate |
|---|---|---|---|---|
| 1 | Fonte oficial dos leads a partir de agora | A Central passa a ser a fonte única da operação comercial e o lugar onde a conversa acontece; a planilha é arquivada; o CRM Pontual continua para suporte; definir a data em que o RespondeChat deixa de ser usado para o comercial (ou se fica só para o Financeiro) | — | G1 |
| 1a | WhatsApp na Central | Contratar a Z-API; quais números conectar (Comercial, Comunicação); usar o número atual ou um novo; número separado para disparo frio (recomendado); quem atende cada número | Configurações → Canais | G6 |
| 1b | Envio automático | Liberar ou não, por canal: régua automática, envio de listas, resposta de agentes de IA. Tudo nasce desligado | Configurações → Canais | G5 |
| 1c | Autonomia dos agentes | Quais agentes entram em modo autônomo e quais ficam em copiloto; se algum pode registrar a decisão de qualificação; para quais etapas podem mover | Configurações → Agentes de IA | — |
| 1d | Funil | Conferir etapas e nomes; decidir se o funil continua depois do agendamento (ativar "Proposta enviada", "Negociação", "Fechado – ganho"); decidir se haverá funil separado por produto | Configurações → Funis e etapas | G2 |
| 1e | Campos | Quais colunas da planilha viram campo; quais são obrigatórios em cada etapa | Configurações → Campos | G2 |
| 1f | Contas de anúncio | Conectar a conta do Meta Ads e a do Google Ads (quem tem o acesso de administrador; quem na agência apoia) | Configurações → Integrações | G6 |
| 2 | Autorização para carregar dados reais | Termo simples assinado pelo Marcelo: quais bases entram (planilha de funil, bases de prospecção, listas), de onde vieram, quem tem acesso, por quanto tempo ficam | libera a carga | G5 |
| 3 | Política de contato para disparo | Quais bases podem receber disparo; intervalo mínimo entre disparos para o mesmo contato; o que conta como recusa; quem responde "onde conseguiu meu contato" e com qual texto | Configurações → Geral; base aprovada | G5 |
| 4 | Retenção | Por quanto tempo manter prospects sem resposta e leads desqualificados; o padrão proposto é 24 meses, com revisão anual | registrado; a limpeza periódica é manual nesta fase | G5 |
| 5 | Uso de IA | Autoriza ou não enviar as conversas e o histórico do lead (sem telefone, e-mail e CNPJ) ao modelo de IA | Configurações → Agentes de IA | — |
| 6 | Régua de cadência | Número de tentativas, intervalos e canais, por origem; prazo do primeiro contato | Configurações → Cadência | G4 |
| 7 | ICP por produto | Revisar e publicar o ICP de Automec, Climb, Hiper Gestão e Hiper Mini | Configurações → ICP | G2 |
| 8 | Catálogos | Conferir origens, produtos, segmentos e motivos | Configurações → Catálogos | G2 |
| 9 | Agendamento | Quem apresenta cada produto; duração; modalidade padrão; quem confirma | Configurações → Catálogos (produto) | G7 |
| 10 | Metas e evento que conta | Escolher entre as referências (20 qualificados, 10 apresentações, CPLQ até R$ 200; ou 25, 8 e R$ 160) ou definir outra; confirmar que o evento principal é a apresentação **agendada e confirmada**, com a realizada medida à parte | Configurações → Metas | G3 |
| 11 | Usuários e papéis | Quem entra, com qual papel; quem é o administrador além do Marcelo | Configurações → Usuários | — |
| 12 | Modelos de mensagem e base aprovada | A Lidiane reescreve os modelos iniciais e cadastra as respostas aprovadas (pode vir do Playbook) | Configurações | — |
| 13 | Captura automática | De onde os leads vão chegar sozinhos (formulário do site, anúncios) e quem na Pontual ou na agência configura | `integracoes.md` | G6 |
| 14 | Nome do sistema | O nome exibido no login e no cabeçalho | Configurações → Geral | — |

Decisão que ficar em aberto não trava a fase: o sistema entra com o valor provisório e a pendência vai para a lista de evoluções, com dono e data.

## Parte 3 — Carga real

Sequência obrigatória. Cada passo tem conferência antes do próximo.

| Passo | O quê | Quem | Conferência |
|---|---|---|---|
| 1 | Cópia de segurança completa do banco | Dev | arquivo guardado, com data; restauração testada em ambiente separado |
| 2 | Remoção dos dados de teste: leads, conversas, mídia, agendamentos, listas e eventos criados pelas fixtures das fases 1 a 4 | Dev | contagem zero em cada coleção operacional; catálogos, régua, modelos, ICP e usuários reais preservados |
| 3 | Importar a aba "CRM - Funil de Vendas" com o perfil pronto | Lidiane, com a consultora | relatório do lote; **conciliação**: contagem por etapa na planilha × contagem por estágio no sistema, com cada diferença explicada |
| 4 | Revisar as linhas rejeitadas e reimportar as corrigidas | Lidiane | arquivo de rejeitadas final vazio ou com justificativa |
| 5 | Resolver as duplicidades apontadas | Lidiane | tela Duplicidades sem pendências |
| 6 | Importar a aba "Leads Hiper - Tráfego Pago" | Lidiane | relatório do parceiro conferido contra a aba |
| 7 | Importar as bases de prospecção em uso | Lidiane | contagem de prospects; suprimidos marcados |
| 8 | Marcar como suprimidos os contatos que já recusaram (linhas vermelhas e "sem interesse") | Lidiane | lista de teste de disparo não inclui nenhum deles |
| 8a | Conectar o(s) número(s) de WhatsApp pela Z-API, **depois** da carga, para que as conversas novas encontrem os leads já cadastrados | Marcelo, com o Dev | canal "conectado"; mensagem de teste recebida e respondida |
| 8b | Conectar as contas de anúncio e sincronizar os últimos 90 dias | Marcelo ou Marketing | dashboard de anúncios com os três meses |
| 9 | Lançar o investimento em mídia dos últimos 3 meses | Marcelo ou Marketing | painel de mídia com os três meses |
| 10 | Definir próxima ação para os leads ativos importados | Lidiane | bloco "Sem próxima ação" da fila vazio |
| 11 | Conferir 20 fichas escolhidas ao acaso contra a planilha | Lidiane | tabela de conferência com 20 linhas |
| 12 | Arquivar a planilha como somente leitura, com a data | Marcelo | captura das permissões |

O guia do passo 3 ao 9, com imagens de onde clicar, é o `importacao-de-dados.md`.

## Parte 4 — Regressão das fases 1 a 3

Antes do roteiro com o cliente, o QA reexecuta os critérios de aceite das fases anteriores no sistema já alterado.

| Fase | Critérios | Observação |
|---|---|---|
| 1 | CA-1-01 a CA-1-09 | CA-1-02 (importação atômica) muda de propósito: a importação parcial passa a ser o padrão, e o modo "só se todas forem válidas" preserva o comportamento antigo. Registrar como alteração aprovada, não como falha. CA-1-04 passa a incluir os estágios novos. |
| 2 | CA-2-01 a CA-2-06 | O painel passa a somar lançamentos manuais. |
| 3 | CA-3-01 a CA-3-07 | CA-3-02 passa a ser atendido pela SPEC-4-008. CA-3-07 muda de propósito: o canal externo deixa de ser sempre bloqueado e passa a ser bloqueado **enquanto a chave de envio do canal estiver desligada**; a tentativa com a chave desligada continua negada e auditada. Registrar como alteração aprovada. |

Resultado: tabela critério × passou, falhou ou alterado de propósito. Falha é corrigida antes da Parte 5.

## Parte 5 — Roteiro ponta a ponta com o champion

Executado pela **Lidiane**, com dado real, sem ajuda — a consultora só observa e anota. Cada passo tem o resultado esperado; o que ela não conseguir fazer sozinha é registrado como defeito ou como ajuste de usabilidade.

| # | Passo | Resultado esperado |
|---|---|---|
| 1 | Entrar e abrir a fila do dia | Vê o que fazer, em ordem |
| 2 | Cadastrar um lead que acabou de chamar no WhatsApp | Menos de 1 minuto; aparece no Kanban e na fila |
| 3 | Tentar cadastrar um contato que já existe | O sistema avisa antes de salvar |
| 4 | Abrir a ficha, escrever uma nota e ver a nota no card do Kanban | Nota visível nos dois lugares |
| 5 | Registrar uma tentativa sem resposta e aceitar a próxima ação proposta | Lead sai da fila de hoje e volta na data proposta |
| 6 | Responder um lead pela tela Conversas, com um modelo e um anexo; pedir uma sugestão ao copiloto, editar e enviar | Mensagens entregues; tentativa registrada sozinha |
| 6a | Receber mensagem de um número desconhecido (número de teste) | Lead criado na etapa de entrada; conversa em "Sem dono" |
| 6b | Acompanhar uma conversa atendida pelo agente de triagem e assumir no meio | Agente pausa quando ela escreve; resumo do agente disponível |
| 7 | Qualificar um lead com o ICP do produto | Lead em "Qualificado", com critérios e evidência |
| 8 | Desqualificar um lead que buscava comprar material | Motivo registrado; aparece no painel por motivo |
| 9 | Colocar um lead em nutrição com retorno em 45 dias | Sai da fila; data registrada |
| 10 | Registrar um pedido de "não me procure mais" | Selo de suprimido; fora da fila e de qualquer lista |
| 11 | Agendar uma apresentação, confirmar e baixar o convite | Lead em "Agendado"; convite abre na agenda com a hora certa |
| 12 | Abrir o briefing da apresentação | Uma página, com o que o apresentador precisa |
| 13 | Reagendar | Histórico preservado; uma apresentação só no indicador |
| 14 | Marcar que a apresentação aconteceu e que a proposta foi enviada, com valor | Registrado; estágio não muda |
| 15 | Montar uma lista de prospecção de 50 contatos | Suprimidos fora; exclusões explicadas |
| 16 | Enviar a lista pela Central para 10 números de teste, com aprovação do Marcelo; pausar e retomar. Exportar a mesma lista em arquivo | Envios espaçados; arquivo de 4 colunas aceito pelo RespondeChat, se ainda estiver em uso |
| 17 | Registrar o retorno: 2 responderam, 1 sem interesse | 2 promovidos a lead; 1 suprimido |
| 18 | Importar uma planilha nova de contatos | Assistente concluído sem ajuda |
| 19 | (Marcelo) Abrir os indicadores do mês e clicar em um número | A lista bate com o número |
| 20 | (Marcelo) Ver a produtividade da semana e o relatório do parceiro | Sem digitação manual; relatório exportado |
| 21 | (Marcelo) Criar um usuário e mudar a régua | Feito pela tela |
| 21a | (Marcelo) Acrescentar uma etapa ao funil, criar um campo e colocá-lo no card | Aparece para todos, sem programação |
| 21b | (Marcelo) Alterar as instruções do agente de triagem, testar na área de teste e publicar | Novo comportamento valendo |
| 21c | (Marcelo) Abrir o dashboard de anúncios e identificar o anúncio com mais leads desqualificados | Número e motivo visíveis; lista abre ao clicar |
| 22 | (Dev) Enviar um lead pela rota de captura, com o formulário de teste | Lead aparece na fila com a origem certa |
| 23 | (Dev) Desconectar o número de WhatsApp e reconectar | Aviso na tela; mensagens na fila saem ao reconectar |

Critério de saída: todos os passos concluídos; no máximo 3 ajustes de usabilidade em aberto, nenhum deles impedindo o trabalho; nenhum defeito que cause perda ou exposição de dado.

## Parte 6 — Material e acompanhamento

| Entrega | Conteúdo | Formato |
|---|---|---|
| Guia rápido da Lidiane | o dia a dia em uma página: fila, tentativa, nota, qualificar, agendar, lista | página de ajuda dentro do sistema (menu do usuário → Ajuda) e PDF |
| Guia do administrador | usuários, catálogos, régua, ICP, base aprovada, metas, integrações, auditoria | idem |
| `importacao-de-dados.md` | já na pasta da fase | — |
| `integracoes.md` | já na pasta da fase | — |
| Roteiro de operação técnica | como fazer e restaurar cópia de segurança; como trocar segredos; o que olhar quando a captura falha; como desfazer uma importação; contatos | `runbook.md` na pasta da fase |
| Treinamento | uma sessão de 60 minutos com a Lidiane e uma de 45 com o Marcelo, gravadas | gravação no Drive |
| Primeira semana | a consultora acompanha 15 minutos por dia; todo problema vira item com dono e prazo | registro diário em `STATUS.md` |

Cópia de segurança: confirmar com a plataforma se há cópia automática diária do banco. Se não houver, o roteiro de operação define a cópia manual semanal e quem a faz.

## Parte 7 — Aceite e encerramento

O recibo final (`recibo-fase-4.md`) contém:

1. a tabela dos critérios CA-4-01 a CA-4-57, cada um com passou ou pendente e o link da evidência;
2. o resultado da regressão das fases 1 a 3;
3. o resultado do roteiro ponta a ponta;
4. a situação de cada gate, G1 a G7: resolvido (com a decisão) ou pendente (com dono e data);
5. a conciliação da carga real;
6. os números da primeira semana de uso: leads cadastrados, tentativas registradas, agendamentos, usuários ativos por dia;
7. a **lista de evoluções** — o que ficou fora, em ordem de valor. Já entram nela: migração do WhatsApp para a API oficial da Meta; outros canais na caixa de conversas (Instagram, e-mail, chat do site); criação automática de evento na agenda; acesso do parceiro ao sistema; feriados na régua; automações com condições encadeadas; campos calculados; limpeza automática por retenção; agente de voz;
8. a decisão de entrada em operação, com data e o nome de quem decidiu (Marcelo);
9. a assinatura do aceite.

Item não testado permanece **pendente** no recibo; ausência de reclamação não conta como aprovação.

Depois do aceite, a pasta `04_fase-atual/` é arquivada em `05_entregas/fase-4/` e o `STATUS.md` registra o encerramento do projeto.

## Cenários

| Cenário | Dado/condição | Resultado esperado |
|---|---|---|
| Principal | Carga da planilha de funil | Contagem por etapa conciliada; diferenças explicadas. |
| Limite | Planilha com linhas em branco no meio e cabeçalho em duas linhas | Importada sem edição manual. |
| Limite | Decisão 5 (IA) negada | Assistente desligado; todo o resto da fase aceito normalmente. |
| Falha | Carga com mapeamento errado | Desfazer importação e repetir; a cópia de segurança do passo 1 é o último recurso. |
| Falha | Roteiro com mais de 3 ajustes em aberto | Corrigir e repetir os passos afetados antes do aceite. |
| Recuperação | Problema grave na primeira semana | Restaurar a cópia de segurança e voltar à planilha arquivada, que está íntegra; a decisão é do Marcelo. |

## Critérios de aceite

- [ ] **CA-4-53:** Não há senha, chave nem token no código, nas migrações ou no histórico em uso: a senha inicial foi trocada, os usuários de teste estão desativados e toda rota nova nega acesso sem login e sem o papel exigido.
- [ ] **CA-4-54:** As decisões da Pontual estão registradas em decisoes-go-live.md, com data, e aplicadas no sistema pela tela; cada gate, de G1 a G7, está resolvido ou pendente com dono e data.
- [ ] **CA-4-55:** A base de produção não tem dados de teste; há cópia de segurança anterior à carga, com restauração testada; a carga real está conciliada — a contagem por etapa bate com a planilha de origem, com cada diferença explicada.
- [ ] **CA-4-56:** A regressão dos critérios das fases 1 a 3 está registrada (passou, falhou ou alterado de propósito); o roteiro ponta a ponta foi executado pela Lidiane e pelo Marcelo com dados reais, sem ajuda, com no máximo três ajustes de usabilidade em aberto.
- [ ] **CA-4-57:** Guia rápido, guia do administrador e roteiro de operação técnica estão entregues; os treinamentos foram feitos e gravados; o recibo final traz evidência por critério, a situação dos gates, a lista de evoluções e a decisão de entrada em operação assinada.

## Provas

| Prova | Como | Evidência |
|---|---|---|
| Segurança | os 8 itens da Parte 1 | relatório com uma linha por item |
| Decisões | `decisoes-go-live.md` preenchido | arquivo |
| Carga | os 12 passos da Parte 3 | relatório de conciliação |
| Regressão | Parte 4 | tabela de critérios |
| Ponta a ponta | Parte 5 | gravação da sessão e tabela dos passos |
| Aceite | Parte 7 | `recibo-fase-4.md` assinado |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.1 | Trocar a senha inicial do usuário real, remover a senha fixa da migração 0001 (passar a ler do cofre de segredos), desativar os usuários de fixture em produção e revisar as regras de acesso direto de todas as coleções | Dev backend | SPEC-4-014 | CA-4-53 | Falha: acesso sem papel e busca por segredos no repositório | relatório dos 8 itens de segurança + tabela coleção × regra | Fase 3 aceita | ☐ | A |
| T4.2 | Conduzir a reunião de decisões e preencher decisoes-go-live.md; aplicar pela tela cada decisão cuja tela já exista e registrar as demais como pendência com dono e data | Consultora + Pontual (Marcelo e Lidiane) | SPEC-4-014 | CA-4-54 | Manual: cada decisão com valor ou pendência | decisoes-go-live.md preenchido e datado + termo de carga de dados | Fase 3 aceita | ☐ | A |
| T4.72 | Fazer a cópia de segurança completa, testar a restauração em ambiente separado e remover os dados de teste das fases 1 a 4, preservando catálogos, funis, campos, régua, modelos, ICP, agentes e usuários reais | Dev backend | SPEC-4-014 | CA-4-55 | Prova: contagem zero nas coleções operacionais; restauração testada | relatório de contagens + registro da restauração | T4.70, T4.69, T4.62, T4.55, T4.54, T4.21, T4.43, T4.39, T4.29, T4.30, T4.71, T4.64, T4.61 aceitas | ☐ | N |
| T4.73 | Executar a carga real na ordem de importacao-de-dados.md: planilha de funil, leads do parceiro, bases de prospecção, suprimidos, anúncios; depois conectar os números de WhatsApp e as contas de anúncio; conciliar e arquivar a planilha | Pontual (Lidiane), com a consultora | SPEC-4-014 | CA-4-55 | Manual: os passos da Parte 3 da SPEC-4-014 | relatório de conciliação + 20 fichas conferidas | T4.72 aceita; T4.2 aceita; T4.1 aceita | ☐ | N |
| T4.74 | Reexecutar os critérios das fases 1 a 3 e executar o roteiro ponta a ponta com dados reais, sem ajuda; corrigir defeitos e repetir os passos afetados | QA + Pontual (Lidiane e Marcelo) | SPEC-4-014 | CA-4-56 | Regressão integral + roteiro da Parte 5 | tabela de critérios + gravação da sessão + tabela dos passos | T4.73 aceita | ☐ | N |
| T4.75 | Entregar o guia rápido, o guia do administrador (com ajuda dentro do sistema) e o roteiro de operação técnica; fazer e gravar os treinamentos; acompanhar a primeira semana de uso | Consultora | SPEC-4-014 | CA-4-57 | Manual: Parte 6 da SPEC-4-014 | guias + gravações + registro diário da primeira semana | T4.74 aceita | ☐ | N |
| T4.76 | Emitir o recibo final da fase com evidência por critério, situação dos gates, conciliação, números da primeira semana e lista de evoluções; colher a decisão de entrada em operação e o aceite; arquivar a fase | Consultora + Pontual (Marcelo) | SPEC-4-014 | CA-4-57 | Manual: Parte 7 da SPEC-4-014 | recibo-fase-4.md assinado | T4.75 aceita | ☐ | N |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
