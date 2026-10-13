# Decisões da Pontual para a entrada em operação

**Projeto:** Pontual Tecnologia — Central de Aquisição e Qualificação de Leads  
**Como usar:** preencher na reunião de decisões (T4.2), com o Marcelo e a Lidiane. Cada decisão tem um valor provisório, que é o que o sistema usa enquanto a linha estiver em branco. Decisão em aberto não trava a fase: fica com dono e data, e vai para a lista de evoluções.

**Data da reunião:** ____/____/______  
**Presentes:** ______________________________________________  
**Registrado por:** __________________________________________

> Este arquivo é de autoria da Pontual e da consultora. Nenhuma linha deve ser preenchida por dedução.

---

## 1. Dados e operação

| # | Decisão | Valor provisório | Decisão da Pontual | Quem aplica, onde | Gate |
|---|---|---|---|---|---|
| 1.1 | A Central passa a ser a fonte oficial dos leads comerciais, e a planilha "CRM - Acompanhamento" é arquivada? Em que data? | planilha continua até a carga | | Marcelo — arquivar a planilha | G1 |
| 1.2 | Autorização para carregar dados reais. Quais bases entram, de onde vieram, quem tem acesso | não autorizado: só dados fictícios | | Marcelo — termo assinado, anexado a este arquivo | G5 |
| 1.3 | Por quanto tempo manter prospects sem resposta e leads desqualificados | 24 meses, revisão anual | | registrado aqui | G5 |
| 1.4 | Usuários do sistema e o papel de cada um; quem é administrador além do Marcelo | Marcelo (admin), Lidiane (SDR) | | Configurações → Usuários | — |
| 1.5 | Nome do sistema, exibido no login e no cabeçalho | o atual | | Configurações → Geral | — |

## 2. Funil, campos e qualificação

| # | Decisão | Valor provisório | Decisão da Pontual | Quem aplica, onde | Gate |
|---|---|---|---|---|---|
| 2.1 | Etapas do funil comercial e os nomes | Novo, Triagem, Em contato, Qualificado, Agendamento solicitado, Agendado, Nutrição, Desqualificado | | Configurações → Funis e etapas | G2 |
| 2.2 | O funil continua depois do agendamento (Proposta enviada, Negociação, Fechado – ganho)? | não; só anotação do resultado | | idem | G2 |
| 2.3 | Funis separados por produto (Hiper, Automec, Certificado)? | um funil só | | idem | G2 |
| 2.4 | Quais colunas da planilha viram campo do lead; quais são obrigatórios e em que etapa | os campos da SPEC-4-001 | | Configurações → Campos | G2 |
| 2.5 | O que aparece no card do Kanban | empresa, contato, origem, produto, próxima ação, última nota | | Configurações → Ficha e card | — |
| 2.6 | Origens, produtos, segmentos e motivos | os dados iniciais da SPEC-4-001 | | Configurações → Catálogos | G2 |
| 2.7 | ICP de cada produto: critérios, quais são obrigatórios | rascunho com segmento, regime tributário, situação da loja, usuários, sistema atual | | Configurações → ICP — **publicar** | G2 |
| 2.8 | Ações automáticas por etapa (ex.: ao qualificar, criar a tarefa de propor horário) | nenhuma | | Configurações → Funis e etapas | — |

## 3. Rotina e agendamento

| # | Decisão | Valor provisório | Decisão da Pontual | Quem aplica, onde | Gate |
|---|---|---|---|---|---|
| 3.1 | Régua de follow-up: quantas tentativas, intervalos, canais; por origem | 3 tentativas; 1, 2 e 4 dias úteis; WhatsApp, ligação, WhatsApp | | Configurações → Cadência | G4 |
| 3.2 | Prazo para o primeiro contato com lead novo | 2 horas úteis (indicação: 1 hora) | | idem | G4 |
| 3.3 | Quem apresenta cada produto; duração; online ou presencial; quem confirma | Lidiane; 60 minutos; online; quem agendou | | Configurações → Catálogos (produto) | G7 |
| 3.4 | Metas do mês: qualificados, apresentações agendadas, apresentações realizadas, custo máximo por qualificado. Referências do escopo: 20, 10 e R$ 200 — ou 25, 8 e R$ 160 | sem meta: o painel mostra só o realizado | | Configurações → Metas | G3 |
| 3.5 | O evento que conta como resultado do projeto | apresentação agendada **e confirmada**; a realizada é medida à parte | | registrado aqui | G3 |

## 4. WhatsApp

| # | Decisão | Valor provisório | Decisão da Pontual | Quem aplica, onde | Gate |
|---|---|---|---|---|---|
| 4.1 | Contratar a Z-API. Quem contrata e quem guarda as credenciais | não contratado | | Marcelo | G6 |
| 4.2 | Quais números conectar (Comercial, Comunicação) | nenhum | | Configurações → Canais | G6 |
| 4.3 | Usar o número atual ou um novo. Número separado para disparo frio? (recomendado) | — | | idem | G6 |
| 4.4 | Quem atende cada número; quem vê todas as conversas | Lidiane | | idem | — |
| 4.5 | Data em que o comercial deixa de usar o RespondeChat (ou se fica só para o Financeiro) | em paralelo até decidir | | registrado aqui | G1 |
| 4.6 | Horário de atendimento e mensagem fora do horário | 9h às 18h, sem mensagem automática | | Configurações → Canais | — |

## 5. Envio automático — tudo nasce desligado

| # | Decisão | Valor provisório | Decisão da Pontual | Quem aplica, onde | Gate |
|---|---|---|---|---|---|
| 5.1 | A régua pode enviar mensagens sozinha? Em quais tentativas, com quais modelos | não; tudo manual | | Configurações → Canais e Cadência | G5 |
| 5.2 | Envio de listas pela Central: permitido? Em qual número, com qual limite diário e ritmo | não; exportação em arquivo | | Configurações → Canais | G5 |
| 5.3 | Política de contato para base fria: quais bases podem receber; intervalo mínimo entre envios ao mesmo contato; o que conta como recusa | 30 dias de intervalo; "sair", "parar", "não tenho interesse", "remover" | | Configurações → Geral | G5 |
| 5.4 | Resposta para "onde você conseguiu meu contato?" | sem resposta aprovada: transbordo para uma pessoa | | Configurações → Base aprovada | G5 |
| 5.5 | Quem aprova listas acima de 50 contatos | Marcelo | | papéis | — |

## 6. Agentes de IA

| # | Decisão | Valor provisório | Decisão da Pontual | Quem aplica, onde | Gate |
|---|---|---|---|---|---|
| 6.1 | Autoriza enviar as conversas e o histórico do lead ao modelo de IA (sem telefone, e-mail e CNPJ)? | não: agentes e copiloto desligados | | Configurações → Agentes de IA | G5 |
| 6.2 | Quais agentes entram: triagem de anúncio, qualificação por produto, agendamento, follow-up, reativação | só o copiloto interno | | idem | — |
| 6.3 | Quais ficam em copiloto (sugerem) e quais em autônomo (enviam) | todos em copiloto | | idem, e a chave do canal | G5 |
| 6.4 | Um agente pode mover o lead de etapa? Para quais etapas | não | | idem | — |
| 6.5 | Um agente pode registrar sozinho a decisão de qualificado ou desqualificado? | não: a decisão é de uma pessoa | | idem | — |
| 6.6 | Um agente pode propor horários e registrar o pedido de apresentação? | não | | idem | G7 |
| 6.7 | Respostas aprovadas (base de conhecimento): quem escreve e quem revisa. Há resposta aprovada para preço? | base vazia: o agente não fala de produto nem de preço | | Configurações → Base aprovada | — |
| 6.8 | Modelos de mensagem: quem reescreve os textos iniciais | textos neutros de fábrica | | Configurações → Modelos de mensagem | — |

## 7. Anúncios e captura

| # | Decisão | Valor provisório | Decisão da Pontual | Quem aplica, onde | Gate |
|---|---|---|---|---|---|
| 7.1 | Conectar a conta do Meta Ads. Quem tem o acesso de administrador; quem na agência apoia | relatório importado | | Configurações → Integrações | G6 |
| 7.2 | Google Ads: pedir o token de desenvolvedor (prazo do Google) e, enquanto isso, importar relatórios | relatório importado | | idem | G6 |
| 7.3 | Padrão de parâmetros de campanha e de códigos nos anúncios (`integracoes.md`, seção 7.4). Quem ajusta os anúncios | sem padrão: leads caem em "Sem anúncio identificado" | | agência | G6 |
| 7.4 | De onde os leads vão chegar sozinhos: formulário do site, anúncios com WhatsApp. Quem mantém o site | só WhatsApp e cadastro manual | | quem mantém o site | G6 |
| 7.5 | Quem, do Marketing ou da agência, recebe acesso com o papel `marketing` | ninguém | | Configurações → Usuários | — |

---

## Pendências que saem da reunião

| # | Decisão em aberto | Dono | Data | Valor provisório em uso |
|---|---|---|---|---|
| | | | | |

## Assinaturas

| Nome | Papel | Data | Assinatura |
|---|---|---|---|
| Marcelo | Champion | | |
| Lidiane | Comercial | | |
| Kim Prestes | Consultora | | |
