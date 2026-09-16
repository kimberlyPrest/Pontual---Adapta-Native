# Fase 1 — Fundação operacional e baseline

## Resultado
Uma central utilizável com login, cadastro/importação controlada, Kanban, fila, ficha de qualificação, timeline e dashboard básico, operada com dados sintéticos/amostra anonimizada e sem depender de APIs externas.

## Inclui
RF-01–08, RF-18, RF-22 e preparação de RF-23/24. Definição provisória e versionada de estágios/motivos; relatório das lacunas G1–G7.

## Demonstração
Importar lote de teste, conciliar duplicidade, receber lead na fila, qualificar manualmente, mover no Kanban, registrar próxima ação e visualizar métricas e timeline.

## Critérios de aceite
- **CA-1-01:** cada papel enxerga apenas ações e dados autorizados em teste positivo e negativo.
- **CA-1-02:** importação inválida é rejeitada sem persistência parcial e gera relatório sanitizado.
- **CA-1-03:** possível duplicidade é sinalizada e a decisão preserva origem e histórico.
- **CA-1-04:** lead percorre novo→triagem→em contato→qualificado/desqualificado/nutrição com regras e motivos.
- **CA-1-05:** todo lead ativo sem próxima ação aparece na fila de exceção.
- **CA-1-06:** timeline registra autor, data e antes/depois de mudança crítica.
- **CA-1-07:** dashboard reconcilia exatamente com o lote demonstrado e separa origem desconhecida.
- **CA-1-08:** baseline é marcado como provisório/inconclusivo até aprovação de G2/G3.
- **CA-1-09:** operação essencial continua sem IA e sem integrações externas.

## Controles transversais de segurança

- Fase 1 usa somente fixtures sintéticas; dados reais exigem gate LGPD com base legal, retenção e classificação de PII.
- Evidências mascaram e-mail, telefone, IDs e qualquer segredo; tokens nunca aparecem em captura, log ou exportação.
- Importação é sempre atômica nesta fase; não existe modo parcial.
- Exportações usam allowlist de campos, autorização por papel e neutralização de `=`, `+`, `-`, `@`, tab e CR.
- Auditoria é append-only na aplicação, inclusive para administrador.

## Fora desta fase

Integrações Google/Meta Ads, canal WhatsApp real, IA respondente, follow-up automático, reativação e agenda integrada pertencem às Fases 2–4 e dependem de G1–G7.

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T1.1 | Criar contas fixture e matriz de autorização server-side | Dev full-stack | SPEC-1-001 | CA-1-01/01a/01b: autorização, sessão, anti-brute-force e contingência governada ou ausente | RED/GREEN acesso permitido | testes de autorização por papel | checks aprovados | ☐ | A |
| T1.2 | Implementar negações auditadas, sessão e exportação protegida | Dev full-stack | SPEC-1-001 | CA-1-01/01a/01b: negação sem vazamento, sessão e auditoria de segurança | Falha/regressão de autorização | log sanitizado de negação e testes | T1.1 aceita | ☐ | B |
| T1.3 | Demonstrar acesso e entregar recibo CA-1-01 | QA | SPEC-1-001 | CA-1-01/01a/01b completos | Regressão integral SPEC-1-001 | recibo CA-1-01 | T1.2 aceita | ☐ | C |
| T1.4 | Criar fixture CSV sintética e validador de esquema seguro | Dev dados | SPEC-1-002 | CA-1-02/02a: schema inválido e fórmula perigosa falham sem persistência parcial | RED/GREEN importação inválida | fixture + testes atomicidade | checks aprovados | ☐ | A |
| T1.5 | Implementar importação idempotente e relatório sanitizado | Dev dados | SPEC-1-002 | CA-1-02/02a: importação atômica, valor não executável e relatório sanitizado | Principal/falha importação | log lote e relatório sanitizado | T1.4 aceita | ☐ | B |
| T1.6 | Implementar detecção e decisão de possível duplicidade | Dev full-stack | SPEC-1-002 | CA-1-03: conflito preserva origem e histórico | Limite/repetição duplicidade | testes lote repetido e conflito | T1.5 aceita | ☐ | C |
| T1.7 | Demonstrar importação, reversão e reconciliação | QA | SPEC-1-002 | CA-1-02/02a/02b e CA-1-03 completos | Regressão integral SPEC-1-002 | recibo CA-1-02/03 | T1.6 aceita | ☐ | D |
| T1.8 | Implementar Kanban com estágios provisórios versionados | Dev frontend | SPEC-1-003 | CA-1-04: lead percorre somente transições válidas | GREEN fluxo principal | capturas e testes de transição | checks aprovados | ☐ | A |
| T1.9 | Implementar ficha de qualificação e motivos obrigatórios | Dev full-stack | SPEC-1-003 | CA-1-04: encerramento exige critérios, motivo e evidência | Limite/falha de qualificação | testes de validação por resultado | T1.8 aceita | ☐ | B |
| T1.10 | Demonstrar três desfechos e regressão do Kanban | QA | SPEC-1-003 | CA-1-04 completo | Regressão integral SPEC-1-003 | recibo CA-1-04 | T1.9 aceita | ☐ | C |
| T1.11 | Implementar fila determinística e exceção sem próxima ação | Dev full-stack | SPEC-1-004 | CA-1-05: lead ativo sem ação aparece na fila | RED/GREEN cobertura de ação | testes da fila por condição | checks aprovados | ☐ | A |
| T1.12 | Implementar timeline imutável das mudanças críticas | Dev backend | SPEC-1-004 | CA-1-06: timeline append-only inclusive contra administrador | RED/GREEN auditoria crítica e negação de delete/update | testes e log sanitizado | T1.11 aceita | ☐ | B |
| T1.13 | Demonstrar fila, próxima ação e timeline | QA | SPEC-1-004 | CA-1-05 e CA-1-06 completos | Regressão integral SPEC-1-004 | recibo CA-1-05/06 | T1.12 aceita | ☐ | C |
| T1.14 | Implementar consultas e cards do dashboard reconciliável | Dev full-stack | SPEC-1-005 | CA-1-07: totais/filtros conferem com lote | RED/GREEN reconciliação | testes com contagem esperada | T1.7 aceita | ☐ | E |
| T1.15 | Implementar estado provisório e exportação segura | Dev full-stack | SPEC-1-005 | CA-1-07a e CA-1-08: exportação autorizada/sanitizada e baseline inconclusivo | Limite baseline inconclusivo | captura de aviso + teste CSV | T1.14 aceita | ☐ | F |
| T1.16 | Provar operação degradada sem IA e APIs | QA | SPEC-1-005 | CA-1-09: operação essencial permanece funcional | Falha/degradação | roteiro gravado e logs | T1.15 aceita | ☐ | G |
| T1.17 | Demonstrar dashboard e emitir recibo final da Fase 1 | QA | SPEC-1-005 | CA-1-07, CA-1-08 e CA-1-09 completos | Regressão integral SPEC-1-005 | recibo CA-1-07/07a/08/09 + relatório das lacunas G1–G7 | T1.16 aceita | ☐ | H |
