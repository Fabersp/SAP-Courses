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
