# Fase 3 — CRM conversacional e qualificação assistida

## Resultado

Atendimento ocorre em interface conversacional dentro da Central, com IA assistiva restrita à base aprovada, transbordo humano explícito, qualificação versionada por produto com ICP vigente, opt-out respeitado e configuração administrativa com publicação e rollback — sem canal real e sem autonomia da IA até os gates aprovarem.

## Inclui

RF-06 (ficha de qualificação), RF-09 (conversa unificada com fixture/importação), RF-10 (transbordo humano), RF-11 (agente assistivo), RF-12 (configuração administrativa versionada), RF-15 (supressão), RF-23 (ICP versionado), RN-03 (decisão humana no piloto), RN-04 (fora da base aprovada gera transbordo), RN-05 (lead ativo com responsável/estágio/próxima ação), RN-07 (opt-out prevalece), RN-13 (operação usa versão publicada), RN-14 (falha de integração mantém fila de exceção), RNF-02/03/04/05/08/10. Ativação do canal real depende de G1/G2/G5/G6.

## Demonstração

Importar conversas sintéticas, abrir a conversa unificada de um lead, receber sugestão da IA dentro da base aprovada, provocar pergunta fora da base e ver o transbordo com motivo/contexto, assumir e retomar manualmente, concluir qualificação com ICP vigente, registrar opt-out, publicar uma alteração administrativa com rollback e conferir a timeline completa — tudo com o canal real bloqueado.

## Critérios de aceite

- **CA-3-01:** evento repetido do canal (ou reimportação) não duplica mensagem nem lead; a conversa é exibida por lead em ordem cronológica, sem alterar o histórico original.
- **CA-3-02:** a IA sugere/responde somente dentro da base aprovada publicada; pergunta fora da base gera transbordo com motivo/contexto; nunca improvisa preço, contrato ou compromisso técnico.
- **CA-3-03:** o humano assume e devolve o atendimento de modo explícito, sem disputa com a IA; enquanto o lead está transbordado, nenhuma resposta automática é enviada.
- **CA-3-04:** a qualificação usa a versão vigente do ICP por produto e registra critérios e evidências; alteração de ICP não reclassifica o histórico silenciosamente.
- **CA-3-05:** opt-out interrompe ações automáticas, impede inclusão em follow-up/reativação e aparece na timeline com data/hora e origem.
- **CA-3-06:** toda alteração administrativa passa por rascunho, validação, publicação versionada e rollback; rascunhos não afetam leads em operação.
- **CA-3-07:** indisponibilidade da IA (ou do canal) mantém o atendimento humano funcional; tentativa de ativar canal externo sem G1/G5/G6 é negada e auditada.

## Controles transversais de segurança

- Fase 3 usa somente fixtures de conversa sintéticas; canal real exige G1 (fonte/fronteira), G5 (LGPD, opt-out, retenção) e G6 (contrato técnico do canal) antes de qualquer envio.
- Nenhum envio automático existe nesta fase: a IA sugere, o humano decide; todo envio manual é auditado.
- Base aprovada, ICP e configurações são versionados; rascunhos não afetam leads em operação (RN-13).
- Evidências mascaram telefone, e-mail e IDs; tokens nunca aparecem em captura, log ou exportação.
- Auditoria append-only para transbordos, retomadas, publicações, rollbacks e respostas manuais.

## Fora desta fase

Follow-up automático (Fase 4), reativação segmentada (Fase 4), agendamento integrado (Fase 4), handoff ao apresentador (Fase 4), metas numéricas congeladas (Fase 5, após baseline). A IA não envia mensagens por conta própria em nenhuma fase. Integrações novas de Ads e autonomia de campanhas seguem fora.

## Tasks

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status | Leva |
|---|---|---|---|---|---|---|---|---|---|
| T3.1 | Criar fixture de conversas e validador de esquema | Dev dados | SPEC-3-001 | CA-3-01 | RED/GREEN ingestão inválida | fixture + testes de atomicidade | Fase 2 aceita | ☐ | A |
| T3.2 | Implementar ingestão idempotente de conversas | Dev dados | SPEC-3-001 | CA-3-01 | Principal/falha ingestão | log de lote + relatório sanitizado | T3.1 aceita | ☐ | B |
| T3.3 | Implementar conversa unificada por lead | Dev frontend | SPEC-3-001 | CA-3-01 | GREEN fluxo principal | capturas + testes de ordem cronológica | T3.2 aceita | ☐ | C |
| T3.4 | Implementar base aprovada versionada de respostas | Dev full-stack | SPEC-3-002 | CA-3-02 | RED/GREEN resposta fora da base | testes de versão e publicação | Fase 2 aceita | ☐ | C |
| T3.5 | Implementar motor de sugestão com limite e fallback | Dev full-stack | SPEC-3-002 | CA-3-02 | Limite/falha de resposta | testes de limite e fallback | T3.4 aceita | ☐ | D |
| T3.6 | Implementar transbordo com pausa da IA e retomada explícita | Dev full-stack | SPEC-3-003 | CA-3-03 | RED/GREEN transbordo | testes de pausa e retomada | T3.5 aceita | ☐ | E |
| T3.7 | Implementar fila de transbordo e atribuição ao responsável | Dev frontend | SPEC-3-003 | CA-3-03 | GREEN fluxo principal | capturas da fila + testes | T3.6 aceita | ☐ | F |
| T3.8 | Implementar timeline das conversas (append-only) | Dev backend | SPEC-3-001 | CA-3-01 | RED/GREEN auditoria | testes de timeline e negação de delete | T3.3 aceita | ☐ | F |
| T3.9 | Implementar ICP versionado por produto | Dev full-stack | SPEC-3-004 | CA-3-04 | RED/GREEN versionamento | testes de vigência e versão | Fase 2 aceita | ☐ | G |
| T3.10 | Implementar ficha de qualificação vinculada ao ICP vigente | Dev full-stack | SPEC-3-004 | CA-3-04 | Principal/limite de qualificação | capturas + testes de versão | T3.9 e T3.3 aceitas | ☐ | H |
| T3.11 | Implementar opt-out e supressão | Dev full-stack | SPEC-3-005 | CA-3-05 | RED/GREEN supressão | trilha de opt-out + log de negação | T3.8 aceita | ☐ | H |
| T3.12 | Implementar rascunho, validação e publicação versionada | Dev full-stack | SPEC-3-006 | CA-3-06 | RED/GREEN publicação | trilha de publicação | T3.4 e T3.9 aceitas | ☐ | I |
| T3.13 | Implementar rollback de configuração | Dev full-stack | SPEC-3-006 | CA-3-06 | Limite/falha de rollback | captura do rollback + histórico | T3.12 aceita | ☐ | J |
| T3.14 | Provar degradação segura e bloqueio do canal sem gates | QA | SPEC-3-007 | CA-3-07 | Falha/degradação | roteiro gravado + log de negação | T3.11, T3.12 e T3.13 aceitas | ☐ | K |
| T3.15 | Demonstrar conversa ponta a ponta e emitir recibo final da Fase 3 | QA | SPEC-3-007 | CA-3-01..07 | Regressão integral | recibo CA-3-01..07 + relatório de gates G1/G2/G5/G6 | T3.14 aceita | ☐ | L |
