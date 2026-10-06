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
