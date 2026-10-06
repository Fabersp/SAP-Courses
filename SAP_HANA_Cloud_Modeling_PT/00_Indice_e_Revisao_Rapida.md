# SAP HANA Cloud — Developing Data Models: Índice e Revisão Rápida

Treinamento em português (termos técnicos em inglês) baseado no curso **Developing Data Models with SAP HANA Cloud** (learning.sap.com). Cada unit tem: conteúdo das lições, pontos-chave para a prova, quiz oficial com gabarito comentado e glossário.

> Os gabaritos dos quizzes foram elaborados a partir do conteúdo das lições (o site não exibe as respostas).

## Índice

| Arquivo | Unit | Lições |
|---|---|---|
| U01 | Preparing the Modeling Environment | BAS, Development Spaces, criar/importar projeto, navegação, Joule |
| U02 | Creating Calculation Views | Conceitos, DIMENSION, CUBE, Star Join, TIME, data sources, Data Preview, features comuns, Top View Node |
| U03 | Working with Common Nodes | Projection, Aggregation, Keep Flag, Transparent Filter |
| U04 | Joining Data Sources | Tipos de join, cardinality, filter mapping, otimização de joins |
| U05 | Working with Union Nodes | Union node, union pruning |
| U06 | Creating Data Slices | Minus, Intersect |
| U07 | Ranking Data | Rank node |
| U08 | Embedding Functions | Restricted/Calculated columns, filtros, client, currency conversion |
| U09 | Creating Dynamic Calculation Views | Variables, Input Parameters, Mapping |
| U10 | Implementing Hierarchies | Level, Parent-Child, time-dependent |
| U11 | Developing Custom Logic Using SQL | SQL Console, expressions, hierarchy via SQL, SQLScript, functions |
| U12 | Applying Best Practices | Instantiation, unfolding, boas práticas nos nós |
| U13 | Tools to Check Model Performance | Performance Analysis, Debug Query, SQL Analyzer |
| U14 | Features to Improve Performance | Parallelization, Partitioning |
| U15 | Storing Calculation View Results | Static Cache, Snapshots, MDS Cubes |
| U16 | Additional Productivity Tools | Produtividade, lineage, impact analysis |
| U17 | Working in a Modeling Project | Estrutura, namespace, HDI, deploy, gestão de conteúdo |
| U18 | Managing the Lifecycle | Projeto, dados externos, Git, MTA |
| U19 | Implementing Security | Analytic privileges, roles, masking |
| U20 | Knowledge Graph Engine | Knowledge graphs, ontologies, KGE, SPARQL |

---

## Revisão rápida — 80 fatos que mais caem

### Ferramenta e ambiente
1. Ferramenta recomendada: **SAP Business Application Studio** (baseado em VS Code, roda no **SAP BTP**).
2. Extension essencial: **SAP HANA Calculation View Editor**. SQL Analyzer exige **SAP HANA Performance Tools**.
3. Adicionar extension → dev space **STOPPED**; exportar dev space → **RUNNING**; não remove extension; trial = 2 spaces/1 rodando.
4. Projeto novo → **Start from Template**; colaboração → **Clone from Git**; Import só **.zip/.tar** (e extrai).
5. Joule: `/hana-gen` (projeto), `/hana-modeling-help` (modelagem), `/hdi-gen` (HDI artifacts), `/hana-sql-gen`, `/hana-ai`, `/code-search`, `/sap-help`.

### Calculation views
6. **DIMENSION**: só attributes (master data); top node padrão **Projection**.
7. **CUBE**: measures agregadas pelos attributes pedidos; top node padrão **Aggregation**.
8. **CUBE with Star Join**: une DIMENSION views ao fato; Star Join **só como top view node**; OLAP engine.
9. Shared columns: Name/Label não mudam; **Alias** obrigatório em conflito.
10. TIME dimension gera attributes de tempo a partir de uma data base + tabela de tempo.
11. Data sources: row/column tables, **virtual tables**, calculation views, **SQL views**, **table functions** (não flat file direto).
12. Calculation views são **read-only**, calculam **on-the-fly** e **se adaptam à query**.
13. Semantics node é sempre o topo; top view node logo abaixo; nada acima deles.
14. Null handling sem default: **0** numérico / **vazio** NVARCHAR.
15. Data Preview ≈ **SAP HANA Database Explorer**; usuário técnico **`_RT`**; "Data Preview With Other Database User".

### Nós
16. Projection: selecionar colunas, filtrar, calculated columns, input parameters.
17. Aggregation: SUM padrão; 8 funções; **calculated column calculada DEPOIS** da agregação.
18. Query com GROUP BY **sobrescreve** a função das Semantics.
19. **Keep Flag**: força granularidade (qty × price por ORDER_ID).
20. **Transparent Filter**: evita coluna filtrada no GROUP BY (distinct count em stacked views).
21. Union = **UNION ALL**; + Aggregation acima = **UNION DISTINCT**.
22. Empty Union Behavior: **No Row** (padrão) / **Row with Constants**.
23. Union pruning: **explicit (constant)**, **implicit (Pruning Configuration Table)**, **column-based (Focus Column)**.
24. Configuration table: `= < > <= >= BETWEEN`; mesma coluna **OR**, colunas diferentes **AND**.
25. **Minus** = primeira fonte menos a segunda (**Switch Order**); Intersect/Minus comparam só colunas pedidas.
26. Rank: **Row** (empates diferentes), **Rank** (iguais com gap), **Dense Rank** (sem gap), **Sum** (acumulado, só 1ª sort column); **não agrega**.
27. Rank Percentage = fração 0–1; Target Value por input parameter; **All Values** (QRC 2/2024).

### Joins
28. **Full Outer no Star Join**: só uma dimension e **por último**.
29. **Referential** = Inner + pruning; exige **cardinality** + **Integrity Constraint**.
30. Pruning referential: nada pedido de B + integrity em A + cardinality de B **..1**.
31. **Text Join** = Left Outer 1:1 + **language column (SPRAS)**; não traduz.
32. **Temporal Join**: só em **Star Join**, tipo **Inner**, colunas date/timestamp/integer.
33. **Non-Equi Join**: operadores ≠ "="; Inner/Left/Right/Full.
34. **Dynamic Join**: granularidade variável; **multi-column**; erro se nenhuma coluna de join pedida.
35. Multi-Join Order: **Outside-In / Inside-Out**.
36. Key indicator: **dourado** completo, **tracejado** incompleto, **cinza** perdido.
37. Join pruning padrão **não** ocorre em Inner; precisa Outer/Referential/Text.
38. Optimize Join Columns: Referential/Outer/Text + só colunas de A + coluna de join de A não pedida + cardinality B ..1.
39. Greedy pruning: 1 Disabled / 7 Left / 11 Right / 15 Both; precedência **query > view > join**.
40. Filter mapping: **performance + consistência**; direção padrão bidirecional.

### Colunas, filtros, moeda, parâmetros
41. Restricted column: mesma coluna OR / diferentes AND; simplifica calculated columns; pode usar measures (QRC 3/2023).
42. Calculated column pode ser **attribute ou measure**; não persiste.
43. Filtros em attributes **e** measures; renomeou a coluna → use o **nome de output**.
44. Top node: filtro **antes** do processamento; outros nós: no **output**.
45. **Ignore Multiple Outputs For Filter** libera push-down; ⛔ mostra bloqueio (2026 QRC1).
46. Default Client: Cross-Client / Fixed / **Session Client (padrão)**; `_RT` sem client → preview vazio.
47. Currency: TCUR* (TCURR = taxas, TCURX = decimais); **Amount with Currency Code**; Upon Failure **Fail/NULL/Ignore**; IP de moeda **VARCHAR(5)**.
48. Variable: só **attributes**, filtro **no topo**; não se aplica a measures.
49. Input parameter: `'$$IP$$'` em expressões, **PLACEHOLDER** na SQL; ignorado se não usado.
50. Mapping: Data Sources / Procedures-Scalar Functions / Value help for variables-IPs / Value help for attributes; **sem cross-mapping**.
51. Cascading prompt: value help view do **filho** filtrada por IP mapeado ao **pai**.
52. IP para SQL view/virtual table remota: **SDA (ODBC)**, não SDI.

### Hierarchies e SQL
53. Hierarchies: **Level** e **Parent-Child**; time-dependent só **parent-child**.
54. Orphan nodes: Root Nodes, Error, Ignore, Step Parent.
55. Hierarchy Type: Auto / SQL Hierarchy Views / No Hierarchy Views.
56. Hierarchy via SQL: criar na **DIMENSION**, consumir no **CUBE with Star Join**, **Enable Hierarchies for SQL access**, coluna `<hier>node`.
57. Expressions usam **SQL** (não SQLScript); UDFs em calculated columns e filters (QRC 1/2023).
58. SQLScript: table variables, `:var`, IF/WHILE/FOR; imperativo é mais lento.
59. **Table function** = data source; **scalar function** = valor (IP); functions read-only; **procedures não** são data source.
60. HDI container: query **sem schema**; catalog: **com schema**. Objetos persistentes por **source file**.

### Performance
61. Instantiation: calc engine poda → SQL optimizer.
62. **Unfolding**: tudo em SQL; column view na fonte = não unfolded; `M_FEATURE_USAGE`.
63. Empilhar views é **boa prática**; calcule **alto**, filtre **baixo**; SQL em vez de column engine.
64. Performance Analysis mode: sem deploy; joins, cardinality, partições, tamanho de tabela.
65. Debug Query mode: SQL por nó; usa versão **deployada**.
66. SQL Analyzer: escrever query + **Generate SQL Analyzer Plan File** (.plv); **TRACE ADMIN + INIFILE ADMIN**.
67. **Partition Local Execution**: início em Projection/Aggregation com **tabela**; fim em **Union**; **1 bloco por query**.
68. Partitioning: horizontal, só **column store**, supera limite de **linhas**.
69. Static cache: Enable Cache + unfolding + sem granularity tracking; hint **RESULT_CACHE**; não em view que checa analytic privileges; retention em **minutos**.
70. Snapshot: query sobre a view → snapshot table + procedures; DEFINER/INVOKER; interface view.
71. MDS cube: só **CUBE**; para **SAC via InA**; sem input parameters nem calculated measures; `MANAGE_MDS_CUBE`.

### Projeto, deploy e segurança
72. Workspace → Project → **Module** → Folder.
73. Namespace: `.hdinamespace` (name; subfolder **append**/ignore); `ns::object`.
74. `.hdiconfig`: sem nome; liga suffix → plug-in; `minimum_feature_version`.
75. Fora do container → **synonym** (+ user-provided service + `.hdbgrants`); `.hdbrevokes` processado antes.
76. Deploy de aplicação: `mbt build` → **.mtar**; `.mtaext` para ajustes; Git **não** faz deploy.
77. Analytic privilege: `.hdbanalyticprivilege`; Attribute / SQL Expression / **Dynamic (procedure DEFINER, read-only, sem input, VARCHAR(256))**.
78. Vários analytic privileges na mesma view → **OR**; SELECT só na view do topo.
79. **Design-time roles** (`.hdbrole`) recomendadas; `.hdbroleconfig` para schemas externos.
80. Masking: VARCHAR/NVARCHAR/CHAR/SHORTTEXT; privilégio **UNMASKED**; mask mode Default x Session User. Knowledge Graph: **SPARQL**, RDF/RDFS/OWL.
