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
