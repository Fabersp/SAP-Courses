# Unit 1 — Preparing the Modeling Environment (Preparando o Ambiente de Modelagem)

**Curso:** Developing Data Models with SAP HANA Cloud (SAP Learning)
**Foco da unit:** conhecer a ferramenta **SAP Business Application Studio (BAS)**, os **Development Spaces**, como criar/importar projetos, navegar na interface e usar o **Joule**.

---

## Lição 1.1 — Getting Started with SAP Business Application Studio

**Objetivo:** descrever o SAP Business Application Studio e como ele é usado no desenvolvimento em SAP HANA Cloud.

### O que é o SAP Business Application Studio
- É uma **IDE (Integrated Development Environment)** **web-based**, usada para desenvolver todas as camadas de uma aplicação **full-stack**: interface (UI), camada de aplicação e camada de banco de dados (**database layer**).
- Suporta aplicações **cloud**, **on-premise** e **hybrid**, integrando tecnologias SAP e não-SAP.
- Um dos cenários suportados é **data modeling no SAP HANA Cloud** — o foco deste curso.
- É baseado no **Visual Studio Code** (projeto open-source da Microsoft). Por isso aceita **extensions / plug-ins** — SAP e terceiros fornecem vários para aumentar a produtividade.
- Oferece **editores de texto e gráficos** para criar **development artifacts**, além de uma **CLI (command line interface)** / terminal.
- Tem **suporte nativo a Git** para versionamento do código-fonte — os controles de Git ficam embutidos no BAS.

> **Atenção (prova):** o **Web IDE for SAP HANA** também pode ser usado para modelar no SAP HANA Cloud, **mas não é recomendado**: não terá novas features. **A ferramenta recomendada para data modeling no SAP HANA Cloud é o SAP Business Application Studio.**

### Como obter acesso ao BAS
- O BAS é uma aplicação web cujo back-end roda no **SAP BTP (Business Technology Platform)**.
- Você faz a **subscription** do serviço SAP Business Application Studio no nível do **BTP sub-account**, pelo **BTP Cockpit**.
- Antes de abrir o BAS no navegador há passos de dois tipos:
  1. Passos de **BTP Account / Subaccount** (ex.: subscription do serviço).
  2. Passos de **User membership e authorization** (atribuir ao usuário os **role collections** necessários).
- Concluídos esses passos, o BAS pode ser lançado pelo navegador.

---

## Lição 1.2 — Working with Development Spaces

**Objetivo:** criar um **Development Space** para o conteúdo de modelagem.

### Conceito
- No BAS, cada desenvolvedor tem seu próprio ambiente, formado por um ou mais **Development Spaces** (dev spaces).
- **Não confunda** Development Spaces com **Cloud Foundry spaces** — são coisas diferentes.
- Ao criar um dev space você escolhe o **tipo de aplicação** (ex.: *SAP HANA Native Application*) e as **extensions** (ferramentas).
- **Essencial para modelagem:** incluir a extension **SAP HANA Calculation View Editor**.

### Gerenciando Development Spaces
A **landing page** do BAS lista os dev spaces com:
- Nome do dev space
- Tipo de aplicação (kind of application)
- **Status:** `STOPPED`, `STARTING`, `RUNNING`, etc.
- Data de criação
- ID

- O dev space precisa estar **RUNNING** para você trabalhar nele. Quando está rodando, o nome vira um **hyperlink** para entrar.
- **Cuidado:** todo dev space **para automaticamente após um tempo ocioso (idle)** para poupar recursos. Sempre confira o status na landing page — especialmente se você salvou nos favoritos a URL do space (que contém o ID do space).

### Vários Development Spaces
- Dependendo da subscription do BTP, há limites de **quantidade máxima de spaces** e de **spaces rodando em paralelo**.
- **Conta SAP BTP Trial:** até **2 dev spaces**, e **apenas 1 rodando por vez**. Para trocar, pare o atual primeiro.

### Manutenção
- Depois de criado, o **nome** e o **tipo de aplicação** do dev space **não podem ser alterados**.
- É possível **adicionar extensions opcionais** — mas o dev space precisa estar **parado (STOPPED)**.
- **Não é possível remover** uma extension já atribuída (para preservar os projetos existentes).
- É possível **exportar todo o conteúdo** do dev space como arquivo **.tar.gz** — para isso o dev space precisa estar **RUNNING**. Ao final, o BAS mostra como importar esse conteúdo em outro dev space (de preferência vazio).

---

## Lição 1.3 — Creating a New Project in SAP Business Application Studio

**Objetivo:** criar um projeto de modelagem no BAS.

- Os projetos ficam numa estrutura de pastas específica do seu usuário e do seu dev space.
- Na **Explorer view** você navega no conteúdo do projeto (subpastas e arquivos), abre arquivos no editor, apaga ou move arquivos.
- A faixa (ribbon) no topo da Explorer view mostra o nome da pasta (em maiúsculas). A pasta é aberta como **workspace** — ou seja, configurações específicas do BAS podem ser associadas a ela.
- **Projeto do zero:** use o botão **Start from Template** na aba **Welcome**. O wizard cria um projeto quase vazio, apenas com os arquivos-base.

---

## Lição 1.4 — Importing an Existing Project in SAP Business Application Studio

**Objetivo:** importar um projeto para o workspace.

### Compartilhando conteúdo com Git (método preferido)
- **Git é o método preferido** para compartilhar conteúdo de desenvolvimento num projeto SAP HANA Cloud (importar/exportar com outros desenvolvedores).
- Git permite colaboração e trabalho em paralelo sem um sobrescrever o trabalho do outro — essencial em projetos grandes.
- O BAS inclui Git para gerenciar o ciclo de vida do código **localmente** e também conectar a um **remote repository** corporativo/público.
- Operações Git podem ser feitas pelo **terminal** (usuários experientes) ou pela **Git view** (mais guiada e visual).
- Para **colaborar em projetos grandes**, a melhor forma de trazer conteúdo para o workspace é **Clone from Git**.

### Importando sem Git
Mais simples, mas **sem rastreamento nem controle de versão**.

| Opção de import | O que faz |
|---|---|
| **Import** (Welcome page) / **Import project** (clique direito em área vazia) | Importa **somente arquivos .tar ou .zip** e já os extrai. Depois, a nova pasta pode ser aberta num novo workspace. |
| **File → Upload Files** | Faz upload de arquivos compactados (.tar/.zip) **ou arquivos individuais**. Útil para adicionar arquivos soltos. Se subir um arquivo compactado, você precisa extrair pela linha de comando (`tar` / `unzip`). |
| **Drag and drop** do explorador de arquivos | Copia arquivos/pastas do Windows File Explorer para qualquer pasta visível na Explorer view. |

> **Cuidado:** com **Upload Files** e **drag and drop**, o conteúdo pode **sobrescrever** arquivos existentes (mesmo nome e mesma pasta). Um aviso aparece para cada conflito.

### Exportando sem Git
- **Download:** clique direito em qualquer arquivo/pasta na Explorer view. Vários objetos são compactados num único **.tar**.
- Para baixar a pasta raiz inteira (vários projetos), clique direito na área em branco abaixo da estrutura de arquivos.

---

## Lição 1.5 — Navigating SAP Business Application Studio

**Objetivo:** descrever o layout e as features do BAS relevantes para modelagem.

- O BAS oferece várias formas de fazer a mesma ação: **menu commands**, **keyboard shortcuts**, **quick commands**, etc.
- As **views** à esquerda cobrem as necessidades do desenvolvedor: explorar arquivos, **search/replace**, conectar ao **Git**, **Debug**, etc.
- As views disponíveis dependem das **extensions** (incluídas por padrão conforme o tipo de dev space ou adicionadas como opcionais).
- Algumas views aparecem como ícones à esquerda; outras não — mas você pode abrir qualquer view pelo menu **Open View**.
- Algumas views têm painel próprio; outras aparecem como sub-painel. Ex.: **SAP HANA Projects** e **Java Projects** aparecem dentro da **Explorer view**.
- Você pode ocultar ícones/painéis que não usa.

### Quick Command (Command Palette)
- Forma poderosa de executar ações repetitivas ou pontuais.
- Comandos organizados por categorias: **Cloud Foundry**, **CDS**, **graphical modeling**, etc.
- Assim como as views, os quick commands **dependem das extensions instaladas**.

### Status bar
Mostra:
- **Notificações** e **problems** (warnings ou errors) no código.
- **Cloud Foundry status** e, se houver, o **Cloud Foundry target** (org/space onde você está logado).
- Informações **Git** quando o projeto é Git-enabled — ex.: nome da **branch** atual (checked-out branch).

---

## Lição 1.6 — Working with Joule in SAP Business Application Studio

**Objetivo:** usar o **Joule** dentro do BAS.

### O que é o SAP Joule
- **Joule** é o **copilot de IA** da SAP, integrado nas ferramentas SAP, incluindo o BAS.
- Ajuda a acelerar o desenvolvimento: **gera código**, **explica conceitos** e auxilia em tarefas específicas de SAP.
- Num dev space de **SAP HANA Native Database**, é especialmente útil para **database artifacts**, **SQL**, **CDS models** e lógica de aplicação.

### Como acionar o Joule
- Como **side panel assistant** (chat): clique no **ícone do Joule na Activity Bar**.
- Pelos atalhos da **command palette**.
- Por **inline suggestions** no editor.

### Recursos
- **Chat window** com prompts em linguagem natural.
- **Context awareness:** adapta o escopo do prompt ao conteúdo dos arquivos abertos ou ao código selecionado.
- **Command-based interaction:** você aciona capacidades específicas com **prefixos** (ex.: `/hana-ai`, `/hana-gen`) antes do prompt.

### Comandos (workspace do tipo SAP HANA Native Application)

| Comando | Para que serve |
|---|---|
| `/code-search` | Busca trechos de código no repositório. |
| `/hana-ai` | Extensão da biblioteca Python **HANA Machine Learning (hana-ml)** focada em **GenAI**; integra com HANA ML, **HANA vector engine** e **SAP GenAI Hub SDK**. |
| `/hana-gen` | Assistente SQL de IA especializado em **criar um projeto SAP HANA Cloud Database**. |
| `/hana-modeling-help` | **SAP HANA Modeling Assistant** — ajuda no design de **calculation views**. |
| `/hana-sql-gen` | Ajuda a criar **SQL statements** para o SAP HANA Cloud. |
| `/hdi-gen` | Gerador de definições de **HDI artifacts** (tables, views, roles…) a partir de linguagem natural, seguindo best practices. |
| `/sap-help` | Pesquisa no **SAP Help Portal**. |

> Outros comandos podem aparecer conforme as extensions ativas no dev space.

### Exemplo 1 — Criar um projeto de exemplo
1. Abra o painel do Joule pela Activity Bar.
2. Digite `/hana-gen`.
3. Escolha um modelo proposto (**Flight**, **Inventory Management**, **Bookshop**) ou descreva sua necessidade (tabelas, colunas…) em linguagem natural. O design das tabelas evolui conforme você adiciona requisitos (novos prompts ou editando um anterior).
4. Opcional: proponha um nome para o projeto.
5. Clique **Accept** — o Joule gera os artifacts na **Explorer pane**.

### Exemplo 2 — Ajuda em modelagem
1. Abra o painel do Joule.
2. Digite `/hana-modeling-help`.
3. Pergunte em linguagem natural — ex.: *"How to create a rank column"*, *"How to implement a parent-child hierarchy"*.
4. O resultado é um trecho da documentação SAP HANA, às vezes customizado com os artifacts citados na pergunta.

---

## Pontos-chave para a prova (Unit 1)
- Ferramenta **recomendada** para modelagem no SAP HANA Cloud: **SAP Business Application Studio** (não o Web IDE, não o Database Explorer).
- BAS = baseado em **VS Code**, roda no **SAP BTP**, subscription no **sub-account** via **BTP Cockpit**.
- Extension **essencial** para modelar: **SAP HANA Calculation View Editor**.
- Dev space: trial = 2 spaces / 1 rodando; adicionar extension → space **STOPPED**; exportar → space **RUNNING**; não dá para remover extension nem renomear/mudar o tipo.
- Projeto novo → **Start from Template**. Colaboração em projeto grande → **Clone from Git**. **Import** aceita só **.tar/.zip**.
- **Explorer view:** listar arquivos/pastas e criar projeto a partir de template.
- **Joule:** sugestões de código com IA e integração com ferramentas SAP; precisa de input do desenvolvedor e de internet.

---

## Quiz oficial da Unit 1 — com gabarito comentado
*(Gabarito elaborado a partir do conteúdo das lições.)*

1. **Quais são capacidades do SAP Joule no BAS?** (2 corretas)
   - ✅ It provides AI-powered code suggestions and assistance.
   - ✅ It integrates with SAP development tools.
   - ❌ It operates without the need for developer input. *(depende de prompts do desenvolvedor)*
   - ❌ It can work offline with no internet access.
2. **No SAP HANA Cloud, qual ferramenta é recomendada para data modeling?**
   - ✅ **SAP Business Application Studio** *(Web IDE não evolui mais; Database Explorer não é ferramenta de modelagem)*
3. **O que você pode fazer na Explorer view do BAS?** (2 corretas)
   - ✅ Create a project from a template
   - ✅ List files and folders
   - ❌ Manage SAP HANA artifacts deployment *(isso é feito pela view SAP HANA Projects)*
4. **Para colaborar em projetos grandes, qual a melhor opção para trazer conteúdo ao workspace?**
   - ✅ **Clone from Git**
5. **Qual extension é essencial no dev space para começar a modelar?**
   - ✅ **SAP HANA Calculation View Editor**
6. **Qual opção usar para criar um projeto totalmente novo?**
   - ✅ **Start from Template**

---

## Glossário da Unit 1
- **IDE (Integrated Development Environment):** ambiente integrado de desenvolvimento.
- **SAP BTP (Business Technology Platform):** plataforma de nuvem da SAP onde o BAS roda.
- **Sub-account / BTP Cockpit:** subconta do BTP / console de administração do BTP.
- **Development Space (dev space):** ambiente isolado do desenvolvedor no BAS, com extensions específicas.
- **Extension / plug-in:** ferramenta adicional instalada no dev space.
- **Workspace:** pasta aberta no BAS à qual se associam configurações.
- **Explorer view:** painel de arquivos/pastas do projeto.
- **Quick Command / Command Palette:** caixa de comandos rápidos.
- **Status bar:** barra inferior com notificações, problemas, status Cloud Foundry e Git.
- **Joule:** copilot de IA da SAP.
- **HDI (HANA Deployment Infrastructure):** infraestrutura de deploy de artifacts em containers (detalhada nas units seguintes).
