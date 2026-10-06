# SPEC-4-005 — Handoff estruturado ao apresentador e recibo final da Fase 4

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack / QA  
**Origem no escopo:** RF-17, RN-01, RNF-02, RNF-04  
**Degrau da solução:** construção mínima — o handoff é um resumo estruturado gerado a partir dos dados já existentes na Central (qualificação, ICP, conversa, agendamento); nenhum dado novo é coletado.

## Resultado observável

Ao confirmar um agendamento, o sistema gera um handoff estruturado ao apresentador contendo produto, contexto do lead, critérios de qualificação e pendências — sem expor dado desnecessário (dados mascarados conforme permissão); o apresentador acessa o handoff sem ler a conversa integral. A fase fecha com demonstração ponta a ponta e recibo CA-4-01..07.

## Limites e dependências

- **Inclui:** geração de handoff a partir do agendamento confirmado, campos produto/contexto/critérios/pendências, máscara de dados por permissão, trilha de acesso, demonstração ponta a ponta da fase e recibo final.
- **Fora de escopo:** envio automático do handoff por canal externo, materiais de apresentação, eventos pós-agendamento (RN-01).
- **Entradas e pré-condições:** SPEC-4-004 aceita (agendamento confirmado); qualificação/ICP da Fase 3.
- **Saídas/artefatos:** coleção `handoffs` + recibo `evidencias/recibo-fase-4.md`.
- **Risco e plano B:** se campos de contexto forem insuficientes, o handoff permite complemento manual auditado.
- **Rollback:** handoff pode ser regenerado; acessos permanecem auditados.

## Fluxo e regras

1. Registrar RED (handoff expõe dado além do necessário — comportamento que deve falhar).
2. Implementar geração de handoff no agendamento confirmado: produto, contexto, critérios (ICP vigente), pendências.
3. Aplicar máscara por permissão: apresentador vê o necessário; PII além do mínimo é ocultado (RNF-04).
4. Registrar trilha de acesso ao handoff.
5. Demonstrar ponta a ponta (régua → segmento → disparo simulado → agendamento → handoff) e emitir recibo CA-4-01..07 com relatório de gates G4–G7.

| Cenário | Dado/condição | Resultado esperado | Recuperação |
|---|---|---|---|
| Principal | Agendamento confirmado com qualificação completa | Handoff completo e mascarado | Complemento manual auditado |
| Limite | Qualificação incompleta | Handoff com pendências explícitas | Não bloqueia o agendamento |
| Falha | Acesso sem permissão ao handoff | Negação auditada | RNF-02 |

## Checklist de execução

- [ ] RED registrado antes da implementação.
- [ ] Handoff com os 4 blocos provado (produto, contexto, critérios, pendências).
- [ ] Máscara por permissão demonstrada.
- [ ] Demonstração ponta a ponta gravada/registrada.
- [ ] Recibo CA-4-01..07 emitido com relatório de gates.

## Critérios de aceite

- [ ] **CA-4-07:** handoff contém produto, contexto, critérios e pendências sem expor dado desnecessário.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED CA-4-07 | handoff sem máscara antes da entrega | fixture + geração | dado além do necessário exposto (falha esperada) | `evidencias/CA-4-07-red` |
| GREEN CA-4-07 | agendamento confirmado + geração | fixture + UI | handoff com 4 blocos, mascarado, com trilha de acesso | `evidencias/CA-4-07-green` |
| REFACTOR/REGRESSÃO | demonstração ponta a ponta + regressão F1–F3 | roteiro completo | CA-4-01..07 no recibo; CAs das fases anteriores passando | `evidencias/recibo-fase-4.md` |

**Dados/fixtures:** reusa `fixtures/agendamentos-fase4.csv` + qualificações sintéticas da Fase 3.  
**Caminhos de erro obrigatórios:** qualificação incompleta, acesso sem permissão, dado sensível no handoff, regeneração.  
**Evidência exigida:** handoff gerado, trilha de acesso, roteiro da demonstração, recibo CA-4-01..07.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.16 | Implementar geração de handoff com máscara por permissão | Dev full-stack | SPEC-4-005 | CA-4-07 | RED/GREEN máscara | handoff + testes de máscara | T4.15 aceita | ☐ | J |
| T4.17 | Demonstrar ponta a ponta e emitir recibo final da Fase 4 | QA | SPEC-4-005 | CA-4-01..07 | Regressão integral | recibo + relatório de gates G4–G7 | T4.12, T4.15 e T4.16 aceitas | ☐ | K |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
