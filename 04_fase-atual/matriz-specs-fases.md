# Matriz de SPECs e Fases — Fase 4 (fase final)

| SPEC | Resultado observável | RFs | RNs | RNFs | CAs | Gates | Tasks |
|---|---|---|---|---|---|---|---|
| SPEC-4-001 | Lead com campos reais, catálogos e funis/etapas como dados | RF-02, RF-04, RF-06, RF-07, RF-16 | RN-01, RN-02, RN-05, RN-08 | — | CA-4-01 a CA-4-03 | G1, G2 | T4.4, T4.5, T4.6, T4.7, T4.8 |
| SPEC-4-002 | Cadastro manual, aviso de duplicidade e captura automática | RF-01, RF-02, RF-03, RF-22 | RN-02, RN-14, RN-15 | — | CA-4-04 a CA-4-06 | G6 | T4.22, T4.23, T4.25 |
| SPEC-4-003 | Assistente de importação que aceita as planilhas reais, com desfazer | RF-03, RF-10, RF-19, RF-22 | RN-02, RN-14, RN-15 | RNF-05, RNF-09 | CA-4-07 a CA-4-09 | G1, G5 | T4.26, T4.27, T4.28, T4.29, T4.30, T4.31 |
| SPEC-4-004 | Ficha completa do lead, Kanban de trabalho e lista de leads | RF-04, RF-05, RF-06, RF-07, RF-08, RF-09, RF-10, RF-15, RF-23 | RN-03, RN-05, RN-06, RN-07 | — | CA-4-10 a CA-4-15 | G2 | T4.9, T4.10, T4.11, T4.12, T4.13, T4.14, T4.15, T4.16 |
| SPEC-4-005 | Fila do dia, tentativas e régua de cadência | RF-05, RF-07, RF-08, RF-13 | RN-05, RN-07, RN-08, RN-13 | — | CA-4-16 a CA-4-19 | G4 | T4.32, T4.33, T4.34, T4.35 |
| SPEC-4-006 | Bases de prospecção, listas com exclusões e exportação | RF-10, RF-13, RF-14, RF-15 | RN-03, RN-07 | — | CA-4-20 a CA-4-23 | G5 | T4.50, T4.51, T4.52, T4.53 |
| SPEC-4-007 | Agendamento com confirmação, briefing, convite e resultado | RF-11, RF-16, RF-17 | RN-01, RN-09, RN-10 | — | CA-4-24 a CA-4-26 | G7 | T4.36, T4.37, T4.38, T4.39 |
| SPEC-4-008 | Agentes de IA configuráveis pela interface e copiloto | RF-09, RF-10, RF-11, RF-23 | RN-03, RN-04, RN-13, RN-14 | RNF-03, RNF-10 | CA-4-27 a CA-4-32 | G5 | T4.56, T4.57, T4.58, T4.59, T4.60, T4.61, T4.62 |
| SPEC-4-009 | Funil, produtividade, dashboard de anúncios, parceiro e metas | RF-12, RF-13, RF-18, RF-19, RF-20, RF-21 | RN-02, RN-11, RN-12 | — | CA-4-33 a CA-4-37 | G3, G6 | T4.63, T4.64, T4.65, T4.66, T4.67, T4.68, T4.69 |
| SPEC-4-010 | Usuários, catálogos, régua, modelos, ICP, base aprovada e auditoria | RF-12, RF-14, RF-15, RF-23, RF-24 | RN-13 | RNF-01, RNF-02, RNF-08 | CA-4-38 a CA-4-40 | G2, G4 | T4.40, T4.41, T4.42, T4.43 |
| SPEC-4-011 | Chaves, registro de eventos, fila de exceção e painel de situação | RF-09, RF-22 | RN-14 | RNF-05, RNF-10 | CA-4-41 a CA-4-42 | G6 | T4.24, T4.70, T4.71 |
| SPEC-4-012 | Funis, etapas, campos, ficha, card e automações pela interface | RF-03, RF-06, RF-12, RF-14 | RN-13 | — | CA-4-43 a CA-4-46 | G2 | T4.17, T4.18, T4.19, T4.20, T4.21 |
| SPEC-4-013 | WhatsApp dentro da Central pela Z-API; envio de listas e de régua | RF-09, RF-10, RF-13, RF-15 | RN-04, RN-05, RN-07, RN-14 | RNF-05, RNF-10 | CA-4-47 a CA-4-52 | G5, G6 | T4.3, T4.44, T4.45, T4.46, T4.47, T4.48, T4.49, T4.54, T4.55 |
| SPEC-4-014 | Segurança, decisões, carga real, validação ponta a ponta e aceite | RF-15, RF-24 | — | RNF-01, RNF-02, RNF-03, RNF-04 | CA-4-53 a CA-4-57 | G1–G7 | T4.1, T4.2, T4.72, T4.73, T4.74, T4.75, T4.76 |

## Cobertura de critérios de aceite

| CA | SPEC | Tasks de prova |
|---|---|---|
| CA-4-01 | SPEC-4-001 | T4.4, T4.5 |
| CA-4-02 | SPEC-4-001 | T4.6 |
| CA-4-03 | SPEC-4-001 | T4.7, T4.8 |
| CA-4-04 | SPEC-4-002 | T4.22 |
| CA-4-05 | SPEC-4-002 | T4.23 |
| CA-4-06 | SPEC-4-002 | T4.25 |
| CA-4-07 | SPEC-4-003 | T4.26, T4.27, T4.30 |
| CA-4-08 | SPEC-4-003 | T4.28 |
| CA-4-09 | SPEC-4-003 | T4.29, T4.31 |
| CA-4-10 | SPEC-4-004 | T4.9, T4.10, T4.16 |
| CA-4-11 | SPEC-4-004 | T4.11 |
| CA-4-12 | SPEC-4-004 | T4.12 |
| CA-4-13 | SPEC-4-004 | T4.13 |
| CA-4-14 | SPEC-4-004 | T4.14 |
| CA-4-15 | SPEC-4-004 | T4.15 |
| CA-4-16 | SPEC-4-005 | T4.32 |
| CA-4-17 | SPEC-4-005 | T4.33 |
| CA-4-18 | SPEC-4-005 | T4.34 |
| CA-4-19 | SPEC-4-005 | T4.35 |
| CA-4-20 | SPEC-4-006 | T4.50 |
| CA-4-21 | SPEC-4-006 | T4.51 |
| CA-4-22 | SPEC-4-006 | T4.52 |
| CA-4-23 | SPEC-4-006 | T4.53 |
| CA-4-24 | SPEC-4-007 | T4.36, T4.39 |
| CA-4-25 | SPEC-4-007 | T4.37 |
| CA-4-26 | SPEC-4-007 | T4.38 |
| CA-4-27 | SPEC-4-008 | T4.57, T4.59 |
| CA-4-28 | SPEC-4-008 | T4.56, T4.60 |
| CA-4-29 | SPEC-4-008 | T4.58, T4.60 |
| CA-4-30 | SPEC-4-008 | T4.61 |
| CA-4-31 | SPEC-4-008 | T4.56 |
| CA-4-32 | SPEC-4-008 | T4.62 |
| CA-4-33 | SPEC-4-009 | T4.63 |
| CA-4-34 | SPEC-4-009 | T4.64 |
| CA-4-35 | SPEC-4-009 | T4.68 |
| CA-4-36 | SPEC-4-009 | T4.65, T4.66, T4.67 |
| CA-4-37 | SPEC-4-009 | T4.69 |
| CA-4-38 | SPEC-4-010 | T4.40 |
| CA-4-39 | SPEC-4-010 | T4.41, T4.42 |
| CA-4-40 | SPEC-4-010 | T4.43 |
| CA-4-41 | SPEC-4-011 | T4.24, T4.70 |
| CA-4-42 | SPEC-4-011 | T4.70, T4.71 |
| CA-4-43 | SPEC-4-012 | T4.19 |
| CA-4-44 | SPEC-4-012 | T4.17, T4.18 |
| CA-4-45 | SPEC-4-012 | T4.20 |
| CA-4-46 | SPEC-4-012 | T4.21 |
| CA-4-47 | SPEC-4-013 | T4.3, T4.44 |
| CA-4-48 | SPEC-4-013 | T4.45, T4.46, T4.47 |
| CA-4-49 | SPEC-4-013 | T4.45, T4.48 |
| CA-4-50 | SPEC-4-013 | T4.46, T4.49 |
| CA-4-51 | SPEC-4-013 | T4.54, T4.55 |
| CA-4-52 | SPEC-4-013 | T4.48 |
| CA-4-53 | SPEC-4-014 | T4.1 |
| CA-4-54 | SPEC-4-014 | T4.2 |
| CA-4-55 | SPEC-4-014 | T4.72, T4.73 |
| CA-4-56 | SPEC-4-014 | T4.74 |
| CA-4-57 | SPEC-4-014 | T4.75, T4.76 |

## Tasks por leva

| Leva | Assunto | Tasks | Quantidade |
|---|---|---|---|
| A | Fundação: segurança, decisões, contratações e modelo de dados | T4.1, T4.2, T4.3, T4.4, T4.5, T4.6, T4.7, T4.8 | 8 |
| B | Ficha do lead, Kanban e lista | T4.9, T4.10, T4.11, T4.12, T4.13, T4.14, T4.15, T4.16 | 8 |
| C | Personalização: campos, funis, automações e layout | T4.17, T4.18, T4.19, T4.20, T4.21 | 5 |
| D | Entrada de leads e base das integrações | T4.22, T4.23, T4.24, T4.25 | 4 |
| E | Importação de dados | T4.26, T4.27, T4.28, T4.29, T4.30, T4.31 | 6 |
| F | Fila do dia e cadência | T4.32, T4.33, T4.34, T4.35 | 4 |
| G | Agendamento e briefing | T4.36, T4.37, T4.38, T4.39 | 4 |
| H | Administração | T4.40, T4.41, T4.42, T4.43 | 4 |
| I | Conversas pelo WhatsApp | T4.44, T4.45, T4.46, T4.47, T4.48, T4.49 | 6 |
| J | Prospecção, listas e envios | T4.50, T4.51, T4.52, T4.53, T4.54, T4.55 | 6 |
| K | Agentes de IA | T4.56, T4.57, T4.58, T4.59, T4.60, T4.61, T4.62 | 7 |
| L | Indicadores e dashboard de anúncios | T4.63, T4.64, T4.65, T4.66, T4.67, T4.68, T4.69 | 7 |
| M | Central de integrações | T4.70, T4.71 | 2 |
| N | Entrada em operação e aceite | T4.72, T4.73, T4.74, T4.75, T4.76 | 5 |

## Tasks que dependem de pessoas ou de terceiros

| Task | Quem | O quê |
|---|---|---|
| T4.2 | Consultora + Pontual (Marcelo e Lidiane) | Conduzir a reunião de decisões e preencher decisoes-go-live.md |
| T4.3 | Pontual (Marcelo) | Contratar a Z-API e criar uma instância por número |
| T4.73 | Pontual (Lidiane), com a consultora | Executar a carga real na ordem de importacao-de-dados.md: planilha de funil, leads do parceiro, bases de prospecção, suprimidos, anúncios |
| T4.74 | QA + Pontual (Lidiane e Marcelo) | Reexecutar os critérios das fases 1 a 3 e executar o roteiro ponta a ponta com dados reais, sem ajuda |
| T4.75 | Consultora | Entregar o guia rápido, o guia do administrador (com ajuda dentro do sistema) e o roteiro de operação técnica |
| T4.76 | Consultora + Pontual (Marcelo) | Emitir o recibo final da fase com evidência por critério, situação dos gates, conciliação, números da primeira semana e lista de evoluções |
