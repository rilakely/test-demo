# Couche 1 — Données (au 2026-09-24)

**Résultat : aucun projet ne passe les 4 critères (0 / 64 candidats).**

Règle appliquée : au moindre doute sur un critère, c'est un échec.
Limite de la recherche : les preuves viennent surtout des extraits de résultats de recherche. La plupart des pages (arXiv, TechCrunch, sites des éditeurs) n'ont pas pu être ouvertes à cause du proxy.

## 1. Acquisition et droits (14 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Passerelle HTTP 402 / pay-per-crawl multi-CDN | a, b, d | TollBit est déjà intégré chez Fastly, Akamai et DataDome ; Cloudflare le fait aussi |
| Attribution des réponses RAG pour royalties « pay per use » | a, b, d | Déjà couvert par ProRata Gist, Microsoft PCM (02/2026) et Cloudflare pay per use (07/2026) |
| Audit tiers des déclarations d'usage faites par les IA | a, b, d | Aucun budget ; Cloudflare, Microsoft PCM et Sureel ont déjà un reporting |
| Attribution paramétrique (sources apprises par le modèle) | a, d | Sureel racheté par Warner (06/2026) ; Musical AI et Vermillio déjà présents |
| Filtre de régurgitation non littérale | a, d | Azure protected material, Google RECITATION, Patronus |
| Audit de mémorisation avant lancement | a, b, d | Fait en interne par les labs ; CopyShield en open source |
| Unlearning à la demande pour takedown | a, d | Hirundo ; les filtres de sortie suffisent aux déployeurs |
| Générateur du résumé AI Act (art. 53) | a, b, d | Simple formulaire, faisable en interne |
| Moteur de conformité opt-out côté crawler | a, b, d | Fait en interne par les labs ; Spawning |
| Registre d'opt-out pour les créateurs | a, b, d | Spawning DNTR, RSL Collective |
| Vérification cryptographique des agents | a, b, d | Web Bot Auth chez Cloudflare, AWS, Akamai, HUMAN, Vercel |
| API web temps réel pour agents | b, d | Tavily racheté par Nebius (275 M$) ; Exa, Parallel, Firecrawl |
| Place de marché de licences de contenu | b, d | Human Native racheté par Cloudflare, Calliope par Protege |
| Rails de paiement agent → contenu | a, b, d | Fondation x402 (Coinbase, Cloudflare, Google, Visa, AWS) |

## 2. Données humaines et environnements RL (11 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Audit d'exploitabilité des environnements RL | a, d | HUD ; BenchJack en open source ; QA interne des labs |
| Moniteur de reward hacking sur les rollouts | a, d | Docent (Transluce) en open source ; HUD |
| Détection d'usage de LLM par les annotateurs | a, d | Prolific le fait ; Mercor utilise Insightful ; détection interne |
| Validation de rubrics | a, d | Scale ; Docent |
| Arbitrage du désaccord entre experts | a, d | Fait en interne par les labs et les fournisseurs |
| Clones d'applications d'entreprise pour le RL | b, d | Fleet (60 M$ ARR), Deeptune racheté par Mercor |
| Simulateur d'utilisateurs calibré | a, d | Veris AI ; VISTA en open source |
| Assainisseur de dépôts pour environnements de code | a, d | Trivial à faire en interne |
| Poste de travail sécurisé pour annotateurs | a, b, d | Relève de la sécurité générique |
| Place de marché d'environnements | a, b, d | Prime Intellect Environments Hub, OpenEnv |
| Nouvelle place de marché d'experts | b, d | Mercor, Handshake, Micro1, Surge, Turing |

## 3. Synthétique et curation (14 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Données tabulaires synthétiques | b, d | Gretel racheté par NVIDIA, Hazy par SAS, YData par KPMG, MOSTLY AI par Syntho ; Delphix (09/2026) |
| Audit de confidentialité par MIA | a, c, d | Aucun budget ; obligations « haut risque » de l'AI Act reportées à 12/2027 |
| Déduplication sémantique GPU | a, c, d | NeMo Curator, datatrove |
| Optimiseur de mélange de données | a, d | Olmix en open source, DatologyAI, fait en interne |
| Classifieur de qualité qui préserve la diversité | a, d | Nemotron-CC, DatologyAI |
| Décontamination sémantique des benchmarks | a, d | Fait en interne par les acheteurs (OpenAI) |
| Registre de distillabilité des modèles | a, b, d | OpenRouter le fait déjà |
| Détection des attaques par distillation | a, b, d | Anthropic l'a construit en interne |
| Distillation clé en main | a, b, d | Amazon Bedrock Model Distillation, NeMo Data Designer |
| Détection de LLM dans les données « humaines » | a, d | Cleanlab racheté par Handshake |
| Audit de « reward-hackability » | a, d | Proximal ; QA interne des labs |
| Générateur du résumé AI Act | a, b, d | Faisable en interne |
| Moniteur d'effondrement (model collapse) | a, b, d | Data Designer |
| Vérification des traces de raisonnement | a, b, d | Recettes open source |

## 4. Données d'entreprise pour RAG et agents (11 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Parsing VLM haute précision | a, b, d | Modèles ouverts < 2B à plus de 96 sur OmniDocBench ; Databricks et Snowflake le font nativement |
| Routeur OCR / VLM | a, b, d | Reducto |
| Chunking en service | a, b, c, d | Chonkie en open source |
| Synchronisation des ACL au niveau du chunk | a, d | Paragon, Merge, Auth0 FGA, Microsoft, Glean |
| Remédiation du sur-partage avant Copilot | a, b, d | SharePoint Advanced Management inclus dans Copilot ; Varonis |
| Passerelle de masquage PII | b, d | Prompt Security → SentinelOne, Lakera → Check Point, Pangea → CrowdStrike |
| CDC vers index vectoriel | a, b, d | Databricks Delta Sync, Snowflake Cortex Search, CocoIndex |
| Couche sémantique auto-générée | a, d | Snowflake Semantic View Autopilot, Databricks Metric Views |
| Lignage des réponses d'IA | a, b, c, d | Collibra, Atlan |
| Effacement RGPD vérifiable dans les index vectoriels | a, d | Faisable en interne ; éditeurs de bases vectorielles, Transcend, OneTrust |
| Passerelle MCP de gouvernance | b, d | Oasis → Cyera (~1 Md$), Natoma → Snowflake |

## 5. Données multimodales et physiques (14 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Décodage vidéo GPU + dataloader | a, b, d | TorchCodec, DALI, NeMo Curator |
| Lakehouse multimodal à accès aléatoire | a, b, d | LanceDB, Nimble, Vortex |
| Cache stockage → GPU | a, b, d | Alluxio, VAST, Weka |
| Checkpoint asynchrone | a, b, d | PyTorch DCP, NVRx |
| Curation et captioning vidéo en service | a, b, d | NeMo Curator |
| Lake de données robot MCAP → LeRobot | a, b, d | Foxglove, Rerun, Encord ; conversion en open source |
| Curation de démonstrations par fonctions d'influence | a, d | L'argent va à la collecte ; Foxglove et Encord ; méthode facile à internaliser |
| QA temps réel de la téléopération | a, d | Claru |
| Anonymisation de vidéo égocentrique | a, b, d | brighter AI racheté par Milestone ; EgoBlur |
| Retargeting de vidéo humaine vers des actions robot | a, d | Config ; NVIDIA GR00T-Dreams |
| Recherche de scénarios de conduite par VLM | a, d | Applied Intuition Basis, Scale |
| Auto-annotation LiDAR | a, b, d | Segments.ai racheté par Uber ; Kognic, Scale |
| Résumé AI Act à partir du lignage | a, d | lakeFS/DVC, Securiti |
| Trajectoires robot synthétiques | a, b, d | NVIDIA GR00T-Dreams |

## Constat

La valeur de cette couche est captée par trois types d'acteurs :

- **Plateformes de données** (Databricks, Snowflake, NVIDIA) : elles intègrent la fonction nativement, souvent gratuitement ou en open source.
- **Fournisseurs complets** (Mercor, Scale, Cloudflare, cybersécurité) : ils rachètent les spécialistes.
- **Labs** : ils gardent le contrôle qualité et l'outillage en interne.
