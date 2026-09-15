# SPEC-1-002 — Importação assistida e deduplicação de amostra

**Fase:** 1  
**Status:** planejada  
**Dono:** Desenvolvimento/Dados, com validação do champion  
**Origem no escopo:** RF-001, RF-002, RF-010; Fase 1  
**Degrau da solução:** construção mínima — provar importação segura em amostra antes de qualquer migração.

## Contexto e decisões fechadas

- **Estado atual:** bases são copiadas entre Google Sheets, Excel/SharePoint e RespondeChat; regras de identidade e precedência não estão homologadas.
- **Estado desejado:** importar uma amostra, preservar origem, sinalizar possíveis duplicidades e rejeitar linhas inválidas sem perder o lote original.
- **Decisões já fechadas:** nenhuma migração integral; dados originais não serão apagados.
- **Bloqueios:** nenhum para a amostra de teste; chave de produção e precedência definitiva ficam fora desta fase.

## Resultado observável

Uma importação de amostra auditável em que cada linha fica `importada`, `duplicidade para revisão` ou `rejeitada`, com origem e motivo preservados.

## Limites e dependências

- **Inclui:** upload/ingestão de arquivo de teste, validação, relatório de lote, possível duplicidade e rollback do lote.
- **Fora de escopo:** migração integral, envio ao RespondeChat, enriquecimento externo e deduplicação automática definitiva.
- **Entradas e pré-condições:** arquivo anonimizado e dicionário de campos aprovado para o teste.
- **Saídas/artefatos:** relatório do lote, registros de teste e lista de revisão.
- **Dependências e responsáveis:** cliente fornece amostra; desenvolvimento implementa; champion decide casos ambíguos.
- **Atores e permissões mínimas:** administrador importa; gestão revisa; SDR não importa produção.
- **Superfícies afetadas:** importador e ambiente de teste.
- **Risco e plano B:** se não houver chave confiável, somente sinalizar coincidências; não consolidar automaticamente.
- **Rollback ou reversão:** remover o lote identificado e restaurar o estado anterior do ambiente de teste.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| arquivo anonimizado → lote de teste | arquivo fornecido pelo cliente | empresa, contato, telefone/e-mail, produto, origem | administrador autenticado | lote com ID; reprocessar não duplica o mesmo lote | linha inválida vai para relatório |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-4 | linha sem campo obrigatório | rejeitar com motivo | não descartar o arquivo original | RF-010 |
| RN-5 | possível coincidência de identidade | separar para revisão | não consolidar sem regra aprovada | AC-011 |

## Fluxo e regras

1. Administrador seleciona arquivo de teste.
2. Sistema cria ID de lote e valida cabeçalho/linhas.
3. Sistema classifica linhas válidas, inválidas e possíveis duplicidades.
4. Sistema importa apenas válidas não bloqueadas para o ambiente de teste.
5. Sistema gera relatório e permite rollback do lote.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | amostra válida | registros importados com origem | relatório do lote |
| Limite | telefone/e-mail coincidente | revisão manual, sem fusão automática | decidir depois |
| Falha | cabeçalho ausente/arquivo corrompido | lote não é aplicado | corrigir arquivo e reenviar novo ID |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC e escopo definitivo, seções 4, 5 e Fase 1.
2. **Alterar somente:** importação e relatório no ambiente de teste.
3. **Não alterar:** sistemas legados, dados originais e regras de produção.
4. **Executar nesta ordem:** RED com arquivo inválido; validação; importação mínima; GREEN; rollback.
5. **Parar e pedir validação quando:** a execução tentar consolidar dados de produção ou escolher uma regra definitiva de identidade.
6. **Estado válido ao parar:** lote identificável, relatório gerado e rollback disponível.

## Checklist de execução

- [ ] amostra anonimizada recebida;
- [ ] dicionário de campos conferido;
- [ ] lote válido, inválido e duplicado testados;
- [ ] relatório e rollback exercitados;
- [ ] evidência e decisão dos casos ambíguos registradas.

## Critérios de aceite

- [ ] **CA-1-006:** arquivo válido gera lote identificado e registros com origem preservada.
- [ ] **CA-1-007:** linha inválida é rejeitada com motivo sem interromper silenciosamente o lote.
- [ ] **CA-1-008:** possível duplicidade não é fundida sem decisão aprovada.
- [ ] **CA-1-009:** rollback remove somente o lote de teste e preserva o arquivo original.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | arquivo inválido e lote duplicado | executar `F1-IMPORT-01` | falha antes do importador | log |
| GREEN | arquivo válido com três tipos de linha | executar `F1-IMPORT-01/02` | CA-1-006/007/008 passam | relatório |
| REFACTOR/REGRESSÃO | rollback e reprocessamento do mesmo lote | executar `F1-IMPORT-03` | CA-1-009 passa e não duplica | log/captura |

**Dados/fixtures:** arquivo anonimizado com uma linha válida, uma inválida, uma possível duplicidade e uma origem desconhecida.  
**Caminhos de erro obrigatórios:** arquivo corrompido, coluna ausente, lote repetido e permissão insuficiente.  
**Evidência exigida:** arquivo hash/ID do lote, relatório e aceite da revisão humana.

## Handoff e operação

- **Como demonstrar:** importar amostra → revisar classificação → consultar relatório → executar rollback.
- **Como operar depois:** administrador executa somente lotes autorizados.
- **Como monitorar:** lotes com rejeição, duplicidade ou falha de rollback.
- **Pendência conhecida:** chave e precedência definitivas.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T04 | Definir fixture e validação do formato da amostra | Dados/Cliente | SPEC-1-002 | CA-1-006/007 | F1-IMPORT-01 | fixture + relatório | amostra anonimizada | ☐ |
| F1-T05 | Implementar classificação, importação e relatório | Desenvolvimento | SPEC-1-002 | CA-1-006/007/008 | F1-IMPORT-02 | lote + relatório | F1-T04 aprovada | ☐ |
| F1-T06 | Implementar e provar rollback/reprocessamento | Desenvolvimento/QA | SPEC-1-002 | CA-1-009 | F1-IMPORT-03 | log de rollback | F1-T05 aprovada | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
