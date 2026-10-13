# Fase 4 — Central completa em operação real (fase final)

> **Esta é a última fase do projeto.** Ela substitui o recorte anterior da Fase 4 (follow-up, reativação e agendamento em simulação) e absorve o que seria a Fase 5 (consolidação, validação ponta a ponta e entrada em operação).

## Por que o recorte mudou

As fases 1 a 3 entregaram um backend sólido — 41 rotas, auditoria, versionamento, idempotência — e quase nenhuma tela: 13 dessas rotas são usadas por alguma página. Não é possível cadastrar um lead pela interface, o lead não tem os campos da operação, o funil termina em "Qualificado" e nenhum dado real entrou no sistema. O plano anterior da Fase 4 acrescentaria mais backend em simulação.

Esta fase entrega o produto que a Pontual pediu nas reuniões: **um CRM comercial personalizável, com o WhatsApp dentro, agentes de IA configuráveis e o dashboard dos anúncios**, operando com os dados reais da empresa.

## Resultado

Ao final, a equipe comercial da Pontual trabalha o dia inteiro na Central, sem planilha:

1. os leads entram sozinhos (WhatsApp, site, anúncios) ou por cadastro e importação, já com origem e campanha;
2. a conversa com o lead acontece dentro da Central, pelo WhatsApp da empresa;
3. um agente de IA, configurado pelo administrador na tela, faz a triagem e a qualificação e chama uma pessoa na hora certa;
4. o funil, as etapas, os campos do lead e o card do Kanban são definidos pelo administrador, sem programação;
5. a fila do dia diz o que fazer, a régua de follow-up funciona — uma tentativa por vez — e o agendamento da apresentação fica registrado com briefing;
6. o dashboard mostra, até o nível do anúncio, quanto foi gasto e o que aconteceu com cada lead.

## O que a fase entrega, por área

| Área | Entrega | SPEC |
|---|---|---|
| Dados | lead com os campos reais; catálogos; funis e etapas como dados | SPEC-4-001 |
| Entrada | cadastro em menos de 1 minuto; aviso de duplicidade; captura automática | SPEC-4-002 |
| Importação | assistente que aceita as planilhas como elas são; desfazer | SPEC-4-003 |
| Trabalho | ficha completa do lead; Kanban com busca, filtros e arrastar; lista de leads | SPEC-4-004 |
| Rotina | fila do dia; registro de tentativas; régua de cadência | SPEC-4-005 |
| Prospecção | bases frias fora do funil; listas com exclusões; exportação | SPEC-4-006 |
| Agendamento | apresentação com confirmação; briefing; convite; resultado | SPEC-4-007 |
| IA | agentes criados e ajustados pela interface; copiloto | SPEC-4-008 |
| Gestão | funil, produtividade, dashboard de anúncios, parceiro, metas | SPEC-4-009 |
| Administração | usuários, catálogos, régua, modelos de mensagem, ICP, base aprovada, auditoria | SPEC-4-010 |
| Integrações | chaves, registro de eventos, fila de exceção, painel de situação | SPEC-4-011 |
| Personalização | funis, etapas, campos, ficha, card e automações pela interface | SPEC-4-012 |
| Conversas | WhatsApp dentro da Central pela Z-API; envio de listas e de régua | SPEC-4-013 |
| Operação | segurança, decisões, carga real, validação, treinamento, aceite | SPEC-4-014 |

Documentos de apoio, nesta pasta:

| Arquivo | Para quê |
|---|---|
| `integracoes.md` | contrato de cada integração (Z-API, Meta Ads, Google Ads, captura do site, Google Sheets, agenda, RespondeChat) e a lista de conferência para ligar tudo |
| `importacao-de-dados.md` | guia da carga: o que entra, em que ordem, mapeamento coluna a coluna, erros comuns, conferência |
| `decisoes-go-live.md` | as decisões que a Pontual precisa tomar, para preencher na reunião |
| `modelos/` | modelos de arquivo para importação |
| `fixtures/` | dados fictícios para os testes |
| `matriz-specs-fases.md` | rastreabilidade: requisito → SPEC → critério → task |
| `tasks.json` | as tasks, no formato do Maestro |

## Inclui

RF-01 a RF-24 do PRD na parte que ainda não tinha tela ou dado real, com destaque para: RF-02/03 (entrada e deduplicação), RF-04/05/08 (Kanban, fila, timeline), RF-06/23 (qualificação e ICP), RF-09/10/11 (conversa, transbordo, agente), RF-12/14 (administração pela interface), RF-13/14 (follow-up e reativação), RF-15 (supressão), RF-16/17 (agendamento e handoff), RF-18 a RF-21 (indicadores, mídia, recomendações, feedback), RF-22 (importação). RN-01 a RN-15 e RNF-01 a RNF-10 continuam valendo; as alterações de propósito estão na seção "O que muda em relação às fases anteriores".

## Demonstração

Roteiro ponta a ponta com dados reais, executado pela Lidiane e pelo Marcelo (SPEC-4-014, Parte 5): um lead chega pelo WhatsApp a partir de um anúncio; o agente de triagem conversa e preenche a ficha; a Lidiane assume, qualifica com o ICP, agenda a apresentação e gera o briefing; o Marcelo vê o lead no dashboard, na linha do anúncio que o trouxe; em seguida, o Marcelo acrescenta uma etapa ao funil, cria um campo e ajusta as instruções do agente, pela tela.

## Critérios de aceite

**SPEC-4-001 — Lead com campos reais, catálogos e funis/etapas como dados**

- **CA-4-01:** Todo lead que já existia continua visível depois da migração, com nome, telefone e e-mail nos campos novos; nenhuma tela nem resposta de rota mostra a palavra "fictício"; o telefone é guardado normalizado e exibido formatado.
- **CA-4-02:** Origem, produto, segmento e motivo vêm de catálogos: valor fora do catálogo não é aceito em formulário e, em importação ou integração, cai em "Desconhecida", "A definir" ou "A classificar", de forma visível.
- **CA-4-03:** As colunas do Kanban vêm dos dados do funil, não do código; mover um lead sem cumprir a regra da categoria ou um campo obrigatório da etapa é negado com a lista do que falta; o histórico anterior permanece intacto.

**SPEC-4-002 — Cadastro manual, aviso de duplicidade e captura automática**

- **CA-4-04:** Um usuário cria um lead pelo botão "Novo lead" em menos de 1 minuto, com três informações obrigatórias (empresa ou contato, telefone, origem), e o lead aparece no Kanban e na fila sem recarregar a página.
- **CA-4-05:** Ao informar telefone, CNPJ ou e-mail que já existe, a tela mostra o registro existente antes de salvar e oferece abrir, anotar no existente ou criar mesmo assim com motivo; a tela Duplicidades permite mesclar dois registros sem perder notas, tentativas nem histórico.
- **CA-4-06:** Uma chamada válida à rota de captura cria o lead com origem, campanha e parâmetros de campanha em até 5 segundos; a repetição com a mesma chave de idempotência não duplica; chave inválida é negada e registrada.

**SPEC-4-003 — Assistente de importação que aceita as planilhas reais, com desfazer**

- **CA-4-07:** O usuário importa um arquivo CSV ou XLSX, um texto colado do Excel ou uma planilha Google, com colunas em qualquer ordem e com qualquer nome, mapeando as colunas na tela (inclusive para campos personalizados), e vê a prévia já normalizada antes de confirmar.
- **CA-4-08:** A planilha real "CRM - Funil de Vendas" entra com o perfil pronto, sem edição manual do arquivo: etapas, fontes, tentativas (data e canal) e observações viram etapa, origem, tentativas e nota.
- **CA-4-09:** Linhas inválidas não bloqueiam as válidas: o usuário importa as válidas, baixa as rejeitadas com o motivo por linha, decide as duplicidades em grupo e pode desfazer o lote inteiro enquanto os leads não tiverem sido trabalhados; mídia e conversas têm tela de importação.

**SPEC-4-004 — Ficha completa do lead, Kanban de trabalho e lista de leads**

- **CA-4-10:** A ficha do lead abre a partir do Kanban, da fila, da busca, da lista e das conversas; todo campo é editável no lugar e cada alteração fica na aba Atividades com autor, data, valor anterior e novo; a busca global encontra o lead por nome, telefone em qualquer formato, e-mail ou CNPJ.
- **CA-4-11:** Uma nota escrita na ficha aparece na ficha, no card do Kanban e na fila, sem recarregar; notas não são apagadas.
- **CA-4-12:** A aba Atividades reúne, em ordem cronológica e com filtro por tipo, mudanças de etapa e de campo, notas, tentativas, mensagens, transbordos, agendamentos, ações de IA e de automação.
- **CA-4-13:** Qualificar, desqualificar e nutrir são feitos na ficha, com o ICP vigente do produto e motivo do catálogo; faltando critério obrigatório, a tela mostra a pendência e não avança; a versão do ICP usada fica registrada e o histórico não é reclassificado.
- **CA-4-14:** O opt-out é registrado em dois cliques, com origem e evidência; o lead suprimido mostra o selo em toda tela e sai das filas, listas e envios; transbordo aberto aparece na ficha e na fila, com assumir e resolver.
- **CA-4-15:** O Kanban tem seletor de funil, busca, filtros e arrastar entre colunas; o card mostra o conteúdo configurado, a próxima ação com atraso e a última nota; soltar em etapa que exige dados abre o diálogo pedindo só o que falta.

**SPEC-4-005 — Fila do dia, tentativas e régua de cadência**

- **CA-4-16:** Registrar uma tentativa de contato leva no máximo três cliques (canal, resultado, salvar); contadores e última interação atualizam na hora.
- **CA-4-17:** Depois de uma tentativa sem resposta, o sistema propõe a próxima ação pela régua (dia, hora e canal), dentro da janela e dos dias úteis; o usuário aceita ou ajusta; a régua é alterada pelo administrador na tela, sem código.
- **CA-4-18:** A tela Minha fila mostra, em ordem de prioridade, transbordos, novos sem primeiro contato com o prazo, agendamentos a confirmar, atrasados, de hoje, retornos de nutrição, sem próxima ação e a corrigir; cada item é resolvido na própria fila; nenhum lead ativo fica fora dela.
- **CA-4-19:** Ao esgotar as tentativas, o sistema pergunta o que fazer (encerrar, nutrir com data, tentar de novo); nada é encerrado sem confirmação humana; lead em nutrição reaparece na fila na data de retorno.

**SPEC-4-006 — Bases de prospecção, listas com exclusões e exportação**

- **CA-4-20:** Bases de prospecção ficam em tela própria, fora do Kanban e da fila, até haver resposta; "Promover a lead" leva um clique e preserva origem, etiquetas e histórico.
- **CA-4-21:** A lista mostra, antes de ser usada, os critérios, o tamanho, os excluídos por motivo e a prévia dos contatos; as exclusões obrigatórias (suprimido, telefone inválido, já cliente, em negociação) não podem ser desligadas e são reaplicadas no servidor na hora do uso.
- **CA-4-22:** A exportação gera um arquivo com exatamente quatro colunas — Nome, Número, E-mail, Etiquetas — aceito pelo RespondeChat sem edição; cada exportação fica registrada (quem, quando, quantos, etiqueta) e marcada na timeline dos contatos; exportar de novo não duplica registros.
- **CA-4-23:** O retorno é registrado em grupo (respondeu, sem interesse, número inválido); "sem interesse" cria a supressão e tira o contato de toda lista futura; "respondeu" oferece promover a lead.

**SPEC-4-007 — Agendamento com confirmação, briefing, convite e resultado**

- **CA-4-24:** Um lead só conta como agendado com data, hora, apresentador e confirmação registrada; agendamento sem confirmação aparece como pendência na fila e não entra no indicador; a tela Agenda mostra a semana por apresentador.
- **CA-4-25:** Reagendar, cancelar, registrar falta e registrar que aconteceu criam eventos novos sem apagar os anteriores; os indicadores contam uma apresentação por série; o resultado depois da apresentação é anotação e não cria fluxo.
- **CA-4-26:** O briefing da apresentação cabe em uma página, com produto, contexto, critérios do ICP atendidos, pendências e histórico resumido, sem dado desnecessário; o convite .ics abre no Outlook e no Google Agenda com a hora correta e é atualizado ao reagendar.

**SPEC-4-008 — Agentes de IA configuráveis pela interface e copiloto**

- **CA-4-27:** O administrador cria, instrui, testa e publica um agente de IA pela tela, sem código; a área de teste mostra a resposta e as ações que o agente faria, sem enviar mensagem nem gravar no lead; a publicação exige os cenários prontos executados; a versão anterior pode ser restaurada.
- **CA-4-28:** Em conversa real, o agente em modo autônomo se identifica como assistente virtual, conduz o roteiro de qualificação em conversa, preenche os campos permitidos com a evidência e executa só as capacidades ligadas; o que não estiver ligado é bloqueado no servidor e registrado.
- **CA-4-29:** O agente só afirma o que está no conhecimento aprovado; preço, contrato ou prazo fora dele, pedido de falar com uma pessoa, recusa, mídia não interpretada ou limite atingido geram transbordo com motivo e resumo, e o agente para de escrever; mensagem de uma pessoa na conversa pausa o agente.
- **CA-4-30:** O copiloto entrega, na ficha e na conversa, o resumo do histórico com referências em até 15 segundos, a sugestão de mensagem editável e a sugestão de qualificação por critério como rascunho; a decisão registrada é sempre de uma pessoa, e fica gravado se ela seguiu a sugestão.
- **CA-4-31:** De fábrica, todo agente nasce em copiloto, o envio automático do canal nasce desligado e a decisão de qualificação é humana; telefone, e-mail e CNPJ não são enviados ao modelo; com a IA indisponível ou desligada, todas as telas continuam funcionando.
- **CA-4-32:** Cada agente tem painel de desempenho (conversas, transbordos por motivo, concordância com a decisão humana, campos corrigidos, uso) e uma lista de conversas para revisar; pausar um agente o interrompe na hora em todas as conversas.

**SPEC-4-009 — Funil, produtividade, dashboard de anúncios, parceiro e metas**

- **CA-4-33:** O painel de funil mostra, por período, origem, produto e responsável, entradas, primeiro contato, qualificados, agendamentos e apresentações realizadas, com taxas e tempos; clicar em qualquer número abre a lista dos leads que o compõem, e a contagem da lista é igual ao número.
- **CA-4-34:** A produtividade diária e semanal por usuário (mensagens, ligações, follow-ups, respostas, reuniões agendadas e realizadas) é calculada automaticamente a partir das tentativas, mensagens e agendamentos, sem digitação.
- **CA-4-35:** O dashboard de anúncios mostra, em três níveis (campanha, conjunto, anúncio), investimento, impressões, cliques, leads no CRM, qualificados, agendamentos, CPL, CPLQ e o motivo de desqualificação mais comum; a soma dos filhos é igual ao pai; "Sem anúncio identificado" e "Origem desconhecida" ficam em linhas próprias; metas só aparecem se cadastradas.
- **CA-4-36:** Um lead que chega por anúncio com botão de WhatsApp, ou por página com parâmetros de campanha, é ligado ao anúncio sem ninguém informar; o gasto entra por sincronização, relatório importado ou lançamento manual, sem contagem em dobro; falha de sincronização não derruba o dashboard.
- **CA-4-37:** A devolutiva para o Marketing e o relatório do parceiro são gerados e exportados sem dado pessoal de lead e sem dados de outras origens; o papel marketing não vê nome, telefone nem e-mail em nenhuma tela.

**SPEC-4-010 — Usuários, catálogos, régua, modelos, ICP, base aprovada e auditoria**

- **CA-4-38:** O administrador cria, desativa, redefine a senha e muda o papel de um usuário pela tela; a senha temporária aparece uma única vez e é trocada no primeiro acesso; usuário desativado não entra; cada papel só vê e faz o que lhe cabe, em teste positivo e negativo.
- **CA-4-39:** O administrador altera catálogos, régua, modelos de mensagem, ICP por produto e base aprovada pela interface; a alteração vale para ações novas, sem reclassificar o histórico; a versão anterior de régua, ICP e base aprovada pode ser restaurada.
- **CA-4-40:** A tela de auditoria lista os eventos críticos (acessos negados, alterações de configuração, importações, exportações, envios, decisões) com filtro por usuário, período e tipo; nenhum evento pode ser alterado ou apagado.

**SPEC-4-011 — Chaves, registro de eventos, fila de exceção e painel de situação**

- **CA-4-41:** A central de integrações mostra a situação de cada integração e o último evento; o administrador cria, testa e revoga chaves de acesso, e o valor da chave aparece uma única vez.
- **CA-4-42:** Um evento que falha vai para a fila de exceção com o motivo em linguagem clara e pode ser corrigido e reprocessado, ou descartado com motivo; com todas as integrações desligadas, cadastro, importação, fila e agendamento continuam funcionando.

**SPEC-4-012 — Funis, etapas, campos, ficha, card e automações pela interface**

- **CA-4-43:** O administrador cria um funil, acrescenta, renomeia, reordena e desativa etapas e publica, tudo pela tela; o Kanban de todos os usuários reflete a mudança; as proteções impedem deixar o funil sem etapa de entrada, de agendamento ou de encerramento; a versão anterior pode ser restaurada.
- **CA-4-44:** Um campo personalizado, de qualquer um dos tipos, criado pela tela, passa a valer sem programação na ficha, no cadastro, no mapeamento da importação, na captura, nos filtros, no card, nos modelos de mensagem e nos agentes; excluído por engano, pode ser restaurado com os valores em até 30 dias.
- **CA-4-45:** As ações automáticas configuradas em uma etapa são executadas quando um lead entra nela e ficam na timeline marcadas como automáticas; uma ação que falha não impede a movimentação e aparece para o administrador; importação em massa não dispara envio de mensagem nem agente.
- **CA-4-46:** O administrador define as seções da ficha e o conteúdo do card do Kanban por funil; os usuários salvam visões da lista de leads com filtros, colunas e ordenação.

**SPEC-4-013 — WhatsApp dentro da Central pela Z-API; envio de listas e de régua**

- **CA-4-47:** O administrador conecta um número de WhatsApp lendo o QR code na tela da Central; as credenciais da Z-API nunca aparecem em tela nem trafegam para o navegador; a desconexão do número é avisada na tela de conversas e ao administrador.
- **CA-4-48:** Uma mensagem recebida aparece em Conversas em até 3 segundos, sem recarregar; texto, imagem, áudio e documento são enviados e mostram os estados (enviada, entregue, lida, falhou); o mesmo evento recebido duas vezes não duplica; grupos são ignorados.
- **CA-4-49:** Um número desconhecido que escreve vira lead na etapa de entrada do funil, com a origem do canal ou do anúncio; ao lado da conversa, o painel do lead permite editar campos, mover a etapa, anotar e agendar sem sair da tela.
- **CA-4-50:** Nenhuma mensagem — de pessoa, lista, régua, agente ou automação — é enviada a contato suprimido, e a tentativa é negada no servidor e auditada; toda primeira mensagem do dia para um lead é registrada como tentativa, sem digitação.
- **CA-4-51:** O envio de lista pela Central exige aprovação quando previsto, sai com intervalo aleatório dentro do ritmo configurado, respeita o limite diário e pausa sozinho por recusas, falhas ou desconexão; a régua automática envia uma tentativa por vez e cancela os envios pendentes quando o contato responde; com as chaves do canal desligadas, nada automático é enviado.
- **CA-4-52:** A conversa tem dono e situação (aberta, aguardando, resolvida); assumir, transferir e resolver ficam registrados; notas internas aparecem só para a equipe; os modelos de mensagem são inseridos com as variáveis preenchidas.

**SPEC-4-014 — Segurança, decisões, carga real, validação ponta a ponta e aceite**

- **CA-4-53:** Não há senha, chave nem token no código, nas migrações ou no histórico em uso: a senha inicial foi trocada, os usuários de teste estão desativados e toda rota nova nega acesso sem login e sem o papel exigido.
- **CA-4-54:** As decisões da Pontual estão registradas em decisoes-go-live.md, com data, e aplicadas no sistema pela tela; cada gate, de G1 a G7, está resolvido ou pendente com dono e data.
- **CA-4-55:** A base de produção não tem dados de teste; há cópia de segurança anterior à carga, com restauração testada; a carga real está conciliada — a contagem por etapa bate com a planilha de origem, com cada diferença explicada.
- **CA-4-56:** A regressão dos critérios das fases 1 a 3 está registrada (passou, falhou ou alterado de propósito); o roteiro ponta a ponta foi executado pela Lidiane e pelo Marcelo com dados reais, sem ajuda, com no máximo três ajustes de usabilidade em aberto.
- **CA-4-57:** Guia rápido, guia do administrador e roteiro de operação técnica estão entregues; os treinamentos foram feitos e gravados; o recibo final traz evidência por critério, a situação dos gates, a lista de evoluções e a decisão de entrada em operação assinada.

## Gates e decisões do cliente

Os gates G1 a G7 estão abertos desde a análise crítica. Nesta fase eles deixam de ser bloqueios indefinidos e viram **decisões com dono e data**, tomadas na reunião da T4.2 e registradas em `decisoes-go-live.md`.

| Gate | Assunto | Como é resolvido nesta fase | O que fica desligado até a decisão |
|---|---|---|---|
| G1 | Fonte oficial | a Central passa a ser a fonte da operação comercial; planilha arquivada | carga de dados reais |
| G2 | ICP e dicionário | ICP por produto, catálogos, funil e campos revisados e publicados pela tela | nada; vale o provisório |
| G3 | Baseline e metas | metas cadastradas pela Pontual; sem meta, o painel mostra só o realizado | exibição de metas |
| G4 | Cadência | régua confirmada ou alterada pela tela | nada; vale a régua provisória |
| G5 | LGPD, contato e retenção | termo de carga, política de contato, retenção, uso de IA | carga real; envio de listas; régua automática; envio de dados à IA |
| G6 | Contrato técnico dos canais | Z-API para o WhatsApp; APIs ou relatórios para os anúncios (`integracoes.md`) | conexão do número; sincronização dos anúncios |
| G7 | Agenda | apresentador por produto e regras de confirmação; convite `.ics` | nada |

**Padrões seguros:** todo envio automático nasce desligado — régua automática, envio de listas e resposta de agentes são três chaves por canal, ligadas só por decisão registrada. Todo agente nasce em modo copiloto. A decisão de qualificado ou desqualificado nasce humana.

## Controles transversais de segurança

- A senha inicial que está escrita em uma migração é trocada **antes de qualquer outra task** (T4.1); nenhum segredo fica no código.
- Credenciais da Z-API e das contas de anúncio ficam só no servidor; nenhuma rota as devolve; todas as chamadas a sistemas externos saem do backend.
- Dados reais só entram depois do termo de carga; até lá, o ambiente mostra a faixa "Ambiente de teste".
- Nenhum envio, por nenhum caminho, para contato suprimido; a conferência é feita no servidor, na hora do envio.
- O que vai para o modelo de IA não inclui telefone, e-mail nem CNPJ; o texto do contato é tratado como dado, nunca como instrução.
- Auditoria continua sem alteração nem exclusão, inclusive para o administrador, e ganha tela.
- Registros de integração e evidências mascaram telefone, e-mail e documentos.
- Exportações seguem a lista de campos permitidos e a neutralização de fórmulas das fases anteriores.
- O papel `marketing` não vê nome, telefone, e-mail nem conversa de lead.

## O que muda em relação às fases anteriores

Alterações de propósito, para serem registradas como tal na regressão (SPEC-4-014, Parte 4):

| Antes | Agora | Motivo |
|---|---|---|
| Importação atômica: uma linha inválida rejeita o lote (CA-1-02) | importação parcial por padrão, com arquivo de rejeitadas; o modo "só se todas forem válidas" continua disponível | a planilha real tem defeitos; rejeitar tudo impede a carga |
| Funil fixo no código, com transições em linha reta (CA-1-04) | etapas configuráveis; regras por categoria de etapa | a operação precisa pular e voltar etapas, e o funil é da Pontual |
| Campos `*_ficticio`, e-mail obrigatório | campos reais; e-mail opcional | boa parte dos leads não tem e-mail |
| Canal externo sempre bloqueado (CA-3-07) | bloqueado enquanto a chave de envio do canal estiver desligada | o WhatsApp passa a operar dentro da Central |
| IA só sugere, por comparação de palavras (CA-3-02) | agentes com modelo de linguagem; modo autônomo opcional, desligado de fábrica | pedido da Pontual; os limites de conteúdo (RN-04) são mantidos |
| Disparo governado em simulação (plano anterior da Fase 4) | envio real de listas, com aprovação, ritmo, limite e pausa automática | a simulação não resolvia a dor |
| Fase 5 separada | incorporada à SPEC-4-014 | esta é a última fase |

## Fora desta fase

API oficial do WhatsApp (Meta); outros canais de conversa (Instagram, e-mail, chat do site); grupos do WhatsApp; proposta, contrato, assinatura e faturamento (Autentique, Conexa); implantação, suporte e pós-venda; acesso do parceiro ao sistema; alteração de campanhas pelo sistema; automações com condições encadeadas; campos calculados; agente de voz. Tudo isso entra na lista de evoluções do recibo final.

## Ordem de execução

A fase é grande. As levas são feitas na ordem; dentro de uma leva, as tasks podem andar em paralelo quando as pré-condições permitirem. Três tasks **não esperam** as demais e começam na primeira semana: T4.1 (segurança), T4.2 (decisões do cliente) e T4.3 (contratações e acessos externos, que têm prazo de terceiros).

Marcos sugeridos, para acompanhamento com o cliente:

| Marco | Levas | O que o cliente já consegue ver |
|---|---|---|
| 1 — O lead real | A a D | cadastrar, ver a ficha, mover no Kanban, personalizar funil e campos |
| 2 — A carga e a rotina | E a H | importar a planilha, trabalhar a fila, agendar, administrar |
| 3 — A conversa | I e J | atender pelo WhatsApp dentro da Central; listas e envios |
| 4 — A inteligência | K e L | agentes de IA; dashboard de anúncios |
| 5 — Em operação | M e N | integrações ligadas, carga real, validação e aceite |

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.1 | Trocar a senha inicial do usuário real, remover a senha fixa da migração 0001 (passar a ler do cofre de segredos), desativar os usuários de fixture em produção e revisar as regras de acesso direto de todas as coleções | Dev backend | SPEC-4-014 | CA-4-53 | Falha: acesso sem papel e busca por segredos no repositório | relatório dos 8 itens de segurança + tabela coleção × regra | Fase 3 aceita | ☐ | A |
| T4.2 | Conduzir a reunião de decisões e preencher decisoes-go-live.md; aplicar pela tela cada decisão cuja tela já exista e registrar as demais como pendência com dono e data | Consultora + Pontual (Marcelo e Lidiane) | SPEC-4-014 | CA-4-54 | Manual: cada decisão com valor ou pendência | decisoes-go-live.md preenchido e datado + termo de carga de dados | Fase 3 aceita | ☐ | A |
| T4.3 | Contratar a Z-API e criar uma instância por número; ativar o token de segurança da conta; criar o usuário do sistema e o token de leitura do Meta Ads; pedir o token de desenvolvedor do Google Ads; conferir os conectores habilitados no Skip | Pontual (Marcelo) | SPEC-4-013 | CA-4-47 | Manual: itens 1 a 4 da lista de conferência de integracoes.md | credenciais entregues por canal seguro + protocolo do pedido ao Google + lista de conectores | Fase 3 aceita | ☐ | A |
| T4.4 | Criar a migração aditiva da coleção leads com os campos reais, copiar os valores dos campos *_ficticio, normalizar telefones e criar os índices | Dev backend | SPEC-4-001 | CA-4-01 | RED/GREEN: contagem por etapa antes e depois | tabela antes × depois + lista de telefones a corrigir | Fase 3 aceita | ☐ | A |
| T4.5 | Implementar os normalizadores compartilhados (telefone, CNPJ, e-mail, data, moeda, texto seguro), usados por formulário, importação e integração | Dev full-stack | SPEC-4-001 | CA-4-01 | RED/GREEN: exemplos da SPEC e 10 casos de erro | saída dos testes unitários | Fase 3 aceita | ☐ | A |
| T4.6 | Criar a coleção de catálogos com os dados iniciais (origens, produtos, segmentos, motivos, etiquetas) e a rota de leitura; converter origens e produtos antigos para códigos do catálogo | Dev backend | SPEC-4-001 | CA-4-02 | Principal/limite: valor fora do catálogo | catálogos carregados + leads antigos convertidos | T4.4 aceita | ☐ | A |
| T4.7 | Criar as coleções de funis e etapas com as categorias do sistema e o funil padrão; trocar a lista fixa de estágios pela leitura dos dados; reescrever a rota de mudança de etapa para validar por categoria e por campos obrigatórios | Dev backend | SPEC-4-001 | CA-4-03 | RED/GREEN: uma movimentação válida e uma inválida por categoria | respostas das rotas + busca no código sem etapas fixas | T4.4 aceita | ☐ | A |
| T4.8 | Ajustar as rotas existentes de Kanban, fila, métricas e exportação aos campos novos e às categorias de etapa, mantendo os campos de resposta antigos | Dev backend | SPEC-4-001 | CA-4-03 | Regressão: critérios das fases 1 a 3 que usam essas rotas | lista de critérios com passou ou falhou | T4.7 aceita; T4.6 aceita | ☐ | A |
| T4.9 | Reorganizar a navegação: menu lateral por papel, rotas novas com redirecionamento das antigas, cabeçalho com busca global e sino de pendências, página inicial por papel; retirar da tela de login os recursos que não existem | Dev frontend | SPEC-4-004 | CA-4-10 | Principal: menu de cada papel; limite: endereço proibido | capturas do menu por papel + busca pelos três formatos de telefone | T4.8 aceita | ☐ | B |
| T4.10 | Criar as rotas de leitura e edição do lead e a ficha (cabeçalho e aba Resumo) com edição no lugar, controle de edição concorrente e registro de antes e depois | Dev full-stack | SPEC-4-004 | CA-4-10 | RED/GREEN: edição concorrente; principal: editar 3 campos | gravação + aba Atividades com as 3 edições | T4.9 aceita | ☐ | B |
| T4.11 | Criar as notas do lead (coleção, rotas, caixa de nota na ficha, fixar, marcar como incorreta) e mostrar a última nota no card do Kanban e na fila | Dev full-stack | SPEC-4-004 | CA-4-11 | Principal: nota aparece nos três lugares; falha: tentativa de apagar | capturas da ficha, do card e da fila | T4.10 aceita | ☐ | B |
| T4.12 | Criar a rota e a aba Atividades, que reúnem em ordem cronológica mudanças, notas, tentativas, mensagens, transbordos, agendamentos, IA e automações, com filtro por tipo | Dev full-stack | SPEC-4-004 | CA-4-12 | Principal: um item de cada tipo | captura da aba com os filtros | T4.10 aceita | ☐ | B |
| T4.13 | Criar a aba Qualificação: ICP vigente do produto, respostas por critério, pendências e as decisões Qualificar, Desqualificar e Nutrir com motivo do catálogo, usando as rotas de qualificação e de ICP já existentes | Dev full-stack | SPEC-4-004 | CA-4-13 | RED/GREEN: critério obrigatório faltando; principal: as três decisões | capturas + registros das fichas com a versão do ICP | T4.10 aceita; T4.6 aceita | ☐ | B |
| T4.14 | Levar para a ficha o registro de opt-out e a remoção por administrador, o selo de suprimido em todas as telas e a faixa de transbordo com abrir, assumir e resolver, usando as rotas já existentes | Dev full-stack | SPEC-4-004 | CA-4-14 | Falha: ação em lead suprimido; principal: opt-out em dois cliques | capturas da ficha, do Kanban e da fila + auditoria | T4.10 aceita | ☐ | B |
| T4.15 | Refazer o Kanban: seletor de funil, colunas a partir dos dados, busca, filtros, arrastar e soltar com diálogo do que falta, card com próxima ação, atraso e última nota, carregamento por coluna e atualização em tempo real | Dev frontend | SPEC-4-004 | CA-4-15 | Principal: arrastar; limite: soltar em etapa que exige dados; acessibilidade por teclado | gravação com o mouse e só com o teclado | T4.10 aceita; T4.11 aceita | ☐ | B |
| T4.16 | Criar a tela Leads: tabela com colunas escolhidas, filtros no endereço, ordenação, paginação, seleção múltipla e ações em massa (responsável, próxima ação, etiqueta, exportar) | Dev full-stack | SPEC-4-004 | CA-4-10 | Principal: filtro e ação em massa; limite: ação em massa de sdr em lead de outro | capturas + registro das ações na auditoria | T4.10 aceita | ☐ | B |
| T4.17 | Criar os campos personalizados (definições, valores no lead, validação por tipo), as configurações dos campos de sistema e a rota que descreve os campos para as telas | Dev backend | SPEC-4-012 | CA-4-44 | RED/GREEN: valor inválido por tipo | testes por tipo + resposta da rota de campos | T4.10 aceita | ☐ | C |
| T4.18 | Criar a tela Campos: criar, editar, ordenar, ocultar, excluir e restaurar; e fazer a ficha, o cadastro, os filtros e a exportação se desenharem a partir da rota de campos | Dev full-stack | SPEC-4-012 | CA-4-44 | Principal: um campo de cada tipo nos lugares previstos; falha: mudar tipo com valores, excluir campo em uso | tabela lugar × funcionou + capturas das recusas | T4.17 aceita | ☐ | C |
| T4.19 | Criar o editor de funis e etapas: rascunho, criação, renomeação, reordenação, cor, alerta de lead parado, campos obrigatórios, duplicar funil, prévia do impacto, publicação atômica com movimentação de leads, versões e restauração, com as proteções | Dev full-stack | SPEC-4-012 | CA-4-43 | Principal: criar etapa e funil; falha: as proteções; recuperação: restaurar versão | gravação + capturas das mensagens de proteção | T4.7 aceita; T4.17 aceita; T4.15 aceita | ☐ | C |
| T4.20 | Implementar as ações automáticas ao entrar na etapa (próxima ação, responsável fixo ou em rodízio, etiqueta, preencher campo, transbordo, notificar), o registro de execução e a tela de falhas; deixar previstas as ações de enviar modelo e acionar agente, que passam a funcionar nas levas I e K | Dev backend | SPEC-4-012 | CA-4-45 | Principal: três ações em uma etapa; falha: ação que falha não impede a movimentação; limite: importação em massa | timeline de três leads (movido à mão, importado, por integração) | T4.19 aceita | ☐ | C |
| T4.21 | Criar a configuração das seções da ficha e do card do Kanban por funil, com pré-visualização, e as visões salvas da lista de leads (pessoais e compartilhadas) | Dev full-stack | SPEC-4-012 | CA-4-46 | Principal: mudar o card e salvar uma visão | capturas do card antes e depois + visão compartilhada vista por outro usuário | T4.18 aceita; T4.15 aceita; T4.16 aceita | ☐ | C |
| T4.22 | Criar o botão Novo lead e o formulário rápido e completo, com a rota de criação, os campos personalizados obrigatórios e a criação da primeira próxima ação | Dev full-stack | SPEC-4-002 | CA-4-04 | Principal: cadastro cronometrado; limite: telefone sem DDD | gravação com o tempo + lead no Kanban e na fila | T4.18 aceita | ☐ | D |
| T4.23 | Implementar a regra única de duplicidade (CNPJ, telefone, e-mail, nome com cidade), a verificação em tempo real no formulário, a tela Duplicidades e a mesclagem de registros | Dev full-stack | SPEC-4-002 | CA-4-05 | RED/GREEN: mesmo telefone em três formatos; principal: mesclar com notas e tentativas | capturas do aviso + ficha do registro principal depois da mesclagem | T4.22 aceita | ☐ | D |
| T4.24 | Criar as chaves de acesso (com o valor guardado só como resumo criptográfico), o registro de eventos de integração com mascaramento, o limite de chamadas e as rotas de criar, revogar e testar chave | Dev backend | SPEC-4-011 | CA-4-41 | RED/GREEN: chave revogada; limite: chamada acima do limite | respostas das rotas + registro de eventos mascarado | Fase 3 aceita | ☐ | D |
| T4.25 | Implementar a rota de captura de lead: validação, idempotência, origem e parâmetros de campanha, campos personalizados, tratamento de contato existente e de contato suprimido, resposta rápida com fila de exceção | Dev backend | SPEC-4-002 | CA-4-06 | Principal, repetida, sem chave e inválida, com fixtures/captura-fase4.json | tabela chamada × resposta + eventos registrados | T4.24 aceita; T4.23 aceita | ☐ | D |
| T4.26 | Criar o motor de importação: lote em rascunho, validação por linha com os normalizadores, prévia, gravação em blocos com retomada, importação parcial, detecção de arquivo repetido | Dev backend | SPEC-4-003 | CA-4-07 | RED/GREEN: linhas defeituosas conhecidas da fixture; falha: queda no meio da gravação | relatório do lote + contagem sem duplicatas depois da retomada | T4.23 aceita; T4.17 aceita | ☐ | E |
| T4.27 | Criar o assistente de importação em cinco passos: arquivo CSV ou XLSX e texto colado, linha do cabeçalho, mapeamento de colunas com sugestão e valores fixos, correspondência dos valores de catálogo e de etapa, prévia e relatório; histórico de importações | Dev frontend | SPEC-4-003 | CA-4-07 | Principal: arquivo e colar com o mesmo resultado; falha: arquivo sem cabeçalho | gravação do assistente + relatórios idênticos | T4.26 aceita | ☐ | E |
| T4.28 | Criar os perfis de mapeamento prontos (planilha de funil, base de prospecção, lista RespondeChat, leads Hiper, modelo padrão) com as conversões de etapa, fonte, tentativas com data e canal, observações e a regra NUTRIR; permitir salvar perfis | Dev full-stack | SPEC-4-003 | CA-4-08 | Principal: fixtures/leads-planilha-fase4.csv sem editar o arquivo | relatório do lote + três fichas conferidas | T4.27 aceita | ☐ | E |
| T4.29 | Implementar o passo de duplicidades em grupo, o arquivo de linhas rejeitadas com motivo, a definição do primeiro contato em lote e o desfazer do lote | Dev full-stack | SPEC-4-003 | CA-4-09 | Recuperação: desfazer lote com e sem leads já trabalhados | arquivo de rejeitadas + contagem antes e depois de desfazer | T4.28 aceita | ☐ | E |
| T4.30 | Acrescentar a entrada por Google Sheets com o conector do Skip (conta do próprio usuário) e a tela Minhas conexões; ocultar a opção quando o conector não estiver habilitado | Dev full-stack | SPEC-4-003 | CA-4-07 | Principal: planilha Google; limite: conector não habilitado | captura da importação + captura do estado sem conector | T4.27 aceita | ☐ | E |
| T4.31 | Criar as telas de importação de mídia e de conversas sobre as rotas existentes, com o mesmo assistente e os perfis de relatório do Meta Ads e do Google Ads; aceitar telefone para localizar o lead nas conversas | Dev full-stack | SPEC-4-003 | CA-4-09 | Principal: fixtures/midia-fase4.csv; limite: relatório com linhas de título e de total | relatórios dos lotes | T4.27 aceita | ☐ | E |
| T4.32 | Criar as tentativas de contato: coleção sem alteração nem exclusão, rotas, diálogo de registro com os desfechos por resultado e os contadores no lead | Dev full-stack | SPEC-4-005 | CA-4-16 | RED/GREEN: data futura e lead suprimido; principal: três cliques | gravação + contadores atualizados | T4.10 aceita; T4.14 aceita | ☐ | F |
| T4.33 | Implementar a régua de cadência (padrão e por origem, com versões) e o cálculo da proposta de próxima ação com dias úteis e janela de horário | Dev backend | SPEC-4-005 | CA-4-17 | RED/GREEN: fixtures/cadencias-fase4.csv (dia útil, janela, fim de semana, última tentativa) | saída dos testes | T4.32 aceita | ☐ | F |
| T4.34 | Criar a tela Minha fila com os blocos em ordem de prioridade, o prazo do primeiro contato, as ações no próprio item (conversar, registrar tentativa, adiar) e o seletor de fila para gestor e administrador | Dev full-stack | SPEC-4-005 | CA-4-18 | Principal: resolver um item de cada bloco; prova de cobertura: leads ativos × soma dos blocos | tabela com os dois números + gravação | T4.33 aceita; T4.11 aceita | ☐ | F |
| T4.35 | Implementar o fim da cadência (perguntar, encerrar, nutrir, tentar de novo), a nutrição com data de retorno e o retorno à fila com novo ciclo de tentativas | Dev full-stack | SPEC-4-005 | CA-4-19 | Limite: última tentativa; principal: retorno de nutrição na data | capturas do diálogo + lead de volta à fila | T4.34 aceita | ☐ | F |
| T4.36 | Criar o agendamento: coleção de eventos, formulário, validação dos campos obrigatórios no servidor, estados solicitado e confirmado, ajuste da etapa do lead e cartão do agendamento na ficha | Dev full-stack | SPEC-4-007 | CA-4-24 | RED/GREEN: agendamento sem confirmação não conta; falha: sem apresentador ou sem data | testes com fixtures/agendamentos-fase4.csv + capturas | T4.13 aceita; T4.7 aceita | ☐ | G |
| T4.37 | Implementar reagendar, cancelar, registrar falta e registrar que aconteceu, sem apagar eventos, com a contagem por série; e o bloco de resultado depois da apresentação | Dev full-stack | SPEC-4-007 | CA-4-25 | RED/GREEN: tentativa de apagar evento; principal: reagendar conta uma apresentação | lista de eventos do lead + indicador | T4.36 aceita | ☐ | G |
| T4.38 | Criar o briefing da apresentação (com o que cada papel pode ver e o registro de acesso) e o convite .ics com fuso e identificador estável | Dev full-stack | SPEC-4-007 | CA-4-26 | Principal: abrir como apresentador e como outro usuário; convite no Outlook e no Google Agenda; reagendar e reimportar | capturas dos dois briefings + dos dois calendários | T4.37 aceita | ☐ | G |
| T4.39 | Criar a tela Agenda nas visões semana e lista, com filtros, cores por situação, contadores e aviso de conflito de horário | Dev frontend | SPEC-4-007 | CA-4-24 | Principal: semana com compromissos em todas as situações; limite: conflito | capturas das duas visões | T4.37 aceita | ☐ | G |
| T4.40 | Criar a tela e as rotas de usuários: criar com senha temporária mostrada uma vez, troca obrigatória no primeiro acesso, editar, desativar com transferência de leads, redefinir senha; incluir o papel marketing na matriz de autorização e nas rotas | Dev full-stack | SPEC-4-010 | CA-4-38 | RED/GREEN: desativar o último administrador; prova por papel: menu e três endereços proibidos | tabela papel × rota × resultado + gravação do ciclo do usuário | T4.9 aceita | ☐ | H |
| T4.41 | Criar as telas de Catálogos, Cadência (com simulação e versões) e Modelos de mensagem (com variáveis, inclusive de campos personalizados, e pré-visualização), e as preferências gerais | Dev full-stack | SPEC-4-010 | CA-4-39 | Principal: alterar origem, régua e modelo pela tela e usar na operação; recuperação: restaurar régua | gravação + registros na auditoria | T4.33 aceita; T4.18 aceita | ☐ | H |
| T4.42 | Criar as telas de ICP por produto e de Base aprovada (rascunho, validar, publicar, versões, restaurar) sobre as rotas existentes, com os rascunhos iniciais de ICP | Dev frontend | SPEC-4-010 | CA-4-39 | Principal: publicar nova versão do ICP; limite: histórico não reclassificado | capturas + ficha antiga com a versão em que foi feita | T4.13 aceita | ☐ | H |
| T4.43 | Criar a rota e a tela de auditoria, reunindo os registros de segurança, de lead, de conversa, de configuração, de importação, de exportação e de integração, com filtros, mascaramento e exportação | Dev full-stack | SPEC-4-010 | CA-4-40 | Falha: tentativa de alterar ou apagar evento; principal: localizar alterações feitas nas tasks anteriores | capturas com os filtros + resposta negada | T4.40 aceita | ☐ | H |
| T4.44 | Criar os canais e o adaptador da Z-API em um único arquivo do backend: guardar credenciais só no servidor, conectar por QR code com renovação, situação, desconectar, reiniciar, registrar o endereço de eventos; e a tela Canais | Dev backend | SPEC-4-013 | CA-4-47 | Principal: conectar um número de teste; falha: credenciais não aparecem em rota nem no navegador; desconexão avisada | gravação da conexão + inspeção das respostas e do tráfego | T4.3 aceita; T4.24 aceita; T4.40 aceita | ☐ | I |
| T4.45 | Implementar o recebimento dos eventos da Z-API: validação do segredo e da instância, idempotência por identificador de mensagem, mensagens de texto e mídia (baixando os arquivos), mudança de estado, mensagens enviadas pelo celular, grupos ignorados, lead novo para número desconhecido e dados do anúncio de origem | Dev backend | SPEC-4-013 | CA-4-48, CA-4-49 | RED/GREEN: mesmo evento três vezes; principal e limite com fixtures/zapi-eventos-fase4.json | contagem de mensagens + lead criado com origem e anúncio | T4.44 aceita; T4.23 aceita | ☐ | I |
| T4.46 | Implementar a fila de envio e o envio de texto e mídia pela Z-API, com estados, novas tentativas, conferência de supressão no servidor, conferência de número com WhatsApp e mensagens em espera quando o canal estiver desconectado | Dev backend | SPEC-4-013 | CA-4-48, CA-4-50 | RED/GREEN: envio a contato suprimido; falha: canal desconectado; limite: servidor reiniciado no meio | registros da fila + negação auditada | T4.45 aceita; T4.14 aceita | ☐ | I |
| T4.47 | Criar a tela Conversas em três colunas com atualização em tempo real: lista com abas, situação, filtros, busca e não lidas; conversa com balões, estados, mídia e eventos do lead; campo de escrever com anexo; e usar o mesmo componente na aba Conversa da ficha | Dev frontend | SPEC-4-013 | CA-4-48 | Principal: trocar dez mensagens de cada tipo com um número de teste | gravação com os estados das mensagens | T4.46 aceita | ☐ | I |
| T4.48 | Completar o atendimento: painel do lead ao lado da conversa (editar, mover etapa, anotar, agendar), assumir, transferir, resolver e reabrir, notas internas na conversa, modelos de mensagem pelo campo de escrever, horário de atendimento e mensagem fora do horário | Dev full-stack | SPEC-4-013 | CA-4-49, CA-4-52 | Principal: atender uma conversa do início ao agendamento sem sair da tela | gravação + registros de dono e situação | T4.47 aceita; T4.41 aceita; T4.36 aceita | ☐ | I |
| T4.49 | Registrar automaticamente a tentativa na primeira mensagem do dia para um lead e atualizar o resultado quando o contato responde; mostrar a conversa não respondida como pendência na fila; trocar os botões de WhatsApp da ficha, da fila e do Kanban por Conversar | Dev backend | SPEC-4-013 | CA-4-50 | Principal: enviar e receber; limite: várias mensagens no mesmo dia geram uma tentativa | produtividade do dia sem digitação + fila com a pendência | T4.48 aceita; T4.34 aceita | ☐ | I |
| T4.50 | Criar a tela de bases de prospecção (registros do tipo prospect), com filtros, etiquetas, contadores e a promoção a lead, individual e em massa | Dev full-stack | SPEC-4-006 | CA-4-20 | Principal: promover; limite: prospect não aparece no Kanban nem na fila | capturas das três telas | T4.28 aceita | ☐ | J |
| T4.51 | Criar o construtor de listas: fontes, critérios com contagem, prévia e exclusões obrigatórias e configuráveis calculadas no servidor, remoção manual e limite de contatos | Dev full-stack | SPEC-4-006 | CA-4-21 | Prova com fixtures/base-prospeccao-fase4.csv: exclusões conhecidas | tabela de exclusões conferida com o esperado | T4.50 aceita; T4.32 aceita | ☐ | J |
| T4.52 | Implementar a exportação no formato de quatro colunas (configurável), o congelamento dos membros, o registro da exportação e as tentativas de origem lista, sem duplicar ao exportar de novo | Dev full-stack | SPEC-4-006 | CA-4-22 | Limite: exportar duas vezes; contato suprimido entre a prévia e a exportação | arquivo comparado com modelos/modelo-respondechat.csv + registros | T4.51 aceita | ☐ | J |
| T4.53 | Criar a aba Retornos: marcar em massa respondeu, sem interesse e número inválido, colar lista de telefones, supressão automática e conclusão da lista | Dev full-stack | SPEC-4-006 | CA-4-23 | Principal: sem interesse gera supressão e tira o contato de listas em rascunho | prévia de nova lista sem o contato | T4.52 aceita | ☐ | J |
| T4.54 | Implementar o envio de lista pela Central: escolha de canal, modelo, início, ritmo e limite diário; aprovação; processamento pelo agendador com intervalo aleatório; andamento em tempo real; pausar, retomar, cancelar; pausa automática; preenchimento automático dos retornos | Dev backend | SPEC-4-013 | CA-4-51 | Prova com 20 números de teste: intervalos no registro, pausa e retomada, pausa automática por recusa | horários de envio + gravação do andamento | T4.53 aceita; T4.46 aceita; T4.41 aceita | ☐ | J |
| T4.55 | Implementar o modo automático da régua: tentativa marcada como automática envia o modelo na data calculada, com as conferências antes de cada envio, uma tentativa por vez e cancelamento quando o contato responde; respeitar a chave do canal | Dev backend | SPEC-4-013 | CA-4-51 | RED/GREEN: chave do canal desligada; prova com régua de minutos em ambiente de teste | registro das tentativas, uma por vez + cancelamento ao responder | T4.49 aceita; T4.41 aceita | ☐ | J |
| T4.56 | Criar o motor de agentes no backend sobre o recurso de IA do Skip: montagem do contexto com mascaramento, regras fixas, capacidades como ferramentas conferidas no servidor, limites, tempo máximo, registro de cada execução e rota de situação; registrar o bloqueio se o recurso não estiver disponível | Dev backend | SPEC-4-008 | CA-4-28, CA-4-31 | RED/GREEN: capacidade desligada é bloqueada; teste do mascaramento com telefone, e-mail e CNPJ no texto; tentativa de instrução no texto do contato | saída dos testes + registros de bloqueio | T4.17 aceita; T4.7 aceita; T4.2 aceita | ☐ | K |
| T4.57 | Criar a lista e o editor de agentes: identidade, instruções com variáveis, roteiro de qualificação ligado a campos e ao ICP, capacidades, quando atua, limites e transbordo, modelo e custo; rascunho, versões, publicar, restaurar, pausar; modelos prontos de agente | Dev full-stack | SPEC-4-008 | CA-4-27 | Principal: criar um agente a partir de um modelo pronto; recuperação: restaurar versão | gravação | T4.56 aceita; T4.42 aceita | ☐ | K |
| T4.58 | Implementar o conhecimento do agente: seleção de grupos da base aprovada, textos de apoio colados ou enviados, divisão em trechos, consulta dos trechos relevantes e marcação de conteúdo sensível | Dev backend | SPEC-4-008 | CA-4-29 | Prova: cinco perguntas dentro e cinco fora do conhecimento | tabela pergunta × comportamento | T4.57 aceita | ☐ | K |
| T4.59 | Criar a área de teste (conversa simulada que mostra as ações que seriam feitas, sem enviar nem gravar) e os oito cenários prontos; condicionar a publicação à execução dos cenários | Dev full-stack | SPEC-4-008 | CA-4-27 | Prova com fixtures/agentes-cenarios-fase4.json; falha: publicar sem rodar os cenários | tabela cenário × resultado | T4.58 aceita | ☐ | K |
| T4.60 | Ligar os agentes às conversas: gatilhos e prioridade, espera para juntar mensagens, modo copiloto e modo autônomo, execução das ações, transbordos com motivo e resumo, pausa quando uma pessoa escreve, devolver à IA, limites; ativar as ações de etapa enviar modelo e acionar agente | Dev backend | SPEC-4-008 | CA-4-28, CA-4-29 | Prova: dez conversas com números de teste, do primeiro contato ao transbordo ou à qualificação; prioridade humana | gravação + registros de execução | T4.59 aceita; T4.48 aceita; T4.20 aceita | ☐ | K |
| T4.61 | Criar o painel Assistente na ficha e na conversa: resumir com referências, sugerir mensagem por objetivo, sugerir qualificação por critério como rascunho, e o registro do que foi feito com cada sugestão; alimentar o briefing | Dev full-stack | SPEC-4-008 | CA-4-30 | Principal: um uso de cada função; falha: IA desligada | capturas + registros de uso | T4.56 aceita; T4.13 aceita; T4.38 aceita | ☐ | K |
| T4.62 | Criar o painel de desempenho por agente e a lista de conversas para revisar; avisar e pausar ao atingir o limite de uso | Dev full-stack | SPEC-4-008 | CA-4-32 | Principal: indicadores depois das dez conversas de teste | captura do painel | T4.60 aceita | ☐ | K |
| T4.63 | Criar o painel de funil por categoria e por etapa, com filtros, modos por atividade e por safra, tabelas por origem e campanha, motivos, tempos, lacunas e o detalhamento que abre a lista de leads | Dev full-stack | SPEC-4-009 | CA-4-33 | Prova de reconciliação: cinco números × contagem da lista | tabela com os cinco pares | T4.37 aceita; T4.35 aceita; T4.16 aceita | ☐ | L |
| T4.64 | Criar o painel de produtividade diária e semanal por usuário, calculado das tentativas, mensagens e agendamentos, com exportação | Dev full-stack | SPEC-4-009 | CA-4-34 | Prova: conjunto conhecido de tentativas e mensagens | captura conferida com o esperado | T4.49 aceita; T4.37 aceita | ☐ | L |
| T4.65 | Criar o modelo de dados de anúncios (contas, árvore campanha–conjunto–anúncio, números diários) e a atribuição do lead ao anúncio pelos cinco sinais, gravada na criação; códigos de campanha e correspondência de nomes | Dev backend | SPEC-4-009 | CA-4-36 | RED/GREEN: um lead por sinal, com as fixtures da Z-API e da captura | tabela lead × sinal × linha | T4.45 aceita; T4.25 aceita; T4.31 aceita | ☐ | L |
| T4.66 | Implementar a sincronização do Meta Ads: conta e credencial no cofre, leitura diária dos últimos 7 dias e inicial de 90, paginação, eventos e erros na central de integrações, atualização sob demanda | Dev backend | SPEC-4-009 | CA-4-36 | Prova: três dias comparados com o painel da Meta; falha: token inválido | tabela plataforma × Central + cartão com o erro | T4.65 aceita; T4.3 aceita | ☐ | L |
| T4.67 | Implementar a entrada do Google Ads: perfil de relatório importado gravando nos números diários e, com o token aprovado ou o conector habilitado, a sincronização diária | Dev backend | SPEC-4-009 | CA-4-36 | Prova: relatório de um mês conferido com o painel do Google; limite: mesmo dia por sincronização e por relatório | tabela plataforma × Central + aviso de sobreposição | T4.65 aceita | ☐ | L |
| T4.68 | Criar o dashboard de anúncios: números do topo com comparação, gráfico de evolução, tabela em três níveis com colunas escolhidas, qualidade por anúncio, linhas de sem anúncio e origem desconhecida, lançamento manual, recomendações explicáveis com decisão registrada | Dev full-stack | SPEC-4-009 | CA-4-35 | Prova dos três níveis: soma dos filhos igual ao pai; CPLQ com fixtures/midia-fase4.csv | cálculo à mão × tela | T4.66 aceita; T4.67 aceita; T4.63 aceita | ☐ | L |
| T4.69 | Atualizar o feedback para o Marketing com dado real e a devolutiva do mês, criar o relatório do parceiro com exportação e o cadastro de metas | Dev full-stack | SPEC-4-009 | CA-4-37 | Prova de privacidade: percorrer como marketing; relatório do parceiro sem dados de outras origens | capturas + arquivo exportado | T4.68 aceita; T4.40 aceita | ☐ | L |
| T4.70 | Criar a central de integrações: painel de situação com um cartão por integração (captura, WhatsApp, Meta Ads, Google Ads, Google Sheets, agenda, IA), tela de chaves, registro de eventos e fila de exceção com corrigir, reprocessar e descartar | Dev full-stack | SPEC-4-011 | CA-4-41, CA-4-42 | Prova: três falhas diferentes — corrigir uma, reprocessar uma, descartar uma; operação com tudo desligado | capturas + gravação da operação sem integrações | T4.44 aceita; T4.66 aceita; T4.25 aceita; T4.56 aceita | ☐ | M |
| T4.71 | Implementar a rota genérica de recebimento de mensagens para outros sistemas, com idempotência e as regras de lead novo, supressão e recusa | Dev backend | SPEC-4-011 | CA-4-42 | Prova: executar os exemplos de integracoes.md contra o ambiente | tabela chamada × resposta | T4.24 aceita; T4.45 aceita | ☐ | M |
| T4.72 | Fazer a cópia de segurança completa, testar a restauração em ambiente separado e remover os dados de teste das fases 1 a 4, preservando catálogos, funis, campos, régua, modelos, ICP, agentes e usuários reais | Dev backend | SPEC-4-014 | CA-4-55 | Prova: contagem zero nas coleções operacionais; restauração testada | relatório de contagens + registro da restauração | T4.70, T4.69, T4.62, T4.55, T4.54, T4.21, T4.43, T4.39, T4.29, T4.30, T4.71, T4.64, T4.61 aceitas | ☐ | N |
| T4.73 | Executar a carga real na ordem de importacao-de-dados.md: planilha de funil, leads do parceiro, bases de prospecção, suprimidos, anúncios; depois conectar os números de WhatsApp e as contas de anúncio; conciliar e arquivar a planilha | Pontual (Lidiane), com a consultora | SPEC-4-014 | CA-4-55 | Manual: os passos da Parte 3 da SPEC-4-014 | relatório de conciliação + 20 fichas conferidas | T4.72 aceita; T4.2 aceita; T4.1 aceita | ☐ | N |
| T4.74 | Reexecutar os critérios das fases 1 a 3 e executar o roteiro ponta a ponta com dados reais, sem ajuda; corrigir defeitos e repetir os passos afetados | QA + Pontual (Lidiane e Marcelo) | SPEC-4-014 | CA-4-56 | Regressão integral + roteiro da Parte 5 | tabela de critérios + gravação da sessão + tabela dos passos | T4.73 aceita | ☐ | N |
| T4.75 | Entregar o guia rápido, o guia do administrador (com ajuda dentro do sistema) e o roteiro de operação técnica; fazer e gravar os treinamentos; acompanhar a primeira semana de uso | Consultora | SPEC-4-014 | CA-4-57 | Manual: Parte 6 da SPEC-4-014 | guias + gravações + registro diário da primeira semana | T4.74 aceita | ☐ | N |
| T4.76 | Emitir o recibo final da fase com evidência por critério, situação dos gates, conciliação, números da primeira semana e lista de evoluções; colher a decisão de entrada em operação e o aceite; arquivar a fase | Consultora + Pontual (Marcelo) | SPEC-4-014 | CA-4-57 | Manual: Parte 7 da SPEC-4-014 | recibo-fase-4.md assinado | T4.75 aceita | ☐ | N |

## Levas

- **A (Fundação: segurança, decisões, contratações e modelo de dados):** T4.1, T4.2, T4.3, T4.4, T4.5, T4.6, T4.7, T4.8
- **B (Ficha do lead, Kanban e lista):** T4.9, T4.10, T4.11, T4.12, T4.13, T4.14, T4.15, T4.16
- **C (Personalização: campos, funis, automações e layout):** T4.17, T4.18, T4.19, T4.20, T4.21
- **D (Entrada de leads e base das integrações):** T4.22, T4.23, T4.24, T4.25
- **E (Importação de dados):** T4.26, T4.27, T4.28, T4.29, T4.30, T4.31
- **F (Fila do dia e cadência):** T4.32, T4.33, T4.34, T4.35
- **G (Agendamento e briefing):** T4.36, T4.37, T4.38, T4.39
- **H (Administração):** T4.40, T4.41, T4.42, T4.43
- **I (Conversas pelo WhatsApp):** T4.44, T4.45, T4.46, T4.47, T4.48, T4.49
- **J (Prospecção, listas e envios):** T4.50, T4.51, T4.52, T4.53, T4.54, T4.55
- **K (Agentes de IA):** T4.56, T4.57, T4.58, T4.59, T4.60, T4.61, T4.62
- **L (Indicadores e dashboard de anúncios):** T4.63, T4.64, T4.65, T4.66, T4.67, T4.68, T4.69
- **M (Central de integrações):** T4.70, T4.71
- **N (Entrada em operação e aceite):** T4.72, T4.73, T4.74, T4.75, T4.76
