# Unit 9 — Creating Dynamic Calculation Views (Variables, Input Parameters e Parameter Mapping)

**Foco da unit:** passar valores dinâmicos em runtime com **Variables** e **Input Parameters**, e **mapear** parâmetros entre views, procedures/functions e value help views (**cascading prompts**).

---

## Lição 9.1 — Implementing Variables

### Conceito
- **Variable** = passa valores dinâmicos para **filtrar attribute columns**.
- Quando um front-end chama a view, ela pede um valor (normalmente um **prompt**) para completar uma cláusula **WHERE** adicionada à query **sobre** a view:
  ```sql
  SELECT <columns> FROM <calculation_view_name>
  WHERE (("COUNTRY" = 'JP')) GROUP BY <group_by columns>;
  ```
- Variables **sempre filtram no topo** (aplicadas na query sobre a view).
- **Variables NÃO se aplicam a measures** — só attributes.

### Criação
- No **Semantics node**, aba **Parameters**.
- A definição inclui:
  - **View/Table for Value Help** + **Reference Column (Attribute):** de onde vem a lista de valores.
  - **Selection Type:** **Single Value**, **Interval** ou **Range**.
  - **Multiple Entries:** permite várias ocorrências.
  - **Mandatory** e/ou **Default Value**.
  - **Apply Filter:** em qual(is) attribute(s) da view o filtro é aplicado.
- Por padrão, a value help é a **própria calculation view** (scan da coluna). Alternativa: uma tabela/view dedicada (ex.: master data `CUSTOMERS`).
- Se a value help é a própria view, a reference column é automaticamente o attribute do Apply Filter (pode mudar, se consistente).
- Runtime:
  - **Mandatory:** o usuário **precisa** informar valores/ranges/intervalos.
  - **Não mandatory:** se nada for informado, retorna **todos os dados** sem filtro.
- Na Semantics você vê qual variable está em qual attribute; **uma variable pode ser atribuída a vários attributes**.
- **Value Help Dialog** mostra os valores possíveis no preview. No Data Preview, **From/To** aparecem mesmo se a variable não for range.

### Value help — boas práticas
- A lista da própria view pode ser **enorme** (ex.: todos os funcionários). Prefira uma **view/tabela externa com lista restrita** — melhor para o usuário e para **performance**.

### Value help baseada em hierarchy
- Se o attribute tem hierarchies, você pode indicar uma hierarchy na variable → o usuário navega na árvore em vez de uma lista plana.
- **Parent-child:** o attribute da variable deve ser o **parent** (não o child).
- **Level:** o attribute deve ser o **nível folha (leaf, mais baixo)**.

---

## Lição 9.2 — Implementing Input Parameters

### Conceito
- Também passa valores dinâmicos em runtime, mas com **mais usos** do que a variable (que só filtra attributes).
- Usos:
  - **Calculated Column:** ex.: previsão de vendas = ano atual × aumento informado pelo usuário (ex.: 11%).
  - **Restricted Column:** vendas do ano escolhido (ex.: 2019) e do anterior.
  - **Filter:** ordens com total > $10.000 (limite informado em runtime) — **filtro em measure**; produtos com nome contendo 'GTR'.
  - **Controle de onde o filtro é aplicado:** com input parameter você escolhe o **nó**; variables sempre filtram no topo.
  - Alimentar parâmetros de **scalar/table functions** (requer mapping).
- Filtrar só por attributes: **variables** são mais simples (não precisa escrever expressão) e **mais suportadas** pelos front-ends.

### Tipos de Input Parameter

| Tipo | Uso |
|---|---|
| **Direct: Currency** | Usuário escolhe moeda origem/destino (currency conversion). |
| **Direct: Date** | Data digitada ou escolhida num **calendário**. |
| **Direct: Unit of Measure** | Unidade de medida escolhida. |
| **Static List** | Lista **predefinida** de valores. |
| **Column** | Lista de valores de uma coluna da view (ou de uma value help table/view). |
| **Derived From Table** | Valores de uma tabela (não necessariamente na view). |
| **Derived From Procedure / Scalar Function** | Valor vem do output escalar de uma **procedure/scalar function**. |
| **Direct** (sem semantic type) | Entrada livre. |

- Para **Currency** e **Unit of Measure**, a lista vem das tabelas de referência — o **default schema** da view deve contê-las.
- Input parameters suportam **múltiplos valores** (filtros *List of values* / *Not in List*, expressões que aceitam multi-value).
- **Derived from Table** e **Derived from Procedure/Scalar Function** **não geram prompt** — exceto se marcar **Input Enabled** (aí o usuário pode alterar o valor retornado).
- Na execução, são passados via cláusula **PLACEHOLDER**:
  ```sql
  SELECT <columns> FROM <calculation_view_name>
  (placeholder."$$IP_YEAR$$"=>'2019') GROUP BY <group_by columns>;
  ```
- Nem toda aplicação consegue passar PLACEHOLDER → às vezes é preciso usar variables.
- **Um input parameter definido mas não usado numa expressão é ignorado.**

### Exemplo: Gross ou Net
- Input parameter **Static List** `GROSS_OR_NET` (valores Gross/Net; default GROSS).
- Calculated column `AMOUNT` (measure):
  ```
  if('$$GROSS_OR_NET$$'='Gross',"GROSS_AMOUNT","NET_AMOUNT")
  ```
- Input parameter é chamado entre **aspas simples e cifrões duplos**: `'$$NOME$$'`.
- Static List é ruim para listas grandes (precisa editar/rebuild). **Direct** permite erro de digitação/formato. **Melhor:** tipo **Column** com **View/Table Value Help** central → lista reusável e mantida centralmente.

### Datas
- **Direct** + semantic type **Date** + data type **DATE** → calendário no prompt. Para intervalo, use **dois** input parameters (Date From / Date To). Também é possível ranges.

> **Recomendação SAP:** use **input parameters em vez de variables sempre que possível** (mais flexíveis), exceto quando o front-end não suporta input parameters.

---

## Lição 9.3 — Mapping Variables and Input Parameters

### Parameter Mapping — 4 tipos

| Tipo | Descrição |
|---|---|
| **Data Sources** | Mapear input parameters da **data source** (view inferior) para input parameters da view atual. |
| **Procedures/Scalar Functions for input parameters** | Mapear parâmetros de procedure/scalar function usados em input parameters "derived from procedure". |
| **Views for value help for variables/input parameters** | Mapear parâmetros de **views externas de value help** usadas por variables/input parameters (cascading prompts). |
| **Views for value help for attributes** | Mapear para value help views associadas a **attributes**. |

- Feature **Input Parameter/Variables Mapping** na aba **Parameters**. Escolha o tipo no dropdown; à esquerda os parâmetros das views inferiores, à direita os da view atual; arraste para mapear.
- **Só Variable↔Variable e Input Parameter↔Input Parameter** — **não há cross-mapping**.
- **Auto-map:** se os nomes forem iguais, cria e mapeia automaticamente na view atual.
- Desde **QRC 1/2024:** criar um input parameter e **propagá-lo** para views consumidoras no mesmo **HDB module** (e propagar exclusão).

### Push down para views inferiores
- Em stacked models, os parâmetros das views inferiores **são ignorados** a menos que você defina parâmetros no topo e os **mapeie** para baixo.
- Tipo **Data Sources** no Manage Mapping. Empurrar filtros para as fontes **reduz dados transferidos** → **ótimo para performance**.
- Também para passar valor a um calculated column numa view inferior.
- No tipo Data Sources só **input parameter → input parameter**. Variables da view inferior **não** são mapeadas para variables da atual, mas podem ser copiadas via **Extract Semantics** (clique direito na data source → Extract Semantics → aba Variables).
- **SQL views com parâmetros** como fonte: mapeie input parameters da view consumidora (nomes podem ser diferentes). Disponível desde **QRC 4/2021**.
- Desde **QRC 1/2022:** passar input parameters para SQL View/Calculation View acessada remotamente como **Virtual Table** — requer **SDA (Smart Data Access)** com **adapter ODBC**; **não** funciona com **SDI (Smart Data Integration)**. Remoto: SAP HANA Cloud ou SAP HANA on-premise ≥ 2.0 (SQL views com input parameters on-premise a partir de 2.0 SPS02).

### Procedures/Scalar Functions em input parameters
- Quando o valor do input parameter precisa ser **calculado** (não só digitado): procedure/scalar function recebe o valor do usuário e retorna o valor para outro input parameter (tipo **derived from procedure**).

### Value help externa — Cascading Prompts
- Por padrão, a value help vem da própria view (lista irrestrita). Redirecionar para outra tabela/view expõe uma **lista restrita** e melhora a **performance** (não compete com a view principal).
- Ex.: view de produtos com filtro fixo por product group → ineficiente para mudar. Melhor: substituir o filtro fixo por uma **variable de product group** e **mapeá-la** para a variable de product group da view principal. O usuário escolhe o grupo e a lista de produtos é filtrada → **cascading prompts** (pode ter vários níveis: família → grupo → produto).
- Opção no Manage Mapping: **Views for value help for variables/input parameters**.
- Uma view externa baseada em **hierarchy** também é uma solução de cascading.

### Value help para attributes
- Atribuir value help a attributes no Semantics para o usuário filtrar o resultado no front-end; pode usar view externa com variable mapeada para variable da view principal.

---

## Pontos-chave para a prova (Unit 9)
- **Variable:** só filtra **attributes**, sempre **no topo** (WHERE); não se aplica a measures.
- **Input parameter:** filtros em **measures**, calculated/restricted columns, escolha do nó, functions; passado via **PLACEHOLDER** `$$IP$$`.
- Tipos de input parameter (Direct/Currency/Date/UoM, Static List, Column, Derived from Table, Derived from Procedure/Scalar Function).
- Mapping: 4 tipos; sem cross-mapping; auto-map.
- **Cascading prompt:** value help view do parâmetro **filho** (ex.: cidade) com filtro por input parameter, mapeado ao parâmetro **pai** (país).

---

## Quiz oficial da Unit 9 — com gabarito comentado
1. **Usar IP_COUNTRY para filtrar a value help de IP_CITY?** → ✅ **Create a value help view for the IP_CITY parameter, listing cities with a filter on Country using an input parameter. Then map this input parameter to the IP_COUNTRY parameter.**
2. **Variables podem ser definidas em attributes e measures — V/F?** → ✅ **False** (só attributes).
3. **Por que criar input parameter em vez de variable?** (2) → ✅ **To filter data by measures**; ✅ **To pass a dynamic value to an expression in a calculated column**.

---

## Glossário
- **Variable / Input Parameter:** filtro dinâmico de attribute / valor dinâmico multiuso.
- **Value Help / Reference Column:** lista de valores / coluna que fornece os valores.
- **Selection Type:** Single Value, Interval, Range.
- **PLACEHOLDER / $$IP$$:** forma de passar input parameters na SQL.
- **Parameter Mapping / Auto-map / Extract Semantics:** ligação de parâmetros entre objetos.
- **Cascading prompts:** prompts encadeados.
- **SDA / SDI:** Smart Data Access / Smart Data Integration (virtualização/integração de dados remotos).
