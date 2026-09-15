# Revisão Multipersona da Proposta — Pontual Tecnologia

**Data:** 2026-08-28  
**Run:** `rv-pontual-001`  
**Cobertura:** revisor-de-plano, revisor-adversarial, revisor-viabilidade, guardião-de-escopo e explorador-de-alternativas.  
**Estado:** completa, porém preliminar por ausência de check-input aprovado.

## Veredito
O escopo atual identifica corretamente o gargalo antes da apresentação, mas ainda não é executável como contrato. Ele propõe uma plataforma ampla antes de confirmar a fonte da verdade, os acessos e o fluxo real de aquisição/agendamento; também usa metas contraditórias e confunde agendamento com apresentação realizada.

## Achados

### RV-001 — Solução operacional não corrige sozinha a aquisição
- **Gravidade:** grave
- **Cenário:** sistema organiza poucos leads ruins, sem recuperar volume/CPLQ.

### RV-002 — Fonte da verdade e integrações indefinidas
- **Gravidade:** grave
- **Evidência:** MoveDesk, CRM Pontual, RespondeChat e planilha aparecem como controles.

### RV-003 — Critério de sucesso contraditório
- **Gravidade:** grave
- **Evidência:** 20/10/R$200 versus 25/8/R$160; agendada versus realizada.

### RV-004 — Aquisição e agendamento não observados ponta a ponta
- **Gravidade:** grave
- **Ação:** obter campanha→captura e qualificado→agenda.

### RV-005 — Cadência conflitante
- **Gravidade:** moderada
- **Evidência:** três versus sete tentativas.

### RV-006 — WhatsApp, opt-out e LGPD insuficientes
- **Gravidade:** grave
- **Cenário:** contato indevido, bloqueio do número e risco reputacional.

### RV-007 — Escopo amplo demais para o primeiro degrau
- **Gravidade:** moderada
- **Ação:** núcleo origem→qualificação→agendamento.

### RV-008 — ICP não homologado
- **Gravidade:** grave
- **Ação:** matriz por produto e decisão humana no MVP.

### RV-009 — Baseline não comparável/auditável
- **Gravidade:** grave
- **Ação:** 12 meses por canal e dicionário de eventos.

### RV-010 — Deduplicação e migração indefinidas
- **Gravidade:** moderada
- **Ação:** identidade, precedência e prova com amostra.

### RV-011 — Agendamento sem contrato operacional
- **Gravidade:** grave
- **Ação:** evento, calendário, responsável e exceções.

### RV-012 — Responsabilidade fragmentada
- **Gravidade:** moderada
- **Ação:** RACI e ritual Marketing–Comercial.

## Pontos sólidos
- Gargalo sustentado antes da apresentação.
- Fronteira pós-agendamento explicitamente excluída.
- Retrabalhos bem documentados.
- Julgamento humano preservado na qualificação.
- Potencial para eliminar controles paralelos após escolha da fonte oficial.