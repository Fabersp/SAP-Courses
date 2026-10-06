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
