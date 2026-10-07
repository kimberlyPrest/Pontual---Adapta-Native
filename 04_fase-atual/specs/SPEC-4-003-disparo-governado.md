# SPEC-4-003 — Disparo governado com aprovação humana e idempotência

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-13, RF-14, RN-03, RN-07, RN-13, RN-14, RNF-02, RNF-05, G5, G6  
**Degrau da solução:** construção mínima — o disparo é um ato humano assistido: o sistema prepara e registra; o envio real só existe com canal autorizado (G5/G6) e aprovação registrada; sem gates, a operação roda em modo simulação.

## Resultado observável

Um lote de follow-up/reativação preparado a partir da régua publicada e do segmento ativado exige aprovação humana explícita antes do disparo; a reexecução do mesmo lote não duplica envio; toda tentativa de disparo sem canal autorizado ou sem gates aprovados é negada e auditada.

## Limites e dependências

- **Inclui:** preparação de lote a partir de régua/segmento, aprovação humana registrada (quem, quando, o quê), disparo com canal autorizado, idempotência por lote e por contato, trilha de auditoria de cada envio, modo simulação sem gates.
- **Fora de escopo:** autonomia da IA para disparar (nenhuma fase), campanhas de mídia (Fase 2), canal real sem G5/G6.
- **Entradas e pré-condições:** SPEC-4-001 e SPEC-4-002 aceitas; G5 (LGPD/opt-out/retenção) e G6 (contrato técnico do canal) para disparo real — sem eles, somente simulação auditada.
- **Saídas/artefatos:** coleções `dispatch_batches` + `dispatch_events`; evidências RED/GREEN.
- **Risco e plano B:** se G5/G6 atrasarem, o lote fica em "simulação" com relatório; nenhum envio real ocorre.
- **Rollback:** lote pode ser cancelado antes do disparo; envios realizados permanecem auditados (append-only).

## Fluxo e regras

1. Registrar RED (lote sem aprovação dispara — comportamento que deve falhar).
2. Implementar preparação de lote: régua publicada + segmento ativado → lista de destinatários com exclusões aplicadas.
3. Implementar aprovação humana obrigatória: lote pendente até aprovação registrada (RN-03); rejeição registra motivo.
4. Implementar disparo idempotente: reexecução do mesmo lote não gera segundo envio por contato (RNF-05); falha de canal mantém item em fila de exceção (RN-14).
5. Bloquear disparo real sem G5/G6: tentativa é negada e auditada.

| Cenário | Dado/condição | Resultado esperado | Recuperação |
|---|---|---|---|
| Principal | Lote aprovado com canal autorizado | Envios registrados um por contato | Fila de exceção (RN-14) |
| Limite | Reexecução do mesmo lote | Nenhum envio duplicado | Idempotência por lote/contato |
| Falha | Disparo sem aprovação OU sem gates | Negação auditada com motivo | Modo simulação |

## Checklist de execução

- [ ] RED registrado antes da implementação.
- [ ] Aprovação humana obrigatória provada (lote pendente sem ela).
- [ ] Idempotência de reexecução demonstrada.
- [ ] Negação sem gates auditada.
- [ ] Dono e handoff confirmados.

## Critérios de aceite

- [ ] **CA-4-04:** disparo real exige aprovação humana e canal autorizado; reexecução não duplica envio.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED CA-4-04 | lote sem aprovação / reexecução antes da entrega | fixture + disparo | envio sem aprovação ou duplicado (falha esperada) | `evidencias/CA-4-04-red` |
| GREEN CA-4-04 | lote aprovado + reexecução | fixture + disparo | envio único por contato; reexecução não duplica | `evidencias/CA-4-04-green` |
| REFACTOR/REGRESSÃO | disparo sem gates + regressão F3 | todos | negação auditada; CA-3-05/07 continuam passando | `evidencias/regressao-SPEC-4-003` |

**Dados/fixtures:** `fixtures/disparo-fase4.csv` (lote sintético com contatos aprovados/suprimidos/duplicados).  
**Caminhos de erro obrigatórios:** lote sem aprovação, canal não autorizado, gates ausentes, reexecução, falha de canal, contato suprimido no lote.  
**Evidência exigida:** trilha de aprovação, log de envios, log de negação, recibo CA-4-04.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.9 | Implementar preparação de lote a partir de régua e segmento | Dev full-stack | SPEC-4-003 | CA-4-04 | Principal | lote + testes de composição | T4.5 e T4.6 aceitas | ☐ | E |
| T4.10 | Implementar aprovação humana obrigatória do lote | Dev full-stack | SPEC-4-003 | CA-4-04 | RED/GREEN aprovação | trilha de aprovação + testes de pendência | T4.9 aceita | ☐ | F |
| T4.11 | Implementar disparo idempotente com fila de exceção | Dev full-stack | SPEC-4-003 | CA-4-04 | Limite/falha | log de envios + testes de reexecução | T4.10 aceita | ☐ | G |
| T4.12 | Bloquear disparo real sem gates G5/G6 (negação auditada) | Dev backend | SPEC-4-003 | CA-4-04 | Falha | log de negação + testes de bloqueio | T4.11 aceita | ☐ | H |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |