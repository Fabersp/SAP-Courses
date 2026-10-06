# Unit 20 — Getting Started with SAP HANA Cloud Knowledge Graph Engine (Knowledge Graphs)

**Foco da unit:** conceitos de **Knowledge Graph**, **RDF / triples**, **ontologies** e linguagens, o **SAP HANA Cloud Knowledge Graph Engine (KGE)**, suas interfaces (**SPARQL**), arquitetura e casos de uso.

---

## Lição 20.1 — Overview of Knowledge Graphs

### O que é
- **Knowledge graph** = representação **estruturada de fatos** sobre o mundo, descrevendo **entidades** e suas **inter-relações**, em formato de **grafo**.
- Base de assistentes virtuais e buscadores; pode **explicar, avaliar e fundamentar** respostas de modelos de deep learning (ex.: LLMs). Aplicações: busca, perguntas e respostas, recomendações, **explainable AI**.

| Aspecto | Definição |
|---|---|
| **Entities** | **Nós (nodes)** = entidades (pessoas, lugares, coisas, conceitos); **arestas (edges)** = relações. |
| **Relationships** | Conexões entre entidades (ex.: `acted_in` entre ator e filme). |
| **Attributes** | Características de entidades/relações (ex.: filme → director, release_date, genre). |
| **RDF** | **Resource Description Framework**: modelo padrão de intercâmbio de dados na web; usa **URIs** para nomear a relação e as duas pontas → **triple**. Forma um **grafo dirigido e rotulado**. |
| **Triples** | Unidade básica: **subject – predicate – object**. Ex.: *Paris* (subject) – *is the capital of* (predicate) – *France* (object). |
| **Semantic Meaning** | Usam **vocabulários padronizados ou ontologies** para definir tipos de entidades e relações → ideal para IA. |
| **Sources of Data** | Estruturados (bancos), semiestruturados (XML, JSON), não estruturados (texto, via **Named Entity Recognition** e **Relation Extraction**). |
| **Examples** | **DBpedia**, **Wikidata**, **Freebase** (hoje parte do Google Knowledge Graph), grafos de SAP, Amazon, Facebook, Microsoft. |
| **Applications** | Buscadores, chatbots, assistentes digitais, recomendação, integração de dados. |
| **Ontologies** | Estrutura formal do grafo (tipos de entidades, relações, attributes) → consistência e interoperabilidade. |

---

## Ontologies

### Componentes

| Componente | Descrição |
|---|---|
| **Classes (Concepts)** | Categorias/tipos de entidades (ex.: Patient, Doctor, Disease). |
| **Properties (Attributes)** | Características das classes (name, age, medical history). |
| **Relationships (Relations)** | Como as classes se relacionam (Doctor *treats* Patient). |
| **Individuals (Instances)** | Objetos concretos de uma classe (John Doe é um Patient). |
| **Axioms** | Declarações lógicas que restringem a interpretação (regras). |

### Propósitos e benefícios
- **Shared Understanding** (vocabulário comum humano/máquina), **Data Integration**, **Reasoning** (inferência automática), **Knowledge Reuse**, **Semantic Interoperability**.

### Exemplos de ontologies
- **Gene Ontology (GO)** — bioinformática; **FOAF (Friend of a Friend)** — pessoas e relações; **SNOMED CT** — terminologia clínica; **Schema.org** — marcação de páginas web.

### Linguagens de ontologia (cai no quiz!)

| Linguagem | Descrição |
|---|---|
| **RDF** | Representa informação sobre recursos na web. |
| **RDFS (RDF Schema)** | Extensão do RDF com vocabulário para classes e propriedades. |
| **OWL (Web Ontology Language)** | Mais rica/expressiva, sobre RDF/RDFS; definição e raciocínio sobre ontologies. |
| **SKOS** | W3C; tesauros, taxonomias, classificações; menos formal (`skos:Concept`, `skos:broader`, `skos:narrower`, `skos:related`). |
| **DAML+OIL** | Predecessora do OWL (DARPA + projetos da UE). |
| **SHACL (Shapes Constraint Language)** | **Validação** de grafos RDF (`sh:NodeShape`, `sh:minCount`…). |
| **Frame Logic** | Antiga, baseada em **frames** (slots e fillers). |

> **Atenção:** *Axioms* são componentes de ontologia; *Gene Ontology* é uma ontologia (exemplo); *Outline* é feature do BAS — **nenhum é linguagem**.

### Ferramentas
- **Ontology Editors:** ex.: **Protégé**.
- **Reasoning Engines:** ex.: **HermiT**, **Pellet**.

---

## Lição 20.2 — SAP HANA Cloud Knowledge Graph Engine

- Componente do banco SAP HANA (in-memory) para **processamento e análise avançada de grafos**; **estende** o SAP HANA Cloud para lidar com **estruturas de dados complexas e interconectadas**. Interage com o **Index Server**, gerenciamento de sessão e clientes HANA.

### Key Features

| Feature | Descrição |
|---|---|
| **Graph Data Processing** | Modela relações complexas; grafos **dirigidos e não dirigidos** (nós e arestas). |
| **In-Memory Processing** | Alta performance e escalabilidade. |
| **Schema Flexibility** | Estrutura **schema-less** (dados heterogêneos/evolutivos). |
| **Scalability** | Grandes volumes. |
| **Integration with SQL** | Combina queries de grafo com SQL tradicional. |
| **Graph Algorithms** | Embutidos: **shortest path**, **connected components**, **centrality**. |
| **Query Language** | Suporte a linguagens de grafo como **Cypher** (Neo4j) e **PGQL**. |

### Interfaces e APIs

| Interface | Descrição |
|---|---|
| **SPARQL** | **Interface principal** de consulta (endpoint SPARQL). **SPARQL Update** para CRUD. *SPARQL = SPARQL Protocol and RDF Query Language.* |
| **RESTful API** | Ingestão de RDF em **Turtle, RDF/XML, N-Triples, JSON-LD**. |
| **Management Operations** | Criar/apagar/gerenciar graph stores, controles de acesso. |
| **Graph Algorithms APIs** | Executar algoritmos embutidos e customizados. |
| **JavaScript API** | Via APIs cliente do SAP HANA. |
| **SQL Interface** | Extensões SQL para integrar dados relacionais e de grafo. |
| **Event Streaming** | Ingestão/processamento em tempo real. |
| **BI e Visualização** | Integração com **SAP Analytics Cloud** e ferramentas de terceiros. |
| **SAP Data Intelligence** | Integração para usar **machine learning** em dados de grafo. |
| **Cloud Foundry / Kubernetes** | **Service brokers** para Cloud Foundry; **Kubernetes operators** para ciclo de vida. |
| **OData Services** | Expor dados via OData. |

### Arquitetura (Working with Knowledge Graph)
- **Data Ingestion and Storage:** fontes estruturadas, semiestruturadas, não estruturadas; armazenamento otimizado para grafos.
- **Data Modeling com RDF/OWL:** representação semanticamente rica; ontologies.
- **Graph Querying com SPARQL:** consultas que percorrem o grafo; baixa latência.
- **Computation and Analytics:** biblioteca de algoritmos (shortest path, **community detection**, centralidade) e algoritmos customizados.
- **Integration and APIs**, **Security and Governance** (controle de acesso, criptografia, permissões por role, lineage, metadados, qualidade), **User Interface e Visualização** (dashboards via BI), **Scalability and Performance** (escala horizontal, otimização de queries, indexação).

### Primeiros passos
- **Pré-requisitos:** instância **SAP HANA Cloud** ativa; acesso ao **SAP HANA Cloud Central** (antigo SAP HANA Cloud Cockpit); permissões adequadas.
- **Passos:** acessar o SAP HANA Cloud Central → criar/selecionar instância → **habilitar o serviço Knowledge Graph** (se não estiver por padrão) → conectar com cliente SQL (SAP HANA Studio, DBeaver, JDBC/ODBC) → criar tabelas e grafos → usar o engine.
  ```sql
  CREATE COLUMN TABLE VertexTable (id INT PRIMARY KEY, name NVARCHAR(100));
  CREATE COLUMN TABLE EdgeTable (id INT PRIMARY KEY, source INT, target INT, relationship NVARCHAR(100));
  CREATE GRAPH MyGraph (VERTEX TABLE VertexTable, EDGE TABLE EdgeTable);
  INSERT INTO VertexTable VALUES (1, 'Alice');
  INSERT INTO VertexTable VALUES (2, 'Bob');
  INSERT INTO EdgeTable VALUES (1, 1, 2, 'knows');
  SELECT * FROM MyGraph;
  ```
- Configuração adicional: **ontology management** e **data integration**.

---

## Lição 20.3 — Use Cases and Demo

### Casos de uso
- **Master Data Management:** Customer 360, Product master data.
- **Supply Chain:** relacionamento com fornecedores, demand forecasting.
- **Risk & Compliance:** risco financeiro, compliance regulatório.
- **Customer Intelligence:** sales forecasting, **churn prediction**.
- **Healthcare & Life Sciences:** integração de dados de pacientes, pesquisa clínica.
- **Enterprise Search:** busca semântica de documentos, mapeamento de skills.
- **IoT / Smart Manufacturing:** manutenção preditiva, analytics em tempo real.
- **Customer Service:** base de conhecimento de help desk, **chatbots** melhores.
- **Retail:** recomendação de produtos, gestão de estoque.
- **R&D:** gestão de inovação, análise de literatura científica.
- **Setor Público:** serviços ao cidadão, análise de políticas.

### Estudo de caso — indústria farmacêutica
- **Desafio:** muito dado (ensaios clínicos, papers, patentes) difícil de aproveitar.
- **Escopo:** Data Integration; Knowledge Graph Creation; Advanced Querying and Analytics; Collaboration and Data Sharing.
- **Implementação:**
  - Ingestão de fontes (PubMed, ClinicalTrials.gov, GenBank) com **SAP HANA Data Lake** e **SAP Data Intelligence**.
  - **Ontologia** do domínio (genes, proteínas, drogas, doenças, ensaios, patentes) e relações (*gene-protein*, *drug-disease*, *drug-clinical trial*).
  - Consultas **SPARQL**, graph analytics (reposicionamento de drogas, efeitos colaterais, novos alvos), ML para prever eficácia/segurança.
  - Compartilhamento com governança (controle de acesso, criptografia).
- **Resultados:** descoberta de drogas acelerada, dados mais acessíveis, analytics avançado, colaboração, melhores decisões.

### Demo — Ontologia de uma Universidade
- **Classes:** Person, Student, Professor, Course, Department.
- **Relações:** Student *is enrolled in* Course; Professor *teaches* Course; Course *belongs to* Department; Department *has* Professor.
- Apresentado no curso como um **modelo de knowledge graph** (estrutura de classes e relações).

---

## Pontos-chave para a prova (Unit 20)
- KG = entidades (nós) + relações (arestas) + attributes; unidade = **triple** (subject, predicate, object).
- KGs usam **vocabulários padronizados / ontologies**.
- Linguagens de ontologia: **RDF, RDFS, OWL** (+ SKOS, SHACL, DAML+OIL, Frame Logic).
- Componente que estende o HANA para dados interconectados: **Knowledge Graph Engine**.
- Linguagem de consulta RDF no KGE: **SPARQL**.
- ML em dados de grafo: integração com **SAP Data Intelligence**.

---

## Quiz oficial da Unit 20 — com gabarito comentado
1. **Quais são linguagens de ontologia?** (3) → ✅ **RDF**; ✅ **OWL**; ✅ **RDFS**. ❌ Axioms, Gene Ontology, Outline.
2. **Componente central que estende o SAP HANA para estruturas complexas e interconectadas?** → ✅ **Knowledge Graph Engine**.
3. **O que knowledge graphs usam para definir tipos de entidades e relações?** → ✅ **Standardized vocabularies or ontologies**.
4. **O que integrar ao KGE para usar machine learning em dados de grafo?** → ✅ **SAP Data Intelligence**.
5. **Linguagem suportada para dados RDF no KGE?** → ✅ **SPARQL**.
6. **O estudo de caso "Ontology of a University" é um modelo de Knowledge Graph — V/F?** → ✅ **True** (conforme apresentado no curso).

---

## Glossário
- **Knowledge Graph / Node / Edge / Triple (subject-predicate-object).**
- **RDF / RDFS / OWL / SKOS / SHACL.**
- **Ontology / Class / Property / Individual / Axiom.**
- **SPARQL / SPARQL Update / Turtle / N-Triples / JSON-LD.**
- **Graph algorithms:** shortest path, connected components, centrality, community detection.
- **SAP HANA Cloud Central.**
