# Relationship Discovery & Network Analysis Engine: Key Literature and Algorithms

## 1. Overview

This report surveys foundational and modern research relevant to building a **Relationship Discovery & Network Analysis Engine** that uncovers hidden, indirect, and non-obvious relationships across people, organizations, events, assets, and locations, using structured and unstructured data. It emphasizes works that either introduced influential algorithms, defined canonical tasks (link prediction, community detection, entity resolution, graph ML), or inspired real-world graph intelligence platforms (e.g., Palantir-style investigative tools, Neo4j ecosystem, fraud and criminal network analytics).[^1][^2][^3][^4][^5][^6]

The report is organized by the requested topical categories. Within each, a small set of **high‑impact papers or books** is highlighted with structured summaries.

***

## 2. Foundational Social Network Analysis Papers

### 2.1 Social Network Analysis: Methods and Applications (Wasserman & Faust, 1994)

- **Type**: Monograph (book), foundational SNA reference.
- **Research problem**: Formalize social network analysis (SNA) as a rigorous statistical and graph-theoretic framework for studying relations rather than just node attributes.[^7][^8]
- **Key contributions**:
  - Systematic treatment of network data models (directed/undirected, valued, two-mode) and adjacency/relational representations.[^9]
  - Defines centrality, prestige, cohesive subgroups, structural balance, and blockmodeling in a unified framework.[^9]
  - Introduces families of statistical network models (e.g., p* / exponential random graph models) used in later dynamic and relational modeling.[^8]
- **Methodology / algorithms**: Graph-theoretic metrics (degree, betweenness, eigenvector centrality), cohesive subgroup detection, blockmodels, and stochastic models for tie formation.[^9]
- **Main findings**: Demonstrates how network structure strongly shapes outcomes in social and organizational settings and how to estimate and test network hypotheses empirically.[^7]
- **Limitations**: Pre‑big‑data era; focuses on relatively small networks and classical statistics, with limited coverage of large-scale or ML-based methods.[^9]
- **Citation impact**: Over 50k citations, widely regarded as the core SNA methods text.[^10]
- **Why it matters / relevance**: Provides conceptual and mathematical foundations (centrality, cohesion, role equivalence) that underlie relationship discovery, influence analysis, and key actor identification in modern engines.
- **Difficulty**: Intermediate (requires some probability/statistics and linear algebra).
- **Link**: Cambridge University Press page and scans.[^11][^12]

### 2.2 The Structure and Function of Complex Networks (Newman, SIAM Review, 2003)

- **Research problem**: Review and synthesize the emerging science of complex networks (social, technological, biological), connecting empirical patterns with generative models.[^4][^13]
- **Key contributions**:
  - Introduces and popularizes concepts such as small‑world effect, heavy‑tailed degree distributions, clustering, assortativity, and network correlations across domains.[^4]
  - Surveys random graph models (Erdős–Rényi, configuration model) and growth models (preferential attachment) used to model real networks.[^4]
  - Discusses dynamical processes on networks (percolation, epidemics, search) relevant to propagation and reachability analysis.
- **Methodology / algorithms**: Analytical graph models, random graphs, preferential attachment, percolation and epidemic models on networks.[^4]
- **Main findings**: Many real networks share non‑trivial structural regularities (e.g., heavy‑tailed degree, clustering, community structure), incompatible with simple random graphs and requiring richer models.[^2][^14]
- **Limitations**: Predates network embeddings and GNNs; focuses on analytical models more than data-driven ML.[^4]
- **Citation impact**: ~17k citations; canonical review for complex network structure.[^2]
- **Relevance**: Offers a mental model of what real-world graphs look like, which informs model choice, scalability planning, and anomaly definition in an analysis engine.
- **Difficulty**: Intermediate to advanced (mathematical but well explained).
- **Link**: arXiv preprint / SIAM Review PDF.[^15][^4]

### 2.3 Networks: An Introduction (Newman, 2010)

- **Type**: Comprehensive textbook on network theory and applications.[^16]
- **Contributions**:
  - Consolidates graph-theoretic foundations, network statistics, community structure, and processes on networks in a single reference.[^17][^16]
  - Bridges SNA with physics and computer science perspectives, including algorithms and practical analysis methods.
- **Relevance**: Highly suitable as a conceptual and algorithmic reference for a CS student building graph‑centric systems.
- **Difficulty**: Intermediate.
- **Link**: Publisher / Google Books.[^18][^16]

***

## 3. Relationship Discovery and Link Analysis Papers

### 3.1 The Link‑Prediction Problem for Social Networks (Liben‑Nowell & Kleinberg, 2007)

- **Research problem**: Given a snapshot of a social network, predict which new edges will form in the near future using only observed topology.[^3]
- **Key contributions**:
  - Formalizes **link prediction** as a problem defined over network snapshots.[^3]
  - Systematically evaluates node proximity measures (common neighbors, Jaccard, Adamic‑Adar, preferential attachment, Katz, shortest paths) as link predictors on co‑authorship networks.[^19][^3]
  - Shows that sophisticated path-based scores (e.g., Katz) outperform naive measures and that topology alone carries predictive signal.[^3]
- **Methodology / algorithms**:
  - Defines and empirically evaluates multiple similarity indices across held‑out edges.[^3]
  - Uses co‑authorship graphs as case studies with train/test splits.
- **Main findings**: Many missing or future ties can be inferred from topology; there is no single universally best heuristic but Katz and related global measures perform strongly.[^3]
- **Limitations**: Focuses on undirected, static social networks; does not exploit node attributes or temporal dynamics.[^3]
- **Citation impact**: Widely cited core paper in link prediction literature.[^20]
- **Relevance**: Provides baseline heuristics and evaluation paradigm for link prediction modules in a relationship discovery engine.
- **Difficulty**: Beginner to intermediate.
- **Link**: Wiley / PDF mirror.[^21][^3]

### 3.2 A Survey of Link Prediction in Complex Networks (Martínez, Berzal, Cubero, ACM CSUR, 2016)

- **Research problem**: Systematically review link prediction methods across complex networks and unify the taxonomy of heuristics and learning-based approaches.[^6][^22]
- **Key contributions**:
  - Proposes a taxonomy that groups methods into similarity‑based (local, quasi-local, global), probabilistic, and ML-based approaches.[^22][^6]
  - Summarizes classical indices (Common Neighbors, Adamic–Adar, Resource Allocation, Katz, random walks) and more advanced models.[^6]
  - Discusses evaluation metrics (AUC, precision@k) and dataset characteristics.
- **Methodology**: Literature meta‑analysis, algorithm classification, comparative discussion of strengths/weaknesses.[^6]
- **Main findings**: No universal winner; performance depends on network sparsity, noise, and heterogeneity; structural heuristics often competitive with more complex models.[^6]
- **Limitations**: Predates most GNN‑based link predictors; focuses on pre‑deep-learning methods.[^6]
- **Citation impact**: Recognized survey frequently cited as starting point in link prediction studies.[^23]
- **Relevance**: Helps design the algorithm portfolio and baselines for link prediction and hidden relationship discovery.
- **Difficulty**: Beginner to intermediate.
- **Link**: ACM DL / preprints.[^24][^23]

### 3.3 Stacking Models for Nearly Optimal Link Prediction in Complex Networks (Ghasemian et al., PNAS, 2020)

- **Research problem**: Understand performance variation across many link prediction algorithms and design a method that approaches the optimal predictor.[^25]
- **Key contributions**:
  - Empirical benchmark of **203 algorithms** across **550 real-world networks** from diverse domains.[^25]
  - Shows no single heuristic dominates; performance varies widely depending on network structure.[^25]
  - Proposes a stacked ensemble that combines multiple predictors to achieve near-optimal performance on synthetic and real networks.[^25]
- **Methodology**: Large-scale empirical study, ensembling methods, comparisons across network classes.
- **Main findings**: Social networks are generally easier to predict than biological/technological ones; ensembles outperform any single predictor.[^25]
- **Limitations**: Focuses on classical methods and ensembles, not GNN-based models.[^25]
- **Relevance**: Motivates building **ensembles of link predictors** rather than relying on a single metric in production engines.
- **Difficulty**: Intermediate.
- **Link**: PNAS article / arXiv preprint.[^26][^25]

### 3.4 Mining Complex Trees for Hidden Fruit: A Graph–Based Computational Solution to Detect Latent Criminal Networks (D. Durrant, thesis / monograph)

- **Research problem**: Detect “latent” criminal networks from heterogeneous investigative data by combining entity resolution and link discovery.[^27]
- **Key contributions**:
  - Proposes an R-based computational solution integrating **entity resolution**, **link prediction**, and **knowledge discovery** for criminal intelligence.[^27]
  - Shows substantial improvements in entity resolution accuracy over competitors (F‑measure ~0.986 vs 0.872) and high‑accuracy detection of hidden criminal links (accuracy ≈ 0.8).[^27]
- **Methodology / algorithms**:
  - Custom entity resolution pipeline leveraging probabilistic matching and relational context.[^27]
  - Link prediction and inference based on structural and attribute features, tuned for investigative datasets.
- **Main findings**: Joint optimization of entity resolution and link prediction can surface non-obvious criminal structures that are missed by rule‑based or isolated approaches.[^27]
- **Limitations**: Case study–driven; details about algorithms and reproducible code may be harder to generalize.
- **Relevance**: Very close conceptually to the desired engine: demonstrates how to integrate ER + link analysis for intelligence applications.
- **Difficulty**: Intermediate.
- **Link**: University repository PDF.[^27]

***

## 4. Community Detection and Network Structure

### 4.1 Community Detection in Graphs (Fortunato, Physics Reports, 2010)

- **Research problem**: Provide a comprehensive review of algorithms and issues in detecting community structure in graphs.[^28][^29]
- **Key contributions**:
  - Defines community structure and discusses quality functions (e.g., modularity) and their limitations (resolution limit).[^30]
  - Surveys partition-based, hierarchical, spectral, statistical-mechanical, and overlapping community methods.[^28]
  - Reviews benchmarking, significance testing, and comparison of community finding algorithms.[^28]
- **Methodology**: Extensive literature survey (103 pages, 400+ references) with conceptual unification.[^28]
- **Main findings**: No single algorithm is universally best; modularity optimization is popular but has resolution and degeneracy issues; generative models (stochastic block models) offer principled alternatives.[^28]
- **Limitations**: Focuses mostly on static, unweighted or simple weighted graphs; predates dynamic and GNN-based community detection.[^28]
- **Citation impact**: 14k+ citations; standard reference for community detection.[^31]
- **Relevance**: Critical for building modules that detect fraud rings, cells, or functional communities from large graphs.
- **Difficulty**: Intermediate.
- **Link**: arXiv and ScienceDirect versions.[^29][^28]

### 4.2 Detecting Clusters/Communities in Social Networks (Kaiser et al., 2017)

- **Research problem**: Introduce a community detection method based on Cohen’s κ as a similarity measure between nodes, and compare with other algorithms.[^32]
- **Key contributions**:
  - Proposes κ-based similarity measure for node pairs and clusters κ values to detect communities.[^32]
  - Performs one of the broadest comparative evaluations across simulated and real networks, benchmarking against eight community detection algorithms.[^32]
- **Methodology**: Similarity-based clustering, empirical evaluation.
- **Main findings**: κ-based method is consistently among top performers in classification quality across datasets.[^32]
- **Limitations**: Less widely used than modularity-based methods; computational costs not optimized for very large graphs.[^32]
- **Relevance**: Illustrates how similarity metrics + clustering can form alternative community modules when modularity is problematic.
- **Difficulty**: Intermediate.
- **Link**: Open-access article on PMC.[^32]

### 4.3 Characterizing the Interactions Between Classical and Community‑Aware Centrality Measures (Orman et al., Scientific Reports, 2021)

- **Research problem**: Study how centrality measures that incorporate community structure differ from classical centralities in evaluating node importance.[^33]
- **Key contributions**:
  - Shows that incorporating community information can yield more effective centrality measures for certain tasks (e.g., spreading, robustness).[^33]
  - Provides empirical evidence that local vs global centralities and community-aware variants behave differently across networks.[^33]
- **Relevance**: For a relationship discovery engine, centrality drives ranking of key actors and chokepoints; this paper motivates community-aware ranking.
- **Difficulty**: Intermediate.
- **Link**: Nature Scientific Reports article.[^33]

***

## 5. Dynamic and Temporal Network Analysis

### 5.1 Modeling and Detecting Change in Temporal Networks via a Dynamic Degree‑Corrected Stochastic Block Model (Wilson et al., 2016)

- **Research problem**: Detect structural changes (e.g., anomalies, regime shifts) in temporal networks using a dynamic DCSBM.[^34]
- **Key contributions**:
  - Extends the degree-corrected stochastic block model to a dynamic setting, with parameters evolving over time.[^34]
  - Combines parametric random graph modeling with statistical process monitoring to detect local and global structural changes.[^34]
- **Methodology**: Dynamic stochastic block modeling, control charts/statistical process monitoring, application to U.S. Senate co‑voting network.[^34]
- **Main findings**: Approach effectively identifies periods of cohesion/polarization in political networks and can detect both localized and global network changes.[^34]
- **Limitations**: Requires fitting parametric models; may be computationally intensive for very large networks.
- **Relevance**: Provides a principled model-based foundation for **temporal change detection** and alerting in dynamic relationship graphs (e.g., sudden formation of suspicious clusters).
- **Difficulty**: Advanced (statistical modeling background helpful).
- **Link**: arXiv preprint.[^34]

### 5.2 Temporal Networks in Biology and Medicine: A Survey on Models, Algorithms, and Tools (Bodini et al., 2022)

- **Research problem**: Survey temporal graph models and algorithms, especially in biomedical contexts.[^35]
- **Key contributions**:
  - Provides formal definitions of temporal graphs (time-ordered snapshots and other models) and discusses core algorithms.[^35]
  - Reviews applications of temporal networks in biology/medicine, demonstrating the need to model node/edge evolution.[^35]
- **Relevance**: Offers general temporal graph modeling concepts (snapshots vs continuous time) transferable to fraud or intelligence networks.
- **Difficulty**: Beginner to intermediate.
- **Link**: Open-access survey.[^35]

### 5.3 A Comprehensive Survey of Dynamic Graph Neural Networks (Zhao et al., PVLDB, 2024)

- **Research problem**: Systematically review dynamic graph neural networks (DGNNs) and their models, frameworks, and benchmarks.[^36]
- **Key contributions**:
  - Covers 81 DGNN models and 12 training frameworks, with a taxonomy based on time representation and update mechanisms.[^36]
  - Provides experimental comparisons of nine representative models and three frameworks on standard datasets, focusing on convergence, efficiency, and memory usage.[^36]
- **Relevance**: Essential for designing **temporally aware GNN components** for link prediction, node classification, and anomaly detection over evolving networks.
- **Difficulty**: Intermediate to advanced.
- **Link**: arXiv/HTML version.[^36]

***

## 6. Knowledge Graph and Entity Resolution Papers

### 6.1 Collective Entity Resolution in Relational Data (Bhattacharya & Getoor, TKDD, 2007)

- **Research problem**: Resolve entities in relational data where references are ambiguous and attributes alone are insufficient.[^37][^38]
- **Key contributions**:
  - Introduces **collective entity resolution**, using co‑occurrence and relational information jointly with attributes.[^38]
  - Proposes a relational clustering algorithm that iteratively refines entity clusters using the cluster assignments of related references.[^38]
- **Methodology / algorithms**:
  - Relational clustering with similarity measures over both attributes and neighboring entities.[^38]
  - Probabilistic generative model with Gibbs sampling to infer latent groups and entities.[^37][^38]
- **Main findings**: Collective resolution significantly outperforms attribute-only baselines across real and synthetic datasets; structural properties of data strongly affect gains.[^37][^38]
- **Limitations**: Developed for moderate-sized relational datasets; not designed for web-scale graphs.
- **Citation impact**: Widely cited; foundational ER work.[^39][^40]
- **Relevance**: Directly supports **entity resolution** modules that precede relationship discovery, ensuring that graph nodes represent real entities rather than fragmented identifiers.
- **Difficulty**: Intermediate.
- **Link**: ACM TKDD / LINQS preprint.[^40][^39]

### 6.2 Domain‑Independent Data Cleaning via Analysis of Entity–Relationship Graph (Kalashnikov & Mehrotra, TODS, 2006)

- **Research problem**: Improve reference disambiguation and data cleaning by exploiting entity-relationship graphs rather than just attribute similarity.[^41][^42]
- **Key contributions**:
  - Represents databases as entity–relationship graphs and uses graph structure to improve disambiguation quality.[^43]
  - Proposes RelDC, a framework that analyzes inter-object relationships in addition to object features.[^43][^38]
- **Methodology**: Graph-based similarity, iterative linkage, and analysis of neighborhood structure to detect duplicates/variants.
- **Main findings**: Demonstrates improved disambiguation accuracy over attribute-only methods in multiple domains.[^43]
- **Relevance**: Motivates building ER and cleaning on top of the same graph infrastructure used for relationship discovery.
- **Difficulty**: Intermediate.
- **Link**: ACM TODS / UCI PDF.[^42][^43]

### 6.3 Unsupervised Graph‑Based Entity Resolution for Complex Entities (Fan et al., ACM TODS, 2023)

- **Research problem**: Entity resolution for **complex entities** (multi-record, structured) using unsupervised graph-based techniques.[^44]
- **Key contributions**:
  - Proposes a graph-based ER framework that links records of complex entities without labeled data.[^44]
  - Shows improvements (up to ~29% in precision/recall) over state-of-the-art ER techniques across seven real-world datasets.[^44]
- **Methodology**: Graph modeling of records and attributes, unsupervised similarity propagation, and clustering.
- **Relevance**: Aligns with large-scale, heterogeneous knowledge graphs where annotation is scarce.
- **Difficulty**: Intermediate.
- **Link**: ACM article.[^44]

### 6.4 Entity Resolution Surveys and Knowledge Graph Alignment

- **End‑to‑End Entity Resolution for Big Data: A Survey (2019)** and **Blocking and Filtering Techniques for Entity Resolution: A Survey (ACM CSUR 2020)** summarize blocking, matching, and scaling approaches for big data ER, including learning-based methods.[^45]
- **Entity Resolution: Past, Present and Yet‑to‑Come (EDBT 2020)** reviews the evolution of ER and open challenges.[^45]
- **A Survey: Knowledge Graph Entity Alignment Research Based on Graph Embedding (AI Review 2024)** focuses on aligning entities across KGs via embeddings, relevant to multi‑source graph integration.[^45]

These surveys help design a scalable ER pipeline and entity alignment components for knowledge-graph based engines.

***

## 7. Graph Machine Learning and Graph Neural Networks

### 7.1 A Comprehensive Survey on Graph Neural Networks (Wu et al., IEEE TNNLS, 2019/2020)

- **Research problem**: Provide a unified overview and taxonomy of GNNs across data mining and ML.[^5][^46]
- **Key contributions**:
  - Taxonomy into recurrent, convolutional, autoencoder, and spatial–temporal GNNs.[^5]
  - Reviews learning paradigms (supervised, semi-supervised, unsupervised, self-supervised, few‑shot) on graphs.[^5]
  - Summarizes applications (social, recommender, chemistry, knowledge graphs) and benchmark datasets.[^5]
- **Relevance**: Baseline map for selecting appropriate GNN architectures for link prediction, node classification, and graph classification in the engine.
- **Difficulty**: Intermediate.
- **Link**: arXiv / IEEE Xplore.[^46][^5]

### 7.2 Graph Neural Networks: A Review of Methods and Applications (Zhou et al., 2020)

- **Research problem**: Another broad GNN survey with focus on models and applications.[^47]
- **Key contributions**:
  - Reviews GNN variants by computation modules and training strategies.[^47]
  - Emphasizes practical applications and performance across tasks.
- **Relevance**: Complements Wu et al. by connecting model design with application performance, helpful for system engineers.
- **Link**: Neurocomputing article.[^47]

### 7.3 Representation Learning on Graphs / Network Representation Learning Surveys

- **Representation Learning on Graphs: Methods and Applications (Hamilton et al., 2017)** and **A Comprehensive Survey of Graph Embedding (Cai et al., 2017)** review graph embedding methods (DeepWalk, node2vec, LINE, etc.) and their use in link prediction and clustering.[^48]
- **Network Representation Learning: A Survey (Zhang et al., IEEE TBigData, 2018)** focuses on NRL algorithms with detailed taxonomy and evaluation.[^49][^48]

These works connect **unsupervised representation learning** and modern GNNs, foundational for embedding-based relationship discovery from large graphs.

### 7.4 Graph Neural Networks for Link Prediction (Zhang & Chen et al., ICLR line of work; ELPH/BUDDY, 2023)

- **Representative papers**:
  - **Revisiting Graph Neural Networks for Link Prediction (Zhang et al., ICLR 2021)** analyzes GAEs vs subgraph-based SEAL methods, showing that learning from labeled subgraphs yields more expressive link representations and large gains on OGB benchmarks.[^50]
  - **Graph Neural Networks for Link Prediction with Subgraph Sketching – ELPH/BUDDY (Chamberlain et al., 2023)** proposes ELPH and BUDDY, full-graph methods that approximate subgraph GNNs using **subgraph sketches** and hashing, achieving SOTA link prediction with far better scalability.[^51]
- **Key contributions**:
  - Establish that naive aggregation of node embeddings often underperforms, and **subgraph‑level** modeling or approximations are crucial for expressive link prediction.[^50][^51]
  - Introduce scalable designs (BUDDY) that precompute features and avoid per‑edge subgraph extraction.[^51]
- **Relevance**: Highly relevant for a production‑grade link prediction module that must scale while maintaining accuracy.
- **Difficulty**: Advanced.

### 7.5 Explainable GNNs and Link Prediction

- **Self‑Explainable Graph Neural Networks for Link Prediction (Wang et al., 2023)** introduces a framework where the model simultaneously produces accurate link predictions and **explicit explanations** via K important neighbors.[^52]
- **Accumulated Local Effects and Graph Neural Networks for Link Prediction (2026)** adapts model-agnostic ALE to visualize how node feature values influence GNN link predictions, comparing exact vs approximate ALE for GCN/GAT.[^19]

These works are important for **Explainable Graph AI**, enabling investigators to understand why a relationship was inferred, a key requirement in intelligence and compliance contexts.

***

## 8. Fraud Detection and Intelligence Network Papers

### 8.1 Link Prediction in Criminal Networks: A Tool for Criminal Intelligence Analysis (Berlusconi et al., PLOS One, 2016)

- **Research problem**: Apply link prediction to real criminal networks to infer missing ties, under high noise and incompleteness.[^1]
- **Key contributions**:
  - Works on a real investigation dataset and defines “marginal” links (removed during investigative preprocessing), then uses their topological features as proxies for missing edges.[^1]
  - Shows that node similarity measures (e.g., common neighbors, Adamic–Adar) provide effective characterization for missing links in this criminal context.[^1]
- **Methodology**: Topological feature analysis, similarity indices, validation via judicial documents.[^1]
- **Main findings**: Predicted links often correspond to actors with high likelihood of co‑participation in illicit activities, supporting investigative utility.[^1]
- **Limitations**: Single case study; relies mainly on topology.[^1]
- **Relevance**: Demonstrates direct use of link prediction in criminal intelligence; strong template for evaluating hidden relationship discovery.
- **Difficulty**: Beginner to intermediate.
- **Link**: PLOS One article.[^1]

### 8.2 Robust Link Prediction in Criminal Networks: Sicilian Mafia Case Study (Calderoni et al., Expert Systems with Applications, 2020)

- **Research problem**: Investigate robustness of link prediction strategies in noisy, incomplete criminal networks.[^53]
- **Key contributions**:
  - Compares link prediction algorithms (including Katz) on meeting vs telephone call networks extracted from judicial documents.[^53]
  - Shows that algorithms leveraging full graph topology (e.g., Katz) remain accurate even on sparse networks, but performance is sensitive to data incompleteness.[^53]
- **Relevance**: Highlights the impact of missing data and network type on link prediction quality, central to designing an operational engine.
- **Difficulty**: Intermediate.
- **Link**: Study summarized by the ROXANNE project article.[^53]

### 8.3 Structural Analysis of Criminal Networks and Predicting Hidden Links (various works)

- Several papers (e.g., “Structural Analysis of Criminal Network and Predicting Hidden Links”) focus on combining SNA measures (centrality, clustering) with link prediction to identify brokers and hidden ties in criminal organizations.[^54][^53]
- They typically use real law-enforcement data and measure how predictive techniques can guide disruption strategies.

### 8.4 Using Social Network Analysis for Fraud Detection (Clerx, TU Delft thesis, 2019; and follow-up 2022 paper)

- **Research problem**: Explore how SNA can support fraud detection and how value is created from data to inspection decisions.[^55][^56]
- **Key contributions**:
  - Shows SNA’s potential for risk analysis, threat assessment, role identification, and evidence building in fraud contexts.[^56]
  - Emphasizes organizational and institutional factors in turning network analytics into operational value.
- **Relevance**: Connects technical SNA with the governance and workflow aspects of deploying an engine in regulatory agencies.

### 8.5 Social Network Analytics for Supervised Fraud Detection in Insurance (Óskarsdóttir et al., Risk Analysis, 2022)

- **Research problem**: Integrate social network analytics into supervised fraud detection for insurance claims.[^57]
- **Key contributions**:
  - Constructs networks linking claims via shared entities (e.g., claimants, garages), then extracts SNA features for supervised ML models.[^57]
  - Demonstrates improved fraud detection performance by augmenting traditional features with network-level attributes.[^57]
- **Relevance**: Shows how to convert raw claim data into graph features feeding ML classifiers—a pattern highly transferable to fintech fraud engines.

### 8.6 Network Analytics for Identifying Fraud Rings and Systemic Risk (Modepalli, JISEM, 2025)

- **Research problem**: Propose a comprehensive network analytics framework for detecting fraud rings and systemic risk in financial institutions.[^58][^59]
- **Key contributions**:
  - Combines network construction, community detection (Louvain), centrality, anomaly detection, and visualization for risk management.[^59][^58]
  - Highlights that graph-based anomaly detection achieves high accuracy and lower false positives versus rules.[^58]
- **Relevance**: Provides an end‑to‑end template similar to commercial graph fraud platforms.

### 8.7 Graph‑Based Fraud Detection and GNN Approaches

Curated lists, such as **Graph based Fraud Detection Papers** and **Awesome Graph/Transformer Fraud Detection**, catalog many GNN-based fraud detection works, including **H2‑FDetector**, **HGsuspector**, **HitFraud**, and **GraphRAD**.[^60][^61][^62]

- **H2‑FDetector: A GNN‑based Fraud Detector with Homophilic and Heterophilic Connections (2022)** explicitly models both homophily and heterophily for fraud detection, improving robustness in transaction graphs where fraudulent nodes connect both to each other and to benign nodes.[^62]
- These works showcase end‑to‑end graph-based fraud systems close to large‑scale industrial deployments.

***

## 9. Modern AI‑Powered Graph Intelligence and Explainability

### 9.1 Explainable Graph Neural Networks

The **awesome-graph-explainability-papers** list aggregates key surveys and methods on GNN explainability, including:

- **Explainability in Graph Neural Networks: A Taxonomic Survey (TPAMI 2022)**.
- **Trustworthy Graph Neural Networks: Aspects, Methods and Trends (Proceedings of the IEEE 2024)**.
- **Graph-Based Explainable AI: A Comprehensive Survey (preprint 2024)**.[^63]

These surveys:

- Classify explanation methods (gradient-based, perturbation, subgraph extraction, counterfactuals) and evaluation metrics.[^63]
- Discuss trust, robustness, privacy, and fairness, all critical for investigative and regulatory settings.

They are complemented by task-specific works such as **Self‑Explainable GNNs for Link Prediction** (see §7.5), which provide concrete mechanisms for interpretable relationship inference.[^52]

### 9.2 Temporal and Dynamic GNNs for Intelligence

The dynamic GNN survey (Zhao et al., 2024) and related works on dynamic network embedding (e.g., dynnode2vec, dyngraph2vec, Continuous-Time Dynamic Network Embeddings) offer methods for capturing evolving structures crucial in criminal/fraud networks.[^64][^36]

These models support tasks like **temporal link prediction**, **early anomaly detection**, and **change-point detection** in streaming graphs, aligning with operational relationship discovery systems.

### 9.3 Graph Summarization with GNNs

- **A Comprehensive Survey on Graph Summarization with Graph Neural Networks (2024, IEEE TAI)** reviews deep learning approaches for summarizing large graphs, including recurrent, convolutional, autoencoder, and attention-based GNNs.[^65]
- It discusses benchmarks and metrics for summarization, along with reinforcement learning approaches to improve summary quality.[^65]

This is relevant for **scalability** and **visualization**, where investigators need condensed yet informative views of large relationship graphs.

### 9.4 Knowledge Graphs, ER, and LLM‑Enhanced Intelligence

Recent industry and research pieces, e.g., **Combining Entity Resolution and Knowledge Graphs** and GraphAware’s work on **using KGs with LLMs for crime analysis**, describe how ER, KG construction (Neo4j, Senzing, Linkurious), and large language models combine to speed up investigative workflows.[^66][^67]

These works highlight practical integration patterns: ingesting heterogeneous records, resolving entities, building a knowledge graph, running graph algorithms, and surfacing results via visual tools and LLM‑based assistants.

***

## 10. Survey Papers and Literature Reviews (Cross‑Cutting)

Beyond category-specific surveys already mentioned, the following are particularly important as “maps of the territory”:

- **Community Detection in Graphs (Fortunato, 2010)** – community algorithms and evaluation.[^28]
- **Community Detection in Networks: A User Guide (Fortunato & Hric, 2016)** – practical guide for choosing and evaluating methods.[^68]
- **A Survey of Link Prediction in Complex Networks (Martínez et al., 2016)** – link prediction taxonomy.[^22][^6]
- **Link Prediction on Complex Networks: An Experimental Survey (Su et al., 2022)** – proposes a new taxonomy, provides comprehensive experimental comparisons.[^69]
- **Network Representation Learning: A Survey (Zhang et al., 2018)** – network embedding methods.[^48][^49]
- **Heterogeneous Network Representation Learning: A Unified Framework with Survey and Benchmark (Yang et al., TKDE 2020)** – heterogeneous network representation with benchmarks.[^70][^71]
- **A Comprehensive Survey on Graph Neural Networks (Wu et al., TNNLS 2019/2020)** and **Graph Neural Networks: A Review of Methods and Applications (Zhou et al., 2020)** – core GNN surveys.[^47][^5]
- **A Comprehensive Survey of Dynamic Graph Neural Networks (Zhao et al., 2024)** – dynamic GNNs.[^36]
- **Entity Resolution surveys** (End‑to‑End ER for Big Data; Blocking and Filtering Techniques; Entity Resolution: Past, Present and Yet‑to‑Come).[^45]
- **GNN Explainability surveys** listed in the awesome-graph-explainability repo.[^63]

These are crucial for a holistic view of the space and for identifying repeated algorithmic patterns.

***

## 11. Essential Reading Order for a New Researcher

### 11.1 Beginner Level (Conceptual Foundations)

1. **Social Network Analysis: Methods and Applications** – core SNA concepts, centrality, subgroups, roles.[^8]
2. **Networks: An Introduction** – network structure, random graphs, community structure, processes.[^16]
3. **The Structure and Function of Complex Networks** – condensed review of complex network properties and models.[^4]
4. **Community Detection in Graphs (Fortunato, 2010)** – foundations of community detection.[^28]
5. **The Link‑Prediction Problem for Social Networks** – classical link prediction and evaluation.[^3]

### 11.2 Intermediate Level (Algorithms and Tasks)

6. **A Survey of Link Prediction in Complex Networks** – full taxonomy and survey of link prediction methods.[^6]
7. **Link Prediction on Complex Networks: An Experimental Survey** – experimental comparisons and taxonomy.[^69]
8. **Collective Entity Resolution in Relational Data** – collective ER foundations.[^38]
9. **Domain‑Independent Data Cleaning via Analysis of Entity–Relationship Graph** – graph-based data cleaning.[^43]
10. **Heterogeneous Network Representation Learning** survey – embeddings for heterogeneous graphs.[^70]
11. **Network Representation Learning: A Survey** – comprehensive NRL methods.[^49]
12. **Community Detection in Networks: A User Guide** – method selection and evaluation in practice.[^68]

### 11.3 Advanced Level (GNNs, Dynamic Graphs, Fraud/Intelligence)

13. **A Comprehensive Survey on Graph Neural Networks** and **Graph Neural Networks: A Review of Methods and Applications** – GNN landscape.[^5][^47]
14. **Representation Learning on Graphs** and related embedding surveys – deeper dive into graph representation learning.[^48]
15. **Graph Neural Networks for Link Prediction (SEAL, ELPH/BUDDY, etc.)** – advanced link prediction modeling.[^50][^51]
16. **A Comprehensive Survey of Dynamic Graph Neural Networks** – temporal GNNs.[^36]
17. **Link Prediction in Criminal Networks: A Tool for Criminal Intelligence Analysis** and the Sicilian Mafia case study – concrete criminal network applications.[^53][^1]
18. **Social Network Analytics for Supervised Fraud Detection in Insurance** – integrating SNA with supervised fraud models.[^57]
19. **Network Analytics for Identifying Fraud Rings and Systemic Risk** – end‑to‑end fraud ring analytics framework.[^59]
20. **GNN Explainability surveys** + **Self‑Explainable GNNs for Link Prediction** – explainable graph AI.[^52][^63]

This ordering moves from classical graph theory and SNA through specialized tasks (link prediction, community detection, ER) into modern GNN and dynamic/intelligence applications.

***

## 12. Key Algorithms Repeated Across the Literature

Across these works, several algorithmic families recur and form the backbone of relationship discovery and network intelligence systems:

- **Centrality Measures**: Degree, betweenness, closeness, eigenvector, PageRank, and community-aware variants.[^7][^33]
- **Community Detection**: Modularity optimization (e.g., Louvain), spectral clustering, stochastic block models (including degree‑corrected and dynamic variants).[^34][^28]
- **Link Prediction Heuristics**: Common Neighbors, Jaccard, Adamic–Adar, Resource Allocation, Preferential Attachment, Katz index, random walk–based measures (e.g., hitting/commute time, SimRank, superposed random walk), quasi-local methods.[^6][^3]
- **Random Graph and Generative Models**: Erdős–Rényi, configuration model, preferential attachment, (degree‑corrected) stochastic block models.[^4][^34]
- **Network Embeddings / Representation Learning**: DeepWalk, node2vec, LINE, GraRep, TADW, GCN, GAT and other GNNs, as surveyed by NRL and GNN papers.[^48][^5]
- **Entity Resolution Techniques**: Attribute similarity, blocking, relational/collective ER using graph structure, probabilistic generative models, graph-embedding-based alignment.[^45][^38]
- **Dynamic Graph Models**: Snapshot models, temporal random walks, dynamic SBMs, dynamic embeddings, continuous-time dynamic graph models.[^64][^36][^34]
- **Graph-based Fraud/Anomaly Detection**: Dense subgraph / subtensor detection, graph-based anomaly scores, GNN-based fraud detection (e.g., H2‑FDetector, HGsuspector, HitFraud).[^61][^72][^62]

These algorithms map closely to functional modules in a relationship discovery engine: ingestion & ER, graph construction, structural analysis, link prediction, community/ring detection, temporal analytics, ML-based risk scoring, and explainability.

***

## 13. Current Research Trends, Open Challenges, and Opportunities

### 13.1 Trends

- **From heuristics to learned models**: Link prediction and fraud detection are shifting from simple heuristics to GNNs and hybrid models that integrate structural and attribute information.[^62][^51][^5]
- **Dynamic and streaming graphs**: Temporal networks and dynamic GNNs receive growing attention, reflecting real-world needs for continuous monitoring and early warning.[^35][^36]
- **Heterogeneous and attributed graphs**: Many applications now involve multi-entity, multi-relational graphs (users, accounts, devices, merchants), leading to advances in heterogeneous NRL and GNNs.[^71][^70]
- **Explainable Graph AI and Trustworthiness**: Surveys and methods focus on explainability, robustness, fairness, and privacy in GNNs, driven by high-stakes domains like finance, healthcare, and law enforcement.[^19][^63]
- **Integration with Knowledge Graphs and LLMs**: Industry reports highlight combining ER, KGs, and LLMs for analytic workflows, giving investigators natural-language access to graph intelligence.[^67][^66]
- **Scalability and graph summarization**: GNN-based graph summarization and scalable link prediction (ELPH/BUDDY) tackle memory constraints and latency in large graphs.[^51][^65]

### 13.2 Open Challenges

- **Scalable, accurate ER and graph construction**: Maintaining high-quality entity resolution and deduplication at scale across many sources remains difficult; ER errors propagate into all downstream graph analytics.[^38][^45]
- **Robustness to noise and incompleteness**: Criminal and fraud networks are particularly noisy and incomplete; link prediction methods must remain robust under adversarial missingness and sampling biases.[^53][^1]
- **Evaluation on realistic intelligence tasks**: Many models are evaluated on benchmark datasets that may not capture operational constraints (adversaries, changing tactics, legal and interpretability requirements).
- **Explainability suitable for investigators**: Existing explainers often produce low-level subgraphs or feature importance scores; turning these into actionable narratives for analysts is an open UX and HCI challenge.[^52][^63]
- **Dynamic, multi-modal data fusion**: Integrating text, transactions, communications, sensor/geo data into coherent temporal graphs is still a major integration and modeling challenge.[^66][^65]
- **Privacy, ethics, and bias**: Graph intelligence systems risk privacy violations and discriminatory outcomes; trustworthy GNN research is only beginning to address these issues systematically.[^63]

### 13.3 Opportunities for a Relationship Discovery & Network Analysis Engine

For a capstone or research project aligned with the literature and practical systems, promising directions include:

- **Explainable link prediction for fraud/criminal networks**: Implement a SEAL/ELPH/BUDDY-style link predictor with self-explaining mechanisms and evaluate it on semi-realistic fraud or communication graphs.[^50][^51][^52]
- **Collective ER + link analysis pipeline**: Build a pipeline that jointly performs collective entity resolution (inspired by Bhattacharya & Getoor) and link prediction to discover hidden rings, with emphasis on measuring how ER quality affects downstream link predictions.[^38][^27]
- **Dynamic risk scoring using DGNNs**: Apply dynamic GNN architectures to temporal transaction or communication data, producing evolving risk scores and alerts with explicit change-point detection.[^36][^34]
- **Graph summarization dashboards**: Use GNN-based graph summarization to generate multi-level summaries (communities, key paths) that can be visualized for investigators in a Neo4j/Linkurious-style dashboard.[^65]
- **Benchmarking classical vs GNN fraud detection**: Compare performance and explainability of classical SNA + gradient boosted trees vs GNN-based detectors on public fraud-like datasets or synthetic benchmarks.[^61][^57]

These directions synthesize foundational SNA, ER, community detection, GNNs, dynamic graphs, and explainability into cohesive, practically motivated research that is directly relevant to modern graph intelligence platforms.

---

## References

1. [Link Prediction in Criminal Networks: A Tool for Criminal Intelligence Analysis](https://journals.plos.org/plosone/article?id=10.1371%2Fjournal.pone.0154244) - The problem of link prediction has recently received increasing attention from scholars in network s...

2. [The Structure and Function of Complex Networks - Semantic Scholar](https://www.semanticscholar.org/paper/The-Structure-and-Function-of-Complex-Networks-Newman/e6c4925fb114d13a8568f88957c167c928f0c9f1) - The Structure and Function of Complex Networks. @article{Newman2003TheSA, title={The Structure and F...

3. [The link‐prediction problem for social networks - Wiley Online Library](https://onlinelibrary.wiley.com/doi/abs/10.1002/asi.20591) - Liben-Nowell, D. and Kleinberg, J. (2007), The link-prediction problem for social networks. J. Am. S...

4. [The structure and function of complex networks - cond-mat - arXiv](https://arxiv.org/abs/cond-mat/0303516) - SIAM Review 45, 167-256 (2003) ... Access Paper: View a PDF of the paper titled The structure and fu...

5. [[1901.00596] A Comprehensive Survey on Graph Neural Networks](https://arxiv.org/abs/1901.00596) - In this survey, we provide a comprehensive overview of graph neural networks (GNNs) in data mining a...

6. [A Survey of Link Prediction in Complex Networks | Semantic Scholar](https://www.semanticscholar.org/paper/A-Survey-of-Link-Prediction-in-Complex-Networks-Mart%C3%ADnez-Berzal/03b15e0ffa62d2626b8051e738ef7399447122d9) - A Survey of Link Prediction in Complex Networks · Víctor Martínez, F. Berzal, J. Cubero · Published ...

7. [Social network analysis : methods and applications | WorldCat.org](https://search.worldcat.org/title/social-network-analysis-methods-and-applications/oclc/30594217) - Covers methods for the analysis of social networks and applies them to examples

8. [Social Network Analysis: Methods and Applications](https://books.google.com/books/about/Social_Network_Analysis.html?id=wsMgAwAAQBAJ) - Social network analysis is used widely in the social and behavioral sciences, as well as in economic...

9. [Social network analysis: methods and applications by Stanley Wasserman & Katherine Faust [1994] {302'.01'1--dc20} : Stanley Wasserman, Katherine Faust : Free Download, Borrow, and Streaming : Internet Archive](https://archive.org/details/SocialnetworkanalysisWassermanFaust1994/SocialnetworkanalysisWassermanFaust1994_144x75) - scan of bookSocial network analysis: methods and applications by Stanley Wasserman & Katherine Faust...

10. [‪Stan Wasserman‬ - ‪Google Scholar‬](https://scholar.google.com/citations?user=XvWu8u8AAAAJ&hl=en) - Social network analysis: Methods and applications. S Wasserman, K Faust. Cambridge University Press,...

11. [Social Network Analysis - Cambridge University Press & Assessment](https://www.cambridge.org/core/books/social-network-analysis/90030086891EB3491D096034684EFFB8) - Social Network Analysis: Methods and Applications reviews and discusses methods for the analysis of ...

12. [[PDF] Social Network Analysis: Methods and Applications](https://www.asecib.ase.ro/mps/Social%20Network%20Analysis%20%5B1994%5D.pdf) - Social network analysis : methods and applications / Stanley. Wasserman, Katherine Faust. p. crn. - ...

13. [The Structure and Function of Complex Networks](https://explore.openaire.eu/search/publication?pid=10.1137%2Fs003614450342480) - The Structure and Function of Complex Networks ... 01 Jan 2003Embargo end date: 31 Dec 2002 English ...

14. [The structure and function of complex networks | Better Evaluation](https://www.betterevaluation.org/tools-resources/structure-function-complex-networks) - Other processes on networks. Sources. Newman, M. E. J. (2003). The structure and function of complex...

15. [[PDF] The structure and function of complex networks - umich.edu](http://www-personal.umich.edu/~mejn/courses/2004/cscs535/review.pdf) - Page 1. The structure and function of complex networks. M. E. J. Newman. Department of Physics, Univ...

16. [Networks: An Introduction - Mark Newman - Google Books](https://books.google.com/books/about/Networks.html?id=LrFaU4XCsUoC) - The scientific study of networks, including computer networks, social networks ... Networks: An Intr...

17. [Complex Systems 535/Physics 508, Fall 2017: Network Theory](http://www-personal.umich.edu/~mejn/courses/2017/cscs535/) - General books on networks: Networks: An Introduction, M. E. J. Newman, Oxford University Press, Oxfo...

18. [Networks: An Introduction | Guide books | ACM Digital Library](https://dl.acm.org/doi/10.5555/1809753) - Networks: An IntroductionMay 2010. Journal cover image. Author: Author Picture ... Advances on Conce...

19. [Accumulated local effects and graph neural networks for link prediction](https://www.nature.com/articles/s41598-026-39000-w) - This study explores the application of Accumulated Local Effects (ALE)22 to GNNs trained for link pr...

20. [The link-prediction problem for social networks | BibSonomy](https://www.bibsonomy.org/bibtex/21c8e2e0414084da508e98c9a8e5372fc/jaeschke) - The link-prediction problem for social networks - Liben-Nowell - 2007 - Journal of the Association f...

21. [[PDF] The Link Prediction Problem for Social Networks - Computer Science](http://web.cs.ucla.edu/~yzsun/classes/2014Spring_CS7280/Papers/Link%20Prediction/p556-liben-nowell.pdf) - The Link Prediction Problem for Social Networks. David Liben-Nowell∗. Jon Kleinberg†. ABSTRACT. Give...

22. [A Survey of Link Prediction in Complex Networks](https://oamonitor.ireland.openaire.eu/rpo/rcsi/search/publication?pid=10.1145%2F3012704) - Link prediction aims to infer the behavior of the network link formation process by predicting misse...

23. [A Survey of Link Prediction in Complex Networks - ACM Digital Library](https://dl.acm.org/doi/10.1145/3012704) - Víctor Martínez, Fernando Berzal, and Juan-Carlos Cubero. 2016. Adaptive degree penalization for lin...

24. [Typing-Monkeys/complex-network-link-prediction - GitHub](https://github.com/Typing-Monkeys/complex-network-link-prediction) - 2016. A Survey of Link Prediction in Complex Networks ACM Comput. Surv. 49, 4, Article 69 (December ...

25. [Stacking models for nearly optimal link prediction in complex networks](https://www.pnas.org/doi/10.1073/pnas.1914950117) - Martínez, F. Berzal, J. C. Cubero, A survey of link prediction in complex networks. ACM Comput. Surv...

26. [[PDF] Stacking Models for Nearly Optimal Link Prediction in Complex ...](https://www.colorado.edu/biofrontiers/2019/10/23/stacking-models-nearly-optimal-link-prediction-complex-networks) - Martínez V, Berzal F, Cubero JC (2017) A survey of link prediction in complex networks. ACM. Computi...

27. [[PDF] a graph-based computational solution to detect latent criminal ...](https://mro.massey.ac.nz/server/api/core/bitstreams/bf6b8843-9be7-43d0-9ab5-8d76f2f74564/content)

28. [[0906.0612] Community detection in graphs - arXiv](https://arxiv.org/abs/0906.0612) - Title:Community detection in graphs. Authors:Santo Fortunato. View a PDF of the paper titled Communi...

29. [[PDF] Physics Reports Community detection in graphs](https://pdodds.w3.uvm.edu/files/papers/others/2010/fortunato2010a.pdf) - Fortunato / Physics Reports 486 (2010) 75–174. 77. Fig. 1. A ... The aim of community detection in g...

30. [Community detection in graphs - ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0370157309002841) - The aim of community detection in graphs is to identify the modules and, possibly, their hierarchica...

31. [‪Santo Fortunato‬ - ‪Google Scholar‬](https://scholar.google.com/citations?user=NDrCCokAAAAJ&hl=en) - Community detection in graphs. S Fortunato. Physics reports 486 (3-5), 75-174, 2010. 14133, 2010. St...

32. [Detecting Clusters/Communities in Social Networks - PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC6103523/) - Cohen’s κ, a similarity measure for categorical data, has since been applied to problems in the data...

33. [Characterizing the interactions between classical and community ...](https://www.nature.com/articles/s41598-021-89549-x) - In this paper, we perform an extensive investigation to get a better understanding of the relationsh...

34. [Modeling and detecting change in temporal networks via a dynamic ...](https://arxiv.org/abs/1605.04049) - In many applications it is of interest to identify anomalous behavior within a dynamic interacting s...

35. [Temporal networks in biology and medicine: a survey on models ...](https://pmc.ncbi.nlm.nih.gov/articles/PMC9803903/) - The use of static graphs for modelling and analysis of biological and biomedical data plays a key ro...

36. [A Comprehensive Survey of Dynamic Graph Neural Networks: Models, Frameworks, Benchmarks, Experiments and Challenges](https://ar5iv.labs.arxiv.org/html/2405.00476) - Dynamic Graph Neural Networks (GNNs) combine temporal information with GNNs to capture structural, t...

37. [Collective Entity Resolution In Relational Data - UMD DRUM](https://drum.lib.umd.edu/items/699e2e14-c5e3-48cf-aa9f-037dbef7a0e2) - Collective Entity Resolution In Relational Data. Collective Entity Resolution In Relational Data ......

38. [[PDF] Collective entity resolution in relational data - Semantic Scholar](https://www.semanticscholar.org/paper/Collective-entity-resolution-in-relational-data-Bhattacharya-Getoor/814f90ef27bfe5a90e118a1df0e24488e75b7939) - Collective entity resolution in relational data. @article ... 2006. TLDR. This work proposes a novel...

39. [Collective entity resolution in relational data - ACM Digital Library](https://dl.acm.org/doi/10.1145/1217299.1217304) - Collective entity resolution in relational data. Authors: Indrajit ... 2006a. Mining graph data. In ...

40. [[PDF] Collective Entity Resolution in Relational Data - LINQS](https://linqs.org/assets/resources/bhattacharya-tkdd07.pdf) - Collective Entity Resolution in Relational Data. · 3. Jim Doe. Jason Doe. Jackie ... 2006a]. The con...

41. [Domain-independent data cleaning via analysis of entity ...](https://dl.acm.org/doi/10.1145/1138394.1138401) - Domain-independent data cleaning via analysis of entity-relationship graph. Authors: Dmitri V. Kalas...

42. [Domain-independent data cleaning via analysis of entity ...](https://www.ics.uci.edu/~dvk/pub/TODS06_dvk.html) - Domain-independent data cleaning via analysis of entity-relationship graph. ACM TODS Journal, Vol. 3...

43. [[PDF] Domain-Independent Data Cleaning via Analysis of Entity ...](https://www.ics.uci.edu/~dvk/CV/pub1.pdf) - In this article, we address the problem of reference disambiguation. Specifically, we consider a sit...

44. [Unsupervised Graph-Based Entity Resolution for Complex Entities](https://dl.acm.org/doi/10.1145/3533016) - In this article, we propose an unsupervised graph-based ER framework that is aimed at linking record...

45. [Entity Resolution, Entity Matching and Entity Alignment.md - GitHub](https://github.com/heathersherry/Knowledge-Graph-Tutorials-and-Papers/blob/master/topics/Entity%20Resolution,%20Entity%20Matching%20and%20Entity%20Alignment.md) - In the query stage, this paper frames the heterogeneous data query problem as a knowledge graph matc...

46. [A Comprehensive Survey on Graph Neural Networks - IEEE Xplore](https://ieeexplore.ieee.org/document/9046288) - In this article, we provide a comprehensive overview of graph neural networks (GNNs) in data mining ...

47. [Graph neural networks: A review of methods and applications](https://www.sciencedirect.com/science/article/pii/S2666651021000012) - In this survey, we conduct a comprehensive review of graph neural networks. For GNN models, we intro...

48. [thunlp/NRLPapers: Must-read papers on network representation ...](https://github.com/thunlp/NRLpapers) - Currently, the implemented models in OpenNE include DeepWalk, LINE, node2vec, GraRep, TADW and GCN. ...

49. [Network Representation Learning: A Survey - IEEE Computer Society](https://www.computer.org/csdl/journal/bd/2020/01/08395024/1hN4aUycB8Y) - We provide a detailed and thorough study of the state-of-the-art network representation learning alg...

50. [Revisiting Graph Neural Networks for Link Prediction - OpenReview](https://openreview.net/forum?id=8q_ca26L1fz) - The authors study the expressive power of Graph Neural Network architectures for the link prediction...

51. [Graph Neural Networks for Link Prediction](https://openreview.net/pdf?id=m1oqEOAozQU)

52. [Self-Explainable Graph Neural Networks for Link Prediction - ar5iv](https://ar5iv.labs.arxiv.org/html/2305.12578) - in this paper, we study a novel problem of self-explainable GNNs for link prediction, which can simu...

53. [Link prediction algorithms to enhance criminal network analysis](https://roxanne-euproject.org/news/blog/link-prediction-algorithms-to-enhance-criminal-network-analysis) - A recent study shows that link prediction can predict links among criminals, although the effectiven...

54. [[PDF] Structural Analysis of Criminal Network and Predicting Hidden Links ...](https://arxiv.org/pdf/1507.05739.pdf)

55. [Using Social Network Analysis for Fraud Detection](https://repository.tudelft.nl/record/uuid:5662ef20-f05c-440f-b0b0-20f623a6d43e)

56. [Using Social Network Analysis for Fraud Detection: Tracing the Path from Data to Value](https://www.academia.edu/85953464/Using_Social_Network_Analysis_for_Fraud_Detection_Tracing_the_Path_from_Data_to_Value) - Recent incidents in the food and beverage industry show that consumer goods remain vulnerable to tam...

57. [Social Network Analytics for Supervised Fraud Detection in Insurance](https://pubmed.ncbi.nlm.nih.gov/33547691/) - Insurance fraud occurs when policyholders file claims that are exaggerated or based on intentional d...

58. [Network Analytics for Identifying Fraud Rings and Systemic Risk](https://jisem-journal.com/index.php/journal/article/download/12641/5883/21275)

59. [Network Analytics for Identifying Fraud Rings and Systemic Risk](https://jisem-journal.com/index.php/journal/article/view/12641) - Network Analytics for Identifying Fraud Rings and Systemic Risk. Article Sidebar. PDF. Published: Au...

60. [safe-graph/graph-fraud-detection-papers](https://github.com/safe-graph/graph-fraud-detection-papers) - A curated list of Graph/Transformer-based fraud, anomaly, and outlier detection papers & resources -...

61. [Graph-based-Fraud-Detection-Papers/README.md at main · manjunath5496/Graph-based-Fraud-Detection-Papers](https://github.com/manjunath5496/Graph-based-Fraud-Detection-Papers/blob/main/README.md) - "Be patient with him. If the same quality did not exist in you, you wouldn't notice it in him."― Rob...

62. [H2-FDetector: A GNN-based Fraud Detector with Homophilic and ...](https://dl.acm.org/doi/10.1145/3485447.3512195) - These results highlight the importance of incorporating heterophilic relationships for effective fra...

63. [flyingdoog/awesome-graph-explainability-papers - GitHub](https://github.com/flyingdoog/awesome-graph-explainability-papers) - Papers about the explainability of GNNs Surveys [ACM computing survey. Neural Networks for Link Pred...

64. [Dynamic network embedding survey](https://dl.acm.org/doi/abs/10.1016/j.neucom.2021.03.138)

65. [A comprehensive survey on graph summarization with graph neural ...](https://researchers.mq.edu.au/en/publications/a-comprehensive-survey-on-graph-summarization-with-graph-neural-n/) - Our investigation includes a review of the current state-of-the-art approaches, including recurrent ...

66. [Combining entity resolution and knowledge graphs - Linkurious](https://linkurious.com/blog/entity-resolution-knowledge-graph/) - Entity resolution and knowledge graphs mutually reinforce one another and when combined, offer signi...

67. [Speed up Criminal Network Analysis with LLMs and Knowledge ...](https://graphaware.com/blog/combine-knowledge-graphs-and-llms-to-speed-up-crime-analysis/) - Criminal network analysis can identify criminal leaders with 92% accuracy—a level of precision that ...

68. [[PDF] Community detection in networks: A user guide](https://www.cs.cornell.edu/courses/cs6241/2020sp/readings/Fortunato-2016-guide.pdf) - Fortunato, Community detection in graphs, Phys. Rep. 486 (2010) 75–174. [12] M. Coscia, F. Giannotti...

69. [Link Prediction on Complex Networks: An Experimental Survey - PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC9211798/) - Link prediction plays an important role in complex network analysis in that it can find missing link...

70. [Heterogeneous Network Representation Learning: A Unified ... - PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC10619966/) - Heterogeneous Network Representation Learning: A Unified Framework with Survey and Benchmark. Carl Y...

71. [[PDF] Heterogeneous Network Representation Learning - Computer Science](https://web.cs.ucla.edu/~yzsun/papers/2020_IJCAI_HIN_Survey.pdf) - In this survey, we review the recent progress in heteroge- neous network representation learning and...

72. [Detecting Group Anomalies in Tera-Scale Multi-Aspect Data via ...](https://pmc.ncbi.nlm.nih.gov/articles/PMC8118605/) - ... fraud detection in graphs. See Akoglu et al. (2015) for a survey. 1.3 Organization of the Paper....

