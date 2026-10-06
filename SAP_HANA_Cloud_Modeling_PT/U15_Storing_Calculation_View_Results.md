# Unit 15 — Storing Calculation View Results (Static Cache, Snapshots e MDS Cubes)

**Foco da unit:** três formas de **armazenar resultados**: **Static Cache**, **Snapshots** e **MDS Cubes**.

---

## Lição 15.1 — Implementing Static Cache to Improve Performance

### Conceito
- Guardar o resultado de uma calculation view em **cache** para acelerar queries seguintes. Útil com cenários complexos/grandes volumes. Reduz tempo de resposta **e** consumo de **CPU e memória**.
- Ex.: Query A às 9:10 busca no banco e grava no cache; às 10:30 lê do cache, muito mais rápido.
- Se o cache **não puder** ser usado (query pede outro data set), a query **não falha** — apenas faz o processamento completo.
- Aplique cache a dados **muito processados e reduzidos** (agregações/filtros) → **nós/views do topo**, não a dados brutos.
- **Cache só funciona em views que NÃO checam analytic privileges.** Portanto, defina analytic privileges **só na view do topo** da pilha e o cache na view **mais alta possível abaixo** dela → os privilégios são checados sobre os dados em cache.

### Ajuste fino (aba **Static Cache** em View Properties)
- **Colunas:** cache por coluna; queries que usam um **subconjunto** das colunas cacheadas são atendidas. Recomenda-se cachear só colunas frequentes. **Se nenhuma coluna for especificada, todas são cacheadas.**
- **Filtros:** o cache é usado se o filtro da query for **exatamente** o do cache ou **reduzir mais** os dados.
- **Retention Period:** idade máxima (em **minutos**) dos dados em cache antes de forçar refresh. Ainda **não** existe invalidação automática quando novos dados chegam às tabelas — só expiração por tempo.

### Pré-requisitos
- **Enable Cache** marcado.
- A view pode ser **unfolded** (totalmente traduzível em SQL; verificar com **Explain Plan**).
- **Sem cálculos dependentes de granularidade** (o cache fica no nível da primeira query, o que pode gerar resultados errados depois).

### Invocação
O cache é **desligado por padrão** e só é considerado quando invocado por um hint **RESULT_CACHE**:
- Na query: `SELECT ... WITH HINT (RESULT_CACHE)`
- **Database hint** na view do topo que consome a view cacheada (Execution Hints).
- Parâmetro de configuração: `indexserver.ini -> [result_cache] -> enabled`.
- Se os pré-requisitos não forem atendidos, dá para usar **Force** — mas teste muito.

---

## Lição 15.2 — Creating Snapshots

- **Snapshot** = "fotografia" dos dados em pontos no tempo → guardar **histórico**, ou ter performance sem precisar de tempo real (ex.: snapshot da manhã).

### Processo
1. **Deploy** da view gera a **snapshot table**.
2. São geradas **procedures** para **truncar** a tabela ou **inserir** dados (e criar/dropar a tabela).
3. Execute o snapshot **manualmente** ou **agendado**.
4. Consulte a snapshot table para dados **históricos**.
5. A view continua disponível para dados em **tempo real**.

### Configuração
- **View Properties → aba Snapshots**: crie uma **query sobre a view**; o resultado dela é gravado na snapshot table.
- **Security mode** das procedures: **DEFINER** (quem pode executar a procedure consegue usar) ou **INVOKER** (precisa dar acesso à snapshot table).
- **Cuidado:** chamar o INSERT duas vezes sem truncar gera **duplicatas**.
- **Create Snapshot After Deployment Automatically:** provisiona no deploy (por padrão trunca e insere de novo a cada deploy). **Keep Snapshot During Re-deployment:** não apaga entre deploys.

### Uso
- Selecione da snapshot table para histórico; use a view para tempo real.
- **Interface view** (gerada): faz **union** da snapshot table com a view e usa um **input parameter** para escolher a fonte.

---

## Lição 15.3 — Defining MDS Cubes

### Motivação
- Ex.: modelo no **SAP Analytics Cloud (SAC)** em que usuários fazem drill-downs diferentes (parceiro/canal/mês vs. produto/canal/data). Uma view única e complexa consome muitos recursos; o static cache demora no primeiro acesso e pode não atender drill-downs variados.
- **MDS cube (multidimensional services cube):** armazena dados **pré-processados/pré-agregados** num formato que atende vários drill-downs rapidamente.

### Conceito
- Definido para uma calculation view específica; é um **submodelo** (subconjunto de measures + attributes). No SAC, inclua todos os campos necessários (o modelo SAC vê **só** os campos do MDS cube).
- **Só para views de Data Category CUBE.**
- Pode haver **vários** MDS cubes por view. O **deploy** gera os cubes **sem dados**; depois de carregados, dados pré-agregados ficam numa estrutura **multidimensional, tipo star-join**, otimizada para OLAP (como um "cache persistido predefinido", mas não plano).
- **MDS hints** opcionais — use só se orientado por suporte.
- **MDS Cube ≠ MDS Cube Cache.**
- Dados refletem o momento da carga e **só o que o usuário que carregou podia ver**. **Não atualiza automaticamente** → precisa **recarregar**. **Não serve para tempo real.**

### Comparação
- **Static cache** pode ser mais rápido, mas a query precisa casar **exatamente**. MDS cube atende **mais queries**, faz pós-cálculos (restrições, ranking, fórmulas, exception aggregation do SAC) e agrega rápido. MDS cube é **mais rápido que snapshot**.
- Estrutura acessível por ferramentas baseadas em **metadados MDS** (SAC via **InA**), **não por SQL** → SQL Analyzer/Explain Plan não mostram uso do MDS cube; MDS cubes não aparecem no Database Explorer.
- **Generate Document** no BAS inclui informações dos MDS cubes.

### Gestão
- Procedure built-in **`MANAGE_MDS_CUBE('<JSON>', ?)`** (entrada e saída JSON) para metadados, **load/reload**, **delete**.
  ```json
  { "Cube": { "Command": "Reload",
      "Target": { "DataSource": { "SchemaName": "<Schema>", "ObjectName": "<MDS Cube Name>" } } } }
  ```
- Sucesso → estado **Ready**. Durante reload, o cube continua consultável com dados antigos. Monitorar em **`_SYS_EPM.MDS_METADATA`**.
- **Redeploy** da view apaga os MDS cubes e dados por padrão. Desde **2025 QRC4**: opção **Keep MDS Cube During Deployment** (só se as mudanças mantêm o cube carregável; senão o deploy falha).
- **2026 QRC1:** **incremental loading** — defina um attribute como **Incremental Column** (valor sempre crescente, ex.: `ORDER_TIMESTAMP`); carrega só registros com valor maior que o último; comando **Update** no `MANAGE_MDS_CUBE`.

### Limitações
- **Sem calculated measures** (faça no SAC).
- Suporta **variables**, mas **não input parameters** (input parameter mandatory sem default → deploy falha; converta para variables).
- **Joins com 2+ colunas de join** não suportados.
- Tipos suportados: inteiros, floats, DECIMAL, BOOLEAN, VARCHAR/NVARCHAR/CHAR/NCHAR, DATE/DAYDATE/TIMESTAMP — **não VARBINARY**.
- Features não suportadas desativam a aba **MDS Cubes** (com lista do que impede); se já houver MDS cube, essas features ficam inativas.

### Autorizações
- **`GRANT EXECUTE ON MANAGE_MDS_CUBE TO <DB_USER>;`**
- **CREATE ANY** no schema HDI da view; **SELECT** na fonte do cube.
- **Analytic privileges** da view se aplicam automaticamente aos MDS cubes; os privilégios dos usuários devem conter só colunas presentes no cube.

---

## Comparativo rápido

| | **Static Cache** | **Snapshot** | **MDS Cube** |
|---|---|---|---|
| Objetivo | Performance em queries repetidas | Histórico / foto dos dados | Performance multidimensional (SAC) |
| Atualização | Expira por **retention period** (min) | Procedures INSERT/TRUNCATE (manual/agendado) | **Reload** / incremental via MANAGE_MDS_CUBE |
| Acesso | Transparente via hint RESULT_CACHE | SELECT na snapshot table / interface view | SAC via **InA** (não SQL) |
| Tempo real | Não (enquanto válido) | Não | Não |

---

## Pontos-chave para a prova (Unit 15)
- Static cache → **melhorar performance**; requer Enable Cache + unfolding + sem granularity tracking; invocado por **RESULT_CACHE**; não em views com analytic privileges.
- Snapshot → **guardar resultado de uma query** sobre a view; **performance em dados históricos**; procedures geradas; DEFINER/INVOKER.
- MDS Cube → só **CUBE**; **SAP Analytics Cloud via InA**; persistência otimizada multidimensional (**True**); sem input parameters/calculated measures.

---

## Quiz oficial da Unit 15 — com gabarito comentado
1. **Por que usar snapshot?** (2) → ✅ **To store the results of a query on the calculation view**; ✅ **To improve performance when querying historical data**.
2. **Front-end que se beneficia de MDS cube?** → ✅ **SAP Analytics Cloud (via InA interface)**.
3. **Benefício do static cache?** → ✅ **Improve calculation view performance**.
4. **MDS Cube é otimização de performance com persistência otimizada para queries multidimensionais — V/F?** → ✅ **True**.

---

## Glossário
- **Static cache / RESULT_CACHE / Retention period.**
- **Snapshot table / Interface view / DEFINER-INVOKER.**
- **MDS cube / InA (Information Access) / MANAGE_MDS_CUBE / Incremental Column.**
- **SAC:** SAP Analytics Cloud.
