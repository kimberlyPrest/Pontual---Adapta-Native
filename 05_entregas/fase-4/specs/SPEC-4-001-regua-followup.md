# SPEC-4-001 — Régua de follow-up versionada

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-13, RN-05, RN-08, RN-13, G4  
**Degrau da solução:** construção mínima — reusa o padrão de rascunho→validação→publicação versionada da SPEC-3-006 (config administrativa); nenhum número de cadência é fixado sem homologação (RN-08).

## Resultado observável

O administrador configura uma régua de follow-up (intervalos, horário permitido, limite de tentativas e exceções) como rascunho, valida e publica uma versão; a operação passa a gerar automaticamente a próxima ação de leads inativos conforme a versão publicada, e a execução respeita intervalo, horário, limite e exceções homologados.

## Limites e dependências

- **Inclui:** modelo de régua versionada (rascunho/publicação/rollback), geração de próxima ação por inatividade, execução com janela de horário, limite de tentativas e exceções, trilha de auditoria de cada ação gerada.
- **Fora de escopo:** envio automático de mensagens (CA-4-04, SPEC-4-003), canal real (G5/G6), metas numéricas de cadência sem G4.
- **Entradas e pré-condições:** Fase 3 aceita (rascunho/publicação da SPEC-3-006 e supressões da SPEC-3-005 disponíveis); G4 (cadência/SLA homologados) para valores oficiais — sem G4, a régua roda em modo homologação com fixture.
- **Saídas/artefatos:** coleções `cadence_rules` (versionada) + `cadence_actions`; evidências RED/GREEN.
- **Risco e plano B:** se G4 atrasar, a régua opera com valores marcados como "não homologados" e nenhum disparo real ocorre (CA-4-04 cobre o disparo).
- **Rollback:** publicação versionada com rollback (padrão SPEC-3-006); ações já geradas permanecem auditadas.

## Fluxo e regras

1. Criar fixture de cadências e registrar RED (régua inexistente: lead inativo não gera próxima ação).
2. Implementar modelo de régua versionada: rascunho não afeta operação (RN-13); publicação cria versão; rollback restaura versão anterior.
3. Implementar geração de próxima ação por inatividade: lead ativo sem interação no intervalo configurado recebe próxima ação com responsável, estágio e prazo (RN-05).
4. Implementar execução com janela de horário, limite de tentativas e exceções (lead com exceção registrada não recebe ação).

| Cenário | Dado/condição | Resultado esperado | Recuperação |
|---|---|---|---|
| Principal | Lead inativo além do intervalo, dentro do horário e do limite | Próxima ação gerada e auditada | Corrigir sem alterar contrato |
| Limite | Lead no limite de tentativas OU fora da janela de horário OU com exceção | Nenhuma ação gerada; motivo registrado | Reprocessar após homologação |
| Falha | Publicação inválida (intervalo negativo, horário vazio) | Rascunho rejeitado; versão publicada anterior segue vigente | Rascunho não afeta operação |

## Checklist de execução

- [ ] Fixture conferida (zero PII).
- [ ] RED registrado antes da implementação.
- [ ] Rascunho não gera ação em operação (RN-13) demonstrado.
- [ ] Intervalo, horário, limite e exceções provados nos três cenários.
- [ ] Dono e handoff confirmados.

## Critérios de aceite

- [ ] **CA-4-01:** régua versionada respeita intervalo, horário, limite e exceções homologados.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED CA-4-01 | lead inativo sem régua publicada | fixture + execução | nenhuma ação gerada (comportamento ausente) | `evidencias/CA-4-01-red` |
| GREEN CA-4-01 | régua publicada + lead inativo dentro das regras | fixture + execução | ação gerada com responsável/prazo; limites respeitados | `evidencias/CA-4-01-green` |
| REFACTOR/REGRESSÃO | republicar régua + rollback + regressão F3 | todos | versão anterior restaurada; CA-3-05/06 continuam passando | `evidencias/regressao-SPEC-4-001` |

**Dados/fixtures:** `fixtures/cadencias-fase4.csv` (lead sintético, última interação, intervalo, horário, limite, exceção — zero PII).  
**Caminhos de erro obrigatórios:** intervalo inválido, horário vazio, limite excedido, exceção registrada, rascunho não publicado.  
**Evidência exigida:** log de ações geradas, trilha de publicação/rollback, recibo CA-4-01.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.1 | Criar fixture de cadências e validador de esquema | Dev dados | SPEC-4-001 | CA-4-01 | RED/GREEN esquema inválido | fixture + testes de validação | Fase 3 aceita | ☐ | A |
| T4.2 | Implementar modelo de régua versionada (rascunho/publicação/rollback) | Dev full-stack | SPEC-4-001 | CA-4-01 | RED/GREEN versionamento | testes de versão e RN-13 | T4.1 aceita | ☐ | B |
| T4.5 | Implementar geração de próxima ação por inatividade | Dev full-stack | SPEC-4-001 | CA-4-01 | Principal/limite | log de ações + testes de intervalo | T4.2 e T4.3 aceitas | ☐ | C |
| T4.8 | Implementar execução da régua com horário, limite e exceções | Dev full-stack | SPEC-4-001 | CA-4-01 | Limite/falha | testes de janela/limite/exceção | T4.5 aceita | ☐ | D |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
