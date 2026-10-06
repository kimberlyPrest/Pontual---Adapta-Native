# Fase 4 — Follow-up, reativação e agendamento

## Resultado

Cadências e reativações governadas recuperam oportunidades elegíveis; leads qualificados chegam a agendamento válido com handoff estruturado.

## Inclui

RF-13 (follow-up por inatividade), RF-14 (reativação segmentada), RF-15 (supressão), RF-16 (agendamento), RF-17 (handoff); RN-01, RN-03, RN-05, RN-07, RN-08, RN-09, RN-10, RN-13, RN-14; RNF-02, RNF-04, RNF-05. Ativação do disparo real e da agenda externa depende de G4–G7.

## Demonstração

Configurar régua versionada, gerar follow-up por inatividade, excluir opt-out do segmento de reativação com prévia, aprovar e disparar lote (simulação sem gates), registrar agendamento com confirmação, reagendar preservando histórico e gerar handoff estruturado ao apresentador — fechando com recibo CA-4-01..07.

## Critérios de aceite

- **CA-4-01:** régua versionada respeita intervalo, horário, limite e exceções homologados.
- **CA-4-02:** opt-out, bloqueio ou inelegibilidade impedem follow-up e reativação.
- **CA-4-03:** segmento informa critérios, tamanho, exclusões e prévia antes de ativação.
- **CA-4-04:** disparo real exige aprovação humana e canal autorizado; reexecução não duplica envio.
- **CA-4-05:** `agendado` só conta com data, hora, responsável e confirmação.
- **CA-4-06:** reagendamento/cancelamento preservam histórico e ajustam o indicador sem apagamento.
- **CA-4-07:** handoff contém produto, contexto, critérios e pendências sem expor dado desnecessário.

## Controles transversais de segurança

- Fase 4 usa fixtures sintéticas; disparo real exige G5 (LGPD/opt-out/retenção) e G6 (contrato técnico do canal); agenda externa exige G7; cadência oficial exige G4.
- Nenhum envio automático sem aprovação humana registrada (RN-03); a IA não dispara nada por conta própria.
- Régua, segmento e disparo são versionados; rascunhos não afetam leads em operação (RN-13).
- Evidências mascaram telefone, e-mail e IDs; auditoria append-only para aprovações, envios, negações e acessos a handoff.

## Fora desta fase

Metas numéricas congeladas e medição de KPIs (Fase 5, após baseline); integração bidirecional com agenda externa (depende de G7, vira evolução); campanhas de mídia e autonomia de agentes (fora do programa); eventos pós-agendamento (adjacentes ao limite funcional, RN-01).

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.1 | Criar fixture de cadências e validador de esquema | Dev dados | SPEC-4-001 | CA-4-01 | RED/GREEN esquema inválido | fixture + testes de validação | Fase 3 aceita | ☐ | A |
| T4.2 | Implementar modelo de régua versionada (rascunho/publicação/rollback) | Dev full-stack | SPEC-4-001 | CA-4-01 | RED/GREEN versionamento | testes de versão e RN-13 | T4.1 aceita | ☐ | B |
| T4.3 | Implementar filtros de exclusão obrigatórios (opt-out/bloqueio/inelegibilidade) | Dev full-stack | SPEC-4-002 | CA-4-02 | RED/GREEN supressão | log de exclusões + testes de negação | T4.1 aceita | ☐ | B |
| T4.4 | Criar fixtures de reativação, disparo e agendamentos | Dev dados | SPEC-4-002/003/004 | CA-4-02..05 | RED/GREEN fixtures | 3 fixtures + testes | T4.1 aceita | ☐ | B |
| T4.5 | Implementar geração de próxima ação por inatividade | Dev full-stack | SPEC-4-001 | CA-4-01 | Principal/limite | log de ações + testes de intervalo | T4.2 e T4.3 aceitas | ☐ | C |
| T4.6 | Implementar construtor de segmento com tamanho e exclusões | Dev full-stack | SPEC-4-002 | CA-4-03 | Principal/limite | prévia + testes de contagem | T4.3 aceita | ☐ | C |
| T4.7 | Implementar prévia sanitizada antes de ativação | Dev frontend | SPEC-4-002 | CA-4-03 | GREEN fluxo principal | capturas da prévia + testes de máscara | T4.6 aceita | ☐ | D |
| T4.8 | Implementar execução da régua com horário, limite e exceções | Dev full-stack | SPEC-4-001 | CA-4-01 | Limite/falha | testes de janela/limite/exceção | T4.5 aceita | ☐ | D |
| T4.9 | Implementar preparação de lote a partir de régua e segmento | Dev full-stack | SPEC-4-003 | CA-4-04 | Principal | lote + testes de composição | T4.5 e T4.6 aceitas | ☐ | E |
| T4.10 | Implementar aprovação humana obrigatória do lote | Dev full-stack | SPEC-4-003 | CA-4-04 | RED/GREEN aprovação | trilha de aprovação + testes de pendência | T4.9 aceita | ☐ | F |
| T4.11 | Implementar disparo idempotente com fila de exceção | Dev full-stack | SPEC-4-003 | CA-4-04 | Limite/falha | log de envios + testes de reexecução | T4.10 aceita | ☐ | G |
| T4.12 | Bloquear disparo real sem gates G5/G6 (negação auditada) | Dev backend | SPEC-4-003 | CA-4-04 | Falha | log de negação + testes de bloqueio | T4.11 aceita | ☐ | H |
| T4.13 | Implementar registro de agendamento com campos obrigatórios (RN-09) | Dev full-stack | SPEC-4-004 | CA-4-05 | RED/GREEN campos | testes de validação server-side | Fase 3 aceita | ☐ | G |
| T4.14 | Implementar estados e confirmação do agendamento | Dev full-stack | SPEC-4-004 | CA-4-05 | Principal/limite | capturas + testes de estado | T4.13 aceita | ☐ | H |
| T4.15 | Implementar histórico append-only e indicador sem apagamento | Dev backend | SPEC-4-004 | CA-4-06 | RED/GREEN auditoria | testes de append-only + indicador | T4.14 aceita | ☐ | I |
| T4.16 | Implementar geração de handoff com máscara por permissão | Dev full-stack | SPEC-4-005 | CA-4-07 | RED/GREEN máscara | handoff + testes de máscara | T4.15 aceita | ☐ | J |
| T4.17 | Demonstrar ponta a ponta e emitir recibo final da Fase 4 | QA | SPEC-4-005 | CA-4-01..07 | Regressão integral | recibo + relatório de gates G4–G7 | T4.12, T4.15 e T4.16 aceitas | ☐ | K |

## Levas

- **A (fixture de cadências):** T4.1
- **B (régua versionada, exclusões e fixtures):** T4.2, T4.3, T4.4
- **C (geração de ação e segmento):** T4.5, T4.6
- **D (prévia e execução da régua):** T4.7, T4.8
- **E (preparação de lote):** T4.9
- **F (aprovação humana):** T4.10
- **G (disparo idempotente e agendamento):** T4.11, T4.13
- **H (bloqueio de gates e estados):** T4.12, T4.14
- **I (histórico append-only):** T4.15
- **J (handoff):** T4.16
- **K (recibo):** T4.17
