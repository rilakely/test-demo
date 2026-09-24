# Couche 3 — Inférence (au 2026-09-24)

**Résultat : aucun projet ne passe les 4 critères (0 / 57 candidats).**

Limite : les preuves viennent surtout des extraits de résultats de recherche, avec environ 40 recherches par agent. Le proxy bloquait la plupart des pages sources. Seuls quelques documents GitHub ont été lus directement.

## 1. Moteurs de service et optimisation (11 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Entraînement de drafts EAGLE par client | a, b, d | Baseten, Together ATLAS (gratuit), Speculators, SpecForge |
| Transfert KV en delta pour P/D désagrégé | a, b, d | Fonction ajoutable par Dynamo, llm-d, LMCache (RFC #7861) |
| KV cache partagé entre nœuds | a, b, d | NVIDIA BlueField-4 + 12 fabricants de stockage, LMCache, WEKA |
| Service de quantification NVFP4 + QAD | b, d | NVIDIA ModelOpt ; Neural Magic, SqueezeBits, Eigen rachetés |
| Optimisation d'inférence en service | b, d | Eigen → Nebius (643 M$), CentML → NVIDIA, Modular → Qualcomm |
| Moteur déterministe complet | a, d | Flags natifs vLLM/SGLang ; SR 26-2 exclut la GenAI |
| Vérificateur de conformité des endpoints | a, d | OpenRouter Auto Exacto, K2 Vendor Verifier |
| Sortie structurée pour schémas complexes | a, b, d | XGrammar par défaut, llguidance, dottxt |
| Configurateur P/D | a, b, c, d | Dynamo AIConfigurator, SLA Planner |
| Équilibrage d'experts MoE | a, b, d | EPLB dans vLLM, llm-d |
| Distribution vLLM/SGLang entreprise | a, b, d | Inferact, RadixArk, Red Hat |

## 2. Économie de l'infrastructure (18 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Passerelle LLM multi-fournisseurs | b, d | Portkey → Palo Alto, OpenRouter → Stripe (>7 Md$), LiteLLM |
| Routeur appris qualité/coût | a, b, d | OpenRouter, Not Diamond, Martian |
| Vérification de fidélité des endpoints | a, d | OpenRouter Auto Exacto, Artificial Analysis |
| Audit des tokens cachés facturés | a, d | Aucune demande prouvée ; TEE / comptabilité côté fournisseur |
| Snapshot GPU pour cold start | a, b, d | NVIDIA Dynamo Snapshot, Modal |
| Isolation du GPU fractionnaire | b, c, d | NVIDIA KAI + HAMi |
| Autoscaler piloté par SLO | a, b, c, d | Dynamo SLA Planner |
| Détection de corruption silencieuse GPU | a, d | NVIDIA NVSentinel, Fleet Intelligence |
| Attribution des coûts LLM | a, b, d | Finout, CloudZero, Vantage |
| Refacturation par token en auto-hébergé | a, b, d | OpenCost 1.121.0, Cast AI |
| Metering / facturation en tokens | b, d | Metronome → Stripe (~1 Md$), m3ter → Salesforce |
| Gestion des engagements IA (PTU) | a, d | Azure natif, Finout |
| Indice de prix GPU et dérivés | b, d | Ornn + ICE, Silicon Data + CME |
| Indice de prix des tokens | a, b, c, d | Silicon Data |
| Revente de capacité réservée | a, b, c, d | SF Compute, Compute Exchange |
| Courtier GPU multi-cloud | b, c, d | NVIDIA DGX Cloud Lepton, Shadeform, Vast.ai |
| Valeur résiduelle GPU | a, b, c, d | Silicon Data |
| Inférence souveraine UE | a, b, d | Mistral Compute, OVHcloud |

## 3. Matériel, kernels et edge (12 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Agent de génération de kernels multi-puces | a, d | AMD GEAK, AWS Transform, Mako, Standard Kernel, Wafer |
| Vérificateur de kernels écrits par IA | a, d | Brique interne de chaque générateur |
| Non-régression numérique matériel/compilateur | a, d | Fait en interne par l'acheteur (Anthropic) |
| Traducteur CUDA → ROCm | a, d | Spectral SCALE, AMD, Modular (Qualcomm) |
| PyTorch → TPU hors Google Cloud | a, b, d | Google TorchTPU avec Meta |
| Orchestrateur multi-puces | b, d | Gimlet Labs (3 Md$) |
| SDK LLM sur NPU | a, d | Nexa → Qualcomm, Windows ML, Apple Foundation Models |
| Runtime LLM embarqué auto/robotique | a, b, d | NVIDIA TensorRT Edge-LLM |
| Contrôleur d'énergie par token | a, b, d | NVIDIA Max-Q, WPPS, Mission Control |
| Flexibilité réseau électrique | a, d | Emerald AI, coalition Google/NVIDIA |
| Portage CUDA → Ascend | a, d | Huawei CANN open source |
| Portage de modèles pour ASIC | a, b, d | Piles maison, Mako, Gimlet |

## 4. Runtime des agents (16 candidats)

| Candidat | Critère éliminatoire | Raison |
|---|---|---|
| Optimiseur de taux de cache multi-fournisseurs | a, d | Cache automatique + diagnostics Anthropic, routage GPT-5.6 |
| Compaction de contexte | a, b, d | Native chez OpenAI et Anthropic |
| Mémoire d'agent | a, b, d | AgentCore Memory, Cloudflare, Anthropic, Mem0 |
| Sandbox micro-VM | b, d | E2B, Daytona, Modal, Vercel, Cloudflare, AgentCore |
| Sandbox avec fork | a, b, d | Daytona, Morph |
| VM de bureau pour computer-use | a, b, c, d | Cua, Windows 365 for Agents |
| Proxy de credentials pour sandbox | a, b, d | Cloudflare Outbound Workers, Vercel |
| Navigateur headless vérifié | a, b, d | Browserbase, Kernel, AgentCore Browser |
| Idempotence des appels d'outils | a, d | Patterns Temporal ; facile en interne |
| Exécution durable pour agents | b, d | Temporal (>250 M$ ARR), Inngest, Restate, DBOS |
| Tracing d'agents | a, b, d | Langfuse → ClickHouse, Helicone, Portkey |
| Plafond de budget par exécution | a, b, d | Claude Agent SDK, LiteLLM, Managed Agents |
| Détection de fin de tour (voix) | a, b, d | Deepgram Flux, LiveKit, Pipecat |
| Orchestration voix temps réel | b, d | LiveKit, Pipecat, Vapi, Retell |
| Tiers de KV cache pour agents auto-hébergés | a, b, d | Dynamo, LMCache |
| Moniteur de context rot | a, d | Fonction triviale des plateformes d'observabilité |

## Constat

La valeur se concentre chez quatre groupes d'acteurs :

- **Plateformes d'inférence** : Fireworks, Baseten, Together, Modal.
- **NVIDIA**, qui publie gratuitement Dynamo, KAI et NVSentinel, et rachète CentML, Run:ai, Lepton et Groq.
- **Fabricants de puces** : Qualcomm et AMD.
- **Acheteurs stratégiques** : Stripe, Palo Alto, Salesforce, ClickHouse.

Chaque brique technique isolée devient une fonction gratuite ou rachetée.
