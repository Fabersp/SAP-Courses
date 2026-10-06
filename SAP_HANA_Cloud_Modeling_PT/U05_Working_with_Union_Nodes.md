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
