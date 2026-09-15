# SPEC-1-003 — Painel básico de entradas por origem

**Fase:** 1  
**Status:** planejada  
**Dono:** Desenvolvimento/Produto, com validação da gestão  
**Origem no escopo:** RF-002, RF-012, Fase 1  
**Degrau da solução:** construção mínima — painel derivado dos registros do cockpit, sem integração de mídia.

## Contexto e decisões fechadas

- **Estado atual:** contagens e qualidade são atualizadas manualmente em planilhas; origem nem sempre é confiável.
- **Estado desejado:** gestão vê entradas de teste agrupadas por origem e reconhece registros sem origem.
- **Decisões já fechadas:** painel não atribui resultado a campanha inexistente e não promete performance de mídia.
- **Bloqueios:** nenhum para o painel de teste; eventos oficiais e baseline histórico ficam fora desta fase.

## Resultado observável

Um painel de teste com total de leads, distribuição por origem e quantidade sem origem conhecida, atualizado a partir dos registros do cockpit.

## Limites e dependências

- **Inclui:** totais, filtros de período/origem e indicação de dados incompletos.
- **Fora de escopo:** CPLQ oficial, integração Google/Meta, recomendações de campanha e dashboard executivo completo.
- **Entradas e pré-condições:** dados de teste criados na SPEC-1-001 e catálogo provisório de origens.
- **Saídas/artefatos:** painel e exportação da visão de teste.
- **Dependências:** SPEC-1-001; gestão valida a leitura.
- **Atores/permissões:** gestão consulta; SDR consulta seu recorte; administrador configura catálogo.
- **Superfícies afetadas:** painel da aplicação.
- **Risco/plano B:** se a origem faltar, exibir `Desconhecida` e métrica de incompletude.
- **Rollback:** remover dados de teste sem alterar regras de origem.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| registros do cockpit → painel | registros de teste da SPEC-1-001 | origem, data de criação, etapa | usuário autenticado | consulta somente leitura | estado vazio e origem desconhecida explícitos |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-6 | origem ausente | agrupar em `Desconhecida` | não inferir origem | RF-002 |
| RN-7 | sem registros no período | mostrar zero e estado vazio | não ocultar filtro | RF-012 |

## Fluxo e regras

1. Usuário autorizado abre o painel.
2. Sistema carrega registros do período padrão.
3. Usuário filtra por período/origem.
4. Sistema exibe totais, distribuição e incompletude.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | registros com origens variadas | totais conferem com a base de teste | — |
| Limite | nenhum registro | zero e mensagem de estado vazio | — |
| Falha | origem ausente | `Desconhecida` e alerta de incompletude | corrigir registro na origem |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC, SPEC-1-001 e escopo definitivo, seções 4 e Fase 1.
2. **Alterar somente:** consulta, filtros e visualização do painel de teste.
3. **Não alterar:** cálculo de CPLQ oficial, integrações de anúncio e campanhas.
4. **Executar nesta ordem:** preparar fixture; RED; implementar consulta; GREEN; testar vazio/incompleto.
5. **Parar e pedir validação quando:** a execução tentar publicar fórmula oficial ou conectar mídia de produção.
6. **Estado válido ao parar:** painel somente leitura e coerente com a base de teste.

## Checklist de execução

- [ ] totais conferidos;
- [ ] filtro por período e origem exercitado;
- [ ] estado vazio testado;
- [ ] origem desconhecida evidenciada;
- [ ] permissão de consulta testada;
- [ ] gestão validou a demonstração.

## Critérios de aceite

- [ ] **CA-1-010:** totais do painel correspondem aos registros de teste.
- [ ] **CA-1-011:** filtro por origem e período altera a visão corretamente.
- [ ] **CA-1-012:** origem desconhecida e dados incompletos ficam explícitos.
- [ ] **CA-1-013:** usuário sem permissão não acessa dados protegidos.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | painel sem consulta/filtro | executar `F1-PAINEL-01` | falha nos critérios principais | log |
| GREEN | base de teste com origens | executar `F1-PAINEL-01/02` | CA-1-010/011 passam | captura |
| REFACTOR/REGRESSÃO | vazio, desconhecido e permissão | executar `F1-PAINEL-03` | CA-1-012/013 passam | log/captura |

**Dados/fixtures:** cinco registros de teste, três origens, um desconhecido e dois períodos.  
**Caminhos de erro obrigatórios:** estado vazio, origem ausente e usuário sem permissão.  
**Evidência exigida:** captura do painel, comparação com fixture e aceite da gestão.

## Handoff e operação

- **Como demonstrar:** abrir painel → filtrar período → filtrar origem → mostrar desconhecidos.
- **Como operar depois:** gestão consulta; correções são feitas no cadastro do lead.
- **Como monitorar:** divergência entre contagem do painel e registros.
- **Pendência conhecida:** baseline e fórmula de indicadores de mídia.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T07 | Criar fixture e consulta do painel | Desenvolvimento | SPEC-1-003 | CA-1-010/011 | F1-PAINEL-01/02 | captura + comparação | SPEC-1-001 aprovada | ☐ |
| F1-T08 | Tratar vazio, desconhecido e permissão | Desenvolvimento/QA | SPEC-1-003 | CA-1-012/013 | F1-PAINEL-03 | logs/capturas | F1-T07 aprovada | ☐ |
| F1-T09 | Realizar demonstração e aceite da gestão | Gestão/Champion | SPEC-1-003 | CA-1-010–013 | roteiro completo | aceite registrado | F1-T08 aprovada | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
