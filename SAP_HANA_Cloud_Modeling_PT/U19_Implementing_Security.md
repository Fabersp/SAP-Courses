# Unit 19 — Implementing Security in SAP HANA Modeling (Segurança)

**Foco da unit:** **Analytic Privileges** (attribute, SQL expression, dynamic; hierarchy nodes; nested views), **Design-Time Roles** (`.hdbrole`, `.hdbroleconfig`, `.hdbgrants`) e **Data Masking**.

---

## Lição 19.1 — Defining Analytic Privileges

### Conceito
- **Analytic privileges** controlam o acesso a dados em calculation views **filtrando linhas** pelos valores de um ou mais **attributes** (row-level security).
- Evitam criar cópias da mesma view com filtros diferentes. Ex.: gerentes regionais usam a mesma view, cada um vê só sua região.
- Um analytic privilege pode ser atribuído a **vários usuários** com a mesma necessidade.

### Passo a passo (end-to-end)
1. Criar source file **`.hdbanalyticprivilege`** (New File → nome com extensão → Rename/label).
2. Atribuir as **calculation views** a proteger (qualquer tipo) — lista **Secured Models**.
3. Escolher o **tipo de restrição** e defini-la.
4. Configurar as views para **checar** analytic privileges: propriedade **Apply Privileges = SQL Analytic Privileges**.
5. **Deploy** do analytic privilege.
6. Atribuir o analytic privilege a uma **role**.
7. Atribuir a role ao **usuário**.

### Tipos de restrição (exclusivos — um por analytic privilege)

| Tipo | Como usar | Exemplo |
|---|---|---|
| **Attribute** | Editor de restrições: selecione attributes das views e defina valores | REGION: EMEA; YEAR: Between 2015 e 2017 |
| **SQL Expression** | Expressão SQL **estática** com attributes e valores | `("REGION"='EMEA' AND "YEAR"='2015')` |
| **Dynamic** | **Procedure** que retorna dinamicamente uma expressão SQL (como um WHERE) | `P_DYNAMIC_AP_FOR_REGION` |

### Restrição Attribute — operadores
- **Between**, **ContainsPattern** (com `*`), **=, <=, <, >, >=**, **IsNull / Is Not Null**, **In**.
- Todos (exceto IsNull/Is Not Null) aceitam string vazia `" "`.
- **Só colunas do tipo Attribute** (não Measure).
- Se uma DIMENSION view tem **parent-child hierarchy habilitada para SQL**, pode restringir por **nó da hierarchy**.

### Combinação de restrições
- **Mesma coluna:** várias restrições → **OR**.
- **Colunas diferentes** no mesmo analytic privilege → **AND**. Ex.: YEAR=2017 e COUNTRY=US → só dados com **YEAR=2017 AND COUNTRY=US**.
- **Vários analytic privileges** para a mesma view e usuário → combinados com **OR** → o usuário vê **mais** dados (YEAR=2017 **OR** COUNTRY=US).

### Validity period
- Restrição Attribute pode valer só por um período — baseado na **data da consulta (CURRENT_DATE)**, **não** nos attributes de tempo da view.
- Em SQL Expression: adicione `(CURRENT_DATE BETWEEN '...' AND '...') AND <expressão>`.

### SQL Expression
- Para lógica que o tipo Attribute não expressa (ex.: precedência AND/OR). Ex.: `("COUNTRY" = 'US' OR "YEAR" <= '2016')`.
- **Dica:** comece com Attribute e **converta** para SQL Expression (código gerado). **O contrário não é possível.**

### Dynamic Analytic Privilege
- Condições determinadas **em runtime** por usuário → abordagem **escalável**: **o mesmo analytic privilege serve vários usuários com acessos diferentes**.
- Procedure retorna, por exemplo, `("COUNTRY"='US')` para User1 e `("COUNTRY"='UK' OR "COUNTRY"='FR')` para User2.
- **Propriedades obrigatórias da procedure:**
  - **Read-only**
  - Security mode **DEFINER**
  - **Sem input parameters**
  - **Um único output escalar** do tipo **VARCHAR(256)** ou **NVARCHAR(256)**

### Erros de deploy (checagem de dependência para evitar views desprotegidas)
- View com check de SQL analytic privileges ativado, mas **nenhum** analytic privilege a lista em Secured Models.
- Analytic privilege cujas views **não** têm Apply Privileges = SQL Analytic Privileges.
- View com check **desativado**, mas ainda listada em algum analytic privilege.
- **Melhor prática:** buildar analytic privileges e views **juntos**.
- Após deploy, as views **não podem ser vistas** até o privilege ser concedido (via **design-time role**).

### Restrição por nó de hierarchy
- Usuário com acesso a um nó vê todos os **descendentes** (ex.: nó NA → todos os países da América do Norte).
- Passos: habilitar a view para check de analytic privilege **e** SQL access a hierarchies → criar analytic privilege → Attribute restriction → **Hierarchical Privilege** → escolher hierarchy e nó.
- **Suportado** em **CUBE with Star Join** com check de SQL analytic privilege e SQL access a hierarchy habilitados, e **só para parent-child hierarchies**.

### Segurança em views aninhadas (nested)
Para cada view: o usuário tem **SELECT** na column view? A view checa analytic privileges? O usuário tem analytic privilege para ela?
- **Object privileges:** basta **SELECT na view do topo** — não precisa em views/tabelas inferiores.
- **Analytic privileges:** aplicados em **toda a hierarquia**. Se alguma view da pilha checa analytic privileges e o usuário **não** tem privilégio para ela → **nenhum dado** (não autorizado).
- O output de uma view inferior define o **máximo** que as superiores podem mostrar.
- Analytic privilege **não concede SELECT implícito** — o usuário sempre precisa de **SELECT explícito**.

---

## Lição 19.2 — Defining Roles

### Conceitos de segurança
- Usuários podem ser **donos** de objetos. **Role** = coleção de privilégios, concedida a usuários ou outras roles (**nested roles**). **Privilege** = permite operações em objetos.
- Recomenda-se **gerenciar autorizações via roles** (privilégios diretos ao usuário = muita manutenção).
- Todos os privilégios (diretos e indiretos) são **somados**.
- **Não existe "deny"** — todos os privilégios concedem acesso. A checagem para assim que encontra os privilégios necessários.
- Há **roles predefinidas** (algumas são templates).

### Dois tipos de role
- **Catalog Roles:** runtime puro (sem source file), criadas via **SQL** ou **SAP HANA Cockpit**; privilégios via `GRANT`.
- **Design-Time Roles:** source files no HDB module, extensão **`.hdbrole`**; privilégios concedidos ao **deployar**. **Recomendadas** para acessar objetos locais do container.
- **Uma role por arquivo `.hdbrole`**; role ID (com namespace) única no HDB module.

### `.hdbroleconfig`
- O `.hdbrole` **não pode** conter nomes reais de schema, só **referências lógicas** (ex.: `Ref1`), resolvidas no **`.hdbroleconfig`** → nomes de schemas externos mantidos num só lugar.
- Pode criar manualmente ou gerar pelo editor do `.hdbrole`.
- **Role Editor** abre por padrão ao criar um `.hdbrole`.

### Acesso a schema externo e `.hdbgrants`
- Administrador define **user-provided service** com usuário técnico; o projeto decide quais autorizações vão para quais roles. Para calculation views, o principal é **SELECT** nas fontes.
- **`.hdbgrants`** em três níveis:
  1. Nome do **user-provided service**.
  2. Usuários: **`object_owner`** (dono dos objetos do container) e **`application_user`** (usuários ligados aos módulos da aplicação).
  3. Conjunto de privilégios (sintaxe parecida com `.hdbrole`).
- Um `.hdbgrants` pode listar autorizações de **vários** user-provided services.
- Ex.: SELECT só para object_owner permite criar o synonym, mas sem SELECT para application_user **não** se consegue ver o conteúdo da tabela.

---

## Lição 19.3 — Masking Sensitive Data

### Column Masking
- Define uma **máscara** em colunas sensíveis: só usuários autorizados veem o valor real; os demais veem parte/todo **oculto** (preview ou front-end).
- **Como:** Semantics → aba **Columns** → selecionar coluna → ícone **Data Masking** → expressão **SQL**.
- Tipos suportados: **VARCHAR, NVARCHAR, CHAR, SHORTTEXT**. Funciona com tabelas **ROW e COLUMN**.
- Máscara = string constante (`XXX-XXX-XXX`) ou expressão SQL com funções de string. Ex.:
  ```sql
  LEFT(credit_card,4) || '-XXXX-XXXX-' || RIGHT(credit_card,4)
  ```
  `1111-2222-3333-4444` → `1111-XXXX-XXXX-4444`.

### Privilégio UNMASKED
- Sem o privilégio **UNMASKED**, o usuário vê a máscara. Normalmente incluído numa role do HDB module.
- **Nível schema** (todas as views do container; também tabelas e SQL views):
  ```json
  { "role": { "name": "db::UNMASK_ENTIRE_SCHEMA",
      "schema_privileges": [ { "privileges": ["UNMASKED"] } ] } }
  ```
- **Nível objeto** (objeto específico, com tipo e nome com namespace):
  ```json
  { "role": { "name": "db::UNMASK_EMPLOYEES_PAYMENT",
      "object_privileges": [ { "type": "VIEW", "name": "db::CVD_EMPLOYEES_PAYMENT", "privileges": ["UNMASKED"] } ] } }
  ```
- **UNMASKED não substitui SELECT** (SELECT continua obrigatório).

### Mask Mode (desde QRC 4/2021)
- **Default:** máscara baseada no usuário que **chama** a view com a máscara. Em views inferiores, executadas com privilégios do **Object Owner** (que sempre tem unmasked) → dados não mascarados nas inferiores.
- **Session User:** máscara baseada no **usuário da sessão** que executa a query (relevante em views inferiores de uma pilha).
- Configuração **global da view**; no topo de uma pilha, o mask mode da view do topo **não sobrescreve** o das inferiores.

---

## Pontos-chave para a prova (Unit 19)
- **Analytic privilege** = filtro de **linhas** por attributes; `.hdbanalyticprivilege`; Apply Privileges = **SQL Analytic Privileges**.
- Tipos: **Attribute**, **SQL Expression**, **Dynamic (procedure)**; Attribute → SQL possível, inverso não.
- Mesma coluna OR / colunas diferentes AND / vários privileges OR.
- **Dynamic** = reutilizável para vários usuários com acessos diferentes; procedure read-only, DEFINER, sem input, output VARCHAR/NVARCHAR(256).
- Nested: SELECT só no topo; analytic privilege necessário em toda view que checa.
- **Design-time roles** (`.hdbrole`) recomendadas; `.hdbroleconfig` para schemas.
- **Masking** = esconder caracteres; **UNMASKED** para ver; Default vs Session User.

---

## Quiz oficial da Unit 19 — com gabarito comentado
1. **O que é data masking?** → ✅ **Obscurecer valores de coluna escondendo parte ou todos os caracteres com caracteres substitutos.**
2. **Abordagem recomendada para roles que acessam objetos locais do container?** → ✅ **Design-time Roles**.
3. **O que é um dynamic analytic privilege?** → ✅ **Um analytic privilege reutilizável que pode servir a vários usuários que precisam acessar dados diferentes.**

---

## Glossário
- **Analytic privilege / Secured Models / Apply Privileges.**
- **Attribute restriction / SQL Expression / Dynamic (procedure-based).**
- **Validity period / CURRENT_DATE.**
- **Catalog role / Design-time role (.hdbrole) / .hdbroleconfig / .hdbgrants.**
- **Object privilege (SELECT) / System privilege / Schema privilege.**
- **Data masking / UNMASKED / Mask Mode (Default, Session User).**
