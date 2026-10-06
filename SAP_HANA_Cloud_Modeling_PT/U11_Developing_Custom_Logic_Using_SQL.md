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
