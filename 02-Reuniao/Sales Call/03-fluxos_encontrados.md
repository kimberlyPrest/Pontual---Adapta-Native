# Fluxos encontrados

- Reunião: [kick-off] Adapta Native & Pontual Tecnologia
- Data: 2026-08-11
- Link tl;dv: https://tldv.io/app/meetings/6a7b712759bb83001320b864
- Fonte: transcrição e metadados retornados pela API do tl;dv.

## Observação
Os fluxos abaixo representam o processo atual de aquisição comercial e o processo futuro/desejado de inteligência financeira citado na reunião.

## Processo Atual de Aquisição e Qualificação Comercial (atual)
```mermaid
flowchart TD
    A["Anúncios no Google Ads e Meta Ads"] --> B["Entrada do Lead por Formulário ou WhatsApp"]
    B --> C["Cadastro Automático no CRM MoveDesk"]
    C --> D["Ações do Bot Pontinho e Atuação do SDR"]
    D -->|"Lead Qualificado"| E["Apresentação com Vendedora Lidiane e Time Técnico"]
    D -->|"Sem Contato (até 7 tentativas)"| F["Cadência de Nutrição do CRM"]
    E --> G["Envio de Proposta Comercial (R$ 5k ou R$ 8k)"]
    G --> H["Acompanhamento e Fechamento"]
```

## Integração de Inteligência Financeira e DRE no ERP (desejado)
```mermaid
flowchart TD
    A["Dados Financeiros no ERP Pontual"] --> B["Conexão via API com Ethos / LLM"]
    B --> C["Análise em Tempo Real de DRE e Margens"]
    C --> D["Geração de Recomendações de Precificação e Redução de Despesas"]
    D --> E["Apresentação de Insights Diretos ao Cliente Final"]
```
