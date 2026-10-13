# Importação de dados — guia da carga

**Projeto:** Pontual Tecnologia — Central de Aquisição e Qualificação de Leads  
**Para quem é:** Lidiane e Marcelo, que vão trazer os dados da operação para a Central; e quem implementa, para saber exatamente o que cada planilha precisa virar.  
**O que você vai conseguir com ele:** colocar na Central a planilha de funil, as bases de prospecção, as listas do RespondeChat e os números de anúncio, sem editar os arquivos à mão.

O comportamento do assistente de importação está na SPEC-4-003. A sequência oficial da carga inicial, com as conferências, está na SPEC-4-014, Parte 3.

---

## 1. O que entra e em que ordem

| Ordem | O quê | De onde vem | Vira na Central | Perfil de mapeamento |
|---|---|---|---|---|
| 1 | Funil comercial | aba **CRM - Funil de Vendas** do arquivo "CRM - Acompanhamento - ok" (SharePoint) | leads, com etapa, tentativas e observações | Planilha CRM – Funil de Vendas |
| 2 | Leads do parceiro | aba **Leads Hiper - Trafego Pago** | leads com origem "Parceiro Hiper" | Leads Hiper – tráfego pago |
| 3 | Bases de prospecção | planilha "Base de contatos" (Google Sheets) enviada pela parceira | prospects (ficam fora do Kanban até responderem) | Base de prospecção (cadastro CNPJ) |
| 4 | Listas já disparadas | arquivos no modelo do RespondeChat | prospects com a etiqueta da lista | Lista RespondeChat |
| 5 | Quem já recusou | linhas vermelhas e "Sem interesse" da planilha | contatos suprimidos | (vem junto do item 1; conferir no item 6.4) |
| 6 | Anúncios | relatórios do Meta Ads e do Google Ads, ou sincronização | dashboard de anúncios | Relatório Meta Ads · Relatório Google Ads |
| 7 | Conversas antigas (opcional) | exportação de conversas, se houver | histórico na ficha | Modelo de conversas |

Faça nesta ordem: os leads primeiro, para que prospects e conversas encontrem quem já existe e não dupliquem.

**Não entram:** abas "Rotina Diária", "Resumo Semanal", "KPIs" e os planejamentos mensais. A Central calcula esses números sozinha a partir do que for registrado nela daqui para a frente (SPEC-4-009).

---

## 2. Antes de começar

1. **Autorização.** A carga de dados reais só é liberada depois da reunião de decisões (T4.2). Enquanto aparecer a faixa "Ambiente de teste", use só dados fictícios.
2. **Funil e campos prontos.** Confira em Configurações → Funis e etapas se as etapas estão como a Pontual quer, e em Configurações → Campos se as colunas da planilha que não existem na Central já viraram campo. Coluna sem campo de destino só pode ser ignorada ou ir para as observações.
3. **Catálogos conferidos.** Origens, produtos, segmentos e motivos (Configurações → Catálogos). O que não existir no catálogo cai em "Desconhecida", "A definir" ou "A classificar".
4. **Cores da planilha.** As cores das linhas **não** são importadas — o arquivo exportado não as carrega de forma confiável. Antes de exportar, confira a coluna **Etapa do Funil** de cada grupo:

| Cor da linha | Significado na operação | Como deve estar a coluna Etapa do Funil | Vira na Central |
|---|---|---|---|
| Amarela | negociação ativa, proposta enviada | "Proposta Enviada" | registro histórico de proposta enviada |
| Verde | venda concluída | "Fechado - Ganho" | registro histórico de ganho |
| Laranja | optou por sistema gratuito; retomar depois | **acrescente** a palavra `NUTRIR` no início da coluna Observações | nutrição, com retorno em 45 dias e motivo "Optou por sistema gratuito" |
| Vermelha | disse que não tem interesse | "Sem Interesse" | desqualificado **e suprimido** (não recebe mais contato) |
| Branca | disparo massivo sem resposta | deixe como está; recomenda-se importar como **prospect** (seção 4.3) | prospect |

   A regra do `NUTRIR` existe porque a planilha não tem uma etapa para a linha laranja. É o único ajuste manual pedido, e pode ser feito com o filtro por cor do Excel em poucos minutos.
5. **Uma cópia de segurança** do sistema é feita antes da carga (SPEC-4-014). Qualquer importação também pode ser desfeita (seção 8).

---

## 3. Mapeamento coluna a coluna

O perfil já faz este mapeamento sozinho. A tabela serve para conferir e para ajustar se a planilha tiver mudado.

### 3.1 Planilha CRM – Funil de Vendas

O cabeçalho da planilha tem duas linhas (o grupo em cima, a coluna embaixo). No passo 1 do assistente, indique a **linha de baixo** como cabeçalho.

| Grupo | Coluna na planilha | Vai para | Conversão |
|---|---|---|---|
| — | Data Entrada | Data de entrada | data (`10/07/2026`) |
| — | Nome Empresa | Empresa | texto |
| Informações do Lead | Contato / Decisor | Contato | texto |
| Informações do Lead | Tel / WhatsApp | Telefone | telefone brasileiro; se houver dois números na célula, o segundo vai para Telefone secundário |
| Informações do Lead | E-mail | E-mail | e-mail; inválido fica vazio, com aviso |
| Qualificação | Segmento | Segmento | catálogo (você confirma a correspondência dos valores) |
| Qualificação | Cidade/Bairro | Cidade e Bairro | separa por `/` ou `-`; reconhece a UF se vier junto |
| Qualificação | Sistema Atual | Sistema atual | texto |
| Qualificação | Fonte do Lead | Origem | Tráfego Pago → Google Ads ou Meta Ads se a campanha indicar, senão Desconhecida com o detalhe "Tráfego pago (canal não informado)" · Indicação → Indicação · Disparo Massivo → Disparo massivo · Contato antigo → Contato antigo |
| Atividade Comercial | 1ª Tentativa | Tentativa 1 | lê data e canal: `17/08/2026 Ligação` → ligação em 17/08/2026 |
| Atividade Comercial | 2ª Tentativa | Tentativa 2 | idem |
| Atividade Comercial | 3ª Tentativa | Tentativa 3 | idem |
| Atividade Comercial | Última interação | Última interação | data e canal |
| Etapa do Funil | Etapa do Funil | Etapa | tabela abaixo |
| Etapa do Funil | Produto | Produto | Automec → Automec · Hiper gestão → Hiper Gestão · Hiper Mini → Hiper Mini · Climb → Climb · vazio → A definir |
| Resultado | Valor Proposta (R$) | Valor da proposta | texto, como está (`R$500,00 + R$275,00`) |
| Resultado | Data Fechamento | Data de fechamento | data |
| — | Observações | Nota do lead | texto inteiro vira a primeira nota; se começar com `NUTRIR`, o lead vai para nutrição |

Conversão da coluna Etapa do Funil (para o funil padrão; se o funil foi alterado, o assistente pede a correspondência):

| Na planilha | Etapa na Central | Complemento |
|---|---|---|
| Novo Lead | Novo | |
| Contatado | Em contato | |
| Qualificado | Qualificado | entra com a qualificação marcada como "importada da planilha" |
| Proposta Enviada | Agendado | resultado: proposta enviada; registro histórico |
| Fechado - Ganho | Agendado | resultado: ganho; registro histórico |
| Fechado - Perdido · Perdido - Fechido | Desqualificado | motivo: fechou com concorrente; registro histórico |
| Sem Interesse | Desqualificado | motivo: sem interesse; **suprimido** |
| (vazio) | Novo | |

"Registro histórico" significa que o lead fica consultável, entra nos indicadores do passado e **não** aparece na fila de trabalho.

Se a Pontual ativar as etapas "Proposta enviada", "Negociação" e "Fechado – ganho" no funil, o assistente oferece mandar essas linhas para elas, em vez de "Agendado".

### 3.2 Base de prospecção (cadastro CNPJ)

| Coluna na planilha | Vai para | Observação |
|---|---|---|
| `cnpj` | CNPJ | valida o dígito; CNPJ repetido não duplica |
| `razaosocial` | Empresa | |
| `nomefantasia` | Nome fantasia | |
| `dataAbertura` | campo personalizado "Data de abertura", se existir | senão, ignorar |
| `situacaoCadastral` | filtro | linhas que não estejam ativas são rejeitadas, com o motivo |
| endereço (logradouro, bairro, município, UF) | Bairro, Cidade, UF | |
| telefone | Telefone | linha sem telefone válido é rejeitada (não serve para contato) |
| e-mail | E-mail | |

Valores fixos do lote: **Tipo = Prospect**, **Origem = Disparo massivo** (ou a que fizer sentido), **Etiqueta** com o nome da base (ex.: `base-hiper-2026-10`).

### 3.3 Lista RespondeChat

| Coluna | Vai para |
|---|---|
| `Nome` | Contato |
| `Número` | Telefone |
| `E-mail` | E-mail |
| `Etiquetas` | Etiquetas |

Valor fixo: **Tipo = Prospect**.

### 3.4 Leads Hiper – tráfego pago

Mesmo mapeamento da seção 3.1 para as colunas que existirem. Valores fixos: **Origem = Parceiro Hiper**, **Produto = Hiper Gestão** (ajuste se a coluna de produto existir).

### 3.5 Modelo padrão da Central

Para qualquer outra fonte, use `modelos/modelo-leads.csv`. As colunas do modelo têm os mesmos nomes dos campos e são reconhecidas sem mapeamento. Só **telefone** (ou e-mail) e **empresa ou contato** são obrigatórios.

---

## 4. Passo a passo

### 4.1 Planilha de funil

1. No SharePoint, abra "CRM - Acompanhamento - ok", aba **CRM - Funil de Vendas**. Confira a coluna Etapa do Funil conforme a seção 2, item 4.
2. Escolha **uma** das formas de levar os dados:
   - **Arquivo:** Arquivo → Salvar como → Baixar uma cópia (`.xlsx`).
   - **Colar:** selecione do cabeçalho até a última linha, copie (Ctrl+C).
3. Na Central: **Importar** → cartão **Leads**.
4. **Passo 1:** aba "Arquivo" (escolha o arquivo e a aba "CRM - Funil de Vendas") ou aba "Colar" (Ctrl+V). Indique a linha do cabeçalho.
5. **Passo 2:** o perfil "Planilha CRM – Funil de Vendas" é escolhido sozinho. Confira os três primeiros valores de cada coluna. Confirme a correspondência dos valores de Segmento, Fonte e Etapa.
6. **Passo 3:** leia o resumo. Abra a lista de erros. Erros mais comuns e o que fazer estão na seção 7.
7. **Passo 4:** se aparecerem duplicidades, o padrão ("manter a primeira" dentro do arquivo; "atualizar campos vazios" para quem já existe) serve para a carga inicial.
8. **Passo 5:** **Importar**. Baixe o arquivo de **linhas rejeitadas**, corrija nele mesmo e importe só esse arquivo.
9. No relatório, use **Definir primeiro contato para todos em…** para os leads ativos, escolhendo uma data — ou distribua em alguns dias, para não lotar a fila de um dia só.
10. Faça a conferência da seção 9.

### 4.2 Leads do parceiro

Mesmos passos, com a aba **Leads Hiper - Trafego Pago** e o perfil correspondente. Leads que já entraram pela planilha de funil aparecem como duplicidade; use "atualizar campos vazios".

### 4.3 Bases de prospecção

1. **Importar** → cartão **Base de prospecção**.
2. Aba **Google Sheets**: conecte a sua conta Google (uma vez), cole o endereço da planilha e escolha a aba. Sem o conector: no Google Sheets, Arquivo → Fazer download → `.csv`, e use a aba "Arquivo".
3. Confira o perfil e defina a **etiqueta** do lote.
4. Importe. Os contatos ficam em **Prospecção → Bases**, fora do Kanban.

As linhas brancas da planilha de funil (disparo massivo sem resposta) ficam melhor como prospects: filtre-as na planilha, copie e importe por aqui, com o perfil da planilha de funil e o valor fixo **Tipo = Prospect**.

### 4.4 Listas do RespondeChat

**Importar** → **Base de prospecção** → perfil "Lista RespondeChat". Use uma etiqueta que identifique a lista e a data do disparo. Depois, em Prospecção → Bases, você pode registrar para o lote inteiro a data do último disparo, para que esses contatos respeitem o intervalo mínimo antes de um novo envio.

### 4.5 Quem já recusou

Vem junto da planilha de funil (etapa "Sem Interesse"). Para contatos que estão só em listas: em Prospecção → Retornos, use **Colar uma lista de telefones** e aplique **Sem interesse** a todos de uma vez. Confira no item 6.4.

---

## 5. Anúncios

Três caminhos; escolha o que estiver disponível (detalhes em `integracoes.md`, seção 7).

**Sincronização** — Configurações → Integrações → conectar a conta. A primeira sincronização traz 90 dias. Não há arquivo.

**Relatório do Meta Ads**

1. Gerenciador de Anúncios → aba **Anúncios**.
2. Escolha o período (um mês por vez funciona bem).
3. Detalhamento → Por tempo → **Dia**.
4. Colunas: Nome da campanha, Nome do conjunto de anúncios, Nome do anúncio, Valor usado, Impressões, Cliques (todos), Resultados.
5. Exportar → `.csv`.
6. Na Central: **Importar** → cartão **Mídia** → perfil "Relatório Meta Ads".

| Coluna do relatório | Vai para |
|---|---|
| Dia | data |
| Nome da campanha | campanha |
| Nome do conjunto de anúncios | conjunto |
| Nome do anúncio | anúncio |
| Valor usado (BRL) | investimento |
| Impressões | impressões |
| Cliques (todos) | cliques |
| Resultados | leads informados pela plataforma |

**Relatório do Google Ads**

1. Campanhas → **Anúncios**.
2. Período → Segmentar → Tempo → **Dia**.
3. Colunas: Campanha, Grupo de anúncios, Anúncio, Custo, Impressões, Cliques, Conversões.
4. Fazer download → `.csv`.
5. **Importar** → **Mídia** → perfil "Relatório Google Ads".

O relatório do Google costuma trazer linhas de título no topo e linhas de total no fim; o perfil as reconhece e ignora. Valores vêm com vírgula decimal (`1.234,56`); o perfil converte.

**Lançamento manual** — Anúncios → Lançar investimento: mês, plataforma, campanha, valor. Para o que não tem relatório.

Depois de importar, abra **Anúncios → Campanhas sem correspondência** e ligue os nomes de campanha que vieram dos leads aos da plataforma.

---

## 6. Conferências específicas

| # | O que conferir | Como |
|---|---|---|
| 6.1 | Telefones | na lista de Leads, filtre "telefone a corrigir"; corrija na ficha |
| 6.2 | Origem | em Indicadores, veja quantos ficaram em "Desconhecida"; corrija em massa na lista de Leads, se souber a origem |
| 6.3 | Etapas | compare a contagem por etapa da planilha com a do Kanban (seção 9) |
| 6.4 | Suprimidos | em Prospecção → Listas, monte uma lista de teste com toda a base; a linha "Suprimido" das exclusões deve ter todos os que recusaram |
| 6.5 | Duplicidades | menu Leads → Duplicidades sem pendências |

---

## 7. Erros comuns e o que fazer

| Mensagem | Causa | O que fazer |
|---|---|---|
| Sem telefone e sem e-mail | linha sem forma de contato | completar na planilha de rejeitadas e reimportar; ou deixar de fora |
| Informe o DDD | telefone com 8 ou 9 dígitos | acrescentar o DDD na planilha de rejeitadas |
| Telefone com dígitos demais ou de menos | dois números na mesma célula sem separador, ou número incompleto | separar com `/` |
| CNPJ inválido | dígito verificador não confere | corrigir; ou apagar o CNPJ e importar sem ele |
| Sem nome de empresa e de contato | as duas colunas vazias | preencher uma delas |
| Data ilegível | texto fora de padrão na coluna de data | usar `dd/mm/aaaa` |
| Valor desconhecido em Segmento, Fonte ou Etapa (aviso) | texto que não está no catálogo | no passo 2, apontar para o item certo; ou criar o item em Configurações → Catálogos e voltar |
| E-mail inválido (aviso) | texto sem `@` | o lead entra sem e-mail |
| Este arquivo já foi importado | o mesmo conteúdo foi enviado antes | cancelar; ou seguir, e as linhas caem em duplicidade |
| Não encontrei o cabeçalho | linhas de título acima do cabeçalho | indicar a linha do cabeçalho no passo 1 |
| Caracteres estranhos (Ã£, Ã§) | codificação do `.csv` | trocar a codificação no passo 1 para "Windows-1252", ou usar `.xlsx` ou colar |

Linha rejeitada **não bloqueia** as outras: as válidas entram, e as rejeitadas ficam no arquivo para corrigir.

---

## 8. Desfazer

**Importar → Histórico de importações → abrir o lote → Desfazer importação.**

- Remove o que o lote criou e devolve os campos que ele alterou em registros que já existiam.
- Lead criado pelo lote que **já foi trabalhado** (ganhou nota, tentativa, mudança de etapa, conversa ou agendamento) não é removido; o relatório lista quantos ficaram.
- Por isso: confira o lote **antes** de começar a trabalhar os leads dele.
- Último recurso: a cópia de segurança feita antes da carga.

---

## 9. Conferência final da carga

Preencha e guarde com o recibo da fase.

| Etapa na planilha | Linhas na planilha | Etapa na Central | Leads na Central | Diferença | Explicação |
|---|---|---|---|---|---|
| Novo Lead | | Novo | | | |
| Contatado | | Em contato | | | |
| Qualificado | | Qualificado | | | |
| Proposta Enviada | | Agendado (histórico: proposta) | | | |
| Fechado - Ganho | | Agendado (histórico: ganho) | | | |
| Fechado - Perdido | | Desqualificado (concorrente) | | | |
| Sem Interesse | | Desqualificado (suprimido) | | | |
| Linhas laranja (`NUTRIR`) | | Nutrição | | | |
| **Total** | | | | | |

Diferenças aceitáveis, desde que explicadas: linhas rejeitadas ainda não corrigidas; duplicidades unidas; linhas brancas levadas para prospects.

Depois: abra **20 fichas** escolhidas ao acaso e compare com a linha da planilha — nome, telefone, etapa, tentativas, observação. Anote o resultado.

Por fim, arquive a planilha como somente leitura, com a data no nome.

---

## 10. Modelos

Na pasta `modelos/`:

| Arquivo | Para quê |
|---|---|
| `modelo-leads.csv` | leads de qualquer fonte |
| `modelo-base-prospeccao.csv` | bases de prospecção |
| `modelo-respondechat.csv` | o formato de 4 colunas, de entrada e de saída |
| `modelo-midia.csv` | investimento por dia e por anúncio |
| `modelo-conversas.csv` | histórico de conversas |

Os mesmos arquivos são baixados pelo link **Baixar modelo** de cada cartão da tela Importar.
