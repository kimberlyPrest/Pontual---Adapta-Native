# SPEC-4-004 — Agendamento válido com histórico preservado

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-16, RN-01, RN-09, RN-10, G7  
**Degrau da solução:** construção mínima — registro de agendamento dentro da Central (data, hora, responsável, participantes, produto, contexto); integração com agenda externa é dependência de G7 e fica fora do corte atual.

## Resultado observável

Um lead qualificado é registrado como `agendado` somente com data, hora, responsável pela apresentação e confirmação; reagendamento e cancelamento preservam o histórico completo do evento anterior e ajustam o indicador sem apagamento; o limite funcional do funil encerra no primeiro agendamento válido (RN-01).

## Limites e dependências

- **Inclui:** registro de agendamento (data, hora, responsável, participantes, produto, contexto), estados solicitado/confirmado/reagendado/cancelado, histórico append-only de eventos, indicador `agendado` derivado dos estados válidos.
- **Fora de escopo:** sincronização bidirecional com agenda externa (G7), envio automático de convite, eventos pós-agendamento (adjacentes, RN-01).
- **Entradas e pré-condições:** Fase 3 aceita (lead com qualificação e ICP vigente); G7 (calendário/disponibilidade/confirmação) para regras oficiais — sem G7, registro manual com fixture.
- **Saídas/artefatos:** coleção `appointments` (append-only) + indicador; evidências RED/GREEN.
- **Risco e plano B:** se G7 atrasar, o agendamento é manual na Central e a integração de agenda vira evolução registrada.
- **Rollback:** cancelamento registra evento; nada é apagado (RN-10).

## Fluxo e regras

1. Registrar RED (agendamento sem confirmação conta como `agendado` — comportamento que deve falhar).
2. Implementar registro de agendamento com campos obrigatórios (RN-09): data, hora, responsável, participantes, produto, contexto.
3. Implementar estados: solicitado → confirmado → reagendado/cancelado; `agendado` só conta com confirmação registrada.
4. Implementar histórico append-only: reagendamento/cancelamento criam novo evento referenciando o anterior; indicador recalculado sem apagar nada.

| Cenário | Dado/condição | Resultado esperado | Recuperação |
|---|---|---|---|
| Principal | Agendamento completo com confirmação | Estado `agendado` válido; indicador conta | Corrigir sem alterar contrato |
| Limite | Agendamento sem confirmação OU sem responsável | Não conta como `agendado`; pendência visível | Completar dados |
| Falha | Tentativa de apagar/editar evento anterior | Negação auditada; histórico preservado | RN-10 |

## Checklist de execução

- [ ] RED registrado antes da implementação.
- [ ] Campos obrigatórios (RN-09) validados server-side.
- [ ] Estados e confirmação provados.
- [ ] Histórico append-only demonstrado (reagendamento e cancelamento).
- [ ] Dono e handoff confirmados.

## Critérios de aceite

- [ ] **CA-4-05:** `agendado` só conta com data, hora, responsável e confirmação.
- [ ] **CA-4-06:** reagendamento/cancelamento preservam histórico e ajustam o indicador sem apagamento.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED CA-4-05 | agendamento incompleto antes da entrega | fixture + registro | conta como agendado sem confirmação (falha esperada) | `evidencias/CA-4-05-red` |
| GREEN CA-4-05 | agendamento completo + confirmação | fixture + registro | `agendado` válido com campos RN-09 | `evidencias/CA-4-05-green` |
| GREEN CA-4-06 | reagendar e cancelar | fixture + registro | eventos preservados; indicador ajustado sem apagar | `evidencias/CA-4-06-green` |
| REFACTOR/REGRESSÃO | regressão F3 (timeline/auditoria) | todos | CA-3-01/06 continuam passando | `evidencias/regressao-SPEC-4-004` |

**Dados/fixtures:** `fixtures/agendamentos-fase4.csv` (agendamentos sintéticos: completo, incompleto, reagendado, cancelado).  
**Caminhos de erro obrigatórios:** data/hora ausente, responsável vazio, sem confirmação, cancelamento, reagendamento, tentativa de delete.  
**Evidência exigida:** captura do agendamento, histórico de eventos, indicador recalculado, recibo CA-4-05/06.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.13 | Implementar registro de agendamento com campos obrigatórios (RN-09) | Dev full-stack | SPEC-4-004 | CA-4-05 | RED/GREEN campos | testes de validação server-side | Fase 3 aceita | ☐ | G |
| T4.14 | Implementar estados e confirmação do agendamento | Dev full-stack | SPEC-4-004 | CA-4-05 | Principal/limite | capturas + testes de estado | T4.13 aceita | ☐ | H |
| T4.15 | Implementar histórico append-only e indicador sem apagamento | Dev backend | SPEC-4-004 | CA-4-06 | RED/GREEN auditoria | testes de append-only + indicador | T4.14 aceita | ☐ | I |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |