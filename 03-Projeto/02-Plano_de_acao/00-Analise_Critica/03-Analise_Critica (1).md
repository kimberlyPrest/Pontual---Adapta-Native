# Análise Crítica Preliminar — Pontual Tecnologia

**Data:** 2026-08-28  
**Rota:** profunda — caminhos materialmente diferentes, dependências externas, métricas contraditórias e alto risco de construir uma plataforma antes de validar aquisição e agendamento.  
**Revisores:** revisor-de-plano, revisor-adversarial, revisor-viabilidade, guardião-de-escopo e explorador-de-alternativas.

## Veredito curto
A proposta é uma boa base de descoberta, mas precisa de corte e homologação antes do escopo final. O projeto recomendado é uma **Central de Aquisição e Qualificação**, limitada a origem → triagem → qualificação → fila do SDR → agendamento, com feedback de qualidade para Marketing. Não recomendo começar substituindo todo o CRM/WhatsApp nem prometer que organização operacional, sozinha, recuperará aquisição.

## Achados graves

### AC-001 — O sistema não gera demanda por si só
- **Evidência:** kickoff e briefing localizam a deterioração em Google/Meta e no perfil dos leads; vídeos mostram controles internos.
- **Cenário:** CRM melhor, mas continuam poucos leads qualificados.
- **Fazer:** incluir rastreabilidade de campanha e loop Marketing–Comercial.

### AC-002 — Não existe fonte da verdade definida
- **Evidência:** DMO/kickoff citam MoveDesk; vídeos citam CRM Pontual, RespondeChat e planilha.
- **Cenário:** duplicidade, status conflitante e opt-out não propagado.
- **Fazer:** escolher sistema oficial e estratégia de transição.

### AC-003 — Metas e evento final divergem
- **Evidência:** briefing = 20 qualificados, 10 realizadas, CPLQ ≤R$200; mapa = 25, 8, R$160; consultora termina no agendamento.
- **Cenário:** vereditos opostos sobre sucesso.
- **Fazer:** homologar definição, fórmula, janela, meta e evento final.

### AC-004 — As duas pontas críticas não foram demonstradas
- **Evidência:** campanhas/formulários e agendamento/calendário não aparecem nos vídeos.
- **Cenário:** solução detalha o meio e inventa entrada/saída.
- **Fazer:** mapear campanha→captura e qualificado→agendamento.

### AC-005 — ICP não homologado por produto
- **Evidência:** Playbook em elaboração; regra do Hiper pode não valer para Automec/Climb.
- **Cenário:** classificação exclui bons leads ou infla qualificados.
- **Fazer:** matriz de ICP e decisão humana no MVP.

### AC-006 — WhatsApp sem governança suficiente
- **Evidência:** bases frias, disparos, falha de cadência, pergunta sobre origem e expurgo manual.
- **Cenário:** bloqueio do canal e risco jurídico/reputacional.
- **Fazer:** base legal, elegibilidade, opt-out, retenção, auditoria e contingência.

### AC-007 — Baseline não congelado/auditável
- **Evidência:** números relatados; mapa solicita 12 meses por canal; definições variam.
- **Cenário:** antes/depois mede mudança de critério ou sazonalidade.
- **Fazer:** consolidar baseline com dicionário de eventos.

### AC-008 — Agendamento sem definição operacional
- **Evidência:** não há calendário, disponibilidade técnica, confirmação e reagendamento demonstrados.
- **Cenário:** status “agendado” sem compromisso válido.
- **Fazer:** definir evento, responsável, calendário e exceções.

## Achados moderados

### AC-009 — Cadência contraditória
Três tentativas nos vídeos/escopo versus sete no kickoff/briefing. Validar por origem/produto.

### AC-010 — MVP reúne capacidades demais
Começar por origem, qualificação, fila e agendamento; integrações completas depois.

### AC-011 — Deduplicação/migração carecem de regra
Definir identidade, precedência e teste com amostra antes de migração integral.

### AC-012 — Donos e cadência de decisão incompletos
Definir RACI e ritual Marketing–Comercial.

## Decisões humanas

| ID | Decisão | Opções | Recomendação |
|---|---|---|---|
| D1 | Produto inicial | Cockpit; Central; Plataforma | Central enxuta |
| D2 | Fonte oficial | MoveDesk; CRM Pontual; nova camada | após auditoria técnica |
| D3 | Meta | 20/10/R$200; 25/8/R$160; outra | baseline antes da meta final |
| D4 | Evento final | agendada; realizada | agendada |
| D5 | Cadência | 3; 7; por origem | por origem/temperatura |
| D6 | WhatsApp | integrar; registrar; excluir | registro/supressão primeiro |
| D7 | Integração | APIs; importação assistida | importação + 1 integração confirmada |

## O que está sólido
- Problema principal antes da apresentação.
- Conversão pós-apresentação relatada como forte, a validar.
- Planilhas/etiquetas geram retrabalho e baixa rastreabilidade.
- Qualificação deve continuar humana.
- Proposta, assinatura, implantação, CS e comissões ficam fora.

## Próximo passo
A consultora preenche `04-Analise_do_Consultor.md`, resolve D1–D7 e solicita itens do `05-Check_Input.md`. Só então rodar `escopo-final`.