# Unit 14 — Implementing Features to Improve Performance (Paralelização e Particionamento)

**Foco da unit:** **Partition Local Execution** (bloco de paralelização) e **table partitioning**.

---

## Lição 14.1 — Controlling Parallelization

### Contexto
- O SAP HANA Cloud **sempre tenta paralelizar** automaticamente. Mas, em passos complexos e longos, o optimizer pode manter o processamento **sequencial** por não entender a semântica de negócio (medo de quebrar o resultado).
- Se você garante que o passo pode rodar em paralelo dividindo os dados em **partições semânticas**, pode **forçar** a paralelização.

### Partition Local Execution
- Flag que marca o **início** e o **fim** do trecho do fluxo onde a paralelização é exigida → gera **várias threads** em pontos de gargalo.
- **Início do bloco:** um nó que tenha uma **tabela** como data source — **Projection** ou **Aggregation** node. Não pode ser function nem view.
  - Nas **Properties** desse nó: marque **Partition Local Execution**.
  - Opcional: uma **coluna de particionamento** → uma partição (thread) por **valor distinto** (ex.: COUNTRY → uma thread por país). Escolha colunas com distribuição **uniforme**.
  - Sem coluna definida → usa as regras de **partição da tabela** (se particionada).
- **Fim do bloco:** um **Union node** com **apenas uma** fonte visível (as várias threads não aparecem no editor). Não pode combinar outras fontes nele. Marque **Partition Local Execution** nas Properties.
- Ex.: Projection particiona por Country → Projection `Currency_convert_In_Parallel` roda em 3 partições (India, France, Iceland) → Union junta.

### Restrições
- O bloco **sempre começa** com nó cuja data source é **tabela** (local ou **synonym** para tabela externa). **Não** pode ser calculation view nem table function.
- **Só um bloco de paralelização por query** — não dá para parar e iniciar outro (na mesma view ou na pilha) nem **aninhar** blocos.
- Pode haver **vários nós de início** num mesmo bloco (com colunas de partição diferentes); todas as threads terminam num **único union** final.
- O HANA ainda aplica paralelização adicional dentro das partições lógicas.

### Monitorar
- Calculated column com a expressão column engine **`partitionid()`** dentro do bloco → mostra o número da partição de cada thread.
- Trace: adicione `WITH PARAMETERS ('PLACEHOLDER' = ('$$CE_SUPPORT$$',''))` ao final da query.
- **SQL Analyzer:** os nós paralelizados aparecem **duplicados** conforme o número de partições.
- **Cuidado:** não abuse — **over-parallelizing** prejudica a performance geral e outros processos.

---

## Lição 14.2 — Partitioning Tables

- Em tabelas **column store**, os dados já são separados por colunas (divisão **vertical**). **Table partitioning** divide as **linhas** em blocos (divisão **horizontal**). Ex.: partições por Year (2020, 2021, 2022).
- **Só se aplica a column store tables** (não row store) — por isso recomenda-se column store para grandes volumes consultados.
- Criar partições é normalmente responsabilidade do **administrador** do SAP HANA Cloud: tipos **hash**, **round robin**, **range**; também **sub-partitions**; monitoramento e ajustes.
- O **modelador** fornece informação sobre **como as queries acessam os dados** para que as partições sejam bem definidas.

### Motivos para particionar (complemento)
*(A seção do curso é apresentada em vídeo; resumo dos motivos clássicos documentados pela SAP.)*
- **Limite de ~2 bilhões de linhas** por tabela/partição não particionada em column store → particionar supera esse limite (**é a limitação cobrada no quiz**).
- **Partition pruning:** queries com filtro na coluna de partição leem só as partições relevantes.
- **Paralelismo:** partições processadas em paralelo.
- **Delta merge** mais leve (por partição) e melhor gestão de memória.
- **Distribuição** de carga entre hosts (cenários scale-out).

---

## Pontos-chave para a prova (Unit 14)
- **Partition Local Execution:** início em Projection/Aggregation com **tabela** como fonte; fim em **Union** com uma única fonte.
- **Apenas 1 bloco de paralelização** por query; sem aninhamento.
- Coluna de particionamento opcional (senão usa a partição da tabela).
- `partitionid()`, `$$CE_SUPPORT$$`, SQL Analyzer para monitorar.
- Partitioning = **horizontal**, só **column store**; supera o limite de **número de linhas** numa column-store table.

---

## Quiz oficial da Unit 14 — com gabarito
1. **Quantos blocos de paralelização numa calculation view?** → ✅ **1**.
2. **Que limitação o table partitioning supera?** → ✅ **The number of rows in a column-store table**.

---

## Glossário
- **Parallelization / Thread:** execução paralela / linha de processamento.
- **Partition Local Execution:** flag de início/fim do bloco paralelo.
- **Table partitioning (hash, round robin, range):** divisão horizontal de tabela.
- **Over-parallelizing:** excesso de paralelização.
