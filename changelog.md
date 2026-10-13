# Changelog

## 13/10/2026

- **Fase 4 replanejada como fase final** pela consultora Kim Prestes. O recorte anterior (5 SPECs, 7 critérios, 17 tasks, disparo em simulação) foi substituído; a Fase 5 foi incorporada.
- Motivo: as fases 1 a 3 entregaram 41 rotas de backend e 13 delas com tela; não há cadastro de lead pela interface, o lead não tem os campos da operação, o funil termina em "Qualificado" e nenhum dado real entrou no sistema.
- Geradas e publicadas 14 SPECs: SPEC-4-001 (modelo de dados reais, catálogos e funis como dados), SPEC-4-002 (entrada de leads e captura automática), SPEC-4-003 (importação assistida), SPEC-4-004 (ficha do lead e Kanban), SPEC-4-005 (fila do dia e cadência), SPEC-4-006 (prospecção e listas), SPEC-4-007 (agendamento e briefing), SPEC-4-008 (agentes de IA configuráveis), SPEC-4-009 (indicadores e dashboard de anúncios), SPEC-4-010 (administração), SPEC-4-011 (central de integrações), SPEC-4-012 (CRM personalizável: funis, etapas, campos, card e automações), SPEC-4-013 (conversas pelo WhatsApp com a Z-API), SPEC-4-014 (entrada em operação e aceite).
- 57 critérios de aceite (CA-4-01 a CA-4-57), 76 tasks T4.1–T4.76 em 14 levas (A–N).
- Novos documentos na pasta da fase: `integracoes.md`, `importacao-de-dados.md`, `decisoes-go-live.md`, `modelos/` (5 modelos de arquivo) e `fixtures/` (8 conjuntos de dados fictícios, com os resultados esperados em `fixtures/LEIA-ME.md`).
- Removidos da fase atual: as SPECs e fixtures do recorte anterior (SPEC-4-001 a SPEC-4-005 antigas; fixtures de disparo e de reativação). Continuam no histórico do repositório.
- Alterações de propósito em relação às fases anteriores, a registrar na regressão: importação parcial por padrão (CA-1-02), etapas configuráveis com regras por categoria (CA-1-04), campos reais com e-mail opcional, canal externo bloqueado só enquanto a chave do canal estiver desligada (CA-3-07), agentes com modelo de linguagem (CA-3-02).
- Canal de WhatsApp definido: Z-API. Gate G6 passa a ter contrato técnico documentado.

## 07/10/2026

- Fase 4 (Follow-up, reativação e agendamento) aberta pela consultora Kim Prestes; Fase 3 arquivada em `05_entregas/fase-3/` com conclusão registrada em 07/10/2026.
- Geradas e publicadas 5 SPECs da Fase 4: SPEC-4-001 (régua de follow-up versionada), SPEC-4-002 (segmento de reativação com prévia e exclusões), SPEC-4-003 (disparo governado com aprovação humana e idempotência), SPEC-4-004 (agendamento válido com histórico preservado), SPEC-4-005 (handoff estruturado ao apresentador e recibo final).
- 7 critérios de aceite (CA-4-01..07), 17 tasks T4.1–T4.17 em 11 levas (A–K), fixtures sintéticas (cadências, reativação, disparo, agendamentos).
- CA-4-04 exige aprovação humana e canal autorizado para o disparo real; sem G5/G6 a operação roda em simulação auditada; G4 condiciona valores oficiais da régua; G7 condiciona a agenda externa.

## 06/10/2026

- Fase 3 (CRM conversacional e qualificação assistida) aberta pela consultora Kim Prestes; Fase 2 arquivada em `05_entregas/fase-2/` com conclusão registrada em 06/10/2026.
- Geradas e publicadas 7 SPECs da Fase 3: SPEC-3-001 (conversa unificada e ingestão idempotente), SPEC-3-002 (agente assistivo com base aprovada versionada), SPEC-3-003 (transbordo humano com retomada explícita), SPEC-3-004 (qualificação assistida com ICP versionado), SPEC-3-005 (opt-out e supressão), SPEC-3-006 (configuração administrativa com publicação e rollback), SPEC-3-007 (degradação segura e bloqueio do canal sem gates).
- 7 critérios de aceite (CA-3-01..07), 15 tasks T3.1–T3.15 em 12 levas (A–L), fixture `conversas-fase3.csv` sintética.
- CA-3-07 exige atendimento humano funcional sem a IA e canal externo bloqueado até G1/G5/G6; gates G1/G2/G5/G6 condicionam canal real e ICP definitivo.

## 28/09/2026

- Fase 2 (Inteligência de mídia) aberta pela consultora Kim Prestes; Fase 1 arquivada em `05_entregas/fase-1/`.
- Geradas e publicadas 4 SPECs da Fase 2: SPEC-2-001 (importação de mídia validada), SPEC-2-002 (painel CPLQ reconciliável), SPEC-2-003 (recomendação explicável com aprovação humana), SPEC-2-004 (feedback Marketing–Comercial + degradação segura).
- 6 critérios de aceite (CA-2-01..06), 8 tasks T2.1–T2.8 em 7 levas (A–G), fixture `midia-fase2.csv` sintética.
- CA-2-06 exige que o funil da Fase 1 opere sem a camada de mídia; gates G3/G6 seguem condicionando metas e API de Ads.

## 15/09/2026

- Criadas três SPECs executáveis para a Fase 1.
- Criadas nove tasks da Fase 1.
- Sincronizados índice de SPECs, tasks gerais, Jornada e matriz de rastreabilidade.