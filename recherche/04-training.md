# Couche 4 — Training (au 2026-09-24)

**Résultat : aucun projet ne passe les 4 critères (0 / 31 candidats).**

Chaque candidat est évalué sur les 4 critères, avec une raison pour chacun.

- **P** = réussite, **É** = échec.
- Ordre des colonnes : a / b / c / d.

## 1. Fiabilité de l'entraînement distribué (11)

| Candidat | a | b | c | d |
|---|---|---|---|---|
| Localisation des GPU à corruption silencieuse (rejeu) | É aucun vendeur ni ARR | P les microbenchmarks ratent >60 % des GPU fautifs (OSDI'26) | P OSDI 07/2026, appel OCP 01/2026 | É rerun state machine de Megatron (NVIDIA), SDCHunter interne ByteDance |
| Reprise sans checkpoint / migration live | É Clockwork 20,6 M$ levés, pas d'ARR | É torchft, TorchPass, NVRx | P TorchPass GA 03/2026 | É Clockwork, torchft, NVRx, NVIDIA AJR |
| Attribution des stragglers | É aucun ARR | É NVRx, CoreWeave, AJR | P OSDI'25 | É NVIDIA, CoreWeave |
| Agent de cause racine des blocages NCCL | É aucune offre payante | É Flight Recorder, NCCL Inspector, NVRx | P NVRx 09/2026 | É Meta, NVIDIA, PyTorch |
| Tri des pics de loss + rollback | É aucun vendeur | É (doute) Megatron, NVRx | P ICLR 2026 | É intégré aux frameworks, W&B |
| Orchestrateur multi-datacenter (DiLoCo) | É levée ≠ ARR | É Decoupled DiLoCo, torchft, Psyche | P DeepMind 04/2026 | É Google, Prime Intellect, Nous |
| Ordonnanceur gang / Slurm-sur-K8s | P Run:ai 700 M$ | É KAI, Kueue, Soperator, SUNK | P SchedMD → NVIDIA 12/2025 | É Run:ai et SchedMD rachetés |
| Autotuner de parallélisme | É (doute) CentML plus large | É (doute) outils académiques | P 03/2026 | É CentML racheté puis fermé |
| Burn-in et santé de cluster | É aucun ARR | É NVSentinel, Together, CoreWeave | P 2026 | É NVIDIA, Together, CoreWeave |
| Mesure tierce du goodput (SLA) | É aucun paiement | É lib Google open source, SemiAnalysis | P ClusterMAX 3.0 09/2026 | É SemiAnalysis, GPU Proof |
| Prédiction des flaps IB/NVLink | É aucun ARR | É NVIDIA UFM Cyber-AI | É aucun déclencheur vérifié | É NVIDIA UFM, Clockwork FleetIQ |

## 2. Infrastructure du post-training RL (11)

| Candidat | a | b | c | d |
|---|---|---|---|---|
| Middleware de synchro des poids | É aucun comparable | É API RL natives de vLLM, RDT, checkpoint-engine | P vLLM 05/2026 | É vLLM, Moonshot, Meta, TRL (open source) |
| Accélérateur de rollouts longue traîne | É aucun revenu | É APRIL dans slime, RollPacker | P 2025–2026 | É Together DAS |
| Correcteur de cohérence numérique trainer/inférence | É aucun budget | É TIS/MIS dans veRL, mode bit-exact de vLLM | P 11/2025, 08/2026 | É vLLM, SkyRL, veRL |
| Stabilisation du RL sur MoE (R3) | É aucun comparable | É R2/R3 dans veRL | P 10/2025 | É Megatron |
| Détection de reward hacking pendant le RL | É W&B adjacent ; Osmosis 7 M$ levés | P détection heuristique et contournable | P 11/2025 | É Osmosis, HUD, W&B, Prime Intellect |
| Flotte de sandboxes pour rollouts | P Modal ~300 M$ ARR, >1/3 sandboxes | É Modal : 50 000 sandboxes concurrentes | P CoreWeave 05/2026 | É Modal, E2B, Daytona, CoreWeave |
| RL managé pour entreprises | P Applied Compute ~50 M$ ; Predibase 109 M$ | É plus de 10 offres | P Bedrock 02/2026 | É Tinker, OpenAI, AWS, Fireworks, River, Applied Compute |
| Service de juge / reward model | É aucun comparable | É Fireworks, Bedrock | P 2025 | É intégré chez Fireworks et Bedrock |
| Rollout FP8/NVFP4 | É aucun comparable | É veRL, NeMo-RL, SkyRL | P 2026 | É NVIDIA, Anyscale, ByteDance |
| Rollouts sur GPU hétérogènes | É aucun ARR | É AReaL-Hex, ECHO-2 | P 2025–2026 | É Prime Intellect, AReaL |
| Équilibrage générateur/trainer | É aucun comparable | É PipelineRL, Libra | P 06/2026 | É frameworks, Tinker, Prime |

## 3. Économie et outillage des runs (9)

| Candidat | a | b | c | d |
|---|---|---|---|---|
| Registre de compute/énergie « AI Act » | É aucun vendeur ; ~12 développeurs >10^25 | É (doute) | P sanctions AI Office 08/2026 | É Modulos, Credo AI, W&B, fait en interne par les labs |
| Mesure certifiée énergie/carbone par run | É outils gratuits | É manque réglementaire, pas technique | P 08/2026 | É W&B, CodeCarbon |
| Suivi d'expériences (remplaçant de Neptune) | P W&B >100 M$ ARR (rapporté) | É aucun manque documenté | P fermeture de Neptune 03/2026 | É Pluto (YC), Minfx, W&B, MLflow, Comet |
| Prévision par lois d'échelle / runs proxy | É aucun vendeur | P prévisible dans 39 % des cas seulement | P 2025–2026 | É labs en interne |
| μP-as-a-service | É aucun paiement | P ruptures MoE, weight decay | P 12/2025 | É méthodes ouvertes, faisable en interne |
| Analytique de goodput / stragglers | É aucun ARR | É (doute) NVRx, HyperPod | P 12/2025 | É NVIDIA, AWS, CoreWeave, Clockwork |
| Stockage de checkpoints dédupliqué | É aucun chiffre | É (doute) éditeurs de stockage | P lakeFS/DVC 11/2025 | É lakeFS, VAST, Scality, W&B |
| Estimateur de coût + courtier GPU | É valorisation ≠ ARR | É aucun manque | P 12/2025 | É SF Compute, neoclouds |
| Usine à modèles souverains | É les montants payent l'infrastructure | É aucun manque | P 2026 | É Telekom AI Factory, NVIDIA NeMo |

## Constat

Dans cette couche, la valeur est captée par trois groupes :

- **NVIDIA** : NVRx, Mission Control, UFM, et les rachats de Run:ai et de SchedMD.
- **Les frameworks open source** : torchft, veRL, slime, vLLM, SkyRL.
- **Les plateformes financées** : Modal, Applied Compute, Tinker, W&B/CoreWeave.

Là où le manque technique est réel (corruption silencieuse des GPU, reward hacking, lois d'échelle), aucun budget n'est prouvé, et les acheteurs (labs, néoclouds) le construisent en interne.
