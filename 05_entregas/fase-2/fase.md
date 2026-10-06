# Fase 2 — Inteligência de mídia

## Resultado

Gestão e Marketing enxergam, num único painel, investimento, entradas, qualificados, agendamentos e CPLQ por campanha/origem, com recomendações explicáveis sujeitas à aprovação humana — sem que o sistema altere campanhas autonomamente.

## Inclui

RF-19 (dashboard de mídia), RF-20 (recomendações explicáveis), RF-21 (feedback Marketing–Comercial), RN-02 (origem desconhecida explícita), RN-11 (recomendação com dado, período, hipótese e confiança), RN-12 (metas só após baseline). Preparação dos gates G3 (baseline) e G6 (APIs Google/Meta Ads).

## Demonstração

Importar lote de mídia sintético, consolidar com o funil da Fase 1, exibir CPLQ por campanha, gerar uma recomendação explicável a partir de dados reais do painel, registrar decisão humana (aprovar/rejeitar com motivo) e emitir a visão de feedback Marketing–Comercial.

## Critérios de aceite

- **CA-2-01:** importação de relatório de mídia (CSV/planilha) é validada, idempotente e rejeita lote inválido sem persistência parcial.
- **CA-2-02:** todo lead do funil cruza com a mídia por origem/campanha; origem sem correspondência permanece `desconhecida` e aparece separada no painel (nunca inferida).
- **CA-2-03:** CPLQ por campanha/origem reconcilia exatamente com os lotes importados e com o funil (numerador = qualificados; denominador = investimento do período).
- **CA-2-04:** recomendação exibe dado, período, hipótese e confiança; aprovação humana registrada antecede qualquer ação; o sistema nunca altera campanha por conta própria.
- **CA-2-05:** feedback Marketing–Comercial mostra qualidade por campanha (motivos de desqualificação) e pode ser exportado de forma sanitizada.
- **CA-2-06:** operação essencial do funil (Fase 1) permanece funcional se a camada de mídia falhar ou estiver vazia.

## Controles transversais de segurança

- Dados de mídia são agregados por campanha/origem; nenhum PII de lead entra no painel de mídia.
- Tokens de API (Google/Meta Ads) ficam em segredo gerenciado, nunca em código, log ou exportação; a Fase 2 começa com importação manual e só ativa API após G6.
- Importação de mídia é atômica e idempotente por lote/período; reversão antes da consolidação.
- Exportações seguem allowlist de campos, autorização por papel e neutralização de `=`, `+`, `-`, `@`, tab e CR.
- Auditoria append-only para importações, decisões de recomendação e exportações.

## Fora desta fase

Canal conversacional real (Fase 3), IA respondente (Fase 3), follow-up automático (Fase 4), reativação (Fase 4), agendamento integrado (Fase 4), metas numéricas congeladas (Fase 5, após baseline). A API de Google/Meta Ads só entra após G6 provado; até lá, importação manual de relatórios.

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T2.1 | Criar fixture de mídia sintética e validador de esquema | Dev dados | SPEC-2-001 | CA-2-01 | RED/GREEN importação inválida | fixture + testes de atomicidade | Fase 1 aceita | ☐ | A |
| T2.2 | Implementar importação idempotente de mídia | Dev dados | SPEC-2-001 | CA-2-01 | Principal/falha importação | log de lote + relatório sanitizado | T2.1 aceita | ☐ | B |
| T2.3 | Implementar cruzamento origem/campanha com o funil | Dev full-stack | SPEC-2-002 | CA-2-02 | RED/GREEN cruzamento | testes de correspondência e desconhecida | T2.2 aceita | ☐ | C |
| T2.4 | Implementar cards e filtros do painel de mídia | Dev frontend | SPEC-2-002 | CA-2-02/03 | GREEN fluxo principal | capturas + testes de reconciliação | T2.3 aceita | ☐ | D |
| T2.5 | Implementar cálculo de CPLQ reconciliável | Dev full-stack | SPEC-2-002 | CA-2-03 | RED/GREEN reconciliação | testes com contagem esperada | T2.4 aceita | ☐ | E |
| T2.6 | Implementar recomendação explicável com aprovação humana | Dev full-stack | SPEC-2-003 | CA-2-04 | Limite/falha de recomendação | testes de decisão + auditoria | T2.5 aceita | ☐ | F |
| T2.7 | Implementar visão de feedback Marketing–Comercial | Dev full-stack | SPEC-2-004 | CA-2-05 | GREEN fluxo principal | captura + export sanitizado | T2.5 aceita | ☐ | F |
| T2.8 | Demonstrar painel de mídia e emitir recibo final da Fase 2 | QA | SPEC-2-003/004 | CA-2-01..06 | Regressão integral | recibo CA-2-01..06 + relatório de gates G3/G6 | T2.6 e T2.7 aceitas | ☐ | G |
