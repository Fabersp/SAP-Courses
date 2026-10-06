# Unit 13 — Using Tools to Check Model Performance (Ferramentas de Performance)

**Foco da unit:** **Performance Analysis mode**, **Debug Query mode** e **SQL Analyzer** (HANA SQL Analyzer / plan file `.plv`).

---

## Lição 13.1 — Validating Calculation Views with Performance Analysis Mode

- O editor de calculation views pode **destacar configurações** que podem causar má performance.
- Ativado pelo botão de **velocímetro (speedometer)** na toolbar do editor.
- **Não precisa** fazer build/deploy da view para usar → reage em **design time**.
- Ao ativar, aparece a aba **Performance Analysis** em todos os nós **exceto o Semantics**. Some ao sair do modo.

### Informações mostradas
- Tipo de tabela (**row** ou **column**).
- **Join types** (inner, outer…).
- **Join cardinality** (n:m etc.) e referential integrity / cardinality proposta.
- Se as tabelas são **particionadas**, como, e quantos registros por partição.

### Warnings
- **Tamanho das tabelas** (destaque para tabelas muito grandes).
- **Cardinalities ausentes** ou que **não batem** com os dados atuais.
- **Joins baseados em calculated columns**.
- **Restricted columns baseadas em calculated columns**.
- Ex.: tabela enorme → pensar em **particionar** e filtrar para processar só as partições necessárias.

---

## Lição 13.2 — Debugging Calculation Views with the Debug Query Mode

- Para examinar o **comportamento em runtime** quando a view é chamada por uma query: você inspeciona **cada nó** e vê o **SQL gerado**.
- Ao entrar no modo, é criada uma **query genérica** (todas as colunas, sem filtros) que simula um front-end. Você pode **editá-la** (remover colunas, adicionar filtros).
- Após executar, vê o SQL de **cada nó** e pode **executar o SQL por nó** para ver resultados intermediários. Ex.: ver um **WHERE** sendo adicionado ou como um **PLACEHOLDER** se comporta.
- Pode navegar até o **nível mais baixo** da pilha a partir do topo (não precisa abrir cada view separadamente).
- Muito útil para verificar **pruning** de colunas e de data sources (**union/join pruning**) com diferentes queries.
- **Reset** da debug query no Semantics node (botão ao lado de Execute).
- **Nota:** o Debug Query mode chama a **versão ativa (deployada)** da view. Mudanças não rebuildadas **não aparecem** → é preciso **deploy** antes.

---

## Lição 13.3 — Analyzing Executions with the SQL Analyzer

### Propósito
O Debug mostra o SQL, mas não a **performance real**. O SQL Analyzer responde:
- Tempo total de execução? Tempo de cada passo? Gargalos (bottlenecks)?
- Quantos registros por passo? Em que sequência? Quais operadores?
- Quanto foi **paralelizado**? Quais **engines**? Tudo foi **unfolded**?

- View do BAS: **HANA SQL Analyzer**. Requer a extension **SAP HANA Performance Tools** no dev space (adicionar com o dev space **parado**).
- Conjunto de views, tabelas e gráficos; aba **Overview** (tempo, memória, operadores, data usage, network…); **Plan Graph**; timeline de compilação e execução; tabelas usadas.
- Serve para qualquer SQL (views, **table functions** etc.).

### Como obter a query
- **Mais fácil:** na view **SAP HANA Database Explorer** do BAS → HDI container → clique direito na **column view** → **Generate SQL Statement**. O plan file abre direto no HANA SQL Analyzer.
- Na **Explorer view**: clique direito no arquivo da view → **SQL Editor**; ou **Data Preview** → aba Raw Data → **Edit SQL Statement in SQL Console**.
- No **Database Explorer externo**: HDI container → column view → **Generate SQL Statement**.
- Os dois últimos exigem **baixar** o plan file (`.plv`) e **subir** no BAS.
- Pode modificar a query ou criar do zero.

### Gerar o plan file
1. Coloque a query num **SQL Console** (**não precisa executar**).
2. **Analyze → Generate SQL Analyzer Plan File**; escolha prefixo do nome e **Save** (local fixo).
3. Se gerado no Database Explorer **embutido** → abre direto no HANA SQL Analyzer.
4. Se gerado em outra ferramenta → **Download** do `.plv` → **Upload** para uma pasta do projeto no BAS → HANA SQL Analyzer → **+** para adicionar → selecionar → ver **PLAN GRAPH**.

### Acessar plan files depois
- **Embedded DB Explorer:** pasta `/home/user/.vscode/data/User/globalStorage/sapse.hana-database-explorer/`.
- **External DB Explorer:** HDI container → **Catalog → Database Diagnostic Files → <DB Instance ID> → other** → baixar e subir no BAS.

### Privilégios para o SQL Analyzer
- Pode ser o `_RT` do container ou um usuário clássico; ambos precisam dos **SYSTEM privileges**:
  - **TRACE ADMIN**
  - **INIFILE ADMIN**

---

## Pontos-chave para a prova (Unit 13)
- **Performance Analysis mode:** design time, sem deploy; mostra join/cardinality, partições, tamanho de tabelas, joins em calculated columns → ajuda em **otimização de joins** e **necessidade de particionar**. *(Não mede performance de SQL nem precisão do resultado.)*
- **Debug Query mode:** SQL por nó, pruning; usa a **versão deployada** (precisa deploy).
- **SQL Analyzer:** extension **SAP HANA Performance Tools**; escrever query + **Generate SQL Analyzer Plan File** (não precisa executar nem baixar no embedded); privilégios TRACE ADMIN + INIFILE ADMIN.

---

## Quiz oficial da Unit 13 — com gabarito comentado
1. **Para usar Debug Query Mode é preciso fazer deploy antes — V/F?** → ✅ **True** (usa a versão ativa).
2. **O que se verifica no Performance Analysis Mode?** (2) → ✅ **Join definition optimization**; ✅ **Table partitioning need**.
3. **Passos necessários para visualizar um plan file a partir do Database Explorer embutido?** (2) → ✅ **Write an SQL query**; ✅ **Generate an SQL Analyzer plan file**. ❌ executar; ❌ baixar.

---

## Glossário
- **Performance Analysis mode:** modo de análise de performance em design time.
- **Debug Query mode:** modo de depuração da query por nó.
- **SQL Analyzer / Plan file (.plv) / Plan Graph:** analisador de execução / arquivo de plano / gráfico do plano.
- **Bottleneck:** gargalo.
- **TRACE ADMIN / INIFILE ADMIN:** system privileges necessários.
