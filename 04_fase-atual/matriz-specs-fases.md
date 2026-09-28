# Matriz SPECs–Fase 2–Tasks

| ID | Task | Leva | SPEC | CAs | Dono | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| T2.1 | Criar fixture de mídia sintética e validador de esquema | A | SPEC-2-001 | CA-2-01 | Dev dados | CA-2-01: esquema inválido e fórmula perigosa falham sem persistência parcial | RED/GREEN importação inválida | fixture + testes de atomicidade | Fase 1 aceita | ☐ |
| T2.2 | Implementar importação idempotente de mídia | B | SPEC-2-001 | CA-2-01 | Dev dados | CA-2-01: importação atômica, idempotente e relatório sanitizado | Principal/falha importação | log de lote + relatório sanitizado | T2.1 aceita | ☐ |
| T2.3 | Implementar cruzamento origem/campanha com o funil | C | SPEC-2-002 | CA-2-02 | Dev full-stack | CA-2-02: cruzamento com `desconhecida` explícita | RED/GREEN cruzamento | testes de correspondência e desconhecida | T2.2 aceita | ☐ |
| T2.4 | Implementar cards e filtros do painel de mídia | D | SPEC-2-002 | CA-2-02/03 | Dev frontend | CA-2-02/03: painel com filtros por período/canal | GREEN fluxo principal | capturas + testes de reconciliação | T2.3 aceita | ☐ |
| T2.5 | Implementar cálculo de CPLQ reconciliável | E | SPEC-2-002 | CA-2-03 | Dev full-stack | CA-2-03: CPLQ reconcilia com lotes e funil | RED/GREEN reconciliação | testes com contagem esperada | T2.4 aceita | ☐ |
| T2.6 | Implementar recomendação explicável com aprovação humana | F | SPEC-2-003 | CA-2-04 | Dev full-stack | CA-2-04: 4 campos + decisão humana auditada; sem ação autônoma | Limite/falha de recomendação | testes de decisão + auditoria | T2.5 aceita | ☐ |
| T2.7 | Implementar visão de feedback Marketing–Comercial | F | SPEC-2-004 | CA-2-05 | Dev full-stack | CA-2-05: motivos por campanha + export sanitizado auditado | GREEN fluxo principal | captura + export sanitizado | T2.5 aceita | ☐ |
| T2.8 | Demonstrar painel de mídia e emitir recibo final da Fase 2 | G | SPEC-2-003/004 | CA-2-01..06 | QA | CA-2-01..06 completos + degradação segura (CA-2-06) | Regressão integral | recibo CA-2-01..06 + relatório de gates G3/G6 | T2.6 e T2.7 aceitas | ☐ |
