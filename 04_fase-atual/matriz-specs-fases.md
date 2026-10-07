# Matriz de SPECs e Fases — Fase 4

| SPEC | Resultado observável | RFs | RNs | RNFs | CAs | Gates | Tasks |
|---|---|---|---|---|---|---|---|
| SPEC-4-001 | Régua de follow-up versionada gera próxima ação por inatividade respeitando intervalo, horário, limite e exceções | RF-13 | RN-05, RN-08, RN-13 | RNF-08 | CA-4-01 | G4 | T4.1, T4.2, T4.5, T4.8 |
| SPEC-4-002 | Segmento de reativação com critérios, tamanho, exclusões obrigatórias e prévia sanitizada | RF-14, RF-15 | RN-07 | RNF-02 | CA-4-02, CA-4-03 | G5 | T4.3, T4.4, T4.6, T4.7 |
| SPEC-4-003 | Disparo governado: aprovação humana, canal autorizado, idempotência e bloqueio sem gates | RF-13, RF-14 | RN-03, RN-07, RN-13, RN-14 | RNF-02, RNF-05 | CA-4-04 | G5, G6 | T4.9, T4.10, T4.11, T4.12 |
| SPEC-4-004 | Agendamento válido com estados, confirmação e histórico append-only | RF-16 | RN-01, RN-09, RN-10 | RNF-02 | CA-4-05, CA-4-06 | G7 | T4.13, T4.14, T4.15 |
| SPEC-4-005 | Handoff estruturado mascarado ao apresentador + recibo final da fase | RF-17 | RN-01 | RNF-02, RNF-04 | CA-4-07 | — | T4.16, T4.17 |

## Cobertura de requisitos da fase

| Requisito | SPEC dona | CA | Task(s) |
|---|---|---|---|
| RF-13 Follow-up | SPEC-4-001, SPEC-4-003 | CA-4-01, CA-4-04 | T4.1, T4.2, T4.5, T4.8, T4.9–T4.12 |
| RF-14 Reativação segmentada | SPEC-4-002, SPEC-4-003 | CA-4-03, CA-4-04 | T4.3, T4.4, T4.6, T4.7, T4.9–T4.12 |
| RF-15 Supressão | SPEC-4-002 | CA-4-02 | T4.3 |
| RF-16 Agendamento | SPEC-4-004 | CA-4-05, CA-4-06 | T4.13, T4.14, T4.15 |
| RF-17 Handoff | SPEC-4-005 | CA-4-07 | T4.16, T4.17 |

## Cobertura de critérios de aceite

| CA | SPEC | Tasks de prova |
|---|---|---|
| CA-4-01 | SPEC-4-001 | T4.1, T4.2, T4.5, T4.8 |
| CA-4-02 | SPEC-4-002 | T4.3 |
| CA-4-03 | SPEC-4-002 | T4.6, T4.7 |
| CA-4-04 | SPEC-4-003 | T4.9, T4.10, T4.11, T4.12 |
| CA-4-05 | SPEC-4-004 | T4.13, T4.14 |
| CA-4-06 | SPEC-4-004 | T4.15 |
| CA-4-07 | SPEC-4-005 | T4.16, T4.17 |