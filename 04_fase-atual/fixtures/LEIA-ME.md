# Fixtures da Fase 4

Todos os dados desta pasta são **fictícios**: nomes com o sufixo "Sintético", domínios `example.invalid` e telefones inventados. Servem para os testes das tasks; nenhum deles deve ser usado em produção.

| Arquivo | Usado por | O que testa |
|---|---|---|
| `leads-planilha-fase4.csv` | SPEC-4-001, 4-003, 4-009 | importação no formato da planilha real, com defeitos conhecidos |
| `base-prospeccao-fase4.csv` | SPEC-4-003, 4-006 | base de prospecção e exclusões de lista |
| `midia-fase4.csv` | SPEC-4-003, 4-009 | números diários de anúncio com totais conhecidos |
| `cadencias-fase4.csv` | SPEC-4-005 | cálculo da próxima ação pela régua |
| `agendamentos-fase4.csv` | SPEC-4-007 | estados do agendamento e contagem por série |
| `captura-fase4.json` | SPEC-4-002, 4-011 | rota de captura de lead |
| `zapi-eventos-fase4.json` | SPEC-4-013, 4-009 | eventos da Z-API, inclusive anúncio de origem |
| `agentes-cenarios-fase4.json` | SPEC-4-008 | os oito cenários prontos dos agentes |

## Resultado esperado de `leads-planilha-fase4.csv`

Arquivo separado por ponto e vírgula, com **cabeçalho em duas linhas** (indicar a segunda). 16 linhas de dados, uma delas em branco.

| Resultado | Quantidade | Linhas de dados |
|---|---|---|
| Criados | 11 | 1 a 8, 12, 13, 15 |
| Pulados por duplicidade dentro do arquivo | 1 | 11 (mesmo telefone da linha 1, em outro formato) |
| Rejeitados | 3 | 9 (telefone sem DDD), 10 (sem telefone e sem e-mail), 16 (data de entrada ilegível) |
| Ignorados | 1 | 14 (linha em branco) |

| Etapa (funil padrão) | Quantidade | Linhas |
|---|---|---|
| Novo | 2 | 12, 13 |
| Em contato | 1 | 2 |
| Qualificado | 2 | 1, 15 |
| Agendado (registro histórico) | 2 | 4 (proposta enviada), 5 (ganho) |
| Desqualificado | 3 | 3 e 8 (sem interesse), 6 (fechou com concorrente) |
| Nutrição | 1 | 7 (observação começa com `NUTRIR`) |

Outros resultados a conferir: 2 contatos suprimidos (linhas 3 e 8); 14 tentativas criadas; a linha 15 fica com telefone secundário, segmento "A classificar" e uma tentativa com a data de entrada e o texto original na nota; a linha 12 entra sem e-mail; o nome da linha 13 é gravado sem o prefixo perigoso; "Trafego pago " (linha 3) é reconhecido como o mesmo valor de "Tráfego Pago"; as linhas de tráfego pago sem campanha ficam com origem "Desconhecida" e o detalhe "Tráfego pago (canal não informado)".

## Resultado esperado de `base-prospeccao-fase4.csv`

12 linhas. Criados como prospect: 8 (linhas 1 a 8). Rejeitados: 3 — linha 9 (situação cadastral não ativa), linha 10 (sem telefone), linha 12 (CNPJ inválido). Duplicidade: linha 11 (mesmo telefone da linha 3).

Para a prova de exclusões da lista (SPEC-4-006), prepare antes: registrar "sem interesse" para os prospects 02 e 05; marcar o telefone do prospect 06 como inválido; exportar uma lista só com o prospect 07 há menos de 30 dias. Lista nova sobre a base inteira, com o intervalo padrão de 30 dias: encontrados 8, excluídos 4 (2 suprimidos, 1 telefone inválido, 1 disparo recente), na lista 4 (prospects 01, 03, 04 e 08).

## Resultado esperado de `midia-fase4.csv`

Setembro de 2026. Google: R$ 3.600,00 em uma campanha, dois conjuntos (R$ 1.800,00 cada), três anúncios (R$ 1.200,00, R$ 600,00 e R$ 1.800,00). Meta: R$ 1.200,00 em uma campanha, dois conjuntos (R$ 800,00 e R$ 400,00), três anúncios (R$ 400,00 cada). Total: R$ 4.800,00.

Com 6 leads qualificados atribuídos à campanha do Google e 3 agendamentos: CPLQ de R$ 600,00 e custo por agendamento de R$ 1.200,00.

## Régua usada em `cadencias-fase4.csv`

Régua padrão: 3 tentativas; intervalos de 1, 2 e 4 dias úteis; canais WhatsApp, ligação, WhatsApp; janela das 9h às 18h, de segunda a sexta. Régua de indicação: 5 tentativas; intervalos de 0, 1, 2, 3 e 5 dias úteis; canais alternados, começando por WhatsApp. As datas foram calculadas sem feriados.
