# Fase 4 — Tarefas gerais

Tasks detalhadas com dono, critério, prova e levas em `00-Fase.md`. Especificações em `01-SPECs/`.

| ID | Task | Dono | SPEC | Critério | Leva | Status |
|---|---|---|---|---|---|---|
| T4.1 | Criar fixture de cadências e validador de esquema | Dev dados | SPEC-4-001 | CA-4-01 | A | ☐ |
| T4.2 | Implementar modelo de régua versionada (rascunho/publicação/rollback) | Dev full-stack | SPEC-4-001 | CA-4-01 | B | ☐ |
| T4.3 | Implementar filtros de exclusão obrigatórios (opt-out/bloqueio/inelegibilidade) | Dev full-stack | SPEC-4-002 | CA-4-02 | B | ☐ |
| T4.4 | Criar fixtures de reativação, disparo e agendamentos | Dev dados | SPEC-4-002/003/004 | CA-4-02..05 | B | ☐ |
| T4.5 | Implementar geração de próxima ação por inatividade | Dev full-stack | SPEC-4-001 | CA-4-01 | C | ☐ |
| T4.6 | Implementar construtor de segmento com tamanho e exclusões | Dev full-stack | SPEC-4-002 | CA-4-03 | C | ☐ |
| T4.7 | Implementar prévia sanitizada antes de ativação | Dev frontend | SPEC-4-002 | CA-4-03 | D | ☐ |
| T4.8 | Implementar execução da régua com horário, limite e exceções | Dev full-stack | SPEC-4-001 | CA-4-01 | D | ☐ |
| T4.9 | Implementar preparação de lote a partir de régua e segmento | Dev full-stack | SPEC-4-003 | CA-4-04 | E | ☐ |
| T4.10 | Implementar aprovação humana obrigatória do lote | Dev full-stack | SPEC-4-003 | CA-4-04 | F | ☐ |
| T4.11 | Implementar disparo idempotente com fila de exceção | Dev full-stack | SPEC-4-003 | CA-4-04 | G | ☐ |
| T4.12 | Bloquear disparo real sem gates G5/G6 (negação auditada) | Dev backend | SPEC-4-003 | CA-4-04 | H | ☐ |
| T4.13 | Implementar registro de agendamento com campos obrigatórios (RN-09) | Dev full-stack | SPEC-4-004 | CA-4-05 | G | ☐ |
| T4.14 | Implementar estados e confirmação do agendamento | Dev full-stack | SPEC-4-004 | CA-4-05 | H | ☐ |
| T4.15 | Implementar histórico append-only e indicador sem apagamento | Dev backend | SPEC-4-004 | CA-4-06 | I | ☐ |
| T4.16 | Implementar geração de handoff com máscara por permissão | Dev full-stack | SPEC-4-005 | CA-4-07 | J | ☐ |
| T4.17 | Demonstrar ponta a ponta e emitir recibo final da Fase 4 | QA | SPEC-4-005 | CA-4-01..07 | K | ☐ |
