# Requisitos Consolidados — Pontual Tecnologia

**Data:** 2026-08-28  
**Status:** preliminar — sujeito às decisões humanas e ao check-input  
**Limite do produto:** termina quando a apresentação comercial é agendada e registrada.

## Objetivo e atores
Permitir que Pontual, agência/Marketing, SDR e gestão enxerguem e operem a jornada de cada lead desde a origem até o agendamento, com qualificação rastreável e feedback de qualidade por canal.

## Requisitos

### RQ-001 — Registro único de lead/oportunidade
Localizar ou criar registro sem duplicar empresa/contato. Não inclui cliente 360º pós-venda.

### RQ-002 — Origem e campanha rastreáveis
Toda entrada possui canal; campanha/conjunto/anúncio quando disponíveis. Origem desconhecida permanece explícita.

### RQ-003 — Fila de triagem e SLA
Novos leads aparecem em fila priorizada; atraso é sinalizado. O SLA de primeiro contato ainda precisa ser definido.

### RQ-004 — Qualificação estruturada e humana
Registrar critérios de ICP por produto, contexto, aderência e pendências. O sistema não decide sozinho que o lead é qualificado.

### RQ-005 — Estados e motivos padronizados
Novo, em triagem, em contato, qualificado, desqualificado, nutrição, agendamento solicitado e apresentação agendada; descarte/pausa exigem motivo.

### RQ-006 — Cadência e próxima ação
Registrar tentativas, canal, resultado, data e próxima ação. Resolver contradição entre 3 e 7 tentativas.

### RQ-007 — Recusa e supressão de contato
Recusa explícita impede novas ações elegíveis e mantém trilha de auditoria. Depende de política LGPD, retenção e reativação autorizada.

### RQ-008 — Agendamento da apresentação
Registrar data, hora, responsável, participantes, produto e contexto. Reagendamento/cancelamento/no-show precisam definição.

### RQ-009 — Painel do funil até agendamento
Entradas, qualificados, desqualificados, motivos, agendamentos e tempos por canal/período.

### RQ-010 — Feedback Marketing–Comercial
Visão periódica da qualidade e dos motivos de desqualificação por campanha/origem.

### RQ-011 — Importação e conciliação inicial
Importar amostra/dados homologados, sinalizar duplicidades e preservar origem. Migração integral só após prova.

### RQ-012 — Controle de acesso e auditoria
Acesso por papel; parceiros enxergam apenas sua origem; mudanças críticas são auditáveis.

## Sinais de sucesso — aguardando homologação
- Briefing: ≥20 qualificados/mês, ≥10 apresentações **realizadas**/mês, CPLQ ≤R$200.
- Mapa de processos: 25 qualificados/mês, 8 apresentações/mês, CPLQ R$160.
- Solicitação da consultora: sucesso até o **agendamento**.

Até decisão humana, nenhum desses números deve ser usado como gate automático.

## Fora de escopo
- realização e resultado da apresentação;
- proposta, CPQ, desconto, assinatura e negociação;
- faturamento, implantação, CS e comissões;
- produto financeiro/FP&A;
- compra/otimização autônoma de mídia;
- disparo autônomo por IA;
- substituição total de CRM/WhatsApp sem decisão específica.

## Decisões pendentes
1. Fonte oficial operacional.
2. Meta numérica e evento final.
3. ICP por produto.
4. Cadência e exceções.
5. Canais/integrações e acessos.
6. Calendário e reagendamento.
7. Política de contato, opt-out, retenção e parceiros.
8. Baseline por canal.