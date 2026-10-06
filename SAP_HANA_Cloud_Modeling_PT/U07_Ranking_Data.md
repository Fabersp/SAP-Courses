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
