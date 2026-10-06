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
