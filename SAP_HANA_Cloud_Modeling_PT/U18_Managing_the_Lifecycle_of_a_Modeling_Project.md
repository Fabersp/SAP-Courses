# Unit 18 — Managing the Lifecycle of a Modeling Project (Ciclo de Vida do Projeto)

**Foco da unit:** criar projeto com HDB module, **schema name**, **HDI Cockpit**, acesso a dados externos (**user-provided service**, **.hdbgrants**, **.hdbrevokes**, **synonyms**, cross-container), **Git** e deploy de aplicação (**MTA / .mtar**).

---

## Lição 18.1 — Creating a Project

- Template usual: **SAP HANA Database Project**.
- O HDB module pode definir uma **persistência local**. Num **virtual data model** com calculation views, o HDB module contém os artifacts mesmo que os dados fiquem fora da aplicação.
- A maioria das propriedades do módulo vai para o **`mta.yaml`**, **exceto o namespace**, materializado no **`.hdinamespace`** em `src`.

| Setting | Descrição | Exemplo |
|---|---|---|
| Project Name | Nome do projeto | sales_analytics_project |
| Module Name | Único no `mta.yaml` | core-database-module |
| Type | **hdb** = tipo reservado para HDB module | hdb |
| Path | Local da raiz do módulo no projeto | db |
| Namespace | Opcional — prefixo dos objetos | sales |
| Schema Name | Opcional — nome do schema do HDI container | SAMPLE_SCHEMA |
| SAP HANA Database Version | SAP HANA Service (legado) ou HANA Cloud | — |

### Schema name
- Se **não** especificado, é gerado no deploy: **`<project_name>_<hdi_service_name_in_MTA>_<numeric_increment>`** (ex.: `MYPROJECT_HDI_DB_1`).
- Descobrir o schema: `select CURRENT_SCHEMA from DUMMY;` num SQL console conectado ao container.
- **É possível especificar** um schema no `mta.yaml`, nos parâmetros do serviço **hdi-container** (`config: schema: <YOUR_SCHEMA_NAME>`) → gera `<YOUR_SCHEMA_NAME>_<numeric_increment>`.
- Dá até para forçar o nome exato (parâmetro adicional), mas **não é recomendado** (conflitos entre projetos).

### HDI Cockpit
- Um banco tem muitos containers, com combinações de usuários e roles → difícil gerenciar.
- **HDI Cockpit (HDI Administration):** visibilidade de **todos os containers**, agrupamento em **container groups** e exibição de usuários/roles com acesso.

---

## Lição 18.2 — Enabling Access to External Data

### Princípio de isolamento
- Por padrão, o container tem acesso total aos **seus** objetos, mas **nenhum** acesso a outros schemas (outro container ou schema clássico). Pode-se habilitar.

### Acesso a schema externo (clássico) — passo a passo
1. **User-Provided Service** (criado no BAS): nome + **user/password** de um usuário com privilégios nos objetos externos. É adicionado automaticamente ao **`mta.yaml`** como resource do HDB module.
2. **Arquivo `.hdbgrants`**: o usuário do serviço concede (um subconjunto de) seus privilégios a:
   - **Object Owner:** usuário técnico **dono** dos objetos do container (não o desenvolvedor). Só ele altera/apaga objetos. No deploy, precisa de acesso aos objetos externos referenciados. Precisa poder **conceder** os privilégios (para incluí-los em roles).
   - **Application User:** usado para **bind** do container a aplicações; na modelagem, normalmente o **`..._RT`** (faz data preview etc.). Geralmente não precisa de grant option.
   - Conceda **só o necessário** (o usuário do serviço pode ser compartilhado com outros projetos).
3. **Synonyms** (`.hdbsynonym`): apontam para os objetos externos.

### `.hdbgrants` e `.hdbrevokes`
- `.hdbgrants` **não materializa** objetos no banco; remover um privilégio do arquivo e rebuildar **não revoga**.
- Para revogar: arquivo **`.hdbrevokes`** (mesma estrutura) listando o que revogar + remover do `.hdbgrants`.
- **Cuidado:** no deploy, `.hdbrevokes` é processado **antes** do `.hdbgrants` → o que estiver no grants **sempre** é concedido.
- **Prática comum:** criar **duas roles fora** do container (object owner e application user), concedê-las ao usuário do serviço e referenciar só essas roles no `.hdbgrants`. Mudanças nas roles refletem automaticamente, sem rebuild.

### Synonyms
```json
{
  "HA300::SALES_DATA": { "target": { "object": "SALES_DATA", "schema": "TRAINING" } },
  "HA300::CUSTOMER":   { "target": { "object": "CUSTOMER",   "schema": "TRAINING" } }
}
```
- Três parâmetros: **nome do synonym** (usado no container), **object** (nome real) e **schema** (onde está).
- Editável em texto ou no **synonym editor**.

### Cross-container access
- Semelhante, com diferenças:
  - **Não precisa** de user-provided service se o container externo está no **mesmo space** — adicione o serviço do container externo ao projeto (vai para o `mta.yaml`).
  - Crie **roles dentro do container externo** com os privilégios.
  - O `.hdbgrants` referencia **essas roles** (não privilégios de um usuário técnico).
  - Ainda precisa de **synonyms**.

---

## Lição 18.3 — Using Git to Manage Source Code

### Por quê
- Vários desenvolvedores: não sobrescrever trabalho, usar a versão mais recente, voltar a versões anteriores, desenvolvimento **paralelo** (ex.: patch da versão 2.1.0 enquanto desenvolve features).
- Benefícios: **backup** do código, **histórico completo**, **branching e merging**, **rastreabilidade** (ligar mudanças a ferramentas de projeto).

### Git em resumo
- **Distributed Version Control System (D-VCS)**; cada desenvolvedor tem cópia local do repositório (todo ou parte do histórico).
- Arquitetura entre local e central; repositórios compartilhados internos ou públicos (ex.: **GitHub**).
- Software livre (GNU GPL).
- **Não** é para code review nem **deploy automatizado** — para isso existem **Gerrit** (review) e **Jenkins** (CI/CD).

### Ciclo de vida dos arquivos
- Três áreas: **working directory**, **staging area (INDEX)**, **local repository**.
- Staging = o que entra no próximo **commit** (não faça stage do que não quer commitar).
- **Histórico** = sequência de commits (cada um aponta para o pai; guarda data, autor, arquivos afetados).
- **Branch** = série de commits; tecnicamente um **ponteiro** para um commit. Sempre há pelo menos uma (padrão **master**/main). Ex.: branch `Feature32` criada após C3.

### Git no BAS (CLI)
- Pré-requisitos: conta e repositório no GitHub.
```bash
git config --global user.email "git_email@company.com"
git config --global user.name "user_name"
git init                       # inicializa repo local
git add .                      # stage
git commit -m "First commit"   # commit local
git branch -M main             # branch main
git remote add origin https://<host>/<repo>
git push origin main
```
- Autenticação com **Personal Access Token (PAT)** em vez de senha (só **HTTPS**). Criar: GitHub → Settings → Developer settings → Personal access tokens → Generate (scope **repo**).
- Cache de credenciais: `git config credential.helper cache --timeout=...` / `git config credential.helper store`; limpar: `git config credential.helper ""`.
- Também há **UI** de Git no BAS: init → stage all → commit → push.

### Status de arquivos
- **A** = novo e staged; **U** = untracked/unstaged; **M** = modificado; **D** = deletado; **C vermelho** = conflito (merge/rebase); **C azul** = cópia. Pode stage, deixar unstaged ou **discard**.

### Commit
- Adiciona ao histórico com descrição. Não altera o working directory.
- Boa mensagem: assunto curto (≤50 caracteres, modo imperativo), linha em branco, corpo com o quê e por quê, link para issue (ex.: JIRA).
- **Amend:** substitui o último commit — **não** faça amend em commits já compartilhados.

---

## Lição 18.4 — Deploying an Application

### Deploy de objetos vs. deploy de aplicação
- **Objetos:** durante o desenvolvimento, pelo BAS, no HDI container do dev space (precisa de role de modelador no space).
- **Aplicação:** num **space alvo** (QA, PROD), em momento específico, **fora do BAS**; quem faz não precisa de role de desenvolvedor.
- Aplicações full-stack (HDB, HTML, NodeJS, Java…) = **MTA (multitarget application)**.

### Passos para QA/PROD
1. **Build** dos HDB modules.
2. **Build do projeto inteiro** → gera o arquivo **MTA archive (`.mtar`)**.
3. **Exportar** o `.mtar`.
4. **Deploy** do `.mtar` no ambiente alvo (entre 3 e 4: disponibilizar o arquivo, ex.: ferramenta de transporte).
- O BAS **não participa** do passo 4; não precisa criar/buildar o projeto no destino.

### Arquivos
- **`.mtar`**: tecnicamente um arquivo compactado; nome `<Application ID>_<version>.mtar`, na pasta **`mta_archives`**. Contém **`mtad.yaml`** (equivalente ao mta.yaml para deploy).
- **`.mtaext`** (opcional): extensão em que o administrador do space ajusta parâmetros para o ambiente alvo (nome do schema do container, nome real de user-provided service, de outro HDI container service).
- Gerar `.mtar`: terminal na **raiz do projeto** → **`mbt build`**.
- Nome do HDI container no deploy = nome do resource no `mta.yaml` (padrão `hdi-<HDB_module_name>`).

---

## Pontos-chave para a prova (Unit 18)
- Acesso a objetos externos → **Synonym** (com user-provided service + .hdbgrants).
- Object Owner vs Application User (`_RT`).
- `.hdbrevokes` antes de `.hdbgrants`.
- Cross-container: roles no container externo; sem user-provided service no mesmo space.
- **Git não automatiza deploy** (False).
- Deploy de aplicação gera **.mtar** (`mbt build`); `.mtaext` para ajustes do alvo.
- Schema name **pode** ser definido no mta.yaml (falso que não pode mudar).

---

## Quiz oficial da Unit 18 — com gabarito comentado
1. **Que artifact define acesso a objetos de schema externo?** → ✅ **Synonym**.
2. **Git é usado para automatizar o deploy de runtime objects — V/F?** → ✅ **False**.
3. **Tipo de arquivo gerado no deploy de uma aplicação completa?** → ✅ **.mtar**.
4. **O nome do schema do container é gerado e não pode ser mudado — V/F?** → ✅ **False** (pode ser definido no `mta.yaml`).

---

## Glossário
- **User-Provided Service / .hdbgrants / .hdbrevokes / .hdbsynonym.**
- **Object Owner / Application User.**
- **Cross-container access.**
- **HDI Cockpit / container groups.**
- **Git: working directory, staging/INDEX, commit, branch, merge, push, PAT.**
- **MTA / .mtar / mtad.yaml / .mtaext / mbt build.**
