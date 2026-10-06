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
