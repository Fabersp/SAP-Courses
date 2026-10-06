# Unit 17 — Working in a Modeling Project (Estrutura do Projeto, Deploy e Gestão de Conteúdo)

**Foco da unit:** **workspace / project / module / folder**, **HDB module**, **namespace** (`.hdinamespace`), **HDI** e **HDI containers**, `.hdiconfig`, view **SAP HANA PROJECTS**, **deploy** e erros, import/export, copiar/renomear/mover/apagar, renomear colunas, properties file e **deprecate**.

---

## Lição 17.1 — Project Structure

### Workspace, Projects, Modules, Folders
- **Workspace:** estrutura de arquivos onde você trabalha em um ou mais projetos. Cada usuário tem o seu (pode criar outros).
- Projetos com **nomes diferentes**, cada um numa pasta raiz. Pode abrir um projeto por vez ou criar um **multiroot workspace**.
- Hierarquia: **Workspace → Project → Module → Folders → Objects**. (Diretamente abaixo do projeto ficam os **modules**.)
- Ao criar projeto: escolhe-se um **template**. Para modelagem, o mais simples é **SAP HANA Database**.
- **Modules:** tipos diferentes (Java, HTML, **database**). Pode ter vários, mas para banco geralmente basta **um**. Dê nomes significativos; não crie módulos vazios.
- **Folders** dentro do módulo organizam os source files (ex.: tables, calculation views).

### HDB Module (HANA Database module)
- Contém todos os **design-time files** dos objetos de banco que devem ser deployados com a aplicação. É uma pasta (ex.: `db`).
- Contém a pasta **`src`** (gerada por padrão — onde fica o conteúdo de modelagem) e conteúdo do deploy HDI (`.env`, pasta `node_module` com o **node.js HDI Deployer**).
- É referenciado no descritor do projeto **`mta.yaml`** (com settings como database id ou schema name, nem todos obrigatórios).
- **Principais artifacts:** Tables, Calculation views, Functions, Procedures, Analytic privileges, Synonyms.
- **Persistence layer:** os dados podem ficar numa persistência **definida pela própria aplicação** (dentro do HDB module) ou **externa** (ex.: schema clássico).

### Object identifier e Namespace
- Runtime object: **`<Namespace>::<Runtime Object Name>`** — nome **obrigatório**; namespace **opcional** (organização lógica). Ex.: `HC::CVC_SalesOrders`.
- Os identificadores são **sempre especificados nos design-time objects**.
- **Regras do namespace:**
  - Opcional; um HDB module pode ter nenhum ou vários namespaces.
  - **Igual para todos os objetos da mesma pasta**.
  - Deve estar **explícito** no design-time file e corresponder ao da pasta (explícito ou herdado das pastas pai).

### Arquivo `.hdinamespace`
- Em `src` (criado automaticamente) ou em qualquer subpasta. Padrão: `"<application_name>.<HDB_module_name>"`.

| Elemento | Valores | Padrão |
|---|---|---|
| **"name"** | namespace válido ou `""` (sem namespace) | `"<application_name>.<HDB_module_name>"` |
| **"subfolder"** | **"append"** (acrescenta nomes das subpastas) ou **"ignore"** | **"append"** |

- Permite convenções como `db.data.table` para tabelas e `db.models.dim` / `db.models.cube` para views.
- **Cuidado:** mudar o `.hdinamespace` afeta imediatamente os identificadores, mas as novas regras só valem no container **depois do deploy do próprio `.hdinamespace`** — antes disso o deploy dos objetos falha.
- **Recomendação:** definir as regras de namespace **no início** do projeto.

### Nome do objeto

| Tipo | Objetos por arquivo | Regra |
|---|---|---|
| Calculation views, table functions, procedures, analytic privileges | **Um** objeto core por arquivo | Nome definido no conteúdo e **deve coincidir** com o nome do arquivo (sem extensão) — recomendado |
| Synonyms, table data import, roles | **Um ou vários** por arquivo | Nome só no conteúdo, sem vínculo com o nome do arquivo |

- Uma calculation view gera uma **column view core** + column views filhas (attributes, hierarchies…).

---

## Lição 17.2 — Deploying Models

### SAP HANA Deployment Infrastructure (HDI)
- Conjunto de serviços para fazer deploy de objetos de banco em **HDI containers** isolados.
- **HDI container** = schema de banco com recursos extras (**owner dedicado** + metadados) → "**smart schema**".
- HDI gerencia **dependências**, a **ordem** de deploy e upgrades.
- **Todo o ciclo de vida** (criar, alterar, apagar) é feito **exclusivamente pelo HDI** → você **não** cria tabelas/views com **DDL SQL** direto no container.

### Data sources de calculation views
- Devem existir como **design-time files** no container do HDB module:
  - Objetos "reais" (table, view, table function, virtual table…) com source file no módulo.
  - **Synonyms** para objetos em outros schemas.
- View que referencia objeto criado direto por SQL **não builda**: `The file requires "db://HA300::TABLE_SQL" which is not provided by any file`.

### Propriedades-chave dos HDI containers
- Criado automaticamente no **primeiro deploy** do database module.
- Gera um **schema** no momento da criação; objetos são deployados nele.
- Source files escritos **schema-free**.
- **Referências diretas** a objetos fora do container **não são permitidas** → use **synonyms**.

### `.hdiconfig`
- Arquivo **obrigatório** em `src`, **sem nome** (só a extensão `.hdiconfig` — se nomear, o build falha).
- **Liga cada tipo de source file ao build plug-in** correto (ex.: suffix `hdbcalculationview` → plug-in `com.sap.hana.di.calculationview`).
- Normalmente **um por HDB module**.
- **`minimum_feature_version`:** impede build com versão de plug-ins inferior à exigida (ex.: feito com 1004 não faz deploy onde há 1002).
- Ao importar um projeto, ele traz seu `.hdiconfig`; para usar features novas, **ajuste** a feature version.

### View SAP HANA PROJECTS
- Materializa os HDB modules. Parte superior: estrutura de pastas; deploy em nível de **arquivo, pasta ou módulo inteiro**.
- **Não** dá para mover/copiar/apagar arquivos ali (só abrir).
- **Pending Deployment** marca o que precisa de deploy. Para **apagar** runtime objects de arquivos deletados, faça deploy da **pasta** que os continha.
- Parte inferior: **Database Connections** (HDB module ↔ HDI containers ou serviços externos). Um projeto com HDB module precisa de pelo menos **um HDI container**; você pode **bind** a um novo ou existente.

### Database Explorer e usuário técnico
- Ao adicionar o container no Database Explorer, o acesso é feito por um **technical SBSS user (Service Broker Security Support)**, criado de forma transparente. Se o container consome schema externo, esse usuário também precisa de autorização.

### Deploy
- Função **Deploy** da view SAP HANA PROJECTS. Usa o **HDI container service** (serviço **Cloud Foundry**).
- **Bind** do HDB module a um HDI container service cria o schema (se não existir).
- Escopos: **HDB module inteiro**, **subpasta**, **um ou vários objetos**.
- Não confunda com **`cf deploy`** do Cloud Foundry (aplicação MTA inteira — usado para levar para QA/PROD).
- Sempre há **checagem de dependência end-to-end** (recursiva), mesmo deployando um único objeto.

### Erros comuns de deploy
- Objeto do qual outro depende **não definido** ou **ainda não deployado**.
- Mesmo runtime object definido por **vários** design-time files.
- **Inconsistência de namespace** entre o arquivo e a pasta.
- Serviço para schema externo não definido/indisponível no space.
- **Synonyms** para schema externo ainda não deployados.
- Erro específico do objeto (ex.: **CUBE sem measure**).
- Inconsistência com objeto referenciado (ex.: nome de coluna diferente).

### Contornando uma view que não builda
- **Exportar** a view para lugar seguro e apagá-la do workspace (depois reimportar); ou
- Adicionar **`.txt`** à extensão (`SALES.hdbcalculationview.txt`) — vira texto simples, sem checagem; para desfazer, remova `.txt`.

---

## Lição 17.3 — Manage Modeling Content

### Export / Import
- Útil para versões, sandboxing, mover entre dev spaces, compartilhar (ex.: definições de synonyms).
- **Export:** **File → Download** ou clique direito → Download. Arquivo único = baixado como está; **pasta** = **.tar**; vários itens = um .tar. Projeto inteiro: clique direito na área em branco.
- **Import:**
  - **Drag and drop:** arquivos compactados **não** são extraídos (extrair via terminal).
  - **Upload Files:** **não** extrai.
  - **Import / Import Project:** **.zip e .tar são extraídos**.
- Projeto inteiro (ex.: `proj1.tar`): abra antes a pasta `projects` para que vire `projects/proj1`; depois abra como workspace.

### Regras para copiar, renomear, mover e apagar
- Num HDB module, a definição de um runtime object (`<namespace>::<object_name>`) **não pode existir mais de uma vez**.
- O namespace no arquivo deve corresponder ao da pasta.
- Deploy **checa dependências em todo o módulo**, mas só **deploya os arquivos selecionados**.
- Checagens valem para objetos já buildados + escopo do deploy; arquivo nunca buildado e fora do escopo é ignorado.

**Copiar:** na mesma pasta exige outro nome; o conteúdo não muda → **renomeie os runtime objects** dentro do arquivo (e ajuste namespace). Lembre que o editor continua mostrando o **original**.

**Mover:** mesma regra de namespace → sem impacto nas referências, mas o deploy precisa incluir a **pasta de origem** (para registrar a remoção). Namespace diferente → ajustar e fazer **impact analysis** antes.

**Renomear:** para o HDI = **apagar + criar**. Mantenha nome do arquivo = nome do runtime object (boa prática). Verifique dependências. Se renomear o arquivo mas não o runtime object e buildar só o arquivo → **erro** (nome duplicado); builde **no nível da pasta**.

**Apagar:** a exclusão só é reconhecida ao deployar o **módulo** ou a **pasta** que continha o arquivo (a pasta, mesmo vazia, é "preciosa").
- Antes de apagar um arquivo → **impact analysis**.
- Antes de apagar uma pasta → **deploy parcial da pasta**.
- **Nunca** apague objetos de um HDI container com **SQL**.

### Search/Replace
- **Ctrl+F / Ctrl+H** no editor; **Search view** (com padrões de inclusão/exclusão); **Find in folder**; busca **fuzzy** por nome de arquivo digitando na Explorer.

### Renomear colunas
- **Nó intermediário:** só **output columns** (aba Mapping → Properties → Name, ou aba Columns para várias). Se a coluna ainda não foi mapeada acima, o novo nome sobe; se já foi, a mudança é **local** (acima volta ao nome original).
- **Top node:** nomes usados por front-ends, SQL e outras views. Use **Rename and Adjust References** no Semantics. Desde **QRC 3/2023:** **Propagate Recursively in Consuming Views** (uma coluna por vez).
- Views expostas externamente exigem cuidado especial.

### Generate Properties File
- **Modeling → Generate Properties File**: arquivo com nome e descrição de colunas, input parameters e variables; após build do HDB module, valores vão para **`BIMC_DESCRIPTION`** → permite **tradução** para vários idiomas; ferramentas cliente leem essa tabela.

### Deprecate Calculation Views
- Semantics → **View Properties → General → Deprecation Option**:
  - **Deprecated and Queries Permitted:** queries continuam; o editor mostra **aviso** (ícone/tooltip) se um nó referencia view deprecated — aviso ao **desenvolvedor**.
  - **Deprecated and Queries Blocked** (QRC 1/2024): queries **bloqueadas**, com mensagem customizável em **Deprecation Note** + erro padrão `Instantiation of calculation model failed`.

---

## Pontos-chave para a prova (Unit 17)
- Abaixo do projeto: **Module**.
- Todo runtime object no container precisa de **source file** (HDI gerencia o ciclo de vida).
- Namespace: `.hdinamespace` (name / subfolder append|ignore); definir cedo.
- `.hdiconfig`: liga suffix → plug-in; sem nome; minimum_feature_version.
- Acesso fora do container → **synonym**.
- Deploy: módulo, pasta ou objetos; dependências checadas end-to-end.
- Deprecate = aviso ao **desenvolvedor** para não reutilizar a view.

---

## Quiz oficial da Unit 17 — com gabarito comentado
1. **O que aparece diretamente abaixo do projeto?** → ✅ **Module**.
2. **Runtime object num container sempre tem source file correspondente — V/F?** → ✅ **True**.
3. **Por que marcar deprecate?** → ✅ **Para avisar o desenvolvedor de que não deve usar a view** (não recomendada para reuso).

---

## Glossário
- **HDB module / src / mta.yaml.**
- **Namespace / .hdinamespace / Object identifier (`ns::name`).**
- **HDI / HDI container ("smart schema") / HDI Deployer / build plug-in / .hdiconfig.**
- **SAP HANA PROJECTS view / Pending Deployment / Bind.**
- **SBSS user:** usuário técnico do Database Explorer.
- **BIMC_DESCRIPTION:** tabela de descrições/traduções.
- **Deprecate.**
