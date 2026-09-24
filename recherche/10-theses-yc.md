# Thèses de candidature YC (critères YC réels) — au 2026-09-24

**Critères appliqués :**
- problème prouvé ;
- pourquoi maintenant daté ;
- budget existant dans la catégorie ;
- créneau précis où les acteurs en place sont mal placés ;
- premier produit rapide.

**Les concurrents sont permis.** Ils doivent être nommés, avec l'intuition qui permet de les battre. Toutes les sources viennent des fichiers 03 à 09 et des recherches citées ici.

## 1. Contrôle d'exactitude des GPU pour les neoclouds (corruption silencieuse de données)

- **Créneau.** Neoclouds et loueurs de GPU de taille moyenne, notés par SemiAnalysis ClusterMAX 3.0 (77 fournisseurs évalués, health checks inclus). Les hyperscalers ont leurs outils internes, pas eux.
- **Problème prouvé.**
  - Chez Meta, environ 1 machine sur 1 000 est touchée par des corruptions silencieuses.
  - Llama 3 : 6 incidents en 54 jours sur 16 384 H100.
  - En inférence INT8/INT4, les accumulateurs INT32 n'ont ni parité ni ECC ([arXiv 2609.19743](https://arxiv.org/html/2609.19743)).
- **Pourquoi maintenant.**
  - Appel OCP / IEEE Micro de 01/2026.
  - ClusterMAX 3.0 en 09/2026.
  - Passage massif de l'inférence en quantification basse précision.
- **Budget de la catégorie.** Les neoclouds paient déjà burn-in et tests d'acceptation (Crusoe, Together). Trainy (YC) vend de la santé et de l'observabilité de clusters GPU.
- **Intuition face aux concurrents.**
  - NVIDIA NVSentinel ne surveille que l'ECC, et DCGM ne fait que du diagnostic à la demande.
  - NVIDIA NVCRE et ClusterMAX mesurent la performance, pas l'exactitude numérique.
  - Personne ne contrôle en continu l'exactitude du trafic de production.
- **Premier produit (6-8 semaines).**
  - Requêtes canaris dans vLLM/SGLang.
  - Rejeu bit à bit en mode déterministe (SGLang le permet).
  - Statistiques d'activation par hôte.
  - Quarantaine du GPU fautif.
- **Comparables YC.** Trainy ([YC](https://www.ycombinator.com/companies/trainy)).
- **Risque.** Le moteur RAS de NVIDIA Rubin.

## 2. Détection des attaques réparties sur plusieurs pull requests

- **Créneau.** Dépôts publics qui reçoivent des PR d'agents IA : éditeurs open-core, projets à forte audience.
- **Problème prouvé.**
  - La détection tombe de 50-60 % PR par PR à 16-22 % sur une fenêtre de 24 PR ([PRWeaver](https://arxiv.org/html/2608.02693), 08/2026).
  - Campagne prt-scan : plus de 500 PR malveillantes, 2 paquets npm compromis ([Wiz](https://www.wiz.io/blog/six-accounts-one-actor-inside-the-prt-scan-supply-chain-campaign), depuis le 11/03/2026).
  - hackerbot-claw : 02-03/2026.
- **Pourquoi maintenant.**
  - Afflux de PR générées par IA : GitHub permet de restreindre les PR depuis le 13/02/2026.
  - Copilot peut approuver des PR depuis le 01/09/2026.
- **Budget de la catégorie.**
  - Datadog SAST : 25 $ par committer et par mois.
  - GitHub Code Security : 30 $.
  - CodeRabbit : 50 M$ d'ARR.
  - Socket : valorisé 1 Md$.
- **Intuition face aux concurrents.** Datadog, Apiiro PRevent, Kusari et CodeRabbit analysent PR par PR. DevTrace et Superagent notent le contributeur. Personne ne relie le code de plusieurs PR.
- **Premier produit (4-8 semaines).** Une GitHub App qui maintient un graphe des changements sensibles par dépôt et ré-audite le sous-graphe lié à chaque nouvelle PR.
- **Comparables YC.** Superagent ([YC](https://www.ycombinator.com/companies/superagent)).
- **Risque.** Absorption par Datadog.

## 3. Conformité NERC « AI-native » pour les datacenters nouvellement régulés

- **Créneau.** Opérateurs de datacenters de 50 MW et plus raccordés à 100 kV ou plus. Ils vont devenir des entités enregistrées NERC à partir de 2027, alors qu'ils n'ont aujourd'hui aucun programme de conformité, contrairement aux producteurs d'électricité.
- **Problème prouvé.**
  - 1 500 MW décrochés en Virginie (07/2024).
  - Plus de 3 GW décrochés de PJM le 22/07/2026.
  - ERCOT a gelé la mise sous tension d'environ 250 à 300 projets (08/2026).
- **Pourquoi maintenant.**
  - Alerte NERC de niveau 3 (04/05/2026).
  - FERC impose des standards avant le 31/12/2026 ([POWER](https://www.powermag.com/ferc-orders-mandatory-nerc-reliability-standards-for-data-center-and-other-computational-loads/)).
  - Critères d'enregistrement NERC en consultation du 19/08 au 18/09/2026 ; enregistrement à partir de 2027 ([TRC](https://www.trccompanies.com/insights/nercs-steps-toward-regulating-large-loads/)).
  - ERCOT NOGRR282 (07/2026).
- **Budget de la catégorie.**
  - Conformité NOGRR282 : 0,5 à 1 M$/MW, surtout du matériel.
  - Frais d'étude PUCT : 100 000 $ par demande.
  - Les cabinets Keentel, PSC, EPE, TRC et GDS vendent déjà des études.
  - **Budget d'un programme de conformité : non chiffré.**
- **Intuition face aux concurrents.**
  - Les cabinets vendent des études ponctuelles.
  - GridStrong sert surtout les producteurs renouvelables (plus de la moitié des 25 premiers propriétaires).
  - Les datacenters n'ont ni équipe ni historique de conformité NERC. Il faut un programme complet et continu : enregistrement, standards, preuves, modèles.
- **Premier produit.** Un « service AI-native » (catégorie RFS YC été 2026) qui prend en charge le dossier d'enregistrement et le suivi des exigences d'un opérateur, avec des ingénieurs outillés par IA.
- **Comparables YC.** Catégorie « AI-Native Service Companies » de la RFS été 2026.
- **Risques.** GridStrong, dirigé par l'ex-directeur de l'ingénierie de NERC. Nombre d'entités concernées non publié.

## 4. Exactitude des paiements SNAP pour les États hors contrats en place

- **Créneau.** Les 7 juridictions dont la pénalité est reportée (AK, DE, DC, GA, IL, NM, OR), et les États qui visent le taux FY2027, qui fixe la pénalité FY2030. Il s'agit des États pas encore engagés avec Maximus, Gainwell, Deloitte ou SAS.
- **Problème prouvé.**
  - Taux d'erreur national FY2025 : 10,62 %. Seules 10 juridictions sur 53 sont sous 6 %.
  - Coût en régime permanent pour les États : 11,1 Md$/an. Calcul PolicyEngine sur les données FNS du 24/06/2026.
- **Pourquoi maintenant.** La loi OBBBA §10105 met une part des prestations à la charge des États dès FY2028. Les coûts administratifs passent à 75 % pour les États en FY2027.
- **Budget de la catégorie.**
  - Floride : 4 M$.
  - Virginie-Occidentale : 0,876 M$ (achat d'urgence).
  - Kansas : 3 à 5 M$ évoqués.
  - Extrapolation : 53 à 143 M$/an de dépenses logicielles.
- **Intuition face aux concurrents.** Maximus, Gainwell et Deloitte vendent la revue avant paiement dans leurs propres systèmes. **Aucune faiblesse mesurée de leurs outils n'est publiée : c'est le point faible de cette thèse.**
- **Premier produit.** Un moteur de règles et un LLM sur le fichier QC public (44 891 revues FY2024), pour localiser les causes racines par type d'erreur.
- **Comparables YC.** Catégories « AI for Government » et « Infra for Government Fraud Hunters » de la RFS printemps 2026 ; startups GovTech YC ([liste](https://www.ycombinator.com/companies/industry/GovTech)). Aucune startup YC spécifique au SNAP trouvée.
- **Risques.** Achats publics lents. Contestation par l'ACLU (Michigan, 08/2026).

## Écartées pour cette liste, faute de preuves suffisantes

| Idée | Raison |
|---|---|
| Autorisation préalable des médicaments | Develop Health, Silna, Banjo ; créneau non documenté |
| Migration des fine-tunes OpenAI | Marché en contraction ; W&B en place |
| Conformité SB 243 des applications compagnons | Budget non sourcé |
| Documentation « medically frail » côté cliniques | Budget non sourcé ; Fortuna (YC) déjà sur le côté bénéficiaire |
