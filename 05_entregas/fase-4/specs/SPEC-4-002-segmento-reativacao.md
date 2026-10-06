# SPEC-4-002 — Segmento de reativação com prévia e exclusões

**Fase:** 4  
**Status:** planejada  
**Dono:** Dev full-stack  
**Origem no escopo:** RF-14, RF-15, RN-07, RNF-02  
**Degrau da solução:** construção mínima — reusa as supressões da SPEC-3-005 (opt-out) como filtro obrigatório de todo segmento; nenhum envio real nesta SPEC (disparo é SPEC-4-003).

## Resultado observável

O operador monta um segmento de reativação por critérios (origem, estágio, inatividade, produto), vê critérios, tamanho, exclusões e prévia dos contatos antes de ativar; contatos com opt-out, bloqueio ou inelegibilidade nunca entram no segmento, e a tentativa de incluí-los é negada com motivo registrado.

## Limites e dependências

- **Inclui:** construtor de segmento por critérios, cálculo de tamanho, lista de exclusões (opt-out/bloqueio/inelegibilidade), prévia sanitizada antes de ativar, trilha de auditoria da criação/ativação.
- **Fora de escopo:** disparo real (SPEC-4-003), enriquecimento externo de dados, campanhas de mídia (Fase 2).
- **Entradas e pré-condições:** Fase 3 aceita (supressões da SPEC-3-005); base de leads da Fase 1.
- **Saídas/artefatos:** coleção `reactivation_segments` + prévia; evidências RED/GREEN.
- **Risco e plano B:** se a base elegível for menor que o esperado, o segmento roda com fixture e o relatório de exclusões orienta o cliente.
- **Rollback:** segmento pode ser desativado; exclusões permanecem auditadas.

## Fluxo e regras

1. Criar fixture de base elegível com contatos opt-out/bloqueados e registrar RED (segmento inclui contato suprimido — comportamento que deve falhar).
2. Implementar construtor de segmento por critérios com tamanho calculado.
3. Aplicar filtros de exclusão obrigatórios: opt-out (RN-07), bloqueio, inelegibilidade; registrar motivo de cada exclusão.
4. Exibir prévia sanitizada (critérios, tamanho, exclusões, amostra mascarada) antes de ativar.

| Cenário | Dado/condição | Resultado esperado | Recuperação |
|---|---|---|---|
| Principal | Critérios válidos sobre base elegível | Segmento com tamanho, exclusões e prévia | Corrigir sem alterar contrato |
| Limite | Critério que só retorna contatos suprimidos | Segmento vazio com motivo explícito | Ajustar critérios |
| Falha | Tentativa de incluir contato opt-out manualmente | Negação auditada com motivo | RN-07 prevalece |

## Checklist de execução

- [ ] Fixture conferida (zero PII; contatos suprimidos incluídos de propósito).
- [ ] RED registrado antes da implementação.
- [ ] Exclusões obrigatórias provadas (opt-out, bloqueio, inelegibilidade).
- [ ] Prévia sanitizada demonstrada antes da ativação.
- [ ] Dono e handoff confirmados.

## Critérios de aceite

- [ ] **CA-4-02:** opt-out, bloqueio ou inelegibilidade impedem follow-up e reativação.
- [ ] **CA-4-03:** segmento informa critérios, tamanho, exclusões e prévia antes de ativação.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED CA-4-02 | segmento sem filtro de supressão | fixture + construtor | contato opt-out incluído (falha esperada) | `evidencias/CA-4-02-red` |
| GREEN CA-4-02 | segmento com filtros obrigatórios | fixture + construtor | suprimidos excluídos com motivo; negação auditada | `evidencias/CA-4-02-green` |
| GREEN CA-4-03 | ativar prévia do segmento | fixture + UI | critérios, tamanho, exclusões e amostra mascarada exibidos | `evidencias/CA-4-03-green` |
| REFACTOR/REGRESSÃO | regressão F3 (supressões) | todos | CA-3-05 continua passando | `evidencias/regressao-SPEC-4-002` |

**Dados/fixtures:** `fixtures/reativacao-fase4.csv` (contatos sintéticos, alguns com opt-out/bloqueio/inelegibilidade).  
**Caminhos de erro obrigatórios:** critério inválido, base vazia, contato suprimido, tentativa manual de inclusão.  
**Evidência exigida:** relatório de exclusões, prévia sanitizada, log de negação, recibo CA-4-02/03.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T4.3 | Implementar filtros de exclusão obrigatórios (opt-out/bloqueio/inelegibilidade) | Dev full-stack | SPEC-4-002 | CA-4-02 | RED/GREEN supressão | log de exclusões + testes de negação | T4.1 aceita | ☐ | B |
| T4.4 | Criar fixtures de reativação, disparo e agendamentos | Dev dados | SPEC-4-002/003/004 | CA-4-02..05 | RED/GREEN fixtures | 3 fixtures + testes | T4.1 aceita | ☐ | B |
| T4.6 | Implementar construtor de segmento com tamanho e exclusões | Dev full-stack | SPEC-4-002 | CA-4-03 | Principal/limite | prévia + testes de contagem | T4.3 aceita | ☐ | C |
| T4.7 | Implementar prévia sanitizada antes de ativação | Dev frontend | SPEC-4-002 | CA-4-03 | GREEN fluxo principal | capturas da prévia + testes de máscara | T4.6 aceita | ☐ | D |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
