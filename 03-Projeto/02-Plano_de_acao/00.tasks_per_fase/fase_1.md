# Fase 1 — Tarefas

<!-- fase-format:2 -->

Cada linha é uma tarefa da Jornada de Execução. **Tudo que cabe num card cabe nesta linha** — se um
campo não estiver aqui, ele não tem como ser preenchido, porque é este arquivo que cria a tarefa.

```
- [ ] Título da tarefa @responsável !30/09/2026 #projeto [interno]   <!-- id:… -->
      > descrição da tarefa, uma ou mais linhas
  - [ ] subtarefa (basta indentar 2 espaços)                         <!-- id:… -->
    - [ ] sub-subtarefa (indente mais 2)                             <!-- id:… -->
```

| marcador | o que define | se você não escrever |
|---|---|---|
| `- [ ]` / `- [/]` / `- [x]` | a fazer / em andamento / concluída | a fazer |
| `@nome` | responsável (`@"Nome Composto"` com aspas) | fica **sem responsável** |
| `!dd/mm/aaaa` | prazo | fica **sem prazo** |
| `#projeto` / `#aculturamento` | tipo | Projeto de IA |
| `[interno]` | o cliente **não** vê esta tarefa | o cliente vê |
| `> texto` na linha de baixo | descrição (aparece ao abrir o card) | sem descrição |
| indentar 2 espaços | vira subtarefa da tarefa acima (vale em qualquer profundidade) | tarefa de topo |

Os marcadores só valem **no fim da linha** — `Revisar #3 do contrato` continua sendo um título.
Um título que TERMINA na forma de um marcador sai escapado com `\\` (`Ligar para \\@joao`); a barra é
só para o parser e nunca aparece no card. Você não precisa escrever isso à mão.
Marque `[x]` para concluir e adicione linhas novas à vontade: elas entram no quadro na próxima
sincronização e voltam aqui com o `<!-- id:… -->` preenchido. **Não apague o marcador de id** das
tarefas que já têm um.

- [ ] Implementar login, papéis e shell do cockpit @Desenvolvimento #projeto <!-- id:F1-T01 -->
      > Criar o acesso de teste e a superfície inicial do cockpit conforme a SPEC-1-001. Provar CA-1-001/003 no recorte F1-COCKPIT-01.
- [ ] Implementar cadastro, Kanban e histórico @Desenvolvimento #projeto <!-- id:F1-T02 -->
      > Criar lead de teste, movimentar card e registrar histórico conforme a SPEC-1-001. Parar após F1-COCKPIT-01/02.
- [ ] Validar erros, duplicidade e demonstração @"Champion/QA" #projeto <!-- id:F1-T03 -->
      > Executar os cenários de permissão, entrada inválida, origem desconhecida e duplicidade; registrar demonstração.
- [ ] Definir fixture e validar formato da amostra @"Dados/Cliente" #projeto <!-- id:F1-T04 -->
      > Preparar arquivo anonimizado com linhas válida, inválida, duplicada e origem desconhecida conforme SPEC-1-002.
- [ ] Implementar classificação, importação e relatório @Desenvolvimento #projeto <!-- id:F1-T05 -->
      > Importar a fixture em lote de teste, classificar linhas e gerar relatório sem alterar produção.
- [ ] Implementar e provar rollback/reprocessamento @"Desenvolvimento/QA" #projeto <!-- id:F1-T06 -->
      > Remover somente o lote de teste e reprocessar sem duplicação conforme F1-IMPORT-03.
- [ ] Criar fixture e consulta do painel @Desenvolvimento #projeto <!-- id:F1-T07 -->
      > Exibir totais e filtros por origem/período a partir dos registros de teste conforme SPEC-1-003.
- [ ] Tratar vazio, desconhecido e permissão @"Desenvolvimento/QA" #projeto <!-- id:F1-T08 -->
      > Provar estado vazio, origem Desconhecida e acesso protegido no painel.
- [ ] Realizar demonstração e aceite da gestão @"Gestão/Champion" #projeto <!-- id:F1-T09 -->
      > Executar o roteiro completo do painel e registrar o aceite humano.
