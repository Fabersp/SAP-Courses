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
# Unit 1 — Preparing the Modeling Environment (Preparando o Ambiente de Modelagem)

**Curso:** Developing Data Models with SAP HANA Cloud (SAP Learning)
**Foco da unit:** conhecer a ferramenta **SAP Business Application Studio (BAS)**, os **Development Spaces**, como criar/importar projetos, navegar na interface e usar o **Joule**.

---

## Lição 1.1 — Getting Started with SAP Business Application Studio

**Objetivo:** descrever o SAP Business Application Studio e como ele é usado no desenvolvimento em SAP HANA Cloud.

### O que é o SAP Business Application Studio
- É uma **IDE (Integrated Development Environment)** **web-based**, usada para desenvolver todas as camadas de uma aplicação **full-stack**: interface (UI), camada de aplicação e camada de banco de dados (**database layer**).
- Suporta aplicações **cloud**, **on-premise** e **hybrid**, integrando tecnologias SAP e não-SAP.
- Um dos cenários suportados é **data modeling no SAP HANA Cloud** — o foco deste curso.
- É baseado no **Visual Studio Code** (projeto open-source da Microsoft). Por isso aceita **extensions / plug-ins** — SAP e terceiros fornecem vários para aumentar a produtividade.
- Oferece **editores de texto e gráficos** para criar **development artifacts**, além de uma **CLI (command line interface)** / terminal.
- Tem **suporte nativo a Git** para versionamento do código-fonte — os controles de Git ficam embutidos no BAS.

> **Atenção (prova):** o **Web IDE for SAP HANA** também pode ser usado para modelar no SAP HANA Cloud, **mas não é recomendado**: não terá novas features. **A ferramenta recomendada para data modeling no SAP HANA Cloud é o SAP Business Application Studio.**

### Como obter acesso ao BAS
- O BAS é uma aplicação web cujo back-end roda no **SAP BTP (Business Technology Platform)**.
- Você faz a **subscription** do serviço SAP Business Application Studio no nível do **BTP sub-account**, pelo **BTP Cockpit**.
- Antes de abrir o BAS no navegador há passos de dois tipos:
  1. Passos de **BTP Account / Subaccount** (ex.: subscription do serviço).
  2. Passos de **User membership e authorization** (atribuir ao usuário os **role collections** necessários).
- Concluídos esses passos, o BAS pode ser lançado pelo navegador.

---

## Lição 1.2 — Working with Development Spaces

**Objetivo:** criar um **Development Space** para o conteúdo de modelagem.

### Conceito
- No BAS, cada desenvolvedor tem seu próprio ambiente, formado por um ou mais **Development Spaces** (dev spaces).
- **Não confunda** Development Spaces com **Cloud Foundry spaces** — são coisas diferentes.
- Ao criar um dev space você escolhe o **tipo de aplicação** (ex.: *SAP HANA Native Application*) e as **extensions** (ferramentas).
- **Essencial para modelagem:** incluir a extension **SAP HANA Calculation View Editor**.

### Gerenciando Development Spaces
A **landing page** do BAS lista os dev spaces com:
- Nome do dev space
- Tipo de aplicação (kind of application)
- **Status:** `STOPPED`, `STARTING`, `RUNNING`, etc.
- Data de criação
- ID

- O dev space precisa estar **RUNNING** para você trabalhar nele. Quando está rodando, o nome vira um **hyperlink** para entrar.
- **Cuidado:** todo dev space **para automaticamente após um tempo ocioso (idle)** para poupar recursos. Sempre confira o status na landing page — especialmente se você salvou nos favoritos a URL do space (que contém o ID do space).

### Vários Development Spaces
- Dependendo da subscription do BTP, há limites de **quantidade máxima de spaces** e de **spaces rodando em paralelo**.
- **Conta SAP BTP Trial:** até **2 dev spaces**, e **apenas 1 rodando por vez**. Para trocar, pare o atual primeiro.

### Manutenção
- Depois de criado, o **nome** e o **tipo de aplicação** do dev space **não podem ser alterados**.
- É possível **adicionar extensions opcionais** — mas o dev space precisa estar **parado (STOPPED)**.
- **Não é possível remover** uma extension já atribuída (para preservar os projetos existentes).
- É possível **exportar todo o conteúdo** do dev space como arquivo **.tar.gz** — para isso o dev space precisa estar **RUNNING**. Ao final, o BAS mostra como importar esse conteúdo em outro dev space (de preferência vazio).

---

## Lição 1.3 — Creating a New Project in SAP Business Application Studio

**Objetivo:** criar um projeto de modelagem no BAS.

- Os projetos ficam numa estrutura de pastas específica do seu usuário e do seu dev space.
- Na **Explorer view** você navega no conteúdo do projeto (subpastas e arquivos), abre arquivos no editor, apaga ou move arquivos.
- A faixa (ribbon) no topo da Explorer view mostra o nome da pasta (em maiúsculas). A pasta é aberta como **workspace** — ou seja, configurações específicas do BAS podem ser associadas a ela.
- **Projeto do zero:** use o botão **Start from Template** na aba **Welcome**. O wizard cria um projeto quase vazio, apenas com os arquivos-base.

---

## Lição 1.4 — Importing an Existing Project in SAP Business Application Studio

**Objetivo:** importar um projeto para o workspace.

### Compartilhando conteúdo com Git (método preferido)
- **Git é o método preferido** para compartilhar conteúdo de desenvolvimento num projeto SAP HANA Cloud (importar/exportar com outros desenvolvedores).
- Git permite colaboração e trabalho em paralelo sem um sobrescrever o trabalho do outro — essencial em projetos grandes.
- O BAS inclui Git para gerenciar o ciclo de vida do código **localmente** e também conectar a um **remote repository** corporativo/público.
- Operações Git podem ser feitas pelo **terminal** (usuários experientes) ou pela **Git view** (mais guiada e visual).
- Para **colaborar em projetos grandes**, a melhor forma de trazer conteúdo para o workspace é **Clone from Git**.

### Importando sem Git
Mais simples, mas **sem rastreamento nem controle de versão**.

| Opção de import | O que faz |
|---|---|
| **Import** (Welcome page) / **Import project** (clique direito em área vazia) | Importa **somente arquivos .tar ou .zip** e já os extrai. Depois, a nova pasta pode ser aberta num novo workspace. |
| **File → Upload Files** | Faz upload de arquivos compactados (.tar/.zip) **ou arquivos individuais**. Útil para adicionar arquivos soltos. Se subir um arquivo compactado, você precisa extrair pela linha de comando (`tar` / `unzip`). |
| **Drag and drop** do explorador de arquivos | Copia arquivos/pastas do Windows File Explorer para qualquer pasta visível na Explorer view. |

> **Cuidado:** com **Upload Files** e **drag and drop**, o conteúdo pode **sobrescrever** arquivos existentes (mesmo nome e mesma pasta). Um aviso aparece para cada conflito.

### Exportando sem Git
- **Download:** clique direito em qualquer arquivo/pasta na Explorer view. Vários objetos são compactados num único **.tar**.
- Para baixar a pasta raiz inteira (vários projetos), clique direito na área em branco abaixo da estrutura de arquivos.

---

## Lição 1.5 — Navigating SAP Business Application Studio

**Objetivo:** descrever o layout e as features do BAS relevantes para modelagem.

- O BAS oferece várias formas de fazer a mesma ação: **menu commands**, **keyboard shortcuts**, **quick commands**, etc.
- As **views** à esquerda cobrem as necessidades do desenvolvedor: explorar arquivos, **search/replace**, conectar ao **Git**, **Debug**, etc.
- As views disponíveis dependem das **extensions** (incluídas por padrão conforme o tipo de dev space ou adicionadas como opcionais).
- Algumas views aparecem como ícones à esquerda; outras não — mas você pode abrir qualquer view pelo menu **Open View**.
- Algumas views têm painel próprio; outras aparecem como sub-painel. Ex.: **SAP HANA Projects** e **Java Projects** aparecem dentro da **Explorer view**.
- Você pode ocultar ícones/painéis que não usa.

### Quick Command (Command Palette)
- Forma poderosa de executar ações repetitivas ou pontuais.
- Comandos organizados por categorias: **Cloud Foundry**, **CDS**, **graphical modeling**, etc.
- Assim como as views, os quick commands **dependem das extensions instaladas**.

### Status bar
Mostra:
- **Notificações** e **problems** (warnings ou errors) no código.
- **Cloud Foundry status** e, se houver, o **Cloud Foundry target** (org/space onde você está logado).
- Informações **Git** quando o projeto é Git-enabled — ex.: nome da **branch** atual (checked-out branch).

---

## Lição 1.6 — Working with Joule in SAP Business Application Studio

**Objetivo:** usar o **Joule** dentro do BAS.

### O que é o SAP Joule
- **Joule** é o **copilot de IA** da SAP, integrado nas ferramentas SAP, incluindo o BAS.
- Ajuda a acelerar o desenvolvimento: **gera código**, **explica conceitos** e auxilia em tarefas específicas de SAP.
- Num dev space de **SAP HANA Native Database**, é especialmente útil para **database artifacts**, **SQL**, **CDS models** e lógica de aplicação.

### Como acionar o Joule
- Como **side panel assistant** (chat): clique no **ícone do Joule na Activity Bar**.
- Pelos atalhos da **command palette**.
- Por **inline suggestions** no editor.

### Recursos
- **Chat window** com prompts em linguagem natural.
- **Context awareness:** adapta o escopo do prompt ao conteúdo dos arquivos abertos ou ao código selecionado.
- **Command-based interaction:** você aciona capacidades específicas com **prefixos** (ex.: `/hana-ai`, `/hana-gen`) antes do prompt.

### Comandos (workspace do tipo SAP HANA Native Application)

| Comando | Para que serve |
|---|---|
| `/code-search` | Busca trechos de código no repositório. |
| `/hana-ai` | Extensão da biblioteca Python **HANA Machine Learning (hana-ml)** focada em **GenAI**; integra com HANA ML, **HANA vector engine** e **SAP GenAI Hub SDK**. |
| `/hana-gen` | Assistente SQL de IA especializado em **criar um projeto SAP HANA Cloud Database**. |
| `/hana-modeling-help` | **SAP HANA Modeling Assistant** — ajuda no design de **calculation views**. |
| `/hana-sql-gen` | Ajuda a criar **SQL statements** para o SAP HANA Cloud. |
| `/hdi-gen` | Gerador de definições de **HDI artifacts** (tables, views, roles…) a partir de linguagem natural, seguindo best practices. |
| `/sap-help` | Pesquisa no **SAP Help Portal**. |

> Outros comandos podem aparecer conforme as extensions ativas no dev space.

### Exemplo 1 — Criar um projeto de exemplo
1. Abra o painel do Joule pela Activity Bar.
2. Digite `/hana-gen`.
3. Escolha um modelo proposto (**Flight**, **Inventory Management**, **Bookshop**) ou descreva sua necessidade (tabelas, colunas…) em linguagem natural. O design das tabelas evolui conforme você adiciona requisitos (novos prompts ou editando um anterior).
4. Opcional: proponha um nome para o projeto.
5. Clique **Accept** — o Joule gera os artifacts na **Explorer pane**.

### Exemplo 2 — Ajuda em modelagem
1. Abra o painel do Joule.
2. Digite `/hana-modeling-help`.
3. Pergunte em linguagem natural — ex.: *"How to create a rank column"*, *"How to implement a parent-child hierarchy"*.
4. O resultado é um trecho da documentação SAP HANA, às vezes customizado com os artifacts citados na pergunta.

---

## Pontos-chave para a prova (Unit 1)
- Ferramenta **recomendada** para modelagem no SAP HANA Cloud: **SAP Business Application Studio** (não o Web IDE, não o Database Explorer).
- BAS = baseado em **VS Code**, roda no **SAP BTP**, subscription no **sub-account** via **BTP Cockpit**.
- Extension **essencial** para modelar: **SAP HANA Calculation View Editor**.
- Dev space: trial = 2 spaces / 1 rodando; adicionar extension → space **STOPPED**; exportar → space **RUNNING**; não dá para remover extension nem renomear/mudar o tipo.
- Projeto novo → **Start from Template**. Colaboração em projeto grande → **Clone from Git**. **Import** aceita só **.tar/.zip**.
- **Explorer view:** listar arquivos/pastas e criar projeto a partir de template.
- **Joule:** sugestões de código com IA e integração com ferramentas SAP; precisa de input do desenvolvedor e de internet.

---

## Quiz oficial da Unit 1 — com gabarito comentado
*(Gabarito elaborado a partir do conteúdo das lições.)*

1. **Quais são capacidades do SAP Joule no BAS?** (2 corretas)
   - ✅ It provides AI-powered code suggestions and assistance.
   - ✅ It integrates with SAP development tools.
   - ❌ It operates without the need for developer input. *(depende de prompts do desenvolvedor)*
   - ❌ It can work offline with no internet access.
2. **No SAP HANA Cloud, qual ferramenta é recomendada para data modeling?**
   - ✅ **SAP Business Application Studio** *(Web IDE não evolui mais; Database Explorer não é ferramenta de modelagem)*
3. **O que você pode fazer na Explorer view do BAS?** (2 corretas)
   - ✅ Create a project from a template
   - ✅ List files and folders
   - ❌ Manage SAP HANA artifacts deployment *(isso é feito pela view SAP HANA Projects)*
4. **Para colaborar em projetos grandes, qual a melhor opção para trazer conteúdo ao workspace?**
   - ✅ **Clone from Git**
5. **Qual extension é essencial no dev space para começar a modelar?**
   - ✅ **SAP HANA Calculation View Editor**
6. **Qual opção usar para criar um projeto totalmente novo?**
   - ✅ **Start from Template**

---

## Glossário da Unit 1
- **IDE (Integrated Development Environment):** ambiente integrado de desenvolvimento.
- **SAP BTP (Business Technology Platform):** plataforma de nuvem da SAP onde o BAS roda.
- **Sub-account / BTP Cockpit:** subconta do BTP / console de administração do BTP.
- **Development Space (dev space):** ambiente isolado do desenvolvedor no BAS, com extensions específicas.
- **Extension / plug-in:** ferramenta adicional instalada no dev space.
- **Workspace:** pasta aberta no BAS à qual se associam configurações.
- **Explorer view:** painel de arquivos/pastas do projeto.
- **Quick Command / Command Palette:** caixa de comandos rápidos.
- **Status bar:** barra inferior com notificações, problemas, status Cloud Foundry e Git.
- **Joule:** copilot de IA da SAP.
- **HDI (HANA Deployment Infrastructure):** infraestrutura de deploy de artifacts em containers (detalhada nas units seguintes).
# Unit 2 — Creating Calculation Views (Criando Calculation Views)

**Foco da unit:** conceitos e terminologia de modelagem; tipos de **calculation view** (**DIMENSION**, **CUBE**, **CUBE with Star Join**, **TIME**); **data sources** suportados; **Data Preview**; features comuns (Semantics node, aliases, hidden columns, sorting, null handling, view properties); **Top View Node**.

---

## Lição 2.1 — Understanding Basic Concepts and Terminology

### Measure vs. Attribute

| | **Measure** | **Attribute** |
|---|---|---|
| O que é | Valor **numérico** (preço, quantidade) sobre o qual se aplicam operações aritméticas/estatísticas: soma, média, top N, cálculos. | Elemento **descritivo** (ex.: país) que dá contexto à measure. Ex.: *Sales Revenue by France*. |
| Exemplos | Quantity, Sales Revenue | Product, Customer, Country, Year |

- **Attributes** servem para **filtrar** ou **agregar** as measures. Ex.: "Total de vendas originadas em Hamburgo?", "Receita de Cars em 2022?".

### Dimension
- Uma **dimension** é um **grupo de attributes relacionados**. Ex.: dimension **Product** com product name, category, supplier.
- Dimensions são **reutilizáveis**: depois de criadas, são compartilhadas entre desenvolvedores e usadas para montar **star schemas**.
- Uma dimension contém **key**, **name** e attributes. Use dimensions diferentes para entidades diferentes (Product, Sales Org…).

### Star Schema
- Tipo de modelo muito poderoso; aciona o **OLAP engine** para **slice-and-dice** rápido.
- **Fact table** (tabela fato): pode ser uma tabela, mas muitas vezes é uma combinação de tabelas para representar uma transação (ex.: sales order). As **dimensions** são conectadas à fact table para fornecer informação descritiva (ex.: país do cliente).

### Hierarchy
- Representação estruturada, **em níveis**, de uma organização, lista de produtos, tempo, etc.
- Usada para navegação **drill-down** em grandes volumes. Você pode definir uma ou mais hierarchies numa calculation view.

### Semantics
- Descreve o **significado** de um attribute ou measure, por exemplo:
  - **Valor monetário** → precisa de **currency** (USD, EUR, GBP). Valor sem moeda não tem sentido.
  - **Quantidade, peso, volume, distância** → precisa de **unit of measure**.
- No SAP HANA Cloud você adiciona semantics a measures, attributes ou à view inteira. Muitas vezes essa informação não existe nas tabelas de origem, mas o modelador a adiciona facilmente na calculation view.

### Calculation Views
- **Calculation view é o principal objeto de modelagem do SAP HANA Cloud.**
- Ficam sobre as **tabelas físicas** da camada de banco e são consumidas por relatórios operacionais e aplicações analíticas.
- Propósito: **projetar** dados das tabelas e executar **cálculos** para gerar um result set que responda a uma pergunta de negócio.
- **Benefícios** (cobrados no quiz): **calculam dados ao vivo, on-the-fly** (sem persistir/replicar dados) e **se adaptam automaticamente à query** que as consulta (só processam as colunas pedidas — *pruning*). São definidas graficamente (ou com SQL/SQLScript em casos específicos) — **não** "em qualquer linguagem".

### Design-time vs. Runtime / Deploy
- Você cria o **design-time object** (arquivo `.hdbcalculationview`) no **HDB module** do projeto, no seu workspace.
- Ao fazer **deploy** pelo BAS, o SAP HANA Cloud gera o **runtime object** correspondente (ex.: uma **column view**) num **container** (**HDI container**).
- Um **container** guarda todos os runtime objects relacionados. Um banco SAP HANA Cloud pode ter **muitos containers**.

### Requisitos analíticos vs. transacionais
- Em aplicações **transacionais** (ex.: SAP ERP), os dados nas tabelas físicas são manipulados pelo **application server** (lógica de negócio), que lê e grava direto nas tabelas. Calculation views normalmente não são necessárias, mas a aplicação pode lê-las (ex.: consumir calculation view a partir de **ABAP**).
- **Calculation views são read-only** — **não alteram dados** no banco.
- Aplicações **analíticas** normalmente não gravam; podem **pular o application server** e consultar direto as calculation views, com dados calculados on-the-fly no banco **in-memory**.

---

## Lição 2.2 — Creating Dimension Calculation Views

- **Dimension calculation view** expõe **master data** (dados mestres) de tabelas de origem. Pode combinar várias fontes, definir filtros, calcular attributes adicionais e criar **hierarchies**.
- Objeto **altamente reutilizável e centralizado**, que fornece dados consistentes; normalmente compartilhado com colegas que o consomem em suas views.
- **Só contém attributes — não contém measures.** Toda coluna da fonte é tratada como attribute, inclusive colunas numéricas (preço, salário).
  - Consequência: a query retorna **todos os valores individuais**, sem agregação. Para somar valores, use uma **CUBE** calculation view.
- Para obter **lista distinta** de valores, você pode definir **aggregation** em qualquer coluna (cada valor único aparece uma vez).
- Pode combinar várias tabelas de master data (ex.: join de customer com customer contacts).
- Pode ser consumida diretamente por ferramentas de relatório, mas o mais comum é ser consumida em **cube calculation views**.
- **Compartilhamento:** a mesma dimension **Product** pode ser usada no cube de **purchase order** e no cube de **sales order**.

---

## Lição 2.3 — Creating Cube Calculation Views

### CUBE
- Use o tipo **CUBE** quando a calculation view precisa ter **measures**.
- Por padrão, **as measures são sempre agregadas pelos attributes pedidos na query**. Mesmo que a view tenha muitos attributes, a agregação acontece só pelos solicitados.
  - Ex.: measure *revenue*, attributes *country* e *city*. Query pede só *country* → revenue somado por country. Outra query pede country e city → revenue somado por city e country.
- Otimizada para **ad-hoc analysis** — slice-and-dice imprevisível por qualquer combinação de attributes.

### CUBE with Star Join
- Extensão do cube: permite **conectar dimension calculation views** ao cube, ampliando muito os attributes disponíveis para análise. Ex.: sales cube com product number + dimension Product → descrição, supplier, cor, preço.
- É o **tipo mais avançado** de calculation view, popular em cenários analíticos ad-hoc. Processado por um **OLAP engine** dedicado → alta performance.
- Os joins entre fact e dimensions são definidos no **Star Join node**. Tipos disponíveis:
  - **Referential Join**
  - **Inner Join**
  - **Left Outer Join**
  - **Right Outer Join**
  - **Full Outer Join** (com restrições específicas)
  - **Text Join**
- Estrutura típica: **Join node** (atua como fact table) → **Star Join node** (com várias dimension views) → **Semantics node**.

### Shared Columns (colunas compartilhadas)
Na aba **Columns** do **Semantics node** de um CUBE with Star Join, as colunas se dividem em:
- **Private:** definidas dentro da própria calculation view (measures ou attributes). Controle total.
- **Shared:** definidas "externamente" nas dimension calculation views referenciadas. Menos controle (podem ser compartilhadas com outros cubes), mas é possível **ocultar** as que não precisa.
- Em shared columns, **Name** e **Label não podem ser alterados**, mas você pode definir **Alias Name** e **Alias Label**.
- **Alias Name é obrigatório** quando nomes de colunas das dimensions conflitam entre si ou com nomes de private columns.

---

## Lição 2.4 — Creating Time-Based Dimension Calculation Views

- Existem dois tipos de dimension calculation view: **Standard** e **Time**.
- A **time-based dimension calculation view** gera automaticamente vários attributes de data a partir de uma **data base**:
  - Número do dia da semana (ex.: quarta = dia 3)
  - Número da semana no ano (ex.: 19 fev = semana 7)
  - **Quarter** (12 dez = Q4)
  - **Half-year** (12 dez = segundo semestre)
  - **Fiscal period** (27 abr 2022 = período fiscal 01/22)
- Não é preciso ter todos esses attributes na fonte nem escrever SQL complexo — basta a data base.
- **Caso de uso principal:** permitir que o usuário **agregue measures de um cube por qualquer attribute de data**. O modelador pode criar uma **time hierarchy** para drill-down: ano → semestre → trimestre → mês → semana → dia (e até hora, minuto, segundo).
- Ao criar a view do tipo **TIME**, também é gerada uma **tabela** (ex.: `M_TIME_DIMENSION`) preenchida automaticamente com registros para o **intervalo de datas** escolhido (ex.: 2020–2024). Pode ser **regenerada** a qualquer momento. Essa tabela é a data source da time view, que depois é consumida no cube. Colunas geradas incluem `CALQUARTER`, `CALMONTH`, `CALWEEK`…
- Há diferentes **calendar types** (ex.: **Gregorian** e **Fiscal**).

---

## Lição 2.5 — Choosing a Data Source for a Calculation View

Data source types suportados nas calculation views do SAP HANA Cloud:
- **Row Table**
- **Column Table**
- **Virtual Tables**
- **Calculation Views**
- **SQL Views**
- **Table Functions**

> **Flat files NÃO são data source direto** de calculation view (só acessíveis indiretamente, ex.: via virtual table).

- **Row vs. Column tables:** para o modelador não faz diferença; todas as funções de modelagem funcionam com ambas. **Column tables** são otimizadas para leitura rápida — o tipo mais usado com calculation views.
- **Virtual table:** tabela no SAP HANA Cloud **mapeada para uma remote table** fora dele. Permite alcançar fontes externas (tabelas de outros bancos, flat files, planilhas…). **Atenção à performance** — pode impactar muito o tempo de resposta.
- **Calculation views** como fonte: recomendado desenvolver **em camadas (layers)** para quebrar modelos complexos e **reutilizar** lógica, evitando redundância.
- **SQL Views:** criadas em SQL; mais simples que calculation views, mas permitem combinar fontes e adicionar colunas calculadas.
- **Table Functions:** definem fontes complexas usando **SQLScript**, depois consumidas pela calculation view.
- **Regra geral:** prefira sempre as funcionalidades **gráficas padrão**; recorra a SQL apenas quando necessário.

---

## Lição 2.6 — Checking the Output of a Calculation View

### Data Preview
Duas formas:
1. Clique direito no **top node** de uma calculation view aberta → **Data Preview**.
2. Clique direito numa calculation view **já deployada** na Explorer pane → **Data Preview** (útil para views não abertas).

- O data preview embutido no BAS equivale ao do **SAP HANA Database Explorer**. O Database Explorer pode ser aberto pela pane **SAP HANA Projects**: selecione o **database module** (ex.: `HC300/db`) → **Open HDI Container**.
- No preview você vê o **SQL** gerado: `SELECT` de todas as colunas, sem filtros, incluindo as aggregate functions do top view node. Você pode **editar a query** (remover colunas, adicionar filtros) para testar.

### Abas do Database Explorer (data preview)

| Aba | Mostra | Uso |
|---|---|---|
| **Raw Data** | Todos os dados | Visualização básica |
| **Hierarchies** | Dados de uma **SQL Hierarchy** selecionada | Preview de hierarchy modelada |
| **Analysis** | Attributes e measures selecionados em tabelas ou gráficos | Profiling e análise |

> O Database Explorer **não é ferramenta de reporting**, mas ajuda na modelagem e troubleshooting.

### Qual usuário executa o Data Preview
- Até **QRC 3/2021**: o preview era executado com o **technical user `..._RT`** (runtime user do container), com privilégios amplos nos objetos do container.
- Desde **QRC 4/2021**: você pode escolher **Data Preview** (com o `_RT`) ou **Data Preview With Other Database User** — permite aplicar **session context** (client, **analytic privileges** etc.).
- O "other user" deve ser escolhido entre os usuários das **connections** definidas no Database Explorer; ele precisa estar atribuído a um **SAP HANA Database** (não dá para adicionar um HDI container com outro usuário que não o `_RT`). A lista de connections **não é filtrada** — escolha uma com acesso ao schema do HDI container.

### Execução manual vs. automática
- Até QRC 4/2021 a query de preview era executada automaticamente.
- Desde **QRC 1/2022** há uma opção no BAS para decidir se executa automaticamente ou manualmente — evita rodar uma **query cara** antes de ajustá-la. Procure por **Manual Refresh** nas preferências (pode ser por **User** ou **Workspace**, e por dev space).

### SQL Hierarchies no Data Preview
- Desde **QRC 1/2022**, **SQL hierarchies** modeladas numa calculation view podem ser visualizadas no Data Preview do BAS, sem ferramenta externa; dá para ver e copiar o SQL de navegação da hierarchy.

---

## Lição 2.7 — Working with Common Features of Calculation Views

### Semantics node e Column Semantics
- Toda calculation view tem um **Semantics node** — já vem criado e é **sempre o nó do topo**, em qualquer tipo de view.
- Nele você atribui semantics a cada coluna (comportamento e significado), informação usada pelos **consuming clients**.
- Configuração **obrigatória** de cada coluna: **column type** → **attribute** ou **measure**.
- Opcional: **semantic type** (ex.: **Date**, **Amount with Currency Code**, **Quantity with Unit of Measure**). Ajuda o front-end a formatar corretamente — mas **é responsabilidade do front-end/cliente** usar essa informação.
- Outras configurações por coluna:
  - **Description/label column** associada (ex.: product id → product description).
  - **Hidden** (ocultar coluna).
  - **Variable** atribuída (usuário escolhe valor de filtro em runtime).
  - **Name e Label** alternativos — ex.: ninguém quer ver `MATNR` no relatório; prefira *Material Number*.

### Data Source Aliases
- Se usar **a mesma data source mais de uma vez** na view, cada instância precisa de **nome único** → **alias**.
- O alias fica em **Properties** ao clicar no cabeçalho da data source (aba Mapping).
- O BAS sugere um alias adicionando número incremental (ex.: `HA300::SNWD_BP_1`). **Recomendado:** substituir por um nome significativo.

### Hiding Columns
- Colunas usadas em cálculos mas que **não devem aparecer** no relatório podem ser marcadas como **Hidden** (checkbox no Semantics node).
- Continuam utilizáveis dentro da view (ex.: em calculated columns). Ex.: `FIRST_NAME` e `LAST_NAME` ocultas porque foram combinadas em `FULL_NAME`.

### Label Columns
- Associe a coluna técnica (ex.: Employee ID) à coluna descritiva (Employee Name). Em ferramentas que suportam, a label column aparece como **Text** e a base como **Key**; o usuário pode alternar entre elas.

### Sorting the Data Set
- Você pode definir uma **ordenação do result set** aplicada quando a query do cliente não especifica nenhuma → ordem estável. Útil também no preview.
- É possível ordenar por coluna oculta, mas geralmente não faz sentido.
- **Cuidado:** se a query do cliente define sua própria ordenação (outras colunas ou só parte delas), a ordem definida na view **não é garantida**. Ex.: view ordena por `SO_ID, PRODUCT_ID`; front-end ordena por `PRODUCT_ID` → não necessariamente `PRODUCT_ID, SO_ID`. Para isso, o `ORDER BY` da query deve ter as duas colunas.

### Null Values (Null Handling)
- Attributes e measures podem ter valores nulos (aparecem como `?`). Você pode definir um **default value**.
- Passos: Semantics node → aba **Columns** → selecionar coluna → checkbox **Null Handling** → opcional: **Default Value**.
- Sem default value definido: **0** para colunas numéricas; **string vazia** para `NVARCHAR`.

### Propriedades gerais da calculation view
Ficam na aba **View Properties** do Semantics node, organizadas em: **General**, **Advanced**, **Static Cache**, **Snapshots**.

**General**

| Property | Descrição |
|---|---|
| **Data Category** | Se a view suporta reporting multidimensional (DIMENSION, CUBE…). |
| **Type** | **Standard** ou **Time**. |
| **Run With** | Como aplicar segurança ao executar view script-based. |
| **Default Client** | Como filtrar por **SAP CLIENT** (MANDT). |
| **Apply Privileges** | Se **analytic privileges** se aplicam (BAS suporta apenas **SQL analytic privileges**). |
| **Default Member** | Membro padrão para todas as hierarchies da view. |
| **Deprecate** | Marca views não recomendadas para reuso (ainda suportadas). |
| **Enable Hierarchies for SQL access** | Num CUBE with star join, se hierarchies das dimensions compartilhadas podem ser consultadas via SQL. |
| **End-User View** | Se a view é oferecida como fonte para ferramentas de relatório. |

**Advanced**

| Property | Descrição |
|---|---|
| **Propagate instantiation to SQL Views** | Se a view é referenciada por SQL view ou CDS view, define se ela é **instanciada** (ex.: pruning de colunas não selecionadas) ou executada exatamente como definida. |
| **Analytic View Compatibility Mode** | Join engine ignora joins com cardinalidade **n..m** no star join quando nenhuma coluna de uma das tabelas é consultada. |
| **Ignore Multiple Outputs For Filter** | Otimização: faz **push down** de filtros mesmo se um node é usado em mais de um node superior. |
| **Evaluate Session Variables Early** | Otimiza **union pruning** considerando valores de session variables. |
| **Pruning Configuration Table** | Tabela com as regras de pruning dos **Union nodes**. |
| **Execute in** | Executar no **SQL engine** ou **column engine**. |
| **Execution Hints** | Como os engines do SAP HANA devem tratar a view em runtime. |

---

## Lição 2.8 — Top View Node

- Ao criar uma calculation view, **dois nós** são criados automaticamente:
  1. **Semantics node** (topo): enriquece o result set final (ex.: currency nas measures, **masking** de dados sensíveis). **Obrigatório**; nada pode ser adicionado acima dele.
  2. **Top view node** (logo abaixo): gera o data set calculado **antes** de aplicar as semantics.
- O tipo do top view node padrão depende do tipo de view, e **pode ser trocado**:

| Data category | Top view node padrão | Resultado | Se trocar… |
|---|---|---|---|
| **DIMENSION** | **Projection** | Lista completa de attributes | → **Aggregation**: lista **distinta** de attributes |
| **CUBE** | **Aggregation** | Agrega measures pelos attributes selecionados | → **Projection**: measures **não agregadas**; → **Star Join**: acessa attributes de várias dimensions |

- Junto com a troca de **data category** (em Properties), evita criar uma view nova só para mudar o passo final. O editor tenta preservar o máximo da definição.
- **Não é possível remover** o top view node nem adicionar nós acima dele.
- Uma view muito simples pode não ter nenhum nó abaixo do top view node.
- **Por que o Semantics fica separado acima do top view node:**
  - Gerenciar o top view node como qualquer outro nó do mesmo tipo (aggregation, projection ou **star join — usado exclusivamente como top view node**).
  - Exibir separadamente as informações globais (propriedades, cache e snapshots, lista de colunas incluindo shared dimensions, **column lineage**).
  - Definir configurações adicionais: semantics das measures (currency conversion), **hierarchies**, **variables**, **input parameters**.

---

## Pontos-chave para a prova (Unit 2)
- **Measure** = numérico agregável; **Attribute** = descritivo. **Dimension** = grupo de attributes.
- **DIMENSION view:** só attributes, master data, reusável; top node padrão **Projection**.
- **CUBE:** measures agregadas pelos attributes pedidos; top node padrão **Aggregation**.
- **CUBE with Star Join:** conecta **DIMENSION views** a um fato; OLAP engine; Star Join só como top view node; shared columns → **Alias** obrigatório em conflito.
- **TIME dimension:** gera attributes de tempo a partir de uma data base; gera tabela de tempo.
- Data sources: row, column, virtual tables, calculation views, SQL views, table functions (**não** flat file direto).
- Data Preview abre equivalência com **SAP HANA Database Explorer**; usuário `_RT` vs **Other Database User**.
- **Hidden column** = usada em cálculo mas não exibida.
- Null Handling sem default: 0 (numérico) / vazio (NVARCHAR).
- Calculation views são **read-only**, calculam **on-the-fly** e **se adaptam à query**.

---

## Quiz oficial da Unit 2 — com gabarito comentado

1. **Qual ferramenta é lançada pelo Data Preview de uma calculation view?** → ✅ **SAP HANA Database Explorer**
2. **Papel da dimension calculation view?** → ✅ **To generate a view of master data from one or more tables**
3. **Papel da cube calculation view?** → ✅ **To aggregate measures without the need for dimensions** *(o star schema é papel do CUBE with Star Join)*
4. **Top view node padrão de uma dimension calculation view?** → ✅ **Projection**
5. **Data source types suportados?** (4 corretas) → ✅ Calculation views, ✅ Virtual tables, ✅ Column tables, ✅ Row tables — ❌ Flat files
6. **Benefícios das calculation views?** (2 corretas) → ✅ **They calculate live data on-the-fly**; ✅ **They adapt automatically to the requesting query** — ❌ "any database language"
7. **Por que ocultar colunas?** → ✅ **Quando a coluna é usada num cálculo mas não precisa aparecer no relatório.** *(Para dados sensíveis use masking/privileges, não hidden.)*
8. **Por que criar uma time-based dimension view?** → ✅ **To automatically generate time-related attributes from a base date**

---

## Glossário da Unit 2
- **Calculation view:** principal objeto de modelagem do SAP HANA Cloud (read-only, calculada on-the-fly).
- **Data Category:** DIMENSION, CUBE (com ou sem Star Join).
- **Star schema / Fact table:** modelo com tabela fato central ligada a dimensions.
- **OLAP engine:** engine analítico usado para star joins.
- **Semantics node / Top view node:** nó de enriquecimento final / nó que produz o data set final.
- **Semantic type:** significado da coluna (Date, Amount with Currency Code, Quantity with Unit of Measure).
- **Private / Shared columns:** colunas da própria view / vindas de dimension views.
- **Design-time / Runtime object:** arquivo fonte / objeto gerado no banco após deploy (ex.: column view).
- **HDI container:** container de runtime objects.
- **Virtual table:** tabela local mapeada a uma remote table.
- **Table function:** função SQLScript que retorna tabela.
- **QRC (Quarterly Release Cycle):** ciclo de release trimestral do SAP HANA Cloud.
- **_RT user:** technical runtime user do HDI container.
# Unit 3 — Working with Common Nodes in Calculation Views (Nós comuns: Projection e Aggregation)

**Foco da unit:** os dois nós mais usados — **Projection node** e **Aggregation node** — e como controlar a agregação com **Keep Flag** e **Transparent Filter**.

---

## Lição 3.1 — Using Projection Nodes

- O **Projection node** é um dos tipos de nó mais populares; aparece na maioria das calculation views de todos os tipos.
- Na aba **Mapping** você escolhe quais colunas da fonte são mapeadas para o output.
- Usos típicos do Projection node:
  1. **Selecionar só as colunas necessárias** de uma data source.
  2. Definir **calculated columns**.
  3. Definir **parameters** que pedem valores em runtime (**input parameters / user prompts**).
  4. **Aplicar filtro** sobre a data source.

> Projection **não agrega** e **não faz join**.

---

## Lição 3.2 — Working with Aggregation Nodes

### Propósito
- Aplicar **aggregate functions** às measures com base em um ou vários attributes. O resultado é **agrupado (grouped by)** pelos attributes selecionados, calculando a measure com a função escolhida (**SUM**, **MIN**, **AVG**…).
- A **granularidade** da agregação é definida pelos **attributes mapeados no output** do nó. Ex.: com Country + Order no output → uma linha por par country/order, com `Amount_Total` (SUM) e `Largest_Amount` (MAX = maior item da ordem).

> **Nota (agregação em dois níveis):** se você quer o total e a **maior ordem** por país, não basta tirar a coluna Order — você ainda obteria o **maior item de linha** por país. Solução: primeiro um Aggregation node somando (SUM) por ordem, depois outro Aggregation node por cima aplicando MAX — **stacked aggregation**.

- Se um attribute for excluído num nó superior (na mesma view, numa view consumidora ou na query), por padrão ele é **ignorado** e a view agrega pelos attributes restantes.

### Aggregate functions suportadas
- **Sum** (padrão)
- **Min**
- **Max**
- **Count**
- **Average**
- **Variance**
- **Standard deviation**
- **Median**

> **Cuidado:** usar essas funções com atenção em cenários **stacked** (views empilhadas) para não gerar resultados incorretos (ex.: média de médias).

### Calculated column no Aggregation node
- **Num Aggregation node, a calculated column é sempre calculada DEPOIS da agregação.**
- Se precisar calcular **ANTES** de agregar (ex.: quantidade × preço por linha), defina a calculated column num nó inferior (ex.: **Projection node**) executado antes do Aggregation.

### Agregação definida na query do cliente
Ao executar SQL sobre um CUBE (com ou sem star join), attributes não selecionados são ignorados. Mas a agregação depende de haver **GROUP BY**:
- **Sem GROUP BY:** cada measure é agregada conforme definido nas **Semantics** da view.
- **Com GROUP BY:** a aggregate function escrita na query para cada measure **sobrescreve** a definida nas Semantics. Flexível, mas pode gerar erros — use com cuidado.

> O SQL padrão de **data preview** de um CUBE (BAS e Database Explorer) usa a segunda abordagem (GROUP BY), mostrando a função de cada measure conforme as Semantics.

### Aggregation Attributes (desde QRC 1/2023)
- No **Semantics node** de um CUBE você pode informar, para cada measure, **quais attributes devem ser mantidos** ao agregar.
- Ex.: `GROSS_AMOUNT` não deve ser agregado sem manter `CURRENCY_CODE` (senão soma USD com EUR).
- A informação é usada por ferramentas de relatório via **SAP HANA Analytics Catalog** (**BIMC views**, schema **`_SYS_BI`**); tabela principal: **`BIMC_EXCEPTION_AGGREGATION_ATTRIBUTES`**.
- **Cuidado:** essa configuração **não altera** a execução da agregação pelo calculation engine e **não impede** queries que retornem dados inconsistentes.

---

## Controlando o comportamento do Aggregation node

As colunas pedidas pela query podem alterar como a agregação é executada. Duas features ajudam:

### Keep Flag
- Cenário: sales orders com **quantity** e **price**; cada order = 1 store, 1 customer, 1 produto. Queremos as vendas de *Mouse* em *fevereiro*.
- Dados: 4 ordens, cada uma com quantity 2 e price 5.
  - **Sem Keep Flag:** agrega no nível (Month, Product) → soma quantities = 8, soma prices = 20 → 8 × 20 = **$160 (errado)**.
  - **Com Keep Flag em ORDER_ID:** o cálculo é feito por ORDER_ID (2 × 5 = 10, × 4) → **$40 (correto)**.
- **Keep Flag = true** força o cálculo a acontecer no **nível de granularidade relevante**, mesmo que a query agrupe em nível mais alto.
- Também pode ser definido em **shared columns** no **Star Join node** de um CUBE with Star Join.

### Transparent Filter
- Em alguns cenários, um filtro (`WHERE`) na query força a coluna filtrada a entrar no **GROUP BY**.
- Cenário: contar lojas que venderam mouse para John **ou** Susan. Susan comprou na TigerDirect; John na TigerDirect e Amazon.
  - **Sem Transparent Filter:** calcula StoreCount por produto **e por customer** → 1 + 2 = **3 (errado**, TigerDirect duplicada).
  - **Com Transparent Filter em Customer:** StoreCount = **2 (correto)**.
- **Transparent Filter = true** em todos os modelos/nós que referenciam a coluna impede que ela entre desnecessariamente no GROUP BY.
- Necessário quando:
  - Há **stacked views** com **distinct count measures** nas views inferiores.
  - Queries na view superior têm **filtros em colunas que não são projetadas**.
- **Nota:** definido numa output column de um nó, influencia só o cálculo do **nó superior**. Para ser transparente para uma view consumidora, marque Transparent Filter **em todos os nós, do nó até o topo**.

---

## Pontos-chave para a prova (Unit 3)
- **Projection:** selecionar colunas, filtrar, calculated columns, input parameters.
- **Aggregation:** GROUP BY pelos attributes do output; função padrão **SUM**; 8 funções (Sum, Min, Max, Count, Average, Variance, Std deviation, Median).
- **Calculated column em Aggregation node = depois da agregação.** Para antes → Projection abaixo.
- Query com **GROUP BY** sobrescreve a função das Semantics.
- **Keep Flag:** força granularidade (ex.: ORDER_ID) em cálculos qty × price.
- **Transparent Filter:** evita que coluna filtrada (não projetada) entre no GROUP BY; essencial com **distinct count** em stacked views.

---

## Quiz oficial da Unit 3 — com gabarito comentado
1. **Propósito do Projection node?** (2 corretas) → ✅ To apply filters on the data; ✅ To extract only the required columns from a data source — ❌ aggregate, ❌ join.
2. **Quando a calculated column é computada num aggregation node?** → ✅ **After aggregation**.

---

## Glossário da Unit 3
- **Projection node:** nó para selecionar colunas, filtrar e calcular.
- **Aggregation node:** nó que agrupa e aplica aggregate functions.
- **Granularity:** nível de detalhe do resultado.
- **Stacked views / stacked aggregation:** views/nós empilhados.
- **Keep Flag:** mantém uma coluna no nível de cálculo mesmo não pedida.
- **Transparent Filter:** evita que coluna filtrada entre no GROUP BY.
- **Distinct count:** contagem de valores distintos.
- **BIMC views / _SYS_BI:** catálogo analítico do SAP HANA usado por ferramentas de BI.
# Unit 4 — Joining Data Sources in Calculation Views (Joins em Calculation Views)

**Foco da unit:** tipos de **join** (Inner, Left/Right/Full Outer, **Referential**, **Text**, **Temporal**, **Non-Equi**, **Dynamic**), **cardinality**, **Key indicator**, **Multi-Join**, **filter mapping** e otimizações (**join pruning**, **Optimize Join Columns**, **Greedy Join Pruning**). É uma das units mais pesadas e cobradas.

---

## Lição 4.1 — Using Join Nodes

### Conceitos
- O **Join node** define joins entre duas ou mais data sources.
- O **Star Join node** (em CUBE with Star Join) também define joins entre a fonte principal (**fact**) e as **DIMENSION calculation views**.
- Em cada join você define: **colunas da join condition**, **join type** e, opcionalmente, **cardinality** (com ajuda do **Propose Cardinality**).

### Cenário de exemplo (para entender os tipos)
Tabelas **Sales Order**, **Customer**, **State**:
- Sales Order 8 **não tem** customer cadastrado.
- Customer **TOM não tem** ordens.
- State **TX não tem** descrição.
- **Nenhum** customer mora no **Alabama**.

### Inner Join
- Tipo mais básico: retorna linhas **quando há match nos dois lados**.
- Num CUBE: **fatos podem ser perdidos** (ordens sem customer/estado correspondente somem).

### Left Outer Join
- Retorna **todas as linhas da tabela da esquerda**, mesmo sem match na direita (campos da direita ficam vazios/placeholder).
- Com **design-time filters**: os filtros são aplicados nas duas tabelas e **depois** o join é executado.
- Num CUBE: todas as ordens são retornadas (inclusive sem customer), mas análise por customer/estado pode trazer dados "irrelevantes" (placeholders). TOM não aparece (não tem ordem).

### Right Outer Join
- Retorna **todas as linhas da tabela da direita**, mesmo sem match na esquerda. Ex.: Alabama aparece com placeholders de customer e amount.

### Full Outer Join
- Combina Left + Right: linhas com match + linhas da esquerda sem match + linhas da direita sem match.
- **Cuidado:** suportado **apenas em calculation views**, nos nós **Join** e **Star Join**. No **Star Join**, o full outer join só pode ser definido em **uma** DIMENSION calculation view, que deve **aparecer por último** no Star Join node.

### Referential Join
- Join **otimizado para performance**.
- Princípio: se a **referential integrity** entre as tabelas é garantida, em certas circunstâncias o join **não é executado** (economiza tempo).
- **Referential integrity:** nas colunas do join, sempre existe match em B para cada linha de A (ou o contrário, ou ambos).
- Ex.: CUBE with star join com várias dimensions, joins **n..1**, e o sistema fonte garante que todo fato tem match em todas as dimensions → o engine pode **podar (prune)** as dimensions das quais a query não pede colunas.
- Pode ser usado em **Join node** e **Star Join node**.

| Característica | Descrição |
|---|---|
| Depende de referential integrity | Matches existem entre as tabelas (esq→dir, dir→esq ou ambos). |
| Otimizado para performance | Não é executado quando o resultado sem o join seria igual. |
| **Funciona como Inner Join quando executado** | Se não for podado, roda como **Inner Join**. |

**Configuração do Referential Join:**
- **Join Type** = **Referential**.
- **Cardinality** — **deve** ser especificada; sem ela, o join **não pode ser otimizado**.
- **Integrity Constraint** — direção em que a integridade é garantida:
  - **Left:** toda linha da esquerda tem pelo menos um match na direita.
  - **Right:** toda linha da direita tem pelo menos um match na esquerda.
  - **Both:** ambos.
  - Se Integrity = Left, **não há otimização** se a query pede colunas só da tabela da direita.

**Condições para o Referential Join ser podado (A e B):**
1. **Nenhum campo de B** é solicitado.
2. **Integrity** está em **A**.
3. Cardinality do lado de B é **..1**.

> Se a cardinality do lado B **não** for ..1, o join é **sempre executado** (o número de linhas depende de quantos matches existem em B).

- Ex.: `SNWD_SO_I` (esquerda) com `CVD_PD` (direita), Referential, Integrity Left, n..1 → cada item tem **exatamente um** match → se a query não pede colunas de CVD_PD, o join não é executado.
- **Cuidado:** use com cautela. Se a integridade **não** for garantida, os resultados podem **variar** conforme você seleciona ou não colunas de uma das fontes.

### Text Join
- Permite exibir labels de attributes **no idioma do usuário**. Pré-requisito: a fonte **já tem** os textos em vários idiomas — **calculation views não traduzem**.
- Tecnicamente: comporta-se como **Left Outer Join** com cardinality **1:1**, mais uma **Language Column** (em tabelas SAP, normalmente **`SPRAS`**).
- Em runtime, o **idioma do usuário** é usado como filtro para buscar a descrição na **text table**. Ex.: **`MARA`** (material) + **`MAKT`** (textos de material) com `SPRAS`.

### Temporal Join
- Master data com **histórico** (ex.: vários registros por funcionário, cada um com validade).
- Adiciona uma **temporal condition**: os registros só casam se uma **data** de uma tabela estiver **dentro do intervalo** definido por duas colunas da outra.
- Ex.: Sales Orders × Customer Status (status muda no tempo). **Inner Join** em `ContactID` + condição temporal:
  - **Temporal column:** Date (Sales Orders)
  - **From Column:** DateFrom (Customer Status)
  - **To Column:** DateTo (Customer Status)
  - **Temporal Condition:** **Include Both** (também existem Exclude Both, Include To/Exclude From, Exclude To/Include From)
- **Regras:**
  - Tipos de dado suportados: **timestamp**, **date**, **integer**.
  - Só colunas já mapeadas no output do Star Join node podem ser temporal columns.
  - **Temporal Joins só são suportados no Star Join de CUBE with Star Join**, e o join type deve ser **Inner**.

### Column Key Indicator
- Ícone que destaca colunas que pertencem à **key** da data source (aparece nas abas **Join Definition** e **Mapping**).
- Determinado por: **primary key** (se a fonte é tabela) ou **key columns definidas nas semantics** (se a fonte é calculation view).

| Ícone | Significado |
|---|---|
| **Dourado** | Key **completa**. |
| **Tracejado** | Key **incompleta** (só parte de uma key composta foi projetada). |
| **Cinza** | Operações intermediárias (ex.: **Join**, **Union**, **Table Function**) podem ter removido a propriedade de key. |

- Ex.: Purchase Order (key = PurchaseOrderId → dourado) × Purchase Order Items (key composta PurchaseOrderId + PurchaseOrderItem; item não mapeado → tracejado).

### Join Cardinality
- Define como as linhas das duas tabelas se relacionam. Ex.: Sales Order (esq.) × Customer (dir.) = **n:1** (várias ordens para um cliente; uma ordem não tem vários clientes).
- **Recomendação:** só especifique a cardinality quando tiver **certeza** do conteúdo. Senão, deixe em branco.
- **Propose Cardinality:** o BAS sugere a cardinality analisando as tabelas **no momento** da definição. Se os dados mudarem, ela pode ficar errada. Ex.: dados de teste com só 1 ordem por cliente → sugestão 1..1 (irreal).

### Multi-Join (mais de duas fontes num Join node)
- Um join precisa de no mínimo 2 fontes, mas o Join node aceita **mais de duas** (**multi-join**), com uma **central table**.
- A propriedade **Multi-Join Order** tem as opções **Outside-In** e **Inside-Out**; só se aplica em alguns cenários (precedência entre certos joins).
- Quando a ordem importa:
  - Todos **Inner Joins** → resultado geralmente **igual** independentemente da ordem.
  - Mistura de **Inner** e **Left Outer** → resultado **pode variar** com a ordem.
- O modelador decide entre multi-join num nó ou joins de 2 em 2, considerando comportamento e **legibilidade**.

### Non-Equi Join
- Join cuja condição **não usa `=`**, mas outros operadores (>, <, >=, <=, !=).
- No BAS é um **tipo de nó próprio** onde você define o operador.
- Suportado para: **Inner**, **Left Outer**, **Right Outer**, **Full Outer**.
- O operador pode ser diferente para cada par de colunas.
- Ex.: `ProductsToBeDelivered.id = SUBTASKS.id` **e** `ProductsToBeDelivered.dueOn < SUBTASKS.plannedDate` → sub-tarefas que não terminam a tempo.

### Dynamic Join
- Permite análise em **diferentes níveis de granularidade** com a **mesma** calculation view.
- Ex.: mostrar lado a lado **vendas por produto** e **vendas totais**, por **Country** ou por **Region**. Com **Regular Join** em Country+Region: correto por country, **inconsistente por region**.
- **Com Dynamic Join:** só as colunas de join **pedidas na query** participam do join.
- **Só pode ser definido em multi-column joins** (join com mais de uma coluna).
- Comportamento de agregação:
  - **Regular (static) join:** agregação **depois** do join.
  - **Dynamic join:** se uma coluna de join não é pedida, é feita uma **agregação para removê-la** e depois o join usa só as colunas pedidas.
- **Cuidado:** se **nenhuma** coluna de join for pedida → **runtime error** na query.

---

## Lição 4.2 — Filtering on Join Nodes

- Ao definir um filtro (filter expression) num join node, você pode **mapear o filtro** entre as fontes (**filter mapping**): um filtro numa coluna de uma fonte é aplicado também numa coluna da outra fonte pelo **SQL optimizer**.
- Desde **QRC 4/2023**, filter mapping também funciona em **non-equi joins**.
- Definido graficamente na aba **Join Definition**, que tem dois modos: criar **Join** ou **Filter mapping** ao arrastar colunas. Você pode exibir só joins, só filter mappings ou ambos.
- Deve-se definir a **Direction** (de qual lado o filtro é propagado). Padrão: **Left ↔ Right** (bidirecional). Conector **laranja**, com ícone de filtro e setas.
- **Benefícios:**
  - **Melhor performance:** filtros executados cedo nas duas fontes, **antes do join**. Ex.: o optimizer não sabe que `product` (esq.) = `product_name` (dir.); mapeando o filtro, a fonte da direita é reduzida.
  - **Consistência de dados** em alguns casos.

---

## Lição 4.3 — Optimizing Joins

### Definir cardinalities
- Cardinalities: **1..1**, **1..n**, **n..1**, **n..m**.
- **Recomendado sempre definir** para que o optimizer decida se joins podem ser omitidos em runtime.
- **Propose Cardinality** só disponível quando as fontes são **tabelas** (não calculation views).
- **Cuidado:** a proposta reflete os dados **no momento** e **não se atualiza**. Use dados **realistas**.

### Join pruning — condições
Uma fonte pode ser podada automaticamente quando:
- a) **Nenhum campo** da tabela a ser podada é pedido;
- b) O join type é **Outer, Referential ou Text**;
- c) A cardinality é **..1** para a tabela a podar **ou** só são pedidas measures com **count distinct** ou nenhuma measure.

- Princípio: o optimizer só poda se **não puder afetar o resultado**.
- **Cuidado:** essa poda padrão **não ocorre em Inner Join**. Para otimizar inner join, use **Referential**.

Exemplos (nenhuma coluna da direita pedida):
- **Left outer + 1..1** → a direita R **pode ser podada**.
- **Left outer + 1..n** → **não** pode (pode haver vários matches).
- **Right outer** → **nunca** poda a direita, qualquer que seja a cardinality.

### Referential (de novo, visão de otimização)
- "Um **inner join que suporta otimização**". Com a cardinality correta, retorna sempre o mesmo resultado de um inner join e usa o **Integrity Constraint** para decidir se pode ignorar o join.
- Diferença para Outer/Text: comporta-se como **inner**. Com cardinality **1..1** e Integrity em **Both**, permite otimização **qualquer que seja o lado** sem colunas pedidas.

### Recomendações gerais de join
- **Sempre** mantenha a cardinality.
- Prefira **n:1 e 1:1 para left outer** e **1:n e 1:1 para right outer**.
- **Reduza** o número de colunas de join.
- **Evite** join em **calculated columns**.
- **Evite** **type conversions** em runtime.
- Verifique se **Dynamic Join** e **Optimize Join Columns** podem ser usados.

### Optimize Join Columns
- **Padrão:** a coluna de join é **sempre incluída** no resultado, mesmo que não pedida — para garantir agregação consistente.
- Marcando **Optimize Join Columns**, você diz ao engine que, se o parceiro for podado, o resultado **não muda** se a coluna de join não participar da agregação. Depende das agregações: **SUM não é sensível** ao nível de agrupamento; **MAX/MIN são sensíveis**.
- Condições para podar a coluna de join entre A e B (todas):
  1. Join type **Referential, Outer ou Text** (com **Inner** a view nem compila com essa opção).
  2. Só colunas de **A** são pedidas.
  3. A **coluna de join de A não** é pedida.
  4. Cardinality do lado **B** é **..1**.
- **Cuidado:** depende fortemente da cardinality; se errada → resultados **instáveis**.

### Greedy Join Pruning (desde QRC 3/2022)
- Permite podar uma fonte **não consultada** (nenhuma coluna pedida) **independentemente** de cardinality e join type. Grande ganho de runtime, mas só use quando o resultado **não depende** de executar ou não o join.
- Setting **Greedy Pruning**: **Left**, **Right** ou **Both** (qual lado pode ser podado).
- Não impede os outros mecanismos de pruning.

| Nível | Precedência | Como ativar |
|---|---|---|
| Um join | 3 (menor) | Join Definition → Properties → dropdown **Greedy Pruning** |
| Todos os joins de uma view | 2 | Semantics → View Properties → **Advanced** → **Add Execution Hints** → buscar *pruning* |
| Todas as views numa query SQL | **1 (maior)** | `SELECT ... FROM ... ('PLACEHOLDER' = ('ce_settings', '{"greedy_join_pruning": "<value>"}'))` |

Valores: **1 = Disabled**, **7 = Left**, **11 = Right**, **15 = Both**.
- Hint na **query** sobrepõe a view; setting da **view** sobrepõe o do join.

---

## Pontos-chave para a prova (Unit 4)
- **Inner** perde fatos sem match; **Left Outer** mantém tudo da esquerda.
- **Full Outer** no Star Join: só **uma** dimension e **por último**.
- **Referential = Inner + pruning**; exige **cardinality** e **Integrity Constraint**; poda quando: nada pedido de B, integrity em A, cardinality B = ..1.
- **Text Join** = Left Outer 1:1 + **language column (SPRAS)**; não traduz.
- **Temporal Join** = só em **Star Join**, tipo **Inner**, colunas date/timestamp/integer.
- **Non-Equi Join** = condição sem "=" (match não é por valor exato).
- **Dynamic Join** = granularidade variável, multi-column, erro se nenhuma coluna de join pedida.
- **Multi-Join Order:** **Outside-In** / **Inside-Out**.
- **Filter mapping:** performance + consistência.
- Join pruning **não** ocorre em Inner padrão; objetivo do pruning = **performance**.
- Greedy pruning: valores 1/7/11/15; precedência query > view > join.

---

## Quiz oficial da Unit 4 — com gabarito comentado
1. **O que se obtém ao mapear um filtro num join node?** (2) → ✅ **Data consistency in the result set**; ✅ **Better performance of join execution**.
2. **Opções de multi-join order?** (2) → ✅ **Inside-Out**; ✅ **Outside-In**.
3. **Como o referential join melhora a performance?** → ✅ **By pruning joins**.
4. **Objetivo principal do join pruning?** → ✅ **To improve the performance of a calculation view**.
5. **O que implementar para analisar em diferentes granularidades na mesma view?** → ✅ **Dynamic join**.
6. **Quando implementar non-equi join?** → ✅ **When your match is not on an exact value of a column**.

---

## Glossário da Unit 4
- **Join condition / Join type / Cardinality:** colunas de ligação / tipo / relação de linhas (1..1, 1..n, n..1, n..m).
- **Referential integrity / Integrity Constraint:** garantia de match / direção dessa garantia.
- **Pruning:** eliminar (não executar) uma fonte ou coluna desnecessária.
- **Text table / Language column (SPRAS):** tabela de textos multilíngues / coluna de idioma.
- **Temporal condition:** condição de validade por datas.
- **Non-Equi Join:** join com operadores de comparação.
- **Dynamic Join:** join que usa só as colunas pedidas.
- **Filter mapping:** propagação de filtro entre fontes do join.
- **Greedy Join Pruning:** poda agressiva de fontes não consultadas.
- **Execution Hint / PLACEHOLDER ce_settings:** instruções ao engine.
# Unit 5 — Working with Union Nodes in Calculation Views (Union nodes e Union Pruning)

**Foco da unit:** **Union node** (standard union, union with constant values, Auto Map by Name, Manage Mappings, Empty Union Behavior, UNION ALL vs UNION DISTINCT) e **Union Pruning** (explicit/constant, implicit/configuration table, column-based).

---

## Lição 5.1 — Working with the Union Node

### Conceito
- O **Union node** combina **dois ou mais data sets** num **conjunto comum de colunas**. Ex.: combinar dados **Actual** e **Plan**.
- O mapeamento das fontes para o target pode ser **automático por nome** ou **manual** (drag-and-drop).
- Pode ser usado em **DIMENSION**, **CUBE** e **CUBE with Star Join**.

### Standard Union
- Para cada **target column**, há sempre **uma source column mapeada** em cada fonte.
- A source column **não precisa ter o mesmo nome** do target (ex.: `month` → `period`).
- **Não é preciso mapear todas** as colunas da fonte — algumas podem ficar de fora.
- Útil para **combinar measures de várias fontes numa só coluna**, com um **attribute** que diz o tipo (ex.: Flag = A/Actual, P/Plan) → sem perder o significado.

### Union with Constant Values
- Você fornece um **valor fixo** para preencher uma target column quando a fonte **não tem** coluna para mapear.
- Ex.: fonte A com `Amount_P` (plan), fonte B com `Amount_A` (actual). Sem attribute para diferenciar, mapeie em **colunas separadas** e preencha com **constante 0** a measure ausente de cada lado.

**Qual escolher?**
- Measures em **colunas diferentes** → **union with constant values**.
- Measures numa **única coluna** diferenciadas por um attribute (ex.: *amount type*) → **standard union**.

### Auto Map by Name

| Cenário | Resultado |
|---|---|
| **Nenhuma** data source selecionada (padrão) | **Todas** as colunas de todas as fontes vão para o output; colunas com mesmo nome são mapeadas juntas. |
| **Uma ou mais** fontes selecionadas | Só as colunas das fontes selecionadas vão para o output; colunas das outras fontes só entram se tiverem correspondente (mesmo nome). |

> Selecionar uma fonte antes é útil quando outras fontes têm muitas colunas indesejadas.

### Colunas não mapeadas (Unmapped Columns)
- Nomes diferentes com o mesmo conteúdo (ex.: `CURRENCY` vs `CURRENCY_CODE`) → mapeie **manualmente**.
- Coluna existe só numa fonte → mapeie para o target; na outra fonte, decida entre **null** ou **constant value**. Ex.: segunda fonte só tem clientes ativos → constante `Active`.
- Mesmo com coluna disponível, você pode usar uma constante para **sobrescrever** o valor da fonte.

### Manage Mappings
- Também dá para criar uma **coluna vazia no target** e dar uma constante por fonte quando **nenhuma** fonte tem a coluna. Ex.: tabela A (part-time) e B (full-time) → nova coluna *Employment Status* com `Part Time` / `Full Time`.
- Como: clique direito na target column → **Manage Mappings** → informe source column ou **constant value** por fonte (ex.: `CLIENT` = coluna fonte para ACTUAL, constante `800` para PLAN).

### Empty Union Behavior
- Quando a query pede colunas que existem numa fonte mas não na outra.
- Ex.: fonte A tem Product e Product Group; B só Product. Query pede **só Product Group**.
- Propriedade **Empty Union Behavior** (por data source):
  - **No Row** (padrão): a fonte B não retorna nada.
  - **Row with Constants:** retorna **uma linha** com a constante definida para B e nulls nas demais colunas → você **fica sabendo** que B não conseguiu fornecer dados.
- Para usar: adicione uma **constant column** no output, dê um valor por fonte e mude a propriedade para **Row with Constants**.

### UNION ALL vs UNION DISTINCT
- O Union node retorna **todos os valores, inclusive repetidos** → equivale a **`UNION ALL`** no SQL.
- Para eliminar repetições, coloque um **Aggregation node acima do Union** agregando os attributes → lista distinta → equivale a **`UNION` / `UNION DISTINCT`**.

---

## Lição 5.2 — Implementing Union Pruning

- **Union pruning** melhora muito a performance ao **ignorar fontes desnecessárias** do union.
- Três abordagens: duas **baseadas em valores** (explicit e implicit) e uma **baseada em nomes de colunas** (column-based).

### 1) Constant Pruning (Explicit Pruning)
- Cada fonte recebe uma **coluna constante** que "marca" seu significado (ex.: `plan` / `actual`).
- Se o filtro da query (`WHERE`) **não bate** com a constante de uma fonte, essa fonte é **ignorada** → **explicit pruning**.
- Ex.: coluna `temperature` com `old` (dados antigos) e `new` (atuais); `WHERE temperature = 'old'` poda a fonte atual.
- Pode haver várias constant columns e filtros AND/OR.
- **Ponto-chave:** se a query **não referencia** a constant column, a fonte **sempre** é incluída (sem pruning).
- Simples, mas exige **manutenção** em cada union node de cada view → alto esforço.

### 2) Implicit Pruning (Pruning Configuration Table)
- Regras ficam numa tabela dedicada, a **Pruning Configuration Table**, **fora** da calculation view (definida na propriedade **Pruning Configuration Table** da view).
- Não precisa de constantes no union node.
- Permite **vários valores por coluna** (lógica **OR**), operadores maior/menor e regras em **várias colunas** (lógica **AND**).
- Mais flexível; fácil de atualizar (são registros numa tabela **compartilhada** por todas as views).
- Operadores: **`=`, `<`, `>`, `<=`, `>=`, `BETWEEN`** (BETWEEN = intervalo **fechado**, inclui LOW e HIGH).
- Mesma coluna → **OR**; colunas diferentes → **AND**.
- Tipos suportados: **INT, BIGINT, VARCHAR, NVARCHAR, Date**.
- Ex.: fonte válida se `YEAR = 2008 OR 2010 OR BETWEEN 2012–2014` **AND** `STATUS = 'A' OR 'X'`. Query 2008 + status C → fonte ignorada; 2013 + A → válida.
- **Input parameters** podem direcionar a query apenas para as fontes necessárias.

### 3) Column-based Pruning
- Em vez de valores, verifica **quais colunas a query pede** e se são **importantes** para o resultado.
- Duas partes:
  1. Mapear as colunas necessárias ao output do union.
  2. Definir **Focus Column(s)**.
- Ex.: SALES só é mapeada a partir de `salesOrder2019`. SALES é **Focus Column** → se a query pede SALES e a fonte não fornece, a fonte é ignorada (mesmo que forneça YEAR etc.).
- Várias focus columns, attributes ou measures.

> É possível **combinar** as três abordagens.
> (Relacionado: a propriedade **Evaluate Session Variables Early** otimiza union pruning considerando session variables.)

---

## Pontos-chave para a prova (Unit 5)
- Union = combina data sets numa estrutura comum; **não precisa mapear todas as colunas**; nomes **não** precisam ser iguais; pode usar **constant values**.
- Union node = **UNION ALL**; + Aggregation em cima = **UNION DISTINCT**.
- **Empty Union Behavior:** No Row (padrão) vs **Row with Constants**.
- Union pruning: **Explicit (constants)**, **Implicit (configuration table)**, **Column-based (focus columns)** — **não existe** "cache-based pruning".
- Configuration table: operadores = < > <= >= BETWEEN; mesma coluna OR, colunas diferentes AND.

---

## Quiz oficial da Unit 5 — com gabarito comentado
1. **Afirmativas corretas sobre o union node?** (2) → ✅ **You do not have to select all source columns**; ✅ **You can provide a constant value when a source does not provide a column**. ❌ nomes iguais não são obrigatórios.
2. **Abordagens de union pruning?** (3) → ✅ Explicit pruning using constants; ✅ Column-based pruning; ✅ Implicit pruning with a configuration table. ❌ Cache-based pruning.

---

## Glossário da Unit 5
- **Union node:** combina conjuntos de dados.
- **Standard union / Union with constant values:** mapeamento direto / com valores fixos.
- **Auto Map by Name / Manage Mappings:** mapeamento automático por nome / gestão manual de mapeamento e constantes.
- **Empty Union Behavior:** comportamento quando uma fonte não fornece colunas pedidas.
- **Union pruning:** eliminar fontes do union em runtime.
- **Pruning Configuration Table:** tabela de regras de implicit pruning.
- **Focus Column:** coluna que decide o column-based pruning.
# Unit 6 — Creating Data Slices (Minus e Intersect Nodes)

**Foco da unit:** operações de conjunto (**set operations**) com os nós **Intersect** e **Minus**.

---

## Lição 6.1 — Implementing Minus and Intersect Nodes

### Intersect node
- Seleciona as linhas de uma data source que **também aparecem** em outra data source.
- Você escolhe as **colunas** em que a verificação de interseção é feita.
- Equivale ao `INTERSECT` do SQL.

### Minus node
- Seleciona as linhas que aparecem **numa data source mas não na outra**.
- Também escolhe as colunas da comparação.
- Equivale ao `MINUS` / `EXCEPT` do SQL.
- **A ordem importa:** o output contém os itens da **PRIMEIRA** data source que **NÃO** estão na **SEGUNDA**.
- Para inverter a ordem: clique direito no Minus node → **Switch Order**.

### Comportamento dinâmico
- Intersect e Minus são **dinâmicos**: a comparação usa apenas os attributes **pedidos em runtime**.
- Ex.: `country` e `name` mapeados no output de um Intersect.
  - Query pede **as duas** colunas → a **combinação** country+name determina o resultado.
  - Query pede **só name** → só `name` é comparado → podem voltar **mais linhas**.
- Uma coluna fornecida pelas duas fontes mas **não consultada** em runtime é **ignorada** na comparação.

---

## Pontos-chave para a prova (Unit 6)
- **Intersect** = linhas presentes nas **duas** fontes.
- **Minus** = linhas da **primeira** fonte que **não** estão na segunda; **Switch Order** inverte.
- Comparação feita só nas colunas **requisitadas** pela query (comportamento dinâmico).

---

## Quiz oficial da Unit 6 — com gabarito
1. **Por que implementar um minus node?** → ✅ **To select rows that appear in one data source but not another data source.** *(A outra alternativa descreve o Intersect.)*

---

## Glossário
- **Set operations:** operações de conjunto (union, intersect, minus).
- **Data slice:** recorte/fatia de dados.
- **Switch Order:** inverter a ordem das fontes no Minus.
# Unit 7 — Ranking Data (Rank Node)

**Foco da unit:** o **Rank node** — Top/Bottom N, **Logical Partition**, **aggregation functions** (Row, Rank, Dense Rank, Sum), **Absolute vs Percentage**, **Target Value**, **Offset** e cuidados com agregação antes do ranking.

---

## Lição 7.1 — Implementing Rank Nodes

### Propósito
- Selecionar, num data set, os **top ou bottom 1, 2… n** valores de uma measure, retornando-os com os attributes correspondentes (e outras measures, se preciso).
- Ex.: os **5 produtos com maior receita** (`GROSS_AMOUNT`) **em cada país**. O **país** define uma **Logical Partition**.

> O Rank node **NÃO faz agregação** dos dados de origem.

### Principais settings

| Setting | Para que serve |
|---|---|
| **Aggregation Function** | Cálculo principal executado no data set (Row, Rank, Dense Rank, Sum). |
| **Result Set Direction** | **Top** ou **Bottom** (Down). |
| **Result Set Type** | **Absolute** ou **Percentage**. |
| **Target Value** e **Offset** | Quantas linhas retornar / quantas pular no topo. |
| **Generate Rank Column** | Gerar uma coluna de rank e dar nome a ela. |
| **Logical Partition** | Particionar o data set por uma ou mais colunas antes do ranking. |
| **Dynamic Partition Elements** | Ajustar a partição automaticamente conforme as colunas pedidas pelo nó/view/query superior. |
| **Sort Column** | Colunas usadas para ordenar o data set para o ranking. |

### Particionamento
- A regra (ex.: top 5 vendas) é aplicada **em cada partição** (ex.: cada Country e cada Year).
- Com **Dynamic Partition Elements**, colunas da partição **não pedidas** pela query superior são **ignoradas** → top 5 por país, ou por ano, só mudando a query, **sem redesenhar** a view.

### Aggregation Functions
Duas categorias:
- **Numeram linhas:** **Row**, **Rank**, **Dense Rank**.
- **Calcula valores** da coluna ordenada: **Sum**.

| Função | Descrição |
|---|---|
| **Row** | Numeração padrão (**empates recebem números diferentes**). |
| **Rank** | Ranking "olímpico" (**empates recebem o mesmo rank**, com **lacunas** depois: 1, 2, 2, 4). |
| **Dense Rank** | Enumera todo valor de rank **sem lacunas** (1, 2, 2, 3). |
| **Sum** | **Soma cumulativa** da coluna de ordenação até a linha atual. |

- Row, Rank e Dense Rank só diferem no tratamento de **empates (ties)**.

### Várias Sort Columns
- Para Row, Rank e Dense Rank, são tratadas **em sequência** (ajuda a desempatar).
- **Sum usa só a primeira** Sort Column.

### Target Value
- Define o resultado: um **valor de rank** (ex.: até rank 2) — exceto para **Sum**, em que é um **valor cumulativo** (ex.: até soma acumulada de 200).
- Target value 2 com **Row** → sempre 2 linhas; com **Rank** → pode retornar **mais** de 2 (empates).
- Pode ser **fixo** ou definido em runtime por **input parameter**.
- O Rank node retorna linhas com ranking **≤ target value**.
- Desde **QRC 2/2024:** Target Value = **All Values** → ordena sem eliminar nenhuma linha.

### Percentage em vez de Absolute
- **Result Set Type = Percentage**. Os valores são **frações entre 0 e 1** (50% = **0.5**).
- Exemplos:
  - Os **50% produtos mais vendidos** por cliente (função **Row**).
  - Os produtos mais vendidos que somam **30% do total** por cliente (função **Sum**).

### Offset
- Exclui um número de elementos do **topo** do conjunto ordenado. Ex.: target 2, offset 2 → pula os ranks 1–2 e retorna os próximos 2.
- Ex. estatístico (**interquartile**): Row + Percentage + Target 0.5 + Offset 0.25.

### Agregar antes do ranking?
- Como o Rank node **não agrega**, a estrutura da fonte afeta o resultado.
- Ex.: fonte com Sales Order ID → várias linhas por Country+Product.
  - **Sem agregar:** retorna os produtos/valores das **melhores ordens individuais** por país.
  - **Agregando por Country e Product antes:** retorna os produtos **que mais vendem no total** por país.
- Fonte = **tabela** → talvez precise de um **Aggregation node** abaixo do Rank.
- Fonte = **CUBE calculation view** → a agregação da view é disparada implicitamente pelas colunas pedidas pelo Rank node (coluna mapeada no output mas não consumida pelo nó superior não é pedida).
- **Cuidado:** algumas colunas sobem mesmo sem serem pedidas — ex.: colunas de **sort order** definidas no Semantics da view fonte, ou com **Keep Flag**.
- Processamento posterior pode **mudar a ordem** das linhas. Para manter: `ORDER BY` na coluna de rank na query, ou opção **Sort Result Set** nas semantics.

### Rank column: Attribute ou Measure?
- **Attribute:** simples e menos sujeito a erro (não há tentação de agregar posições de ranking).
- **Measure:** mais flexível. Ex.: com agregação padrão **MAX**, você obtém o total das 5 maiores ordens por país **e** sabe quantas ordens foram somadas (países com menos de 5 ordens).
- Dê um nome explícito se `Rank_Column` (padrão) for genérico demais.

---

## Pontos-chave para a prova (Unit 7)
- Rank = **Top/Bottom N por partição**; **não agrega**.
- **Row** (empates diferentes), **Rank** (empates iguais, com gaps), **Dense Rank** (sem gaps), **Sum** (acumulado; só 1ª sort column).
- **Percentage** usa frações 0–1.
- Target value pode vir de **input parameter**; **All Values** desde QRC 2/2024.
- **Dynamic Partition Elements** = partição se adapta à query.
- Ordenar dados ("sort") **não** é motivo para usar Rank node.

---

## Quiz oficial da Unit 7 — com gabarito comentado
1. **Por que implementar um rank node?** (2) → ✅ **Identificar os produtos mais vendidos em cada região em cada um dos últimos 5 anos** (top N por partição); ✅ **Identificar os cursos que representam os 10% piores em reservas por ano** (bottom + percentage). ❌ Apenas ordenar/sortear valores não exige Rank node.

---

## Glossário
- **Logical Partition:** divisão do data set antes do ranking.
- **Result Set Direction:** Top / Bottom.
- **Ties:** empates na coluna ordenada.
- **Target Value / Offset:** limite de retorno / quantidade pulada do topo.
- **Cumulative sum:** soma acumulada.
# Unit 8 — Embedding Functions in Calculation Views (Restricted, Calculated, Filtros e Currency Conversion)

**Foco da unit:** **Restricted columns**, **Calculated columns** (granularidade, client-side aggregation), **Filters** (onde e quando são aplicados, **filter push-down**, **client-dependent views**) e **Currency Conversion**.

---

## Lição 8.1 — Generating Restricted Columns

- **Restricted column** = coluna gerada a partir de uma **measure filtrada** por um ou mais valores de attributes. Ex.: *Sales Revenue for France*; *Profit for Automotive Parts in 2022*.
- Valor = **0** para linhas que não atendem ao filtro.
- **Benefício:** fornece ao modelador um objeto pronto, já filtrado, que **simplifica a criação de calculated columns**.
- Desde **QRC 3/2023:** a restrição pode ser baseada também em **measures** (ex.: separar o valor da ordem em duas colunas conforme tenha mais ou menos de 10 itens).
- Restricted columns podem servir de **base para outras** restricted columns (ex.: *Quantity for Spain* → *Quantity for Spain in 2021* e *in 2022*). Mudanças na base são **herdadas**.
- Sempre dê uma **descrição significativa** — o usuário **não vê os filtros**.

### Definindo a restrição
- Pode filtrar por **várias colunas**.
- Operadores: **Equal**, **Greater Than**, **Less Than**, **Is Not Null**, etc.
- Pode **ocultar** a restricted column (ex.: usá-la só num calculated column).
- **Lógica:** linhas na **mesma coluna → OR**; conjuntos de **colunas diferentes → AND** — independentemente da ordem.

### Expression (SQL)
- Alternativa: definir/editar a restrição como **Expression** em **SQL** — mais flexível; pode usar colunas, operadores, **input parameters** e **functions** (ex.: extrair YYYY de uma data).
- Ao editar a expression, a definição gráfica (column-based) **não pode mais ser editada**.

---

## Lição 8.2 — Generating Calculated Columns

- Podem ser criadas em **qualquer tipo** de calculation view.
- São calculadas **on the fly em runtime** e **não persistem** o resultado.
- Exemplos: concatenar first + last name (string function); `IF` para tratar vazio (`IF(isnull(DIVISION),'Other',DIVISION)`); gerar **constante** (ex.: `'2022'`) quando uma fonte não tem a coluna necessária para um join.
- Podem ser **aninhadas** (calculated column baseada em outra).
- Podem ser aritméticas (measures) ou manipulação de strings (attributes).

### Por que calcular na view (e não no front-end)
- Aproveita a velocidade do SAP HANA (o banco calcula).
- **Reuso** — não precisa redefinir em cada relatório.
- Permite **esconder cálculos complexos ou sensíveis** (ex.: regra de reajuste salarial).

### Definindo
- Primeiro passo: escolher o **column type** — **measure** ou **attribute**.
- **Expression Editor:** Elements, Operators, Functions (string, matemáticas…) por duplo clique ou drag-and-drop. **Ctrl+Space** para autocompletar.

### Granularidade
- Alguns cálculos não fazem sentido sobre measures **já agregadas** (ex.: soma de unidades × soma de preços ≠ receita total; preço agregado não significa nada).
- Para cálculos que precisam ser feitos **antes da agregação**, crie a calculated column num **nó anterior** à agregação.
- Ex.: calculando no nível correto → total de vendas correto = **98.750**.
- **Cuidado (performance):** minimize cálculos antes da agregação; sempre que possível calcule **depois** da agregação.

### Client Side Aggregation
- Ex.: measure `MAXIMUM_GROSS_AMOUNT`. Marcando **Enable client side aggregation**, você propõe ao cliente de relatório aplicar também a agregação (ex.: **MAX**) do lado do cliente, conforme o dropdown **Client Aggregation**.

---

## Lição 8.3 — Filtering Data

- Filtrar por **attributes** (país, produtos, clientes) ou por **measures** (ordens acima de um valor) — ou ambos (clientes US com mais de 10 ordens no mês: filtra US cedo, mas calcula o total por cliente antes do limite 10).
- Equilíbrio entre **performance** (filtrar cedo) e **consistência** (filtrar cedo demais pode dar resultado errado, especialmente com agregação).

### Hard-coded vs. runtime
- Filtro **fixo** na view (design time) vs. **prompt** flexível (runtime).
- Objetivo: **minimizar transferência de dados** entre banco e front-end. Agregue e filtre **antes** de os dados saírem da camada de dados.

### Filtros nos nós
- Podem ser aplicados em: **Projection, Union, Join, Rank, Aggregation, Star Join**.
- Aba **Filter Expression**, expressão em **SQL**. Ex.: `"COUNTRY"='US' OR "COUNTRY"='CA'`.
- **Cuidado:** se a coluna foi renomeada no output (ex.: `COUNTRY` → `COUNTRY_CODE`), use o **nome de output** (`"COUNTRY_CODE"='US'`). Senão a expressão pode validar, mas o **build falha**.

### Onde/quando o filtro é aplicado
- **Top node:** geralmente aplicado **antes** do processamento do nó (ex.: antes da agregação no Aggregation/Star Join).
- **Outros nós:** geralmente aplicado ao **output** do nó.

### Filter push-down
- Por padrão, nós consumidos por **vários nós** (o fluxo se divide) **bloqueiam** o push-down de filtros, para garantir correção semântica — o filtro fica alto na pilha e grandes volumes sobem.
- Flag **Ignore Multiple Outputs For Filter** (no nó) → libera o push-down. **Cuidado:** verifique se a semântica não quebra.
- Setting equivalente no nível da view: **Semantics → View Properties → Advanced** (quando uma view é consumida várias vezes por outra).
- **SAP HANA Cloud 2026 QRC1:** o cenário mostra um ícone ⛔ no canto superior direito do nó onde o push-down está **bloqueado**. Some ao marcar **Ignore Multiple Outputs For Filter** ou **Allow Filter Push Down**.

### Client-Dependent Views (SAP Client / MANDT)
- **CLIENT** (SAP Client, coluna **MANDT**, "Mandant") isola tipos de dados em sistemas SAP (S/4HANA, BW/4HANA). Valores com 3 dígitos (001, 200, 800). Quase todas as tabelas SAP são client-dependent.
- A calculation view pode usar a coluna de client para **filtragem automática**.
- **Recomendação:** sempre definir **joins na coluna de client** (senão pode juntar client 200 com 800 — produto cartesiano).

Implementação:
1. Em cada tabela fonte relevante, definir a **Client Column** (aba **Mapping** do nó onde a tabela é referenciada).
2. No **Semantics node**, escolher o **Default Client**:
   - **Cross-Client:** todos os dados, qualquer client.
   - **Fixed Client:** número fixo (ex.: 200).
   - **Session Client:** filtro dinâmico pelo client atribuído ao **usuário** (tabela **USERS**). **É o padrão** numa view nova.
3. Atribuir um **Session Client** ao usuário (**SAP HANA Cockpit → User Management** ou SQL).

| Default Client | Sem Client Column | Com MANDT como Client Column |
|---|---|---|
| Cross-Client | Todas as linhas | Todas as linhas |
| Fixed Client 200 | Todas | Só MANDT=200 |
| Session Client (usuário **sem** client) | Todas | **Nenhuma linha** |
| Session Client (usuário com 800) | Todas | Só MANDT=800 |

- **Importante:** no BAS, o preview é executado pelo **technical user `..._RT`**, que **não tem default client** → com Client Column + Session Client, o **Data Preview não traz dados**. Soluções: **Data Preview with Other Database User**, ou consultar a column view no **Database Explorer** com uma conexão de banco clássica (catálogo ou SQL console).

---

## Lição 8.4 — Implementing Currency Conversion

- Organizações operam em várias moedas mas reportam numa só. Calculation views convertem **em runtime**.
- **Por que na view e não no front-end:** **performance** (100% in-memory) e **consistência** entre relatórios (regras definidas uma vez).

### Elementos
- Tabelas de master data de moedas, tipos de taxa e taxas.
- Semantic type **Amount with Currency Code** na measure.
- Settings de conversão por measure.

### Tabelas TCUR* (padrão SAP)

| Tabela | Conteúdo |
|---|---|
| **TCURC** | Currency codes |
| **TCURR** | Exchange rates (taxas) |
| **TCURV** | Exchange rate types |
| **TCURF** | Conversion factors |
| **TCURN** | Quotations |
| **TCURX** | Decimal places in currencies |
| **TCURT** | Currency code names |
| **TCURW** | Usage of exchange rate types |

- **Não vêm** no SAP HANA Cloud por padrão — você precisa disponibilizá-las e atualizá-las (ex.: **synonym** para schema externo ou ferramentas de **data provisioning**).

### Passos
1. Atribuir o semantic type **Amount with Currency Code** à measure (Semantics → Columns).
   - Pode ser usado **sem conversão**, só para informar a moeda ao front-end: defina **Display Currency** e **não** marque Conversion.
2. **Habilitar a conversão**.
3. Definir moeda origem, destino, data de conversão e tipo de taxa.

### Settings principais

| Setting | Descrição | Opções |
|---|---|---|
| **Client** | Client (MANDT) para filtrar as TCUR* | Session Client / Fixed / Column / Input parameter |
| **Source Currency** | Moeda do valor original | Fixed / Column |
| **Target Currency** | Moeda de destino | Fixed / Column / **Input Parameter** |
| **Exchange Type** | Tipo de taxa (spot, média…) | Fixed / Column / Input Parameter |
| **Conversion Date** | Data para achar a taxa | Fixed / Column / Input Parameter |
| **Exchange Rate** | (opcional) coluna com a taxa a usar | — |
| **Data Type** | Tipo do valor convertido (ex.: Decimal(15,2)) | — |
| **Generate** | Cria coluna com a moeda resultante — **nunca exposta** a ferramentas cliente, só a outras views | — |
| **Upon Failure** | Comportamento se falhar | **Fail** (erro), **NULL**, **Ignore** (mantém valor original) |
| **Accuracy** | Como converter | Intermediate rounding / Retain all possible digits |

- Mantenha as TCUR* **atualizadas** (especialmente em **side-by-side**, sincronizadas com o sistema SAP remoto).

### Decimal Shift e Rounding
- Precisão padrão de 2 dígitos nas tabelas SAP ERP. **Decimal shift** ajusta casas conforme **TCURX**. **Rounding** arredonda para os dígitos da moeda destino.
- **Decimal shift back:** necessário se o resultado for interpretado em **ABAP** (que já faz o shift) → evita **shift duplo**.

### Reutilizando settings entre measures
- **Reference:** as settings da measure origem são aplicadas e **não podem ser alteradas** nas outras (sempre consistentes).
- **Copy:** copia, mas sem vínculo — podem ser alteradas livremente.

### Input parameter para moeda
- Para escolher a moeda em runtime: input parameter **Direct**, semantic type **Currency**, data type **VARCHAR(5)** (como nas TCUR*).

---

## Pontos-chave para a prova (Unit 8)
- **Restricted column** = measure filtrada; simplifica calculated columns; mesma coluna OR, colunas diferentes AND.
- **Calculated column** = attribute **ou** measure; runtime, não persistida; cuidado com granularidade.
- Filtros podem ser em **attributes e measures** (falso dizer que só attributes).
- Top node: filtro antes do processamento; demais nós: no output.
- **Ignore Multiple Outputs For Filter** libera push-down.
- **Default Client:** Cross-Client / Fixed / **Session (padrão)**; `_RT` sem client → preview vazio.
- Currency conversion na view: **performance + consistência**; TCUR*; **Amount with Currency Code**; Upon Failure = Fail/NULL/Ignore.

---

## Quiz oficial da Unit 8 — com gabarito comentado
1. **Por que criar uma restricted column?** → ✅ **To generate a filtered column to simplify the creation of calculated columns** *(proteção de dados é papel de analytic privileges/masking)*.
2. **Filtros podem ser definidos em attributes mas não em measures — V ou F?** → ✅ **False**.
3. **Por que converter moeda na calculation view e não no front-end?** (2) → ✅ **Performance is usually better**; ✅ **Ensure consistency of results across reports**.
4. **Uma calculated column pode gerar attribute ou measure?** → ✅ **True**.

---

## Glossário
- **Restricted column:** measure filtrada.
- **Calculated column:** coluna calculada em runtime.
- **Expression Editor:** editor de expressões.
- **Filter Expression / Filter push-down:** filtro em SQL / empurrar o filtro para níveis inferiores.
- **SAP Client (MANDT) / Session Client:** mandante / client atribuído ao usuário.
- **Currency conversion / Exchange rate type / Decimal shift:** conversão de moeda / tipo de taxa / ajuste de casas decimais.
# Unit 9 — Creating Dynamic Calculation Views (Variables, Input Parameters e Parameter Mapping)

**Foco da unit:** passar valores dinâmicos em runtime com **Variables** e **Input Parameters**, e **mapear** parâmetros entre views, procedures/functions e value help views (**cascading prompts**).

---

## Lição 9.1 — Implementing Variables

### Conceito
- **Variable** = passa valores dinâmicos para **filtrar attribute columns**.
- Quando um front-end chama a view, ela pede um valor (normalmente um **prompt**) para completar uma cláusula **WHERE** adicionada à query **sobre** a view:
  ```sql
  SELECT <columns> FROM <calculation_view_name>
  WHERE (("COUNTRY" = 'JP')) GROUP BY <group_by columns>;
  ```
- Variables **sempre filtram no topo** (aplicadas na query sobre a view).
- **Variables NÃO se aplicam a measures** — só attributes.

### Criação
- No **Semantics node**, aba **Parameters**.
- A definição inclui:
  - **View/Table for Value Help** + **Reference Column (Attribute):** de onde vem a lista de valores.
  - **Selection Type:** **Single Value**, **Interval** ou **Range**.
  - **Multiple Entries:** permite várias ocorrências.
  - **Mandatory** e/ou **Default Value**.
  - **Apply Filter:** em qual(is) attribute(s) da view o filtro é aplicado.
- Por padrão, a value help é a **própria calculation view** (scan da coluna). Alternativa: uma tabela/view dedicada (ex.: master data `CUSTOMERS`).
- Se a value help é a própria view, a reference column é automaticamente o attribute do Apply Filter (pode mudar, se consistente).
- Runtime:
  - **Mandatory:** o usuário **precisa** informar valores/ranges/intervalos.
  - **Não mandatory:** se nada for informado, retorna **todos os dados** sem filtro.
- Na Semantics você vê qual variable está em qual attribute; **uma variable pode ser atribuída a vários attributes**.
- **Value Help Dialog** mostra os valores possíveis no preview. No Data Preview, **From/To** aparecem mesmo se a variable não for range.

### Value help — boas práticas
- A lista da própria view pode ser **enorme** (ex.: todos os funcionários). Prefira uma **view/tabela externa com lista restrita** — melhor para o usuário e para **performance**.

### Value help baseada em hierarchy
- Se o attribute tem hierarchies, você pode indicar uma hierarchy na variable → o usuário navega na árvore em vez de uma lista plana.
- **Parent-child:** o attribute da variable deve ser o **parent** (não o child).
- **Level:** o attribute deve ser o **nível folha (leaf, mais baixo)**.

---

## Lição 9.2 — Implementing Input Parameters

### Conceito
- Também passa valores dinâmicos em runtime, mas com **mais usos** do que a variable (que só filtra attributes).
- Usos:
  - **Calculated Column:** ex.: previsão de vendas = ano atual × aumento informado pelo usuário (ex.: 11%).
  - **Restricted Column:** vendas do ano escolhido (ex.: 2019) e do anterior.
  - **Filter:** ordens com total > $10.000 (limite informado em runtime) — **filtro em measure**; produtos com nome contendo 'GTR'.
  - **Controle de onde o filtro é aplicado:** com input parameter você escolhe o **nó**; variables sempre filtram no topo.
  - Alimentar parâmetros de **scalar/table functions** (requer mapping).
- Filtrar só por attributes: **variables** são mais simples (não precisa escrever expressão) e **mais suportadas** pelos front-ends.

### Tipos de Input Parameter

| Tipo | Uso |
|---|---|
| **Direct: Currency** | Usuário escolhe moeda origem/destino (currency conversion). |
| **Direct: Date** | Data digitada ou escolhida num **calendário**. |
| **Direct: Unit of Measure** | Unidade de medida escolhida. |
| **Static List** | Lista **predefinida** de valores. |
| **Column** | Lista de valores de uma coluna da view (ou de uma value help table/view). |
| **Derived From Table** | Valores de uma tabela (não necessariamente na view). |
| **Derived From Procedure / Scalar Function** | Valor vem do output escalar de uma **procedure/scalar function**. |
| **Direct** (sem semantic type) | Entrada livre. |

- Para **Currency** e **Unit of Measure**, a lista vem das tabelas de referência — o **default schema** da view deve contê-las.
- Input parameters suportam **múltiplos valores** (filtros *List of values* / *Not in List*, expressões que aceitam multi-value).
- **Derived from Table** e **Derived from Procedure/Scalar Function** **não geram prompt** — exceto se marcar **Input Enabled** (aí o usuário pode alterar o valor retornado).
- Na execução, são passados via cláusula **PLACEHOLDER**:
  ```sql
  SELECT <columns> FROM <calculation_view_name>
  (placeholder."$$IP_YEAR$$"=>'2019') GROUP BY <group_by columns>;
  ```
- Nem toda aplicação consegue passar PLACEHOLDER → às vezes é preciso usar variables.
- **Um input parameter definido mas não usado numa expressão é ignorado.**

### Exemplo: Gross ou Net
- Input parameter **Static List** `GROSS_OR_NET` (valores Gross/Net; default GROSS).
- Calculated column `AMOUNT` (measure):
  ```
  if('$$GROSS_OR_NET$$'='Gross',"GROSS_AMOUNT","NET_AMOUNT")
  ```
- Input parameter é chamado entre **aspas simples e cifrões duplos**: `'$$NOME$$'`.
- Static List é ruim para listas grandes (precisa editar/rebuild). **Direct** permite erro de digitação/formato. **Melhor:** tipo **Column** com **View/Table Value Help** central → lista reusável e mantida centralmente.

### Datas
- **Direct** + semantic type **Date** + data type **DATE** → calendário no prompt. Para intervalo, use **dois** input parameters (Date From / Date To). Também é possível ranges.

> **Recomendação SAP:** use **input parameters em vez de variables sempre que possível** (mais flexíveis), exceto quando o front-end não suporta input parameters.

---

## Lição 9.3 — Mapping Variables and Input Parameters

### Parameter Mapping — 4 tipos

| Tipo | Descrição |
|---|---|
| **Data Sources** | Mapear input parameters da **data source** (view inferior) para input parameters da view atual. |
| **Procedures/Scalar Functions for input parameters** | Mapear parâmetros de procedure/scalar function usados em input parameters "derived from procedure". |
| **Views for value help for variables/input parameters** | Mapear parâmetros de **views externas de value help** usadas por variables/input parameters (cascading prompts). |
| **Views for value help for attributes** | Mapear para value help views associadas a **attributes**. |

- Feature **Input Parameter/Variables Mapping** na aba **Parameters**. Escolha o tipo no dropdown; à esquerda os parâmetros das views inferiores, à direita os da view atual; arraste para mapear.
- **Só Variable↔Variable e Input Parameter↔Input Parameter** — **não há cross-mapping**.
- **Auto-map:** se os nomes forem iguais, cria e mapeia automaticamente na view atual.
- Desde **QRC 1/2024:** criar um input parameter e **propagá-lo** para views consumidoras no mesmo **HDB module** (e propagar exclusão).

### Push down para views inferiores
- Em stacked models, os parâmetros das views inferiores **são ignorados** a menos que você defina parâmetros no topo e os **mapeie** para baixo.
- Tipo **Data Sources** no Manage Mapping. Empurrar filtros para as fontes **reduz dados transferidos** → **ótimo para performance**.
- Também para passar valor a um calculated column numa view inferior.
- No tipo Data Sources só **input parameter → input parameter**. Variables da view inferior **não** são mapeadas para variables da atual, mas podem ser copiadas via **Extract Semantics** (clique direito na data source → Extract Semantics → aba Variables).
- **SQL views com parâmetros** como fonte: mapeie input parameters da view consumidora (nomes podem ser diferentes). Disponível desde **QRC 4/2021**.
- Desde **QRC 1/2022:** passar input parameters para SQL View/Calculation View acessada remotamente como **Virtual Table** — requer **SDA (Smart Data Access)** com **adapter ODBC**; **não** funciona com **SDI (Smart Data Integration)**. Remoto: SAP HANA Cloud ou SAP HANA on-premise ≥ 2.0 (SQL views com input parameters on-premise a partir de 2.0 SPS02).

### Procedures/Scalar Functions em input parameters
- Quando o valor do input parameter precisa ser **calculado** (não só digitado): procedure/scalar function recebe o valor do usuário e retorna o valor para outro input parameter (tipo **derived from procedure**).

### Value help externa — Cascading Prompts
- Por padrão, a value help vem da própria view (lista irrestrita). Redirecionar para outra tabela/view expõe uma **lista restrita** e melhora a **performance** (não compete com a view principal).
- Ex.: view de produtos com filtro fixo por product group → ineficiente para mudar. Melhor: substituir o filtro fixo por uma **variable de product group** e **mapeá-la** para a variable de product group da view principal. O usuário escolhe o grupo e a lista de produtos é filtrada → **cascading prompts** (pode ter vários níveis: família → grupo → produto).
- Opção no Manage Mapping: **Views for value help for variables/input parameters**.
- Uma view externa baseada em **hierarchy** também é uma solução de cascading.

### Value help para attributes
- Atribuir value help a attributes no Semantics para o usuário filtrar o resultado no front-end; pode usar view externa com variable mapeada para variable da view principal.

---

## Pontos-chave para a prova (Unit 9)
- **Variable:** só filtra **attributes**, sempre **no topo** (WHERE); não se aplica a measures.
- **Input parameter:** filtros em **measures**, calculated/restricted columns, escolha do nó, functions; passado via **PLACEHOLDER** `$$IP$$`.
- Tipos de input parameter (Direct/Currency/Date/UoM, Static List, Column, Derived from Table, Derived from Procedure/Scalar Function).
- Mapping: 4 tipos; sem cross-mapping; auto-map.
- **Cascading prompt:** value help view do parâmetro **filho** (ex.: cidade) com filtro por input parameter, mapeado ao parâmetro **pai** (país).

---

## Quiz oficial da Unit 9 — com gabarito comentado
1. **Usar IP_COUNTRY para filtrar a value help de IP_CITY?** → ✅ **Create a value help view for the IP_CITY parameter, listing cities with a filter on Country using an input parameter. Then map this input parameter to the IP_COUNTRY parameter.**
2. **Variables podem ser definidas em attributes e measures — V/F?** → ✅ **False** (só attributes).
3. **Por que criar input parameter em vez de variable?** (2) → ✅ **To filter data by measures**; ✅ **To pass a dynamic value to an expression in a calculated column**.

---

## Glossário
- **Variable / Input Parameter:** filtro dinâmico de attribute / valor dinâmico multiuso.
- **Value Help / Reference Column:** lista de valores / coluna que fornece os valores.
- **Selection Type:** Single Value, Interval, Range.
- **PLACEHOLDER / $$IP$$:** forma de passar input parameters na SQL.
- **Parameter Mapping / Auto-map / Extract Semantics:** ligação de parâmetros entre objetos.
- **Cascading prompts:** prompts encadeados.
- **SDA / SDI:** Smart Data Access / Smart Data Integration (virtualização/integração de dados remotos).
# Unit 10 — Implementing Hierarchies in Calculation Views (Hierarquias)

**Foco da unit:** **Level** e **Parent-Child hierarchies**, propriedades (node styles, level types, orphan nodes, aggregate all nodes, cycles, root node visibility), **time-dependent hierarchies** e o **Hierarchy Type** gerado.

---

## Lição 10.1 — Modeling Hierarchies

### Conceito
- Hierarchies facilitam a **navegação (drill-down)** em grandes volumes: o usuário clica num nó e ele expande os níveis inferiores. Measures normalmente são **acumuladas** dos níveis inferiores para os superiores.
- Ex.: Catering → Beverage (Juice, Water) / Food (Burgers, Soup).
- Calculation views suportam dois tipos: **Level** e **Parent-Child**.

| | **Level hierarchy** | **Parent-Child hierarchy** |
|---|---|---|
| Estrutura da fonte | **Cada nível numa coluna separada**; as linhas representam **folhas (leaf nodes)** | Colunas de **parent** e **child** do **mesmo data type**; cada linha representa **um nó** |
| Campos | Heterogêneos (podem ter data types diferentes) | Campos distintos definem a relação pai-filho; normalmente **IDs/keys** |
| Exemplo | Region DACH → Country Germany → Bayern, Hamburg | 4 tem pai 2, que tem pai 1; 1 também é pai de 3 |

---

## Level Hierarchies

### Procedimento
1. Selecione as tabelas fonte (ex.: PRODUCT e PRODUCT_GROUP num Join node).
2. Selecione as colunas da view, incluindo as da hierarchy (PRODUCT_ID, PRODUCT_TEXT, PRODUCT_GROUP, PRODUCT_GROUP_TEXT).
3. No **Semantics node** → aba **Hierarchies** → botão **+**.
4. Adicione as colunas **na ordem dos níveis, de cima para baixo**, com a **menor granularidade no nível mais baixo**. Pode definir **sort direction** (asc/desc) por nível.

### Node Styles (formato do node ID)

| Style | Output | Exemplo |
|---|---|---|
| **Level Name** | Nível e nome do nó | `MONTH.JAN` |
| **Name Only** | Só o nome do nó | `JAN` |
| **Name Path** | Nome do nó e seus ancestrais | `FISCAL_2015.QUARTER_1.JAN` |

### Level Types
- Especifica a semântica dos attributes do nível. Ex.: **TIMEMONTHS** (meses), **REGULAR** (sem formatação especial).

### Ordem dos membros
- Dropdown **Order By** + **Sort Direction**.

### Orphan Nodes (nó sem pai) — 4 opções
- **Root Nodes:** vira nó raiz.
- **Error:** a view gera erro.
- **Ignore:** ignorado.
- **Step Parent:** atribuído a um "padrasto" especificado.

---

## Parent-Child Hierarchies

- Primeiro passo: definir os nós — **Child column** e **Parent column**. Ex.: `EMPLOYEE_HIERARCHY` com child `DNUMBER` e parent `PARENT_DNUMBER`.
- Pode ter **vários pares parent-child** para IDs compostos (ex.: CostCenter→ParentCostCenter + ControllingArea→ParentControllingArea).

### Propriedades avançadas
- **Aggregate All Nodes:** se valores de nós intermediários são agregados até o root. Se tiver certeza de que **não há dados lançados em nós agregados**, use **False** → hierarchy executa **mais rápido**.
- **Default Member:** membro padrão; se vazio, todos são default.
- **Orphan Nodes:** como tratar órfãos. Se usar **Step Parent**: o step parent deve **já existir no nível ROOT** e o ID deve seguir o **node style** definido.
- **Root Node Visibility:** se um root node adicional deve ser adicionado (ex.: *Add Root Node if Defined*).
- **Cycles:** como quebrar ciclos (ex.: *Break up at load time*) ou gerar erro.

### Time-Dependent Hierarchies
- Para hierarquias que mudam no tempo (RH/organizações, **BOMs** em materiais).
- **Só em calculation views e só para parent-child hierarchies.**
- A fonte precisa de colunas **Valid From** e **Valid To**. Permite exibir **versões** diferentes da hierarchy.
- Validade:
  - **Intervalo** → dois input parameters: **From Parameter** e **To Parameter**.
  - **Data específica** → um input parameter **Key Date**.

---

## Tipo de hierarchy gerada (Hierarchy Type)
- Ao fazer build/deploy, a hierarchy é **materializada** em tabelas/views no schema do **HDI container** e em **BIMC\*** no schema **`_SYS_BI`** — úteis quando o front-end não gera as relações da hierarchy sozinho.
- Setting **Hierarchy Type** em Semantics → **View Properties → General**:
  - **Auto (padrão):** no SAP HANA Cloud gera **classic SQL views** (SQL hierarchy); no on-premise gera **MDX hierarchies**. No Cloud, Auto = SQL Hierarchy Views. Vantagem: migrar views on-premise→Cloud sem mudar propriedades.
  - **SQL Hierarchy Views:** gera classic SQL views que materializam a SQL hierarchy.
  - **No Hierarchy Views:** gera só os **metadados** (`_SYS_BI.BIMC*`), **não** os membros — para front-ends que geram a hierarchy sozinhos.

| | Auto | SQL Hierarchy Views | No Hierarchy View |
|---|---|---|---|
| Metadados (BIMC* em _SYS_BI) | Sim | Sim | Sim |
| SQL Hierarchy (SQL view no container) | Sim | Sim | **Não** |

- Nome gerado: view `HC300::CVC_SALES_HIER` com hierarchy `PROD_LEV_HIER` → SQL hierarchy view `HC300::CVC_SALES_HIER/PROD_LEV_HIER/sqlh/PROD_LEV_HIER`.

---

## Pontos-chave para a prova (Unit 10)
- Dois tipos: **Level** e **Parent-Child** (Referential/Temporal são **joins**, não hierarchies).
- Level: uma coluna por nível; Parent-child: colunas pai/filho do mesmo tipo.
- Orphan nodes: Root Nodes, Error, Ignore, Step Parent.
- **Time dependency:** só **parent-child** em calculation views; Valid From/To; Key Date ou From/To parameters.
- Aggregate All Nodes = False → mais rápido (se não há valores em nós intermediários).
- Hierarchy Type: Auto / SQL Hierarchy Views / No Hierarchy Views.

---

## Quiz oficial da Unit 10 — com gabarito
1. **Dois tipos de hierarchy modelada em calculation views?** → ✅ **Parent-Child**; ✅ **Level**.

---

## Glossário
- **Drill-down:** navegar do nível alto para detalhes.
- **Leaf node / Root node:** nó folha / nó raiz.
- **Orphan node / Step parent:** nó sem pai / pai substituto.
- **Node style:** formato do ID do nó.
- **Time-dependent hierarchy / Key Date:** hierarquia com validade / data de referência.
- **MDX:** linguagem de consulta multidimensional (on-premise).
# Unit 11 — Developing Custom Logic Using SQL (SQL, SQLScript e Functions)

**Foco da unit:** **SQL Console**, SQL em calculation views (**Expression Editor**, **UDFs**), consulta de hierarchies via SQL, **SQLScript** (variáveis, tipos, lógica imperativa) e **functions** (scalar e table).

---

## Lição 11.1 — Introducing SAP HANA Cloud SQL Console

### Importância do SQL na modelagem
SQL aparece em vários objetos:
- **Expressions** em calculation views
- **Procedures**
- **Functions**
- **SQL-based analytic privileges**

- Serve para implementar **custom logic** quando as features gráficas não bastam. Modeladores também precisam entender o **SQL gerado** pelas views em runtime e usar ferramentas de debug.
- **Cuidado:** SQL **não é alternativa** à modelagem gráfica. **Não** crie function/procedure para algo que uma calculation view gráfica resolve — views geram o próprio SQL conforme a query e fazem **pruning** avançado → melhor performance.

### SQL Console
- Fica no **SAP HANA Database Explorer** (ferramenta **separada** do BAS, abre em nova aba do navegador). Cada SQL Console é uma nova aba; pode abrir quantas quiser.
- Passos: **abrir** → **escrever SQL** → **executar** → **ver resultados**.
- Ideal para instruções **pontuais** (tabela temporária, query de teste). **Não recomendado** para criar objetos persistentes da aplicação. Para isso, use **source files**:
  - Tabela → **`.hdbtable`**
  - Function → **`.hdbfunction`**
  - O deploy do source file gera o runtime object. *(Vantagem: o objeto é transportado junto com a aplicação.)*
- Botão **Open SAP HANA SQL Console**: versão **embutida no BAS**, com a maioria (não todas) das funções.
- Atenção à **database connection** do console — o SQL roda contra ela. Botão **Connect this SQL console to a different database** para trocar. A conexão atual aparece no topo.
- Para abrir: **Open Database in Database Explorer** ao lado da conexão.

### HDI vs. conexão clássica
- **HDI Container:** gerado automaticamente no **primeiro deploy** do database module; é basicamente um **schema oculto**. **Não precisa especificar o schema** nas instruções.
- **SAP HANA database (catalog):** **precisa especificar o schema** (`SELECT * FROM "SCHEMA"."TABLE"`).

---

## Lição 11.2 — Implementing SQL in Calculation Views

- Expressões em calculation views são escritas em **SQL**, em:
  - **Calculated columns**
  - **Filters**
  - **Restricted columns**
- SQL também pode **gerar valores para input parameters** (ex.: top 5 clientes do mês passados como filtro).
- **Expression Editor:** componentes **Elements** (colunas, parâmetros, value help), **Functions** (por categoria: Date, String…), **Operators**; botão para **validar sintaxe**.
- Grande vantagem: muitas **funções prontas** (strings, datas — ex.: dias entre duas datas).
- **Nota:** expressions usam **SQL "puro"**, **não SQLScript**.
- Desde **QRC 1/2023:** **User-Defined Functions (UDFs)** podem ser usadas em expressions de **calculated columns** e **filters** (aba UDFs) — locais ou de outro HDI container via **synonym**.

---

## Lição 11.3 — Querying a Modeled Hierarchy Using SQL

- Hierarchies de calculation views podem ser lidas por **SQL**, acessando nós de qualquer nível como se fossem colunas. Ex.: `SELECT sum(absence_days) ... WHERE hier_node = 'Service'` → níveis inferiores **somados automaticamente (roll-up)**.
- **Setup necessário:**
  1. A hierarchy deve ser criada numa **DIMENSION calculation view**, consumida no **Star Join** de um **CUBE with Star Join** — **só shared hierarchies podem ser lidas por SQL**.
  2. Habilitar **Enable Hierarchies for SQL access** nas propriedades do Semantics do CUBE with star join.
  3. Na aba **Hierarchies** → seção **SQL Access**, localizar o nome da **Node Column** (padrão: `<hierarquia>node`, ex.: `H_GEOnode`; pode ser renomeado).
- Funciona para **level** e **parent-child**.
- O **Hierarchy Expression Parameter** não é usado na versão atual — ignore.
- A node column pode ir no **GROUP BY** → measure agregada por nó.

---

## Lição 11.4 — Working with SQLScript

### O que é
- Linguagem de banco desenvolvida pela SAP, baseada em SQL padrão (**ANSI-92**), com extensões para recursos do SAP HANA.
- Acesso a features específicas: column tables, parameterized information views, delta buffers, múltiplos result sets em paralelo, currency conversion no banco, fuzzy text search, **spatial**, bibliotecas preditivas.
- Leva lógica intensiva em dados **para dentro do banco** (em vez do application server em ABAP/C++).
- Incentiva o paradigma **set-oriented** (conjuntos) em vez de registro a registro — mas permite **loops** e **IF-THEN-ELSE**.
- Permite **quebrar queries complexas** em instruções menores (**table variables**) → código mais simples e mais **paralelismo** pelo optimizer.

Exemplo — editora com mais livros:
```sql
books_per_publisher = SELECT publisher, COUNT(*) AS num_books FROM BOOKS GROUP BY publisher;
publisher_with_most_books = SELECT * FROM :books_per_publisher
  WHERE num_books >= (SELECT MAX(num_books) FROM :books_per_publisher);
```
- A sequência escrita **não é necessariamente** a ordem de execução — o optimizer escolhe o melhor plano, **sem alterar o resultado** (igual às calculation views).

### Variáveis (extensão ao ANSI-92)
- Declarar: `DECLARE <variable_name> <type> [NOT NULL] [= <value>]` — ex.: `DECLARE a int;` / `DECLARE b int = 0;`
- Atribuir: `a = 3;` / `b = select count(*) from baseTable;`
- Usar: **com dois-pontos** `:a` → `b = :a + 5;`

### Tipos de dados

| Classe | ANSI-92 | Específicos SQLScript |
|---|---|---|
| Date/Time | DATE, TIME, TIMESTAMP | **SECONDDATE** |
| Numeric | DECIMAL, INTEGER, SMALLINT, FLOAT, REAL, DOUBLE | **TINYINT, BIGINT, SMALLDECIMAL** |
| Character | CHAR, NCHAR, VARCHAR, NVARCHAR | **ALPHANUM, SHORTTEXT** |
| Binary | BINARY, VARBINARY | — |
| Large Object | — | **BLOB, CLOB, NCLOB, TEXT, BINTEXT** |
| Boolean | — | **BOOLEAN** |
| Spatial | — | **ST_POINT, ST_GEOMETRY** |

### Input parameters de calculation views no SQLScript
```sql
SELECT "PRODUCT", sum("TAXED_AMOUNT")
FROM "_SYS_BIC"."MyPackage/SalesResults"
  ('PLACEHOLDER' = ('$$Tax Amount$$', '17.5'))
GROUP BY "PRODUCT"
```
- Dentro do SQLScript, referencie o parâmetro com **dois-pontos**: `Sales Revenue * :Tax_Amount`.
- Output parameters de SQLScript também podem ser passados à view via **parameter mapping**.

### Lógica imperativa (flow control)
- SQL é **declarativo** (diz o quê, não como) → optimizer decide o plano.
- Às vezes é preciso **imperativo** (ordem, condições, loops) — SQLScript fornece isso, evitando levar processamento para a camada de aplicação.
- **Dica:** lógica imperativa **não tem a mesma performance** que a declarativa (o optimizer perde liberdade). Para melhor performance, **evite**.

```sql
IF <bool-expr1> THEN {stmts1}
{ELSEIF <bool-expr2> THEN {stmts2}}
{ELSE {stmts3}}
END IF

WHILE <bool-stmt> DO {stmts} END WHILE

FOR <loop-var> IN {REVERSE} <start>..<end> DO {stmts} END FOR
```
- `ELSEIF` pode se repetir (simula switch-case). Depois do `ELSE`, não pode haver outro ELSE/ELSEIF.
- `FOR`: se start > end, o corpo não executa; `REVERSE` = ordem decrescente.

---

## Lição 11.5 — Creating and Using Functions

- **Functions** = bloco reutilizável de lógica em **SQLScript**, retornando **um valor** ou **um data set**. Usadas em calculation views quando o gráfico não basta.
- Podem consumir **input parameters** das views → reutilizáveis.
- Tipos: **scalar** e **table**. Ambas em source files **`.hdbfunction`** (mesma extensão).
- **Sempre read-only:** não permitem **DDL** (create/alter table) nem **DML** (update/insert). **Nunca alteram o banco.**
- Functions podem chamar outras functions (design modular).

### Scalar Function
- Uso comum em views: **derivar valores para input parameters**.
- Retorna um ou mais parâmetros, **cada um com um único valor**. Inputs são opcionais (ex.: data atual não precisa; "data de x dias atrás" precisa de x).
- Palavra-chave **`RETURNS`** (ex.: `RESULT_ADD`, `RESULT_MUL` do tipo Double).

### Table Function
- Retorna um **data set tabular** (colunas e linhas). Usada como **data source customizada** em qualquer nó (Projection, Join…) — como uma "tabela dinâmica criada em runtime".
- Palavra-chave **`RETURNS TABLE`**.
- Ex.: ler sales orders e achar ordens abertas recentes, com data vinda de input parameter.
- Os outputs viram colunas de entrada do nó. Inputs da function: defina input parameters na view e **mapeie**.
- Há um nó dedicado: **Table Function node**, com 3 tipos de input mapping:
  - **Data Source Column:** coluna de uma data source → parâmetro da function.
  - **Input Parameter:** input parameter da view → parâmetro da function (valor digitado em runtime).
  - **Constant:** valor fixo.

### Procedures (nota)
- Parecidas com functions, mas **suportam DDL e DML** (alteram o banco).
- **Não podem ser data source** de calculation view.
- Por que o modelador deve conhecer: para definir **analytic privileges dinâmicos** com lógica SQLScript numa procedure.

---

## Pontos-chave para a prova (Unit 11)
- SQL estende a view (expressions, functions) — **não substitui** a modelagem gráfica.
- Expressions (calculated, filters, restricted, valores de input parameters) usam **SQL**, não SQLScript.
- Objetos persistentes → **source files** (`.hdbtable`, `.hdbfunction`) — transportáveis com a aplicação, não SQL Console.
- HDI container: sem schema na query; catalog: com schema.
- Hierarchy via SQL: criar na **DIMENSION**, consumir em **CUBE with Star Join**, habilitar **Enable Hierarchies for SQL access**.
- SQLScript: **table variables** + **IF/WHILE/FOR**; variável usada com `:`.
- **Table function** = data source; **scalar function** = valor (ex.: input parameter). Functions read-only; procedures não podem ser data source.

---

## Quiz oficial da Unit 11 — com gabarito comentado
1. **Como o SQLScript estende o SQL padrão?** (2) → ✅ **Table variables** para quebrar instruções grandes; ✅ **Instruções condicionais IF..THEN..ELSE**. ❌ JavaScript.
2. **Que tipo de function pode ser data source de um nó?** → ✅ **Table Functions**.
3. **Por que conhecer SQL é importante?** (2) → ✅ Desenvolver **custom logic numa SQL function** e consumir na view; ✅ **Estender** a view com **SQL expressions**. ❌ substituir o editor gráfico.
4. **Em que tipo de view criar a hierarchy para ser lida por SQL?** → ✅ **Dimension** (depois consumida no star join do cube).
5. **Por que definir tabela por source file e não CREATE TABLE no console?** → ✅ **A definição é transportada junto com a aplicação**, mantendo os artifacts juntos.
6. **Por que implementar SQL numa view?** (3) → ✅ Gerar valores para input parameters; ✅ Definir filtros de restricted columns; ✅ Definir calculated column. ❌ Level hierarchy (é gráfica).

---

## Glossário
- **SQL Console / Database Explorer:** editor SQL / ferramenta de banco.
- **Catalog connection:** conexão clássica ao banco (precisa schema).
- **Expression Editor / UDF:** editor de expressões / user-defined function.
- **SQLScript:** extensão SAP do SQL.
- **Declarative vs Imperative:** dizer o quê vs dizer como/ordem.
- **Table variable:** variável que guarda um result set.
- **Scalar function / Table function / Procedure.**
- **DDL / DML:** definição de dados / manipulação de dados.
# Unit 12 — Applying Best Practices for Modeling (Boas Práticas de Modelagem)

**Foco da unit:** **instantiation process**, design em camadas, **query unfolding**, value help dedicada e boas práticas nos nós (column pruning, filtros, cálculos, SQL em vez de column engine, agregações).

---

## Lição 12.1 — Implement Recommended Modeling Practices

### Instantiation Process
- O **calculation engine** pré-otimiza as queries antes do **SQL optimizer**.
- Quando uma query é executada, a calculation view é **instanciada** em runtime: o engine simplifica a view num modelo reduzido que atende só ao que a query pede (considera **dynamic join**, **join cardinality**, **union pruning**…).
- Depois o **SQL optimizer** aplica mais otimizações e define o **execution plan** (sequência ideal, passos paralelos).
- O modelo gerado é **podado** de elementos desnecessários (não contém attributes/measures fora da query) e é tecnicamente uma **column view** que referencia um nó específico da view original.
- Resumo: **query + modelo original → modelo de execução otimizado**.

### Recomendações gerais de design
- **Quebre modelos grandes** em calculation views menores, reutilizáveis (evita redundância e manutenção duplicada). Evite **views monolíticas**; identifique padrões recorrentes.
- Desenvolva **em camadas (layers)**: camadas inferiores = views de fundação; superiores adicionam semântica de negócio. Mudanças nas inferiores refletem imediatamente nas superiores.
- **Não se preocupe com performance ao empilhar:** em runtime o SQL optimizer tenta **"achatar" (flatten)** toda a pilha numa única instrução. Use o **SQL Analyzer** para ver o SQL gerado.

### Query Unfolding
- Objetivo: converter cada operação da view num **operador SQL** e todo o fluxo numa **única instrução SQL** = **unfolding**.
- Vantagem: só o **SQL engine** é usado (sem Calculation Engine, Spatial, Graph…).
- Se algum passo não tem operador SQL equivalente, é processado nos engines especializados, que geram **materialized column views** intermediárias lidas pelo SQL → query **parcialmente unfolded**, com menos otimização.
- **Mas:** unfolding pode aumentar o **tempo de compilação** (a query é "desdobrada" a cada nova seleção). Há um **SQL hint** para impedir unfolding por view (Semantics → Properties → **Advanced**) e um **parâmetro global** para todas.
- Verificar: **Analyze > Explain Plan** no SQL Console (operadores e fontes).
  - Fonte = **column view** → **não** unfolded (resultado materializado).
  - Totalmente unfolded → acessa **apenas tabelas** (operador pode aparecer como **TABLE SCAN**).
- Compare **compilation time** vs **execution time**.
- Contar queries não unfolded:
  ```sql
  SELECT FEATURE_NAME, CALL_COUNT FROM "M_FEATURE_USAGE"
  WHERE COMPONENT_NAME = 'CALCENGINE'
    AND FEATURE_NAME IN ('QUERY TOTAL','QUERY UNFOLDING')
  ```

### Views ou tabelas dedicadas para "List of Values"
- Para performance, crie **calculation views/tabelas dedicadas** para a value help de variables e input parameters.
- Vantagens: **value help mais rápida** (não lê a view inteira), **listas consistentes** entre views, suporte a **analytic privileges** (cada usuário vê só os valores permitidos).

---

## Lição 12.2 — Implement Best Practices in Calculation View Nodes

### Column Pruning
- **Não mapeie colunas desnecessárias** ("just in case"). Muitos attributes geram data sets grandes e granulares quando só se precisa de dados agregados. Mantenha conjuntos intermediários **pequenos**.

### Filtros e Calculated Columns
- **Filtre o mais cedo possível** (o mais baixo na pilha).
- **Evite calculated columns em filter expressions**: crie a coluna calculada e filtre num **Projection node acima** (onde a coluna já não é calculada).
- **Calcule o mais alto possível** na pilha → calcular sobre dados **agregados**, não granulares.

### Expressions em SQL (não Column Engine)
- No SAP HANA **on-premise**, expressões podiam ser em **SQL** ou **Column Engine language**. SAP recomenda **só SQL** (column engine **limita o unfolding**).
- No **SAP HANA Cloud**, **só SQL** está disponível.
- Views importadas do on-premise com column engine **ainda rodam**, mas troque para SQL. Ao escolher SQL, o seletor fica **cinza** (não volta).
- Conversões: concatenação `+` → `||`; `if...then` → **`CASE`**.

### Agregações
- **Reduza o data set** com agregações **cedo** na pilha.
- Ao consumir uma view, **não transforme attributes em measures** (e vice-versa).
- Para modelos **OLAP dimensionais**, use **CUBE with Star Join** — não tente fazer OLAP com join nodes comuns (modelo relacional, pior performance).

---

## Pontos-chave para a prova (Unit 12)
- **Empilhar views é boa prática** (falso que se deve concentrar tudo numa view).
- Instantiation: calc engine poda → SQL optimizer planeja.
- **Unfolding** = tudo em SQL; checar com Explain Plan (column view na fonte = não unfolded).
- Calculated columns: **SQL expressions** e **o mais alto possível**; filtros **o mais baixo possível**.
- Value help em views dedicadas.

---

## Quiz oficial da Unit 12 — com gabarito comentado
1. **Melhor prática é evitar empilhar views e pôr toda lógica numa só — V/F?** → ✅ **False**.
2. **Boas práticas para calculated columns?** (2) → ✅ **Use SQL expressions**; ✅ **Move calculations to the highest node possible**.

---

## Glossário
- **Instantiation:** criação do modelo de execução a partir da query.
- **Calculation engine / SQL optimizer:** engines de pré-otimização / otimização final.
- **Query unfolding:** tradução completa em SQL.
- **Explain Plan:** plano de execução.
- **Column pruning:** eliminar colunas não usadas.
- **Column Engine language:** linguagem de expressões legada (on-premise).
# Unit 13 — Using Tools to Check Model Performance (Ferramentas de Performance)

**Foco da unit:** **Performance Analysis mode**, **Debug Query mode** e **SQL Analyzer** (HANA SQL Analyzer / plan file `.plv`).

---

## Lição 13.1 — Validating Calculation Views with Performance Analysis Mode

- O editor de calculation views pode **destacar configurações** que podem causar má performance.
- Ativado pelo botão de **velocímetro (speedometer)** na toolbar do editor.
- **Não precisa** fazer build/deploy da view para usar → reage em **design time**.
- Ao ativar, aparece a aba **Performance Analysis** em todos os nós **exceto o Semantics**. Some ao sair do modo.

### Informações mostradas
- Tipo de tabela (**row** ou **column**).
- **Join types** (inner, outer…).
- **Join cardinality** (n:m etc.) e referential integrity / cardinality proposta.
- Se as tabelas são **particionadas**, como, e quantos registros por partição.

### Warnings
- **Tamanho das tabelas** (destaque para tabelas muito grandes).
- **Cardinalities ausentes** ou que **não batem** com os dados atuais.
- **Joins baseados em calculated columns**.
- **Restricted columns baseadas em calculated columns**.
- Ex.: tabela enorme → pensar em **particionar** e filtrar para processar só as partições necessárias.

---

## Lição 13.2 — Debugging Calculation Views with the Debug Query Mode

- Para examinar o **comportamento em runtime** quando a view é chamada por uma query: você inspeciona **cada nó** e vê o **SQL gerado**.
- Ao entrar no modo, é criada uma **query genérica** (todas as colunas, sem filtros) que simula um front-end. Você pode **editá-la** (remover colunas, adicionar filtros).
- Após executar, vê o SQL de **cada nó** e pode **executar o SQL por nó** para ver resultados intermediários. Ex.: ver um **WHERE** sendo adicionado ou como um **PLACEHOLDER** se comporta.
- Pode navegar até o **nível mais baixo** da pilha a partir do topo (não precisa abrir cada view separadamente).
- Muito útil para verificar **pruning** de colunas e de data sources (**union/join pruning**) com diferentes queries.
- **Reset** da debug query no Semantics node (botão ao lado de Execute).
- **Nota:** o Debug Query mode chama a **versão ativa (deployada)** da view. Mudanças não rebuildadas **não aparecem** → é preciso **deploy** antes.

---

## Lição 13.3 — Analyzing Executions with the SQL Analyzer

### Propósito
O Debug mostra o SQL, mas não a **performance real**. O SQL Analyzer responde:
- Tempo total de execução? Tempo de cada passo? Gargalos (bottlenecks)?
- Quantos registros por passo? Em que sequência? Quais operadores?
- Quanto foi **paralelizado**? Quais **engines**? Tudo foi **unfolded**?

- View do BAS: **HANA SQL Analyzer**. Requer a extension **SAP HANA Performance Tools** no dev space (adicionar com o dev space **parado**).
- Conjunto de views, tabelas e gráficos; aba **Overview** (tempo, memória, operadores, data usage, network…); **Plan Graph**; timeline de compilação e execução; tabelas usadas.
- Serve para qualquer SQL (views, **table functions** etc.).

### Como obter a query
- **Mais fácil:** na view **SAP HANA Database Explorer** do BAS → HDI container → clique direito na **column view** → **Generate SQL Statement**. O plan file abre direto no HANA SQL Analyzer.
- Na **Explorer view**: clique direito no arquivo da view → **SQL Editor**; ou **Data Preview** → aba Raw Data → **Edit SQL Statement in SQL Console**.
- No **Database Explorer externo**: HDI container → column view → **Generate SQL Statement**.
- Os dois últimos exigem **baixar** o plan file (`.plv`) e **subir** no BAS.
- Pode modificar a query ou criar do zero.

### Gerar o plan file
1. Coloque a query num **SQL Console** (**não precisa executar**).
2. **Analyze → Generate SQL Analyzer Plan File**; escolha prefixo do nome e **Save** (local fixo).
3. Se gerado no Database Explorer **embutido** → abre direto no HANA SQL Analyzer.
4. Se gerado em outra ferramenta → **Download** do `.plv` → **Upload** para uma pasta do projeto no BAS → HANA SQL Analyzer → **+** para adicionar → selecionar → ver **PLAN GRAPH**.

### Acessar plan files depois
- **Embedded DB Explorer:** pasta `/home/user/.vscode/data/User/globalStorage/sapse.hana-database-explorer/`.
- **External DB Explorer:** HDI container → **Catalog → Database Diagnostic Files → <DB Instance ID> → other** → baixar e subir no BAS.

### Privilégios para o SQL Analyzer
- Pode ser o `_RT` do container ou um usuário clássico; ambos precisam dos **SYSTEM privileges**:
  - **TRACE ADMIN**
  - **INIFILE ADMIN**

---

## Pontos-chave para a prova (Unit 13)
- **Performance Analysis mode:** design time, sem deploy; mostra join/cardinality, partições, tamanho de tabelas, joins em calculated columns → ajuda em **otimização de joins** e **necessidade de particionar**. *(Não mede performance de SQL nem precisão do resultado.)*
- **Debug Query mode:** SQL por nó, pruning; usa a **versão deployada** (precisa deploy).
- **SQL Analyzer:** extension **SAP HANA Performance Tools**; escrever query + **Generate SQL Analyzer Plan File** (não precisa executar nem baixar no embedded); privilégios TRACE ADMIN + INIFILE ADMIN.

---

## Quiz oficial da Unit 13 — com gabarito comentado
1. **Para usar Debug Query Mode é preciso fazer deploy antes — V/F?** → ✅ **True** (usa a versão ativa).
2. **O que se verifica no Performance Analysis Mode?** (2) → ✅ **Join definition optimization**; ✅ **Table partitioning need**.
3. **Passos necessários para visualizar um plan file a partir do Database Explorer embutido?** (2) → ✅ **Write an SQL query**; ✅ **Generate an SQL Analyzer plan file**. ❌ executar; ❌ baixar.

---

## Glossário
- **Performance Analysis mode:** modo de análise de performance em design time.
- **Debug Query mode:** modo de depuração da query por nó.
- **SQL Analyzer / Plan file (.plv) / Plan Graph:** analisador de execução / arquivo de plano / gráfico do plano.
- **Bottleneck:** gargalo.
- **TRACE ADMIN / INIFILE ADMIN:** system privileges necessários.
# Unit 14 — Implementing Features to Improve Performance (Paralelização e Particionamento)

**Foco da unit:** **Partition Local Execution** (bloco de paralelização) e **table partitioning**.

---

## Lição 14.1 — Controlling Parallelization

### Contexto
- O SAP HANA Cloud **sempre tenta paralelizar** automaticamente. Mas, em passos complexos e longos, o optimizer pode manter o processamento **sequencial** por não entender a semântica de negócio (medo de quebrar o resultado).
- Se você garante que o passo pode rodar em paralelo dividindo os dados em **partições semânticas**, pode **forçar** a paralelização.

### Partition Local Execution
- Flag que marca o **início** e o **fim** do trecho do fluxo onde a paralelização é exigida → gera **várias threads** em pontos de gargalo.
- **Início do bloco:** um nó que tenha uma **tabela** como data source — **Projection** ou **Aggregation** node. Não pode ser function nem view.
  - Nas **Properties** desse nó: marque **Partition Local Execution**.
  - Opcional: uma **coluna de particionamento** → uma partição (thread) por **valor distinto** (ex.: COUNTRY → uma thread por país). Escolha colunas com distribuição **uniforme**.
  - Sem coluna definida → usa as regras de **partição da tabela** (se particionada).
- **Fim do bloco:** um **Union node** com **apenas uma** fonte visível (as várias threads não aparecem no editor). Não pode combinar outras fontes nele. Marque **Partition Local Execution** nas Properties.
- Ex.: Projection particiona por Country → Projection `Currency_convert_In_Parallel` roda em 3 partições (India, France, Iceland) → Union junta.

### Restrições
- O bloco **sempre começa** com nó cuja data source é **tabela** (local ou **synonym** para tabela externa). **Não** pode ser calculation view nem table function.
- **Só um bloco de paralelização por query** — não dá para parar e iniciar outro (na mesma view ou na pilha) nem **aninhar** blocos.
- Pode haver **vários nós de início** num mesmo bloco (com colunas de partição diferentes); todas as threads terminam num **único union** final.
- O HANA ainda aplica paralelização adicional dentro das partições lógicas.

### Monitorar
- Calculated column com a expressão column engine **`partitionid()`** dentro do bloco → mostra o número da partição de cada thread.
- Trace: adicione `WITH PARAMETERS ('PLACEHOLDER' = ('$$CE_SUPPORT$$',''))` ao final da query.
- **SQL Analyzer:** os nós paralelizados aparecem **duplicados** conforme o número de partições.
- **Cuidado:** não abuse — **over-parallelizing** prejudica a performance geral e outros processos.

---

## Lição 14.2 — Partitioning Tables

- Em tabelas **column store**, os dados já são separados por colunas (divisão **vertical**). **Table partitioning** divide as **linhas** em blocos (divisão **horizontal**). Ex.: partições por Year (2020, 2021, 2022).
- **Só se aplica a column store tables** (não row store) — por isso recomenda-se column store para grandes volumes consultados.
- Criar partições é normalmente responsabilidade do **administrador** do SAP HANA Cloud: tipos **hash**, **round robin**, **range**; também **sub-partitions**; monitoramento e ajustes.
- O **modelador** fornece informação sobre **como as queries acessam os dados** para que as partições sejam bem definidas.

### Motivos para particionar (complemento)
*(A seção do curso é apresentada em vídeo; resumo dos motivos clássicos documentados pela SAP.)*
- **Limite de ~2 bilhões de linhas** por tabela/partição não particionada em column store → particionar supera esse limite (**é a limitação cobrada no quiz**).
- **Partition pruning:** queries com filtro na coluna de partição leem só as partições relevantes.
- **Paralelismo:** partições processadas em paralelo.
- **Delta merge** mais leve (por partição) e melhor gestão de memória.
- **Distribuição** de carga entre hosts (cenários scale-out).

---

## Pontos-chave para a prova (Unit 14)
- **Partition Local Execution:** início em Projection/Aggregation com **tabela** como fonte; fim em **Union** com uma única fonte.
- **Apenas 1 bloco de paralelização** por query; sem aninhamento.
- Coluna de particionamento opcional (senão usa a partição da tabela).
- `partitionid()`, `$$CE_SUPPORT$$`, SQL Analyzer para monitorar.
- Partitioning = **horizontal**, só **column store**; supera o limite de **número de linhas** numa column-store table.

---

## Quiz oficial da Unit 14 — com gabarito
1. **Quantos blocos de paralelização numa calculation view?** → ✅ **1**.
2. **Que limitação o table partitioning supera?** → ✅ **The number of rows in a column-store table**.

---

## Glossário
- **Parallelization / Thread:** execução paralela / linha de processamento.
- **Partition Local Execution:** flag de início/fim do bloco paralelo.
- **Table partitioning (hash, round robin, range):** divisão horizontal de tabela.
- **Over-parallelizing:** excesso de paralelização.
# Unit 15 — Storing Calculation View Results (Static Cache, Snapshots e MDS Cubes)

**Foco da unit:** três formas de **armazenar resultados**: **Static Cache**, **Snapshots** e **MDS Cubes**.

---

## Lição 15.1 — Implementing Static Cache to Improve Performance

### Conceito
- Guardar o resultado de uma calculation view em **cache** para acelerar queries seguintes. Útil com cenários complexos/grandes volumes. Reduz tempo de resposta **e** consumo de **CPU e memória**.
- Ex.: Query A às 9:10 busca no banco e grava no cache; às 10:30 lê do cache, muito mais rápido.
- Se o cache **não puder** ser usado (query pede outro data set), a query **não falha** — apenas faz o processamento completo.
- Aplique cache a dados **muito processados e reduzidos** (agregações/filtros) → **nós/views do topo**, não a dados brutos.
- **Cache só funciona em views que NÃO checam analytic privileges.** Portanto, defina analytic privileges **só na view do topo** da pilha e o cache na view **mais alta possível abaixo** dela → os privilégios são checados sobre os dados em cache.

### Ajuste fino (aba **Static Cache** em View Properties)
- **Colunas:** cache por coluna; queries que usam um **subconjunto** das colunas cacheadas são atendidas. Recomenda-se cachear só colunas frequentes. **Se nenhuma coluna for especificada, todas são cacheadas.**
- **Filtros:** o cache é usado se o filtro da query for **exatamente** o do cache ou **reduzir mais** os dados.
- **Retention Period:** idade máxima (em **minutos**) dos dados em cache antes de forçar refresh. Ainda **não** existe invalidação automática quando novos dados chegam às tabelas — só expiração por tempo.

### Pré-requisitos
- **Enable Cache** marcado.
- A view pode ser **unfolded** (totalmente traduzível em SQL; verificar com **Explain Plan**).
- **Sem cálculos dependentes de granularidade** (o cache fica no nível da primeira query, o que pode gerar resultados errados depois).

### Invocação
O cache é **desligado por padrão** e só é considerado quando invocado por um hint **RESULT_CACHE**:
- Na query: `SELECT ... WITH HINT (RESULT_CACHE)`
- **Database hint** na view do topo que consome a view cacheada (Execution Hints).
- Parâmetro de configuração: `indexserver.ini -> [result_cache] -> enabled`.
- Se os pré-requisitos não forem atendidos, dá para usar **Force** — mas teste muito.

---

## Lição 15.2 — Creating Snapshots

- **Snapshot** = "fotografia" dos dados em pontos no tempo → guardar **histórico**, ou ter performance sem precisar de tempo real (ex.: snapshot da manhã).

### Processo
1. **Deploy** da view gera a **snapshot table**.
2. São geradas **procedures** para **truncar** a tabela ou **inserir** dados (e criar/dropar a tabela).
3. Execute o snapshot **manualmente** ou **agendado**.
4. Consulte a snapshot table para dados **históricos**.
5. A view continua disponível para dados em **tempo real**.

### Configuração
- **View Properties → aba Snapshots**: crie uma **query sobre a view**; o resultado dela é gravado na snapshot table.
- **Security mode** das procedures: **DEFINER** (quem pode executar a procedure consegue usar) ou **INVOKER** (precisa dar acesso à snapshot table).
- **Cuidado:** chamar o INSERT duas vezes sem truncar gera **duplicatas**.
- **Create Snapshot After Deployment Automatically:** provisiona no deploy (por padrão trunca e insere de novo a cada deploy). **Keep Snapshot During Re-deployment:** não apaga entre deploys.

### Uso
- Selecione da snapshot table para histórico; use a view para tempo real.
- **Interface view** (gerada): faz **union** da snapshot table com a view e usa um **input parameter** para escolher a fonte.

---

## Lição 15.3 — Defining MDS Cubes

### Motivação
- Ex.: modelo no **SAP Analytics Cloud (SAC)** em que usuários fazem drill-downs diferentes (parceiro/canal/mês vs. produto/canal/data). Uma view única e complexa consome muitos recursos; o static cache demora no primeiro acesso e pode não atender drill-downs variados.
- **MDS cube (multidimensional services cube):** armazena dados **pré-processados/pré-agregados** num formato que atende vários drill-downs rapidamente.

### Conceito
- Definido para uma calculation view específica; é um **submodelo** (subconjunto de measures + attributes). No SAC, inclua todos os campos necessários (o modelo SAC vê **só** os campos do MDS cube).
- **Só para views de Data Category CUBE.**
- Pode haver **vários** MDS cubes por view. O **deploy** gera os cubes **sem dados**; depois de carregados, dados pré-agregados ficam numa estrutura **multidimensional, tipo star-join**, otimizada para OLAP (como um "cache persistido predefinido", mas não plano).
- **MDS hints** opcionais — use só se orientado por suporte.
- **MDS Cube ≠ MDS Cube Cache.**
- Dados refletem o momento da carga e **só o que o usuário que carregou podia ver**. **Não atualiza automaticamente** → precisa **recarregar**. **Não serve para tempo real.**

### Comparação
- **Static cache** pode ser mais rápido, mas a query precisa casar **exatamente**. MDS cube atende **mais queries**, faz pós-cálculos (restrições, ranking, fórmulas, exception aggregation do SAC) e agrega rápido. MDS cube é **mais rápido que snapshot**.
- Estrutura acessível por ferramentas baseadas em **metadados MDS** (SAC via **InA**), **não por SQL** → SQL Analyzer/Explain Plan não mostram uso do MDS cube; MDS cubes não aparecem no Database Explorer.
- **Generate Document** no BAS inclui informações dos MDS cubes.

### Gestão
- Procedure built-in **`MANAGE_MDS_CUBE('<JSON>', ?)`** (entrada e saída JSON) para metadados, **load/reload**, **delete**.
  ```json
  { "Cube": { "Command": "Reload",
      "Target": { "DataSource": { "SchemaName": "<Schema>", "ObjectName": "<MDS Cube Name>" } } } }
  ```
- Sucesso → estado **Ready**. Durante reload, o cube continua consultável com dados antigos. Monitorar em **`_SYS_EPM.MDS_METADATA`**.
- **Redeploy** da view apaga os MDS cubes e dados por padrão. Desde **2025 QRC4**: opção **Keep MDS Cube During Deployment** (só se as mudanças mantêm o cube carregável; senão o deploy falha).
- **2026 QRC1:** **incremental loading** — defina um attribute como **Incremental Column** (valor sempre crescente, ex.: `ORDER_TIMESTAMP`); carrega só registros com valor maior que o último; comando **Update** no `MANAGE_MDS_CUBE`.

### Limitações
- **Sem calculated measures** (faça no SAC).
- Suporta **variables**, mas **não input parameters** (input parameter mandatory sem default → deploy falha; converta para variables).
- **Joins com 2+ colunas de join** não suportados.
- Tipos suportados: inteiros, floats, DECIMAL, BOOLEAN, VARCHAR/NVARCHAR/CHAR/NCHAR, DATE/DAYDATE/TIMESTAMP — **não VARBINARY**.
- Features não suportadas desativam a aba **MDS Cubes** (com lista do que impede); se já houver MDS cube, essas features ficam inativas.

### Autorizações
- **`GRANT EXECUTE ON MANAGE_MDS_CUBE TO <DB_USER>;`**
- **CREATE ANY** no schema HDI da view; **SELECT** na fonte do cube.
- **Analytic privileges** da view se aplicam automaticamente aos MDS cubes; os privilégios dos usuários devem conter só colunas presentes no cube.

---

## Comparativo rápido

| | **Static Cache** | **Snapshot** | **MDS Cube** |
|---|---|---|---|
| Objetivo | Performance em queries repetidas | Histórico / foto dos dados | Performance multidimensional (SAC) |
| Atualização | Expira por **retention period** (min) | Procedures INSERT/TRUNCATE (manual/agendado) | **Reload** / incremental via MANAGE_MDS_CUBE |
| Acesso | Transparente via hint RESULT_CACHE | SELECT na snapshot table / interface view | SAC via **InA** (não SQL) |
| Tempo real | Não (enquanto válido) | Não | Não |

---

## Pontos-chave para a prova (Unit 15)
- Static cache → **melhorar performance**; requer Enable Cache + unfolding + sem granularity tracking; invocado por **RESULT_CACHE**; não em views com analytic privileges.
- Snapshot → **guardar resultado de uma query** sobre a view; **performance em dados históricos**; procedures geradas; DEFINER/INVOKER.
- MDS Cube → só **CUBE**; **SAP Analytics Cloud via InA**; persistência otimizada multidimensional (**True**); sem input parameters/calculated measures.

---

## Quiz oficial da Unit 15 — com gabarito comentado
1. **Por que usar snapshot?** (2) → ✅ **To store the results of a query on the calculation view**; ✅ **To improve performance when querying historical data**.
2. **Front-end que se beneficia de MDS cube?** → ✅ **SAP Analytics Cloud (via InA interface)**.
3. **Benefício do static cache?** → ✅ **Improve calculation view performance**.
4. **MDS Cube é otimização de performance com persistência otimizada para queries multidimensionais — V/F?** → ✅ **True**.

---

## Glossário
- **Static cache / RESULT_CACHE / Retention period.**
- **Snapshot table / Interface view / DEFINER-INVOKER.**
- **MDS cube / InA (Information Access) / MANAGE_MDS_CUBE / Incremental Column.**
- **SAC:** SAP Analytics Cloud.
# Unit 16 — Using Additional Modeling Productivity Tools (Ferramentas de Produtividade)

**Foco da unit:** features que aceleram o design de calculation views e ferramentas de **auditoria** (**Data Lineage**, **Impact Analysis**, **Column Lineage**, **Outline**, **Where-Used**).

---

## Lição 16.1 — Productivity Aids for Efficient Calculation View Development

| Feature | Para que serve |
|---|---|
| **Switching Node Types** | Trocar um tipo de nó por outro sem perder a ligação com a fonte e com nós superiores/inferiores. |
| **Replacing a Data Source** | Trocar a fonte de um nó sem perder output columns, calculated columns etc. nos nós superiores. |
| **Extract Semantics** | Aplicar as semantics de um nó/fonte inferior ao Semantics node da view. |
| **Propagate to Semantics** | Mapear colunas para o output de **todos os nós superiores** até o topo. |
| **Previewing the Output of Intermediate Nodes** | Ver os dados de um nó intermediário para troubleshooting. |
| **Map Input Parameters between nodes** | Mapear input parameters da view para os das views fonte. |

### Switching node types
- **Projection ↔ Aggregation** sem perder as fontes. No **top node**, também é possível trocar para **Star Join**.

### Replacing data sources
- Substituir tabela/view/nó por outra fonte mantendo colunas propagadas e calculated columns funcionando. Aparece um diálogo de **column mapping** (colunas novas → output existente).

### Extract Semantics
- Extrai semantics ricas das fontes sem redefinir: **column labels**, **aggregation functions**, e também **hierarchies** e **variables** das views incluídas.
- As colunas correspondentes precisam existir na view que extrai (não faz sentido extrair hierarchy de produto sem a coluna produto).

### Propagate to Semantics
- Em views com muitos nós empilhados: selecione colunas no output de um nó → clique direito → **Propagate to Semantics** → mapeadas até o topo (menos trabalhoso que nó a nó).

### Preview de nós intermediários
- Requer a view **já deployada** com sucesso. Selecione o **cabeçalho do nó** (não a data source dentro dele — senão vê os dados da fonte original).

### Map Input Parameters entre nós
- Passa o valor e também a lista de valores ao diálogo. No **Manage Mappings** dá para **copiar e mapear** um input parameter da view fonte num clique (não precisa criar antes).

### Outras ajudas
- **Copy and Paste** de nós (só o nó ou o nó com todos os nós abaixo e conexões).
- **Insert a node between two nodes:** arraste o novo nó da paleta e **solte exatamente sobre o link** entre dois nós (só graficamente). Ex.: inserir Projection entre Join e Aggregation para calcular antes de agregar, mantendo o trabalho feito.
- **Navigating to Source Calculation Views:** clique direito na data source (view) no cenário → **Open**.

---

## Lição 16.2 — Working with Modeling Content in a Project

### Data Lineage e Impact Analysis (dependências entre modelos)
- Clique direito no arquivo da view na Explorer → **Show Data Lineage** ou **Open Impact Analysis**. Funcionam de forma **simétrica**:
  - **Data Lineage:** mostra **todas as views e tabelas fonte das quais** a view **depende** (para baixo).
  - **Impact Analysis:** mostra a **cadeia de views que dependem** da view (para cima).
- Duplo clique abre a view na árvore de dependências.
- Desde **QRC 3/2022:** lineage e impact analysis **entre HDI containers**. A visibilidade depende dos privilégios do **`_RT` user**: com **CATALOG READ** vê tudo; senão, só objetos com object privilege (ex.: SELECT).

### Column Lineage (origem de uma coluna)
- No **Semantics node**, escolha uma coluna → **Show Lineage**: destaca todos os nós onde a coluna existe até o nó/fonte de origem (ver aba Mapping para o nome original).
- Em **CUBE with Star Join**, só funciona para **private columns** (não shared).
- Útil quando colunas são **renomeadas**, para saber a origem de calculated/restricted columns, e para não confundir colunas de mesmo nome (ex.: `NODE_KEY` de `SNWD_BP` vs. de `SNWD_AD`).
- Fica ativo até clicar em **Exit**.

### Outline
- Feature **Outline** do editor de calculation views (estrutura da view). Não confundir com a **Outline view** geral do BAS (estrutura de código).

### Where-Used
- No editor, identifica onde um objeto é usado dentro da view — para achar elementos supérfluos ou evitar quebrar consistência.
- Suporta: **Input Parameters**, **Calculated Columns**, **Restricted Columns**.

---

## Pontos-chave para a prova (Unit 16)
- Dependências **entre modelos**: **Data Lineage** (o que eu uso) e **Impact Analysis** (quem me usa). Column lineage e Outline são **dentro** da view.
- **Extract Semantics:** column labels, aggregation functions, **hierarchies**, variables (não calculated columns).
- **Propagate to Semantics:** leva colunas até o topo.
- Preview de nó intermediário exige deploy.

---

## Quiz oficial da Unit 16 — com gabarito
1. **Ferramentas para analisar dependências entre modelos?** (2) → ✅ **Data lineage**; ✅ **Impact analysis**.
2. **Que semantics podem ser extraídas com Extract Semantics?** (2) → ✅ **Column Labels**; ✅ **Hierarchies**.

---

## Glossário
- **Data Lineage / Impact Analysis / Column Lineage.**
- **Extract Semantics / Propagate to Semantics.**
- **Where-Used:** onde o objeto é usado.
- **CATALOG READ:** system privilege de leitura do catálogo.
# Unit 17 — Working in a Modeling Project (Estrutura do Projeto, Deploy e Gestão de Conteúdo)

**Foco da unit:** **workspace / project / module / folder**, **HDB module**, **namespace** (`.hdinamespace`), **HDI** e **HDI containers**, `.hdiconfig`, view **SAP HANA PROJECTS**, **deploy** e erros, import/export, copiar/renomear/mover/apagar, renomear colunas, properties file e **deprecate**.

---

## Lição 17.1 — Project Structure

### Workspace, Projects, Modules, Folders
- **Workspace:** estrutura de arquivos onde você trabalha em um ou mais projetos. Cada usuário tem o seu (pode criar outros).
- Projetos com **nomes diferentes**, cada um numa pasta raiz. Pode abrir um projeto por vez ou criar um **multiroot workspace**.
- Hierarquia: **Workspace → Project → Module → Folders → Objects**. (Diretamente abaixo do projeto ficam os **modules**.)
- Ao criar projeto: escolhe-se um **template**. Para modelagem, o mais simples é **SAP HANA Database**.
- **Modules:** tipos diferentes (Java, HTML, **database**). Pode ter vários, mas para banco geralmente basta **um**. Dê nomes significativos; não crie módulos vazios.
- **Folders** dentro do módulo organizam os source files (ex.: tables, calculation views).

### HDB Module (HANA Database module)
- Contém todos os **design-time files** dos objetos de banco que devem ser deployados com a aplicação. É uma pasta (ex.: `db`).
- Contém a pasta **`src`** (gerada por padrão — onde fica o conteúdo de modelagem) e conteúdo do deploy HDI (`.env`, pasta `node_module` com o **node.js HDI Deployer**).
- É referenciado no descritor do projeto **`mta.yaml`** (com settings como database id ou schema name, nem todos obrigatórios).
- **Principais artifacts:** Tables, Calculation views, Functions, Procedures, Analytic privileges, Synonyms.
- **Persistence layer:** os dados podem ficar numa persistência **definida pela própria aplicação** (dentro do HDB module) ou **externa** (ex.: schema clássico).

### Object identifier e Namespace
- Runtime object: **`<Namespace>::<Runtime Object Name>`** — nome **obrigatório**; namespace **opcional** (organização lógica). Ex.: `HC::CVC_SalesOrders`.
- Os identificadores são **sempre especificados nos design-time objects**.
- **Regras do namespace:**
  - Opcional; um HDB module pode ter nenhum ou vários namespaces.
  - **Igual para todos os objetos da mesma pasta**.
  - Deve estar **explícito** no design-time file e corresponder ao da pasta (explícito ou herdado das pastas pai).

### Arquivo `.hdinamespace`
- Em `src` (criado automaticamente) ou em qualquer subpasta. Padrão: `"<application_name>.<HDB_module_name>"`.

| Elemento | Valores | Padrão |
|---|---|---|
| **"name"** | namespace válido ou `""` (sem namespace) | `"<application_name>.<HDB_module_name>"` |
| **"subfolder"** | **"append"** (acrescenta nomes das subpastas) ou **"ignore"** | **"append"** |

- Permite convenções como `db.data.table` para tabelas e `db.models.dim` / `db.models.cube` para views.
- **Cuidado:** mudar o `.hdinamespace` afeta imediatamente os identificadores, mas as novas regras só valem no container **depois do deploy do próprio `.hdinamespace`** — antes disso o deploy dos objetos falha.
- **Recomendação:** definir as regras de namespace **no início** do projeto.

### Nome do objeto

| Tipo | Objetos por arquivo | Regra |
|---|---|---|
| Calculation views, table functions, procedures, analytic privileges | **Um** objeto core por arquivo | Nome definido no conteúdo e **deve coincidir** com o nome do arquivo (sem extensão) — recomendado |
| Synonyms, table data import, roles | **Um ou vários** por arquivo | Nome só no conteúdo, sem vínculo com o nome do arquivo |

- Uma calculation view gera uma **column view core** + column views filhas (attributes, hierarchies…).

---

## Lição 17.2 — Deploying Models

### SAP HANA Deployment Infrastructure (HDI)
- Conjunto de serviços para fazer deploy de objetos de banco em **HDI containers** isolados.
- **HDI container** = schema de banco com recursos extras (**owner dedicado** + metadados) → "**smart schema**".
- HDI gerencia **dependências**, a **ordem** de deploy e upgrades.
- **Todo o ciclo de vida** (criar, alterar, apagar) é feito **exclusivamente pelo HDI** → você **não** cria tabelas/views com **DDL SQL** direto no container.

### Data sources de calculation views
- Devem existir como **design-time files** no container do HDB module:
  - Objetos "reais" (table, view, table function, virtual table…) com source file no módulo.
  - **Synonyms** para objetos em outros schemas.
- View que referencia objeto criado direto por SQL **não builda**: `The file requires "db://HA300::TABLE_SQL" which is not provided by any file`.

### Propriedades-chave dos HDI containers
- Criado automaticamente no **primeiro deploy** do database module.
- Gera um **schema** no momento da criação; objetos são deployados nele.
- Source files escritos **schema-free**.
- **Referências diretas** a objetos fora do container **não são permitidas** → use **synonyms**.

### `.hdiconfig`
- Arquivo **obrigatório** em `src`, **sem nome** (só a extensão `.hdiconfig` — se nomear, o build falha).
- **Liga cada tipo de source file ao build plug-in** correto (ex.: suffix `hdbcalculationview` → plug-in `com.sap.hana.di.calculationview`).
- Normalmente **um por HDB module**.
- **`minimum_feature_version`:** impede build com versão de plug-ins inferior à exigida (ex.: feito com 1004 não faz deploy onde há 1002).
- Ao importar um projeto, ele traz seu `.hdiconfig`; para usar features novas, **ajuste** a feature version.

### View SAP HANA PROJECTS
- Materializa os HDB modules. Parte superior: estrutura de pastas; deploy em nível de **arquivo, pasta ou módulo inteiro**.
- **Não** dá para mover/copiar/apagar arquivos ali (só abrir).
- **Pending Deployment** marca o que precisa de deploy. Para **apagar** runtime objects de arquivos deletados, faça deploy da **pasta** que os continha.
- Parte inferior: **Database Connections** (HDB module ↔ HDI containers ou serviços externos). Um projeto com HDB module precisa de pelo menos **um HDI container**; você pode **bind** a um novo ou existente.

### Database Explorer e usuário técnico
- Ao adicionar o container no Database Explorer, o acesso é feito por um **technical SBSS user (Service Broker Security Support)**, criado de forma transparente. Se o container consome schema externo, esse usuário também precisa de autorização.

### Deploy
- Função **Deploy** da view SAP HANA PROJECTS. Usa o **HDI container service** (serviço **Cloud Foundry**).
- **Bind** do HDB module a um HDI container service cria o schema (se não existir).
- Escopos: **HDB module inteiro**, **subpasta**, **um ou vários objetos**.
- Não confunda com **`cf deploy`** do Cloud Foundry (aplicação MTA inteira — usado para levar para QA/PROD).
- Sempre há **checagem de dependência end-to-end** (recursiva), mesmo deployando um único objeto.

### Erros comuns de deploy
- Objeto do qual outro depende **não definido** ou **ainda não deployado**.
- Mesmo runtime object definido por **vários** design-time files.
- **Inconsistência de namespace** entre o arquivo e a pasta.
- Serviço para schema externo não definido/indisponível no space.
- **Synonyms** para schema externo ainda não deployados.
- Erro específico do objeto (ex.: **CUBE sem measure**).
- Inconsistência com objeto referenciado (ex.: nome de coluna diferente).

### Contornando uma view que não builda
- **Exportar** a view para lugar seguro e apagá-la do workspace (depois reimportar); ou
- Adicionar **`.txt`** à extensão (`SALES.hdbcalculationview.txt`) — vira texto simples, sem checagem; para desfazer, remova `.txt`.

---

## Lição 17.3 — Manage Modeling Content

### Export / Import
- Útil para versões, sandboxing, mover entre dev spaces, compartilhar (ex.: definições de synonyms).
- **Export:** **File → Download** ou clique direito → Download. Arquivo único = baixado como está; **pasta** = **.tar**; vários itens = um .tar. Projeto inteiro: clique direito na área em branco.
- **Import:**
  - **Drag and drop:** arquivos compactados **não** são extraídos (extrair via terminal).
  - **Upload Files:** **não** extrai.
  - **Import / Import Project:** **.zip e .tar são extraídos**.
- Projeto inteiro (ex.: `proj1.tar`): abra antes a pasta `projects` para que vire `projects/proj1`; depois abra como workspace.

### Regras para copiar, renomear, mover e apagar
- Num HDB module, a definição de um runtime object (`<namespace>::<object_name>`) **não pode existir mais de uma vez**.
- O namespace no arquivo deve corresponder ao da pasta.
- Deploy **checa dependências em todo o módulo**, mas só **deploya os arquivos selecionados**.
- Checagens valem para objetos já buildados + escopo do deploy; arquivo nunca buildado e fora do escopo é ignorado.

**Copiar:** na mesma pasta exige outro nome; o conteúdo não muda → **renomeie os runtime objects** dentro do arquivo (e ajuste namespace). Lembre que o editor continua mostrando o **original**.

**Mover:** mesma regra de namespace → sem impacto nas referências, mas o deploy precisa incluir a **pasta de origem** (para registrar a remoção). Namespace diferente → ajustar e fazer **impact analysis** antes.

**Renomear:** para o HDI = **apagar + criar**. Mantenha nome do arquivo = nome do runtime object (boa prática). Verifique dependências. Se renomear o arquivo mas não o runtime object e buildar só o arquivo → **erro** (nome duplicado); builde **no nível da pasta**.

**Apagar:** a exclusão só é reconhecida ao deployar o **módulo** ou a **pasta** que continha o arquivo (a pasta, mesmo vazia, é "preciosa").
- Antes de apagar um arquivo → **impact analysis**.
- Antes de apagar uma pasta → **deploy parcial da pasta**.
- **Nunca** apague objetos de um HDI container com **SQL**.

### Search/Replace
- **Ctrl+F / Ctrl+H** no editor; **Search view** (com padrões de inclusão/exclusão); **Find in folder**; busca **fuzzy** por nome de arquivo digitando na Explorer.

### Renomear colunas
- **Nó intermediário:** só **output columns** (aba Mapping → Properties → Name, ou aba Columns para várias). Se a coluna ainda não foi mapeada acima, o novo nome sobe; se já foi, a mudança é **local** (acima volta ao nome original).
- **Top node:** nomes usados por front-ends, SQL e outras views. Use **Rename and Adjust References** no Semantics. Desde **QRC 3/2023:** **Propagate Recursively in Consuming Views** (uma coluna por vez).
- Views expostas externamente exigem cuidado especial.

### Generate Properties File
- **Modeling → Generate Properties File**: arquivo com nome e descrição de colunas, input parameters e variables; após build do HDB module, valores vão para **`BIMC_DESCRIPTION`** → permite **tradução** para vários idiomas; ferramentas cliente leem essa tabela.

### Deprecate Calculation Views
- Semantics → **View Properties → General → Deprecation Option**:
  - **Deprecated and Queries Permitted:** queries continuam; o editor mostra **aviso** (ícone/tooltip) se um nó referencia view deprecated — aviso ao **desenvolvedor**.
  - **Deprecated and Queries Blocked** (QRC 1/2024): queries **bloqueadas**, com mensagem customizável em **Deprecation Note** + erro padrão `Instantiation of calculation model failed`.

---

## Pontos-chave para a prova (Unit 17)
- Abaixo do projeto: **Module**.
- Todo runtime object no container precisa de **source file** (HDI gerencia o ciclo de vida).
- Namespace: `.hdinamespace` (name / subfolder append|ignore); definir cedo.
- `.hdiconfig`: liga suffix → plug-in; sem nome; minimum_feature_version.
- Acesso fora do container → **synonym**.
- Deploy: módulo, pasta ou objetos; dependências checadas end-to-end.
- Deprecate = aviso ao **desenvolvedor** para não reutilizar a view.

---

## Quiz oficial da Unit 17 — com gabarito comentado
1. **O que aparece diretamente abaixo do projeto?** → ✅ **Module**.
2. **Runtime object num container sempre tem source file correspondente — V/F?** → ✅ **True**.
3. **Por que marcar deprecate?** → ✅ **Para avisar o desenvolvedor de que não deve usar a view** (não recomendada para reuso).

---

## Glossário
- **HDB module / src / mta.yaml.**
- **Namespace / .hdinamespace / Object identifier (`ns::name`).**
- **HDI / HDI container ("smart schema") / HDI Deployer / build plug-in / .hdiconfig.**
- **SAP HANA PROJECTS view / Pending Deployment / Bind.**
- **SBSS user:** usuário técnico do Database Explorer.
- **BIMC_DESCRIPTION:** tabela de descrições/traduções.
- **Deprecate.**
# Unit 18 — Managing the Lifecycle of a Modeling Project (Ciclo de Vida do Projeto)

**Foco da unit:** criar projeto com HDB module, **schema name**, **HDI Cockpit**, acesso a dados externos (**user-provided service**, **.hdbgrants**, **.hdbrevokes**, **synonyms**, cross-container), **Git** e deploy de aplicação (**MTA / .mtar**).

---

## Lição 18.1 — Creating a Project

- Template usual: **SAP HANA Database Project**.
- O HDB module pode definir uma **persistência local**. Num **virtual data model** com calculation views, o HDB module contém os artifacts mesmo que os dados fiquem fora da aplicação.
- A maioria das propriedades do módulo vai para o **`mta.yaml`**, **exceto o namespace**, materializado no **`.hdinamespace`** em `src`.

| Setting | Descrição | Exemplo |
|---|---|---|
| Project Name | Nome do projeto | sales_analytics_project |
| Module Name | Único no `mta.yaml` | core-database-module |
| Type | **hdb** = tipo reservado para HDB module | hdb |
| Path | Local da raiz do módulo no projeto | db |
| Namespace | Opcional — prefixo dos objetos | sales |
| Schema Name | Opcional — nome do schema do HDI container | SAMPLE_SCHEMA |
| SAP HANA Database Version | SAP HANA Service (legado) ou HANA Cloud | — |

### Schema name
- Se **não** especificado, é gerado no deploy: **`<project_name>_<hdi_service_name_in_MTA>_<numeric_increment>`** (ex.: `MYPROJECT_HDI_DB_1`).
- Descobrir o schema: `select CURRENT_SCHEMA from DUMMY;` num SQL console conectado ao container.
- **É possível especificar** um schema no `mta.yaml`, nos parâmetros do serviço **hdi-container** (`config: schema: <YOUR_SCHEMA_NAME>`) → gera `<YOUR_SCHEMA_NAME>_<numeric_increment>`.
- Dá até para forçar o nome exato (parâmetro adicional), mas **não é recomendado** (conflitos entre projetos).

### HDI Cockpit
- Um banco tem muitos containers, com combinações de usuários e roles → difícil gerenciar.
- **HDI Cockpit (HDI Administration):** visibilidade de **todos os containers**, agrupamento em **container groups** e exibição de usuários/roles com acesso.

---

## Lição 18.2 — Enabling Access to External Data

### Princípio de isolamento
- Por padrão, o container tem acesso total aos **seus** objetos, mas **nenhum** acesso a outros schemas (outro container ou schema clássico). Pode-se habilitar.

### Acesso a schema externo (clássico) — passo a passo
1. **User-Provided Service** (criado no BAS): nome + **user/password** de um usuário com privilégios nos objetos externos. É adicionado automaticamente ao **`mta.yaml`** como resource do HDB module.
2. **Arquivo `.hdbgrants`**: o usuário do serviço concede (um subconjunto de) seus privilégios a:
   - **Object Owner:** usuário técnico **dono** dos objetos do container (não o desenvolvedor). Só ele altera/apaga objetos. No deploy, precisa de acesso aos objetos externos referenciados. Precisa poder **conceder** os privilégios (para incluí-los em roles).
   - **Application User:** usado para **bind** do container a aplicações; na modelagem, normalmente o **`..._RT`** (faz data preview etc.). Geralmente não precisa de grant option.
   - Conceda **só o necessário** (o usuário do serviço pode ser compartilhado com outros projetos).
3. **Synonyms** (`.hdbsynonym`): apontam para os objetos externos.

### `.hdbgrants` e `.hdbrevokes`
- `.hdbgrants` **não materializa** objetos no banco; remover um privilégio do arquivo e rebuildar **não revoga**.
- Para revogar: arquivo **`.hdbrevokes`** (mesma estrutura) listando o que revogar + remover do `.hdbgrants`.
- **Cuidado:** no deploy, `.hdbrevokes` é processado **antes** do `.hdbgrants` → o que estiver no grants **sempre** é concedido.
- **Prática comum:** criar **duas roles fora** do container (object owner e application user), concedê-las ao usuário do serviço e referenciar só essas roles no `.hdbgrants`. Mudanças nas roles refletem automaticamente, sem rebuild.

### Synonyms
```json
{
  "HA300::SALES_DATA": { "target": { "object": "SALES_DATA", "schema": "TRAINING" } },
  "HA300::CUSTOMER":   { "target": { "object": "CUSTOMER",   "schema": "TRAINING" } }
}
```
- Três parâmetros: **nome do synonym** (usado no container), **object** (nome real) e **schema** (onde está).
- Editável em texto ou no **synonym editor**.

### Cross-container access
- Semelhante, com diferenças:
  - **Não precisa** de user-provided service se o container externo está no **mesmo space** — adicione o serviço do container externo ao projeto (vai para o `mta.yaml`).
  - Crie **roles dentro do container externo** com os privilégios.
  - O `.hdbgrants` referencia **essas roles** (não privilégios de um usuário técnico).
  - Ainda precisa de **synonyms**.

---

## Lição 18.3 — Using Git to Manage Source Code

### Por quê
- Vários desenvolvedores: não sobrescrever trabalho, usar a versão mais recente, voltar a versões anteriores, desenvolvimento **paralelo** (ex.: patch da versão 2.1.0 enquanto desenvolve features).
- Benefícios: **backup** do código, **histórico completo**, **branching e merging**, **rastreabilidade** (ligar mudanças a ferramentas de projeto).

### Git em resumo
- **Distributed Version Control System (D-VCS)**; cada desenvolvedor tem cópia local do repositório (todo ou parte do histórico).
- Arquitetura entre local e central; repositórios compartilhados internos ou públicos (ex.: **GitHub**).
- Software livre (GNU GPL).
- **Não** é para code review nem **deploy automatizado** — para isso existem **Gerrit** (review) e **Jenkins** (CI/CD).

### Ciclo de vida dos arquivos
- Três áreas: **working directory**, **staging area (INDEX)**, **local repository**.
- Staging = o que entra no próximo **commit** (não faça stage do que não quer commitar).
- **Histórico** = sequência de commits (cada um aponta para o pai; guarda data, autor, arquivos afetados).
- **Branch** = série de commits; tecnicamente um **ponteiro** para um commit. Sempre há pelo menos uma (padrão **master**/main). Ex.: branch `Feature32` criada após C3.

### Git no BAS (CLI)
- Pré-requisitos: conta e repositório no GitHub.
```bash
git config --global user.email "git_email@company.com"
git config --global user.name "user_name"
git init                       # inicializa repo local
git add .                      # stage
git commit -m "First commit"   # commit local
git branch -M main             # branch main
git remote add origin https://<host>/<repo>
git push origin main
```
- Autenticação com **Personal Access Token (PAT)** em vez de senha (só **HTTPS**). Criar: GitHub → Settings → Developer settings → Personal access tokens → Generate (scope **repo**).
- Cache de credenciais: `git config credential.helper cache --timeout=...` / `git config credential.helper store`; limpar: `git config credential.helper ""`.
- Também há **UI** de Git no BAS: init → stage all → commit → push.

### Status de arquivos
- **A** = novo e staged; **U** = untracked/unstaged; **M** = modificado; **D** = deletado; **C vermelho** = conflito (merge/rebase); **C azul** = cópia. Pode stage, deixar unstaged ou **discard**.

### Commit
- Adiciona ao histórico com descrição. Não altera o working directory.
- Boa mensagem: assunto curto (≤50 caracteres, modo imperativo), linha em branco, corpo com o quê e por quê, link para issue (ex.: JIRA).
- **Amend:** substitui o último commit — **não** faça amend em commits já compartilhados.

---

## Lição 18.4 — Deploying an Application

### Deploy de objetos vs. deploy de aplicação
- **Objetos:** durante o desenvolvimento, pelo BAS, no HDI container do dev space (precisa de role de modelador no space).
- **Aplicação:** num **space alvo** (QA, PROD), em momento específico, **fora do BAS**; quem faz não precisa de role de desenvolvedor.
- Aplicações full-stack (HDB, HTML, NodeJS, Java…) = **MTA (multitarget application)**.

### Passos para QA/PROD
1. **Build** dos HDB modules.
2. **Build do projeto inteiro** → gera o arquivo **MTA archive (`.mtar`)**.
3. **Exportar** o `.mtar`.
4. **Deploy** do `.mtar` no ambiente alvo (entre 3 e 4: disponibilizar o arquivo, ex.: ferramenta de transporte).
- O BAS **não participa** do passo 4; não precisa criar/buildar o projeto no destino.

### Arquivos
- **`.mtar`**: tecnicamente um arquivo compactado; nome `<Application ID>_<version>.mtar`, na pasta **`mta_archives`**. Contém **`mtad.yaml`** (equivalente ao mta.yaml para deploy).
- **`.mtaext`** (opcional): extensão em que o administrador do space ajusta parâmetros para o ambiente alvo (nome do schema do container, nome real de user-provided service, de outro HDI container service).
- Gerar `.mtar`: terminal na **raiz do projeto** → **`mbt build`**.
- Nome do HDI container no deploy = nome do resource no `mta.yaml` (padrão `hdi-<HDB_module_name>`).

---

## Pontos-chave para a prova (Unit 18)
- Acesso a objetos externos → **Synonym** (com user-provided service + .hdbgrants).
- Object Owner vs Application User (`_RT`).
- `.hdbrevokes` antes de `.hdbgrants`.
- Cross-container: roles no container externo; sem user-provided service no mesmo space.
- **Git não automatiza deploy** (False).
- Deploy de aplicação gera **.mtar** (`mbt build`); `.mtaext` para ajustes do alvo.
- Schema name **pode** ser definido no mta.yaml (falso que não pode mudar).

---

## Quiz oficial da Unit 18 — com gabarito comentado
1. **Que artifact define acesso a objetos de schema externo?** → ✅ **Synonym**.
2. **Git é usado para automatizar o deploy de runtime objects — V/F?** → ✅ **False**.
3. **Tipo de arquivo gerado no deploy de uma aplicação completa?** → ✅ **.mtar**.
4. **O nome do schema do container é gerado e não pode ser mudado — V/F?** → ✅ **False** (pode ser definido no `mta.yaml`).

---

## Glossário
- **User-Provided Service / .hdbgrants / .hdbrevokes / .hdbsynonym.**
- **Object Owner / Application User.**
- **Cross-container access.**
- **HDI Cockpit / container groups.**
- **Git: working directory, staging/INDEX, commit, branch, merge, push, PAT.**
- **MTA / .mtar / mtad.yaml / .mtaext / mbt build.**
# Unit 19 — Implementing Security in SAP HANA Modeling (Segurança)

**Foco da unit:** **Analytic Privileges** (attribute, SQL expression, dynamic; hierarchy nodes; nested views), **Design-Time Roles** (`.hdbrole`, `.hdbroleconfig`, `.hdbgrants`) e **Data Masking**.

---

## Lição 19.1 — Defining Analytic Privileges

### Conceito
- **Analytic privileges** controlam o acesso a dados em calculation views **filtrando linhas** pelos valores de um ou mais **attributes** (row-level security).
- Evitam criar cópias da mesma view com filtros diferentes. Ex.: gerentes regionais usam a mesma view, cada um vê só sua região.
- Um analytic privilege pode ser atribuído a **vários usuários** com a mesma necessidade.

### Passo a passo (end-to-end)
1. Criar source file **`.hdbanalyticprivilege`** (New File → nome com extensão → Rename/label).
2. Atribuir as **calculation views** a proteger (qualquer tipo) — lista **Secured Models**.
3. Escolher o **tipo de restrição** e defini-la.
4. Configurar as views para **checar** analytic privileges: propriedade **Apply Privileges = SQL Analytic Privileges**.
5. **Deploy** do analytic privilege.
6. Atribuir o analytic privilege a uma **role**.
7. Atribuir a role ao **usuário**.

### Tipos de restrição (exclusivos — um por analytic privilege)

| Tipo | Como usar | Exemplo |
|---|---|---|
| **Attribute** | Editor de restrições: selecione attributes das views e defina valores | REGION: EMEA; YEAR: Between 2015 e 2017 |
| **SQL Expression** | Expressão SQL **estática** com attributes e valores | `("REGION"='EMEA' AND "YEAR"='2015')` |
| **Dynamic** | **Procedure** que retorna dinamicamente uma expressão SQL (como um WHERE) | `P_DYNAMIC_AP_FOR_REGION` |

### Restrição Attribute — operadores
- **Between**, **ContainsPattern** (com `*`), **=, <=, <, >, >=**, **IsNull / Is Not Null**, **In**.
- Todos (exceto IsNull/Is Not Null) aceitam string vazia `" "`.
- **Só colunas do tipo Attribute** (não Measure).
- Se uma DIMENSION view tem **parent-child hierarchy habilitada para SQL**, pode restringir por **nó da hierarchy**.

### Combinação de restrições
- **Mesma coluna:** várias restrições → **OR**.
- **Colunas diferentes** no mesmo analytic privilege → **AND**. Ex.: YEAR=2017 e COUNTRY=US → só dados com **YEAR=2017 AND COUNTRY=US**.
- **Vários analytic privileges** para a mesma view e usuário → combinados com **OR** → o usuário vê **mais** dados (YEAR=2017 **OR** COUNTRY=US).

### Validity period
- Restrição Attribute pode valer só por um período — baseado na **data da consulta (CURRENT_DATE)**, **não** nos attributes de tempo da view.
- Em SQL Expression: adicione `(CURRENT_DATE BETWEEN '...' AND '...') AND <expressão>`.

### SQL Expression
- Para lógica que o tipo Attribute não expressa (ex.: precedência AND/OR). Ex.: `("COUNTRY" = 'US' OR "YEAR" <= '2016')`.
- **Dica:** comece com Attribute e **converta** para SQL Expression (código gerado). **O contrário não é possível.**

### Dynamic Analytic Privilege
- Condições determinadas **em runtime** por usuário → abordagem **escalável**: **o mesmo analytic privilege serve vários usuários com acessos diferentes**.
- Procedure retorna, por exemplo, `("COUNTRY"='US')` para User1 e `("COUNTRY"='UK' OR "COUNTRY"='FR')` para User2.
- **Propriedades obrigatórias da procedure:**
  - **Read-only**
  - Security mode **DEFINER**
  - **Sem input parameters**
  - **Um único output escalar** do tipo **VARCHAR(256)** ou **NVARCHAR(256)**

### Erros de deploy (checagem de dependência para evitar views desprotegidas)
- View com check de SQL analytic privileges ativado, mas **nenhum** analytic privilege a lista em Secured Models.
- Analytic privilege cujas views **não** têm Apply Privileges = SQL Analytic Privileges.
- View com check **desativado**, mas ainda listada em algum analytic privilege.
- **Melhor prática:** buildar analytic privileges e views **juntos**.
- Após deploy, as views **não podem ser vistas** até o privilege ser concedido (via **design-time role**).

### Restrição por nó de hierarchy
- Usuário com acesso a um nó vê todos os **descendentes** (ex.: nó NA → todos os países da América do Norte).
- Passos: habilitar a view para check de analytic privilege **e** SQL access a hierarchies → criar analytic privilege → Attribute restriction → **Hierarchical Privilege** → escolher hierarchy e nó.
- **Suportado** em **CUBE with Star Join** com check de SQL analytic privilege e SQL access a hierarchy habilitados, e **só para parent-child hierarchies**.

### Segurança em views aninhadas (nested)
Para cada view: o usuário tem **SELECT** na column view? A view checa analytic privileges? O usuário tem analytic privilege para ela?
- **Object privileges:** basta **SELECT na view do topo** — não precisa em views/tabelas inferiores.
- **Analytic privileges:** aplicados em **toda a hierarquia**. Se alguma view da pilha checa analytic privileges e o usuário **não** tem privilégio para ela → **nenhum dado** (não autorizado).
- O output de uma view inferior define o **máximo** que as superiores podem mostrar.
- Analytic privilege **não concede SELECT implícito** — o usuário sempre precisa de **SELECT explícito**.

---

## Lição 19.2 — Defining Roles

### Conceitos de segurança
- Usuários podem ser **donos** de objetos. **Role** = coleção de privilégios, concedida a usuários ou outras roles (**nested roles**). **Privilege** = permite operações em objetos.
- Recomenda-se **gerenciar autorizações via roles** (privilégios diretos ao usuário = muita manutenção).
- Todos os privilégios (diretos e indiretos) são **somados**.
- **Não existe "deny"** — todos os privilégios concedem acesso. A checagem para assim que encontra os privilégios necessários.
- Há **roles predefinidas** (algumas são templates).

### Dois tipos de role
- **Catalog Roles:** runtime puro (sem source file), criadas via **SQL** ou **SAP HANA Cockpit**; privilégios via `GRANT`.
- **Design-Time Roles:** source files no HDB module, extensão **`.hdbrole`**; privilégios concedidos ao **deployar**. **Recomendadas** para acessar objetos locais do container.
- **Uma role por arquivo `.hdbrole`**; role ID (com namespace) única no HDB module.

### `.hdbroleconfig`
- O `.hdbrole` **não pode** conter nomes reais de schema, só **referências lógicas** (ex.: `Ref1`), resolvidas no **`.hdbroleconfig`** → nomes de schemas externos mantidos num só lugar.
- Pode criar manualmente ou gerar pelo editor do `.hdbrole`.
- **Role Editor** abre por padrão ao criar um `.hdbrole`.

### Acesso a schema externo e `.hdbgrants`
- Administrador define **user-provided service** com usuário técnico; o projeto decide quais autorizações vão para quais roles. Para calculation views, o principal é **SELECT** nas fontes.
- **`.hdbgrants`** em três níveis:
  1. Nome do **user-provided service**.
  2. Usuários: **`object_owner`** (dono dos objetos do container) e **`application_user`** (usuários ligados aos módulos da aplicação).
  3. Conjunto de privilégios (sintaxe parecida com `.hdbrole`).
- Um `.hdbgrants` pode listar autorizações de **vários** user-provided services.
- Ex.: SELECT só para object_owner permite criar o synonym, mas sem SELECT para application_user **não** se consegue ver o conteúdo da tabela.

---

## Lição 19.3 — Masking Sensitive Data

### Column Masking
- Define uma **máscara** em colunas sensíveis: só usuários autorizados veem o valor real; os demais veem parte/todo **oculto** (preview ou front-end).
- **Como:** Semantics → aba **Columns** → selecionar coluna → ícone **Data Masking** → expressão **SQL**.
- Tipos suportados: **VARCHAR, NVARCHAR, CHAR, SHORTTEXT**. Funciona com tabelas **ROW e COLUMN**.
- Máscara = string constante (`XXX-XXX-XXX`) ou expressão SQL com funções de string. Ex.:
  ```sql
  LEFT(credit_card,4) || '-XXXX-XXXX-' || RIGHT(credit_card,4)
  ```
  `1111-2222-3333-4444` → `1111-XXXX-XXXX-4444`.

### Privilégio UNMASKED
- Sem o privilégio **UNMASKED**, o usuário vê a máscara. Normalmente incluído numa role do HDB module.
- **Nível schema** (todas as views do container; também tabelas e SQL views):
  ```json
  { "role": { "name": "db::UNMASK_ENTIRE_SCHEMA",
      "schema_privileges": [ { "privileges": ["UNMASKED"] } ] } }
  ```
- **Nível objeto** (objeto específico, com tipo e nome com namespace):
  ```json
  { "role": { "name": "db::UNMASK_EMPLOYEES_PAYMENT",
      "object_privileges": [ { "type": "VIEW", "name": "db::CVD_EMPLOYEES_PAYMENT", "privileges": ["UNMASKED"] } ] } }
  ```
- **UNMASKED não substitui SELECT** (SELECT continua obrigatório).

### Mask Mode (desde QRC 4/2021)
- **Default:** máscara baseada no usuário que **chama** a view com a máscara. Em views inferiores, executadas com privilégios do **Object Owner** (que sempre tem unmasked) → dados não mascarados nas inferiores.
- **Session User:** máscara baseada no **usuário da sessão** que executa a query (relevante em views inferiores de uma pilha).
- Configuração **global da view**; no topo de uma pilha, o mask mode da view do topo **não sobrescreve** o das inferiores.

---

## Pontos-chave para a prova (Unit 19)
- **Analytic privilege** = filtro de **linhas** por attributes; `.hdbanalyticprivilege`; Apply Privileges = **SQL Analytic Privileges**.
- Tipos: **Attribute**, **SQL Expression**, **Dynamic (procedure)**; Attribute → SQL possível, inverso não.
- Mesma coluna OR / colunas diferentes AND / vários privileges OR.
- **Dynamic** = reutilizável para vários usuários com acessos diferentes; procedure read-only, DEFINER, sem input, output VARCHAR/NVARCHAR(256).
- Nested: SELECT só no topo; analytic privilege necessário em toda view que checa.
- **Design-time roles** (`.hdbrole`) recomendadas; `.hdbroleconfig` para schemas.
- **Masking** = esconder caracteres; **UNMASKED** para ver; Default vs Session User.

---

## Quiz oficial da Unit 19 — com gabarito comentado
1. **O que é data masking?** → ✅ **Obscurecer valores de coluna escondendo parte ou todos os caracteres com caracteres substitutos.**
2. **Abordagem recomendada para roles que acessam objetos locais do container?** → ✅ **Design-time Roles**.
3. **O que é um dynamic analytic privilege?** → ✅ **Um analytic privilege reutilizável que pode servir a vários usuários que precisam acessar dados diferentes.**

---

## Glossário
- **Analytic privilege / Secured Models / Apply Privileges.**
- **Attribute restriction / SQL Expression / Dynamic (procedure-based).**
- **Validity period / CURRENT_DATE.**
- **Catalog role / Design-time role (.hdbrole) / .hdbroleconfig / .hdbgrants.**
- **Object privilege (SELECT) / System privilege / Schema privilege.**
- **Data masking / UNMASKED / Mask Mode (Default, Session User).**
# Unit 20 — Getting Started with SAP HANA Cloud Knowledge Graph Engine (Knowledge Graphs)

**Foco da unit:** conceitos de **Knowledge Graph**, **RDF / triples**, **ontologies** e linguagens, o **SAP HANA Cloud Knowledge Graph Engine (KGE)**, suas interfaces (**SPARQL**), arquitetura e casos de uso.

---

## Lição 20.1 — Overview of Knowledge Graphs

### O que é
- **Knowledge graph** = representação **estruturada de fatos** sobre o mundo, descrevendo **entidades** e suas **inter-relações**, em formato de **grafo**.
- Base de assistentes virtuais e buscadores; pode **explicar, avaliar e fundamentar** respostas de modelos de deep learning (ex.: LLMs). Aplicações: busca, perguntas e respostas, recomendações, **explainable AI**.

| Aspecto | Definição |
|---|---|
| **Entities** | **Nós (nodes)** = entidades (pessoas, lugares, coisas, conceitos); **arestas (edges)** = relações. |
| **Relationships** | Conexões entre entidades (ex.: `acted_in` entre ator e filme). |
| **Attributes** | Características de entidades/relações (ex.: filme → director, release_date, genre). |
| **RDF** | **Resource Description Framework**: modelo padrão de intercâmbio de dados na web; usa **URIs** para nomear a relação e as duas pontas → **triple**. Forma um **grafo dirigido e rotulado**. |
| **Triples** | Unidade básica: **subject – predicate – object**. Ex.: *Paris* (subject) – *is the capital of* (predicate) – *France* (object). |
| **Semantic Meaning** | Usam **vocabulários padronizados ou ontologies** para definir tipos de entidades e relações → ideal para IA. |
| **Sources of Data** | Estruturados (bancos), semiestruturados (XML, JSON), não estruturados (texto, via **Named Entity Recognition** e **Relation Extraction**). |
| **Examples** | **DBpedia**, **Wikidata**, **Freebase** (hoje parte do Google Knowledge Graph), grafos de SAP, Amazon, Facebook, Microsoft. |
| **Applications** | Buscadores, chatbots, assistentes digitais, recomendação, integração de dados. |
| **Ontologies** | Estrutura formal do grafo (tipos de entidades, relações, attributes) → consistência e interoperabilidade. |

---

## Ontologies

### Componentes

| Componente | Descrição |
|---|---|
| **Classes (Concepts)** | Categorias/tipos de entidades (ex.: Patient, Doctor, Disease). |
| **Properties (Attributes)** | Características das classes (name, age, medical history). |
| **Relationships (Relations)** | Como as classes se relacionam (Doctor *treats* Patient). |
| **Individuals (Instances)** | Objetos concretos de uma classe (John Doe é um Patient). |
| **Axioms** | Declarações lógicas que restringem a interpretação (regras). |

### Propósitos e benefícios
- **Shared Understanding** (vocabulário comum humano/máquina), **Data Integration**, **Reasoning** (inferência automática), **Knowledge Reuse**, **Semantic Interoperability**.

### Exemplos de ontologies
- **Gene Ontology (GO)** — bioinformática; **FOAF (Friend of a Friend)** — pessoas e relações; **SNOMED CT** — terminologia clínica; **Schema.org** — marcação de páginas web.

### Linguagens de ontologia (cai no quiz!)

| Linguagem | Descrição |
|---|---|
| **RDF** | Representa informação sobre recursos na web. |
| **RDFS (RDF Schema)** | Extensão do RDF com vocabulário para classes e propriedades. |
| **OWL (Web Ontology Language)** | Mais rica/expressiva, sobre RDF/RDFS; definição e raciocínio sobre ontologies. |
| **SKOS** | W3C; tesauros, taxonomias, classificações; menos formal (`skos:Concept`, `skos:broader`, `skos:narrower`, `skos:related`). |
| **DAML+OIL** | Predecessora do OWL (DARPA + projetos da UE). |
| **SHACL (Shapes Constraint Language)** | **Validação** de grafos RDF (`sh:NodeShape`, `sh:minCount`…). |
| **Frame Logic** | Antiga, baseada em **frames** (slots e fillers). |

> **Atenção:** *Axioms* são componentes de ontologia; *Gene Ontology* é uma ontologia (exemplo); *Outline* é feature do BAS — **nenhum é linguagem**.

### Ferramentas
- **Ontology Editors:** ex.: **Protégé**.
- **Reasoning Engines:** ex.: **HermiT**, **Pellet**.

---

## Lição 20.2 — SAP HANA Cloud Knowledge Graph Engine

- Componente do banco SAP HANA (in-memory) para **processamento e análise avançada de grafos**; **estende** o SAP HANA Cloud para lidar com **estruturas de dados complexas e interconectadas**. Interage com o **Index Server**, gerenciamento de sessão e clientes HANA.

### Key Features

| Feature | Descrição |
|---|---|
| **Graph Data Processing** | Modela relações complexas; grafos **dirigidos e não dirigidos** (nós e arestas). |
| **In-Memory Processing** | Alta performance e escalabilidade. |
| **Schema Flexibility** | Estrutura **schema-less** (dados heterogêneos/evolutivos). |
| **Scalability** | Grandes volumes. |
| **Integration with SQL** | Combina queries de grafo com SQL tradicional. |
| **Graph Algorithms** | Embutidos: **shortest path**, **connected components**, **centrality**. |
| **Query Language** | Suporte a linguagens de grafo como **Cypher** (Neo4j) e **PGQL**. |

### Interfaces e APIs

| Interface | Descrição |
|---|---|
| **SPARQL** | **Interface principal** de consulta (endpoint SPARQL). **SPARQL Update** para CRUD. *SPARQL = SPARQL Protocol and RDF Query Language.* |
| **RESTful API** | Ingestão de RDF em **Turtle, RDF/XML, N-Triples, JSON-LD**. |
| **Management Operations** | Criar/apagar/gerenciar graph stores, controles de acesso. |
| **Graph Algorithms APIs** | Executar algoritmos embutidos e customizados. |
| **JavaScript API** | Via APIs cliente do SAP HANA. |
| **SQL Interface** | Extensões SQL para integrar dados relacionais e de grafo. |
| **Event Streaming** | Ingestão/processamento em tempo real. |
| **BI e Visualização** | Integração com **SAP Analytics Cloud** e ferramentas de terceiros. |
| **SAP Data Intelligence** | Integração para usar **machine learning** em dados de grafo. |
| **Cloud Foundry / Kubernetes** | **Service brokers** para Cloud Foundry; **Kubernetes operators** para ciclo de vida. |
| **OData Services** | Expor dados via OData. |

### Arquitetura (Working with Knowledge Graph)
- **Data Ingestion and Storage:** fontes estruturadas, semiestruturadas, não estruturadas; armazenamento otimizado para grafos.
- **Data Modeling com RDF/OWL:** representação semanticamente rica; ontologies.
- **Graph Querying com SPARQL:** consultas que percorrem o grafo; baixa latência.
- **Computation and Analytics:** biblioteca de algoritmos (shortest path, **community detection**, centralidade) e algoritmos customizados.
- **Integration and APIs**, **Security and Governance** (controle de acesso, criptografia, permissões por role, lineage, metadados, qualidade), **User Interface e Visualização** (dashboards via BI), **Scalability and Performance** (escala horizontal, otimização de queries, indexação).

### Primeiros passos
- **Pré-requisitos:** instância **SAP HANA Cloud** ativa; acesso ao **SAP HANA Cloud Central** (antigo SAP HANA Cloud Cockpit); permissões adequadas.
- **Passos:** acessar o SAP HANA Cloud Central → criar/selecionar instância → **habilitar o serviço Knowledge Graph** (se não estiver por padrão) → conectar com cliente SQL (SAP HANA Studio, DBeaver, JDBC/ODBC) → criar tabelas e grafos → usar o engine.
  ```sql
  CREATE COLUMN TABLE VertexTable (id INT PRIMARY KEY, name NVARCHAR(100));
  CREATE COLUMN TABLE EdgeTable (id INT PRIMARY KEY, source INT, target INT, relationship NVARCHAR(100));
  CREATE GRAPH MyGraph (VERTEX TABLE VertexTable, EDGE TABLE EdgeTable);
  INSERT INTO VertexTable VALUES (1, 'Alice');
  INSERT INTO VertexTable VALUES (2, 'Bob');
  INSERT INTO EdgeTable VALUES (1, 1, 2, 'knows');
  SELECT * FROM MyGraph;
  ```
- Configuração adicional: **ontology management** e **data integration**.

---

## Lição 20.3 — Use Cases and Demo

### Casos de uso
- **Master Data Management:** Customer 360, Product master data.
- **Supply Chain:** relacionamento com fornecedores, demand forecasting.
- **Risk & Compliance:** risco financeiro, compliance regulatório.
- **Customer Intelligence:** sales forecasting, **churn prediction**.
- **Healthcare & Life Sciences:** integração de dados de pacientes, pesquisa clínica.
- **Enterprise Search:** busca semântica de documentos, mapeamento de skills.
- **IoT / Smart Manufacturing:** manutenção preditiva, analytics em tempo real.
- **Customer Service:** base de conhecimento de help desk, **chatbots** melhores.
- **Retail:** recomendação de produtos, gestão de estoque.
- **R&D:** gestão de inovação, análise de literatura científica.
- **Setor Público:** serviços ao cidadão, análise de políticas.

### Estudo de caso — indústria farmacêutica
- **Desafio:** muito dado (ensaios clínicos, papers, patentes) difícil de aproveitar.
- **Escopo:** Data Integration; Knowledge Graph Creation; Advanced Querying and Analytics; Collaboration and Data Sharing.
- **Implementação:**
  - Ingestão de fontes (PubMed, ClinicalTrials.gov, GenBank) com **SAP HANA Data Lake** e **SAP Data Intelligence**.
  - **Ontologia** do domínio (genes, proteínas, drogas, doenças, ensaios, patentes) e relações (*gene-protein*, *drug-disease*, *drug-clinical trial*).
  - Consultas **SPARQL**, graph analytics (reposicionamento de drogas, efeitos colaterais, novos alvos), ML para prever eficácia/segurança.
  - Compartilhamento com governança (controle de acesso, criptografia).
- **Resultados:** descoberta de drogas acelerada, dados mais acessíveis, analytics avançado, colaboração, melhores decisões.

### Demo — Ontologia de uma Universidade
- **Classes:** Person, Student, Professor, Course, Department.
- **Relações:** Student *is enrolled in* Course; Professor *teaches* Course; Course *belongs to* Department; Department *has* Professor.
- Apresentado no curso como um **modelo de knowledge graph** (estrutura de classes e relações).

---

## Pontos-chave para a prova (Unit 20)
- KG = entidades (nós) + relações (arestas) + attributes; unidade = **triple** (subject, predicate, object).
- KGs usam **vocabulários padronizados / ontologies**.
- Linguagens de ontologia: **RDF, RDFS, OWL** (+ SKOS, SHACL, DAML+OIL, Frame Logic).
- Componente que estende o HANA para dados interconectados: **Knowledge Graph Engine**.
- Linguagem de consulta RDF no KGE: **SPARQL**.
- ML em dados de grafo: integração com **SAP Data Intelligence**.

---

## Quiz oficial da Unit 20 — com gabarito comentado
1. **Quais são linguagens de ontologia?** (3) → ✅ **RDF**; ✅ **OWL**; ✅ **RDFS**. ❌ Axioms, Gene Ontology, Outline.
2. **Componente central que estende o SAP HANA para estruturas complexas e interconectadas?** → ✅ **Knowledge Graph Engine**.
3. **O que knowledge graphs usam para definir tipos de entidades e relações?** → ✅ **Standardized vocabularies or ontologies**.
4. **O que integrar ao KGE para usar machine learning em dados de grafo?** → ✅ **SAP Data Intelligence**.
5. **Linguagem suportada para dados RDF no KGE?** → ✅ **SPARQL**.
6. **O estudo de caso "Ontology of a University" é um modelo de Knowledge Graph — V/F?** → ✅ **True** (conforme apresentado no curso).

---

## Glossário
- **Knowledge Graph / Node / Edge / Triple (subject-predicate-object).**
- **RDF / RDFS / OWL / SKOS / SHACL.**
- **Ontology / Class / Property / Individual / Axiom.**
- **SPARQL / SPARQL Update / Turtle / N-Triples / JSON-LD.**
- **Graph algorithms:** shortest path, connected components, centrality, community detection.
- **SAP HANA Cloud Central.**
