# Guia de Uso no NotebookLM + Prompts Prontos

Material: **Developing Data Models with SAP HANA Cloud** — treinamento em português com termos técnicos em inglês, para preparação da prova de certificação.

---

## 1. Como montar o notebook

1. Crie **um notebook** chamado `SAP HANA Cloud – Data Modeling (PT)`.
2. Faça upload de **todos os arquivos `.md`** (00 a U20) como **sources**. O NotebookLM aceita Markdown.
3. Para gerar algo **de um tópico só** (vídeo/áudio por unit): no painel **Sources**, **desmarque tudo** e marque apenas o arquivo da unit desejada (ex.: `U04_Joining_Data_Sources.md`). O NotebookLM gera usando só as sources selecionadas.
4. Para revisões gerais/simulados: deixe **todas** as sources marcadas.

> Dica: units grandes (U04 Joins, U08 Funções/Filtros/Moeda, U09 Variables/Input Parameters, U17 Projeto/Deploy, U19 Segurança) rendem melhor em **2 vídeos/áudios** — use o "foco" sugerido na tabela da seção 4.

---

## 2. Instruções do Chat (Configure chat → Custom)

Cole isto em **Configure chat / Personalizar** para que todas as respostas sigam o padrão:

```
Você é um instrutor sênior de SAP HANA Cloud preparando um aluno brasileiro para a prova de certificação de data modeling (curso "Developing Data Models with SAP HANA Cloud").
Regras:
- Responda sempre em português do Brasil.
- Mantenha SEMPRE os termos técnicos em inglês exatamente como aparecem na SAP (ex.: calculation view, Star Join, Referential Join, input parameter, analytic privilege, HDI container, union pruning). Na primeira ocorrência, explique o termo entre parênteses em português.
- Baseie-se somente nas sources do notebook. Se algo não estiver nelas, diga isso.
- Destaque o que costuma ser cobrado na prova (pegadinhas, valores, defaults, limites, "só funciona em...").
- Quando útil, termine com 2 perguntas rápidas de verificação no estilo da prova (múltipla escolha, indicando quantas corretas), com gabarito no final.
```

---

## 3. Prompts-modelo

Substitua `[UNIT]` e `[FOCO]` pelos valores da tabela da seção 4.

### 3.1 Video Overview (customização)
```
Crie uma videoaula em português do Brasil sobre [UNIT], para quem vai fazer a prova de certificação SAP de data modeling no SAP HANA Cloud.
Mantenha os termos técnicos em inglês (calculation view, node, join, etc.) e explique cada um na primeira vez que aparecer.
Foco: [FOCO].
Estrutura: 1) por que isso existe / problema que resolve; 2) conceitos e definições; 3) como configurar no SAP Business Application Studio (passo a passo); 4) exemplo prático com dados simples; 5) pegadinhas e pontos mais cobrados na prova; 6) resumo final em 5 tópicos.
Use diagramas simples (fluxo de nós, tabelas de comparação). Tom didático e objetivo.
```

### 3.2 Audio Overview — Deep Dive (podcast)
```
Podcast em português do Brasil, dois apresentadores, sobre [UNIT]. Um deles é um consultor SAP experiente e o outro está estudando para a prova de certificação de SAP HANA Cloud data modeling e faz as perguntas que um aluno faria.
Mantenha os termos técnicos em inglês e explique-os de forma natural.
Foco: [FOCO].
Inclua exemplos do dia a dia de projetos (vendas, clientes, produtos), compare conceitos parecidos que confundem na prova e termine com um "lightning round" de 5 perguntas e respostas rápidas.
```

### 3.3 Audio Overview — Brief (revisão rápida, ~5 min)
```
Resumo em áudio, em português do Brasil, de [UNIT] para revisão no dia da prova. Apenas o essencial: definições-chave, valores padrão (defaults), limites, regras do tipo "só funciona em..." e as respostas do quiz oficial explicadas. Termos técnicos em inglês.
```

### 3.4 Audio Overview — Debate (para fixar diferenças)
```
Debate em português do Brasil entre dois especialistas SAP sobre quando usar cada opção em [UNIT]. Foco: [FOCO]. Cada lado defende uma abordagem e no final chegam a um consenso com a regra prática que a prova cobra. Termos técnicos em inglês.
```

### 3.5 Study Guide / Relatório
```
Crie um guia de estudo em português do Brasil para [UNIT], com: objetivos, mapa dos conceitos, tabela de comparação dos itens parecidos, lista de "pegadinhas de prova", e 10 questões de múltipla escolha no estilo SAP (indicando "Choose the correct answer" ou "There are N correct answers"), com gabarito comentado no final. Mantenha termos técnicos em inglês.
```

### 3.6 Flashcards
```
Gere 25 flashcards de [UNIT]. Frente: pergunta curta em português contendo o termo técnico em inglês. Verso: resposta objetiva (1–2 linhas) com o detalhe que a prova cobra (default, limite, tipo de nó, propriedade, extensão de arquivo).
```

### 3.7 Quiz / Simulado
```
Monte um simulado de 20 questões no estilo da certificação SAP sobre [UNIT ou "todas as units"]. Misture "Choose the correct answer" e "There are 2/3 correct answers". Use os termos em inglês nas alternativas, enunciado em português. Mostre o gabarito comentado só no final, explicando por que cada alternativa errada está errada.
```

### 3.8 Mind Map
Use o botão **Mind Map** com todas as sources marcadas para ter o mapa do curso; depois repita com uma unit selecionada para o mapa detalhado do tópico.

### 3.9 Perguntas úteis no chat
- `Compare Variables x Input Parameters numa tabela e diga 5 situações em que só o input parameter resolve.`
- `Liste todos os tipos de join e em que tipo de nó/view cada um é suportado.`
- `Quais são todas as condições de pruning (join pruning, referential, optimize join columns, greedy, union pruning)? Faça uma tabela.`
- `Liste todas as extensões de arquivo citadas no curso (.hdbcalculationview, .hdbrole, .hdbgrants...) e para que serve cada uma.`
- `Quais features mudaram por QRC (versão)? Faça uma linha do tempo.`
- `Faça 10 perguntas de "verdadeiro ou falso" com as maiores pegadinhas do curso inteiro.`

---

## 4. Plano de vídeos/áudios por unit (valores para [UNIT] e [FOCO])

| # | Source | [UNIT] | [FOCO] sugerido |
|---|---|---|---|
| 1 | U01 | Unit 1 – Preparing the Modeling Environment | SAP Business Application Studio, Development Spaces (limites e manutenção), criar/importar projeto, Git vs import, Joule e seus comandos (/hana-gen, /hana-modeling-help…) |
| 2a | U02 | Unit 2 – Creating Calculation Views (parte 1) | measure x attribute, dimension, star schema, tipos de calculation view (DIMENSION, CUBE, CUBE with Star Join, TIME), shared columns e alias |
| 2b | U02 | Unit 2 – Creating Calculation Views (parte 2) | data sources suportados, Data Preview e usuário _RT, Semantics node, hidden/label columns, null handling, view properties e Top View Node |
| 3 | U03 | Unit 3 – Projection e Aggregation Nodes | aggregate functions, calculated column antes/depois da agregação, GROUP BY na query, Keep Flag e Transparent Filter com exemplos numéricos |
| 4a | U04 | Unit 4 – Joins (parte 1) | Inner, Outer, Full Outer no Star Join, Referential Join (cardinality, integrity constraint, condições de pruning), Text Join e Temporal Join |
| 4b | U04 | Unit 4 – Joins (parte 2) | Key indicator, cardinality/Propose Cardinality, Multi-Join, Non-Equi Join, Dynamic Join, filter mapping, Optimize Join Columns e Greedy Join Pruning |
| 5 | U05 | Unit 5 – Union Nodes | standard union x constant values, Auto Map by Name, Empty Union Behavior, UNION ALL x DISTINCT, os 3 tipos de union pruning |
| 6 | U06 + U07 | Units 6 e 7 – Minus, Intersect e Rank Node | Minus/Intersect dinâmicos, Rank node: partition, Row x Rank x Dense Rank x Sum, Percentage, Target Value, Offset, agregar antes de rankear |
| 7 | U08 | Unit 8 – Restricted e Calculated Columns, Filtros | lógica OR/AND, granularidade dos cálculos, onde o filtro é aplicado, filter push-down, client-dependent views (Session Client) |
| 8 | U08 | Unit 8 – Currency Conversion | tabelas TCUR*, Amount with Currency Code, settings (Upon Failure, Generate, Accuracy), decimal shift, Reference x Copy |
| 9a | U09 | Unit 9 – Variables e Input Parameters | diferenças, tipos de input parameter, PLACEHOLDER, uso em calculated columns e filtros em measures |
| 9b | U09 | Unit 9 – Parameter Mapping | 4 tipos de mapping, push down para views inferiores, SQL views/virtual tables, cascading prompts |
| 10 | U10 | Unit 10 – Hierarchies | Level x Parent-Child, orphan nodes, propriedades, time-dependent hierarchy, Hierarchy Type |
| 11 | U11 | Unit 11 – SQL, SQLScript e Functions | SQL Console e HDI x catalog, expressions, hierarchy via SQL, SQLScript (variáveis, IF/WHILE/FOR), scalar x table function |
| 12 | U12 | Unit 12 – Best Practices | instantiation, query unfolding, camadas, column pruning, filtrar cedo/calcular alto, SQL em vez de column engine |
| 13 | U13 | Unit 13 – Ferramentas de Performance | Performance Analysis mode, Debug Query mode, SQL Analyzer (plan file, privilégios) |
| 14 | U14 | Unit 14 – Paralelização e Particionamento | Partition Local Execution (regras e restrições), table partitioning |
| 15 | U15 | Unit 15 – Static Cache, Snapshots e MDS Cubes | pré-requisitos e invocação do cache, processo de snapshot, MDS cube (SAC/InA, limitações, MANAGE_MDS_CUBE) |
| 16 | U16 | Unit 16 – Produtividade e Auditoria | switch node, replace data source, extract/propagate semantics, data lineage x impact analysis, column lineage, where-used |
| 17a | U17 | Unit 17 – Estrutura do Projeto | workspace/project/module, HDB module, namespace e .hdinamespace, nomes de runtime objects |
| 17b | U17 | Unit 17 – Deploy e Gestão de Conteúdo | HDI container, .hdiconfig, SAP HANA PROJECTS, erros de deploy, copiar/mover/renomear/apagar, deprecate |
| 18 | U18 | Unit 18 – Ciclo de Vida | schema name, user-provided service, .hdbgrants/.hdbrevokes, synonyms, cross-container, Git, MTA/.mtar |
| 19a | U19 | Unit 19 – Analytic Privileges | tipos de restrição, combinação OR/AND, dynamic analytic privilege, hierarchy nodes, nested views |
| 19b | U19 | Unit 19 – Roles e Masking | design-time roles, .hdbroleconfig, .hdbgrants, data masking, UNMASKED, mask mode |
| 20 | U20 | Unit 20 – Knowledge Graph Engine | triples/RDF, ontologies e linguagens, KGE, SPARQL, interfaces e casos de uso |

---

## 5. Roteiro de estudo sugerido (2 semanas)

| Dia | O que fazer |
|---|---|
| 1 | U01–U02: ler + vídeo 1, 2a, 2b |
| 2 | U03 + U04 parte 1 |
| 3 | U04 parte 2 + flashcards de joins |
| 4 | U05–U07 |
| 5 | U08 (2 vídeos) |
| 6 | U09 (2 vídeos) + U10 |
| 7 | Simulado 1 (Units 1–10) |
| 8 | U11–U12 |
| 9 | U13–U15 |
| 10 | U16–U17 |
| 11 | U18–U19 |
| 12 | U20 + Simulado 2 (Units 11–20) |
| 13 | Simulado geral + revisar erros com o chat |
| 14 | Áudios "Brief" de todas as units + revisão do arquivo 00_Revisao_Rapida |
