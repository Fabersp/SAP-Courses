# Unit 6 — Creating Data Slices (Minus e Intersect Nodes)

**Foco da unit:** operações de conjunto (**set operations**) com os nós **Intersect** e **Minus**.

---

## Lição 6.1 — Implementing Minus and Intersect Nodes

### Intersect node
- Seleciona as linhas de uma data source que **também aparecem** em outra data source.
- Você escolhe as **colunas** em que a verificação de interseção é feita.
- Equivale ao `INTERSECT` do SQL.

### Minus node
- Seleciona as linhas que aparecem **numa data source mas não na outra**.
- Também escolhe as colunas da comparação.
- Equivale ao `MINUS` / `EXCEPT` do SQL.
- **A ordem importa:** o output contém os itens da **PRIMEIRA** data source que **NÃO** estão na **SEGUNDA**.
- Para inverter a ordem: clique direito no Minus node → **Switch Order**.

### Comportamento dinâmico
- Intersect e Minus são **dinâmicos**: a comparação usa apenas os attributes **pedidos em runtime**.
- Ex.: `country` e `name` mapeados no output de um Intersect.
  - Query pede **as duas** colunas → a **combinação** country+name determina o resultado.
  - Query pede **só name** → só `name` é comparado → podem voltar **mais linhas**.
- Uma coluna fornecida pelas duas fontes mas **não consultada** em runtime é **ignorada** na comparação.

---

## Pontos-chave para a prova (Unit 6)
- **Intersect** = linhas presentes nas **duas** fontes.
- **Minus** = linhas da **primeira** fonte que **não** estão na segunda; **Switch Order** inverte.
- Comparação feita só nas colunas **requisitadas** pela query (comportamento dinâmico).

---

## Quiz oficial da Unit 6 — com gabarito
1. **Por que implementar um minus node?** → ✅ **To select rows that appear in one data source but not another data source.** *(A outra alternativa descreve o Intersect.)*

---

## Glossário
- **Set operations:** operações de conjunto (union, intersect, minus).
- **Data slice:** recorte/fatia de dados.
- **Switch Order:** inverter a ordem das fontes no Minus.
