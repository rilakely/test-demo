# Couche 2 — Modèle (au 2026-09-24)

**Résultat : aucun projet ne passe les 4 critères (0 / 59 candidats).**

**Limite de la recherche :** le quota de recherche web partagé par les agents (200) a été épuisé pendant cette couche. Chaque agent a fait environ 30 à 50 recherches. Les verdicts d'échec reposent sur des preuves déjà trouvées. Aucun candidat n'a échoué seulement faute de recherche.

## 1. Adaptation des modèles (12 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Migration des fine-tunes OpenAI vers l'open-weight | a, d | OpenAI arrête le fine-tuning (05/2026 → 01/2027), mais OpenPipe, qui faisait exactement cette migration, a été absorbé par CoreWeave ; Azure garde les fine-tunes jusqu'à 10/2027 ; faisable en interne |
| Portage automatique d'adaptateurs LoRA vers une nouvelle base | a, d | Aucun budget ; Apple et Google l'intègrent ; Datawizz |
| RFT/GRPO managé | b, d | Bedrock, Azure, Fireworks, Together, Tinker ; Predibase racheté par Rubrik (109 M$) |
| QA des graders et des récompenses en RFT | a, d | Sepal racheté par Mercor ; équipes internes |
| Serving multi-LoRA | a, b, c, d | LoRAX, vLLM ; Fireworks au prix du modèle de base |
| Garde-fou de régression des fine-tunes | a, d | Nova Forge ; fonction triviale pour les plateformes |
| Distillation vers un petit modèle | a, b, d | Bedrock, Vertex, Databricks TAO |
| Merging-as-a-service | a, b, c, d | MergeKit (Arcee) |
| Cycle de vie des adaptateurs on-device | a, d | Apple, Google, Datawizz |
| Apprentissage continu en production | a, b, d | Adaption Labs, Applied Compute ; Cursor le fait en interne |
| Modèle open-weight « à personnaliser » | a, b, d | Inkling (Thinking Machines), Arcee |
| Kit AI Act pour qui modifie un modèle | a, b, d | Seuil d'un tiers du compute rarement atteint |

## 2. Évaluation, vérification, certification (11 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Sentinelle de dérive des modèles fournisseurs | a, b, d | AWS AgentCore, Dynatrace/Arize, Braintrust |
| Certification des hébergeurs open-weight | a, b, d | K2 Vendor Verifier (Moonshot), OpenRouter Auto Exacto |
| Validation des LLM juges | a, d | Open source (prometheus-eval, verdict) |
| Benchmarks experts privés | a, d | Vals AI, Mercor APEX, GDPval |
| Évaluation de trajectoires d'agents | a, b, d | AgentCore, Vertex, Patronus |
| Évaluations indétectables par le modèle | a, d | Apollo, METR, équipes internes, AIUC |
| Détection de triche aux tests dans les PR | a, d | Cursor+Graphite ; faisable en interne |
| Preuve formelle du code généré | a, d | Axiom, Harmonic |
| Cabinet d'audit agréé (SB 813) | a, b, d | Cadre volontaire, registre en 2029 ; AIUC, KPMG |
| Données d'évaluation pour les assureurs | a, b, d | Armilla, AIUC |
| Outillage « haut risque » AI Act | a, b, c, d | Échéance reportée à 12/2027 |

## 3. Sécurité du modèle (11 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Scanner sémantique de fichiers de modèles | b, d | Protect AI racheté par Palo Alto (635 M$), HiddenLayer, JFrog, modelscan |
| Imposer la signature des modèles | a, b, d | Sigstore model-transparency, NVIDIA NGC |
| AI-BOM | a, b, d | Snyk, Cisco (open source), Wiz, JFrog |
| Scanner de backdoors | a, d | Microsoft l'a publié (02/2026) |
| Inférence confidentielle gérée | a, b, d | Azure, Google Private AI Compute, OpenPCC, Phala, Tinfoil |
| Protection des poids déployés chez un tiers | a, b, d | Architecture de référence NVIDIA, Fortanix |
| Preuve de localisation / détection d'interposeur | a, d | Proof of Cloud Alliance ; les hyperscalers sont les data centers |
| Middleware anti-canaux auxiliaires | a, d | `cache_salt` dans vLLM ; bourrage par OpenAI, Azure, Mistral |
| Audit des fuites de raisonnement chiffré | a, b, d | Corrigé par les fournisseurs (08/2026) |
| Anti-distillation avec renseignement partagé | a, d | Aucun budget ; les labos le construisent en interne |
| Marquage conforme à l'article 50 de l'AI Act | a, b, d | SynthID, Imatag, Truepic, Steg.AI |

## 4. Interprétabilité et contrôle (11 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Garde-fou par sondes en sidecar vLLM | a, b, d | vllm-lens (UK AISI), Goodfire |
| API de steering | a, b, d | L'API Ember de Goodfire a été archivée (10/2025) ; le prompting fait mieux (AxBench) |
| Sondes d'hallucination | a, b, d | Sondes sous Apache-2.0, Goodfire RLFR |
| Moniteur par activations pour raisonnement latent | a, d | Réservé au détenteur des poids |
| Moniteur des transcriptions d'agents | a, b, d | Docent (Transluce) |
| CI de comparaison entre versions de modèle | a, d | diffing-toolkit, persona vectors |
| Motifs de refus ECOA par LLM | a, c, d | Les LLM ne décident pas du crédit ; le CFPB a retiré ses circulaires |
| Explication art. 86 AI Act | a, c, d | Annexe III reportée à 12/2027 ; Fiddler |
| Audit de biais mécaniste | a, b, c, d | FairPlay, Fiddler |
| SAE-as-a-service | a, b, d | Les SAE font moins bien que les sondes ; Gemma Scope 2 ouvert |
| Audit de la conscience d'être évalué | a, d | Fait en interne ; Transluce, Apollo, AISI |

## 5. Modèles spécialisés (14 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| API de prévision généraliste | a, b, d | BigQuery AI.FORECAST, Chronos-2 sur SageMaker |
| Demande intermittente | a, d | o9, Blue Yonder, RELEX, Kumo |
| Série temporelle sous licence ouverte | a, b, d | Chronos-2 et Toto sous Apache 2.0 |
| Modèle de fondation tabulaire | b, d | Kumo racheté par Nvidia (>400 M$), Prior Labs par SAP (>1 Md€) |
| Modèle relationnel multi-tables | b, d | Kumo (Nvidia) |
| Tableurs | a, d | Copilot in Excel, Claude for Excel |
| Texte vers CAO | a, d | Autodesk neural CAD |
| Lecture de plans d'ingénierie | a, d | CADDi |
| Placement-routage de circuits imprimés | a, b, d | Cadence Allegro X AI, Quilter |
| Design de puces | b, d | ChipStack racheté par Cadence ; Ricursive, Cognichip |
| Vibrations brutes | a, d | Augury, Tractian, Siemens |
| Embeddings géospatiaux | a, b, d | Google AlphaEarth, Esri |
| Météo IA pour traders d'énergie | a, b, d | Jua, WindBorne, Silurian |
| Métriques d'observabilité | a, b, d | Datadog Toto |

## Constat

La couche modèle est captée par trois forces :

- **Détenteurs des poids** : labs et clouds, qui font en interne la sécurité, les sondes et l'évaluation.
- **Plateformes qui rachètent les spécialistes** : Dynatrace, Cisco, Palo Alto, Nvidia, SAP, Cadence, CoreWeave.
- **Open source**, qui banalise l'outillage.
