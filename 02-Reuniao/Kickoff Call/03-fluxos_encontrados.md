# Fluxos encontrados

- Reunião: [kick-off] Adapta Native & Pontual Tecnologia
- Data: 2026-08-11
- Link tl;dv: https://tldv.io/app/meetings/6a7b712759bb83001320b864
- Fonte: transcrição e metadados retornados pela API do tl;dv.

## Observação
O fluxo abaixo descreve o processo atual de aquisição e atendimento comercial relatado na reunião.

## Processo Comercial e de Aquisição de Leads (atual)
```mermaid
flowchart TD
    A["Anúncios Google Ads / Meta Ads / Indicações"] --> B["Entrada do Lead via WhatsApp ou Formulário"]
    B --> C["Integração Automática no CRM MoveDesk"]
    C --> D["Tentativas de Contato e Nutrição pelo Bot Pontinho e SDR"]
    D -- "Sem sucesso após 7 tentativas" --> E["Envio para Quadrante de Nutrição Periódica no CRM"]
    D -- "Contato com sucesso" --> F["Qualificação do Lead pelo SDR"]
    F -- "Lead Qualificado" --> G["Apresentação Demonstrativa com SDR e Apoio Técnico"]
    G --> H["Confecção e Envio da Proposta Comercial"]
    H --> I["Follow-up e Convencimento Comercial"]
    I --> J["Fechamento e Assinatura do Contrato"]
```
