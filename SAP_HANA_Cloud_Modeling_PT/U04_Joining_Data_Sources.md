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
