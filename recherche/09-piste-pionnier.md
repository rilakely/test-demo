# Piste pionnier (au 2026-09-24)

Critères :
- **P1.** Personne ne vend la fonction exacte, vérifié par au moins 3 recherches ciblées et par lecture du code des projets amont concernés.
- **P2.** Un acheteur précis a le problème et paie déjà pour des choses voisines.
- **P3.** Un premier produit peut être construit en 4 à 8 semaines et vendu sans levée de fonds.

**Résultat : 2 pistes retenues (1 et 2), avec réserves. Les pistes 3 et 4 ont été écartées après revue : pas de preuve de paiement pour la 3, demande suspendue à une règle non finalisée pour la 4. Aucune piste retenue n'a encore de preuve directe d'achat.**

## Pistes retenues (classées)

### 1. Détection en ligne des GPU qui calculent faux (SDC) dans les parcs d'inférence

- **Problème.** Un GPU défectueux renvoie des résultats faux sans aucun signal d'erreur.
  - Chez Meta, environ 1 machine sur 1 000 est touchée.
  - L'entraînement de Llama 3 a connu 6 incidents SDC en 54 jours.
  - En inférence quantifiée INT8/INT4, les accumulateurs INT32 des tensor cores n'ont ni parité ni ECC ([arXiv 2609.19743](https://arxiv.org/html/2609.19743), 09/2026).
- **Acheteur.** Neoclouds et loueurs de GPU de taille moyenne, fournisseurs d'API d'inférence, clusters d'entreprise.
- **Pourquoi personne ne le fait.**
  - NVIDIA NVSentinel ne surveille que l'ECC.
  - DCGM ne fait que du diagnostic à la demande.
  - NVIDIA Cluster Readiness Engine et SemiAnalysis ClusterMAX ne testent pas l'exactitude numérique.
  - Le seul projet qui fait la fonction exacte, CTO92/Sentinel, est un alpha avec 1 étoile. Code lu sur GitHub.
  - Tout le reste relève de la recherche : [SCOUT](https://arxiv.org/pdf/2608.11034), [ITHICA](https://arxiv.org/pdf/2605.15638).
- **Premier produit (6-8 semaines).** Un agent qui combine :
  - des requêtes canaris à réponse connue injectées dans vLLM ou SGLang ;
  - un rejeu bit à bit sur deux GPU en mode déterministe ;
  - des statistiques d'activation par hôte ;
  - une mise en quarantaine automatique du GPU fautif.
- **Premier client plausible.** Un neocloud qui veut se démarquer sur la notation ClusterMAX 3.0, qui évalue les health checks de 77 fournisseurs.
- **Risques.**
  - NVIDIA : le moteur RAS de Rubin promet une surveillance continue en production, et NVIDIA co-signe le livre blanc OCP « SDC in AI ».
  - Aucun incident d'inférence chiffré n'est public.
  - Prouver le taux de détection demande de vrais GPU défectueux (cartes renvoyées au fabricant).
- **À valider avant de construire.** Entretiens avec 5 neoclouds ou loueurs, et un accès à des GPU renvoyés en RMA.

### 2. Suivi d'intention malveillante répartie sur plusieurs pull requests

- **Problème.** Une attaque découpée en plusieurs PR individuellement anodines passe les relecteurs.
  - Détection de 16 à 22 % en revue de la fenêtre complète, contre 50 à 60 % PR par PR ([PRWeaver](https://arxiv.org/html/2608.02693), 08/2026).
  - Campagne réelle prt-scan : plus de 500 PR malveillantes en 6 vagues depuis le 11/03/2026 et 2 paquets npm compromis ([Wiz](https://www.wiz.io/blog/six-accounts-one-actor-inside-the-prt-scan-supply-chain-campaign)).
  - Précédent xz : 12 à 24 mois de contributions légitimes avant la porte dérobée.
- **Acheteur.** Éditeurs qui maintiennent de gros dépôts publics : entreprises open-core, fondations.
  - Budget voisin : 12,5 M$ de subventions Alpha-Omega/OpenSSF (03/2026).
- **Pourquoi personne ne le fait.**
  - Datadog, Apiiro PRevent et Kusari analysent chaque PR isolément.
  - DevTrace et Superagent notent le contributeur, pas le lien entre les codes de plusieurs PR.
  - Aucun dépôt GitHub ne fait la corrélation entre PR.
  - Les textes de SafeDep et Codacy sont des cadres de réflexion, pas des produits.
- **Premier produit (4-8 semaines).** Une GitHub App qui tient, par dépôt, un graphe des changements sensibles : réseau, exec, CI, scripts de build, binaires de test. À chaque nouvelle PR, elle ré-audite le sous-graphe lié.
- **Risques.**
  - Datadog ou Superagent peuvent l'ajouter comme simple fonction.
  - Les mainteneurs open source paient peu.
  - Aucun incident multi-PR confirmé dans la nature, hormis le schéma xz.
- **À valider avant de construire.**
  - Obtenir le jeu de données PRWeaver.
  - Entretiens avec 5 équipes AppSec d'éditeurs open-core.
  - Vérifier l'éligibilité à Alpha-Omega.

### 3. Audit de reproduction non littérale d'œuvres protégées (intrigue, personnages, paraphrase)

- **Problème.** Les filtres existants ne détectent que le verbatim :
  - Azure protected material : correspondance textuelle, anglais seulement ;
  - Gemini RECITATION ;
  - Patronus CopyrightCatcher.

  Or le juge Stein (SDNY, 27/10/2025) a jugé qu'un résumé reprenant cadre, intrigue et personnages peut être substantiellement similaire.
- **Acheteur.** Éditeurs et cabinets d'avocats qui constituent des preuves. Pas un filtre temps réel pour les déployeurs : le seuil juridique est trop flou et les faux positifs trop coûteux.
- **Pourquoi personne ne le fait.**
  - Aucun outil prêt à l'emploi : [arXiv 2606.31250](https://arxiv.org/html/2606.31250v1), [TianPan](https://tianpan.co/blog/2026-04-19-ai-output-copyright-trap-llm-generated-code).
  - [Copyright Detective](https://arxiv.org/html/2602.05252v1) et [CopyShield](https://arxiv.org/abs/2609.01161) restent des travaux de recherche.
  - Concurrent le plus proche : Vermillio TraceID, avec des « soft bindings » sémantiques et un partenariat avec l'AAP (éditeurs américains). Rien ne montre qu'il couvre l'intrigue et les personnages dans du texte généré par LLM.
- **Premier produit (6 semaines).** Sur un catalogue fourni par le client : extraction des événements et personnages de ses œuvres, sondage du modèle ou de l'application visés, puis score de chevauchement façon CopyBench et rapport de preuves.
- **Risques.** Vermillio, et aucune preuve directe qu'un éditeur paierait.
- **À valider avant de construire.** Entretiens avec 3 éditeurs et 3 cabinets côté demandeurs.

### 4. Contrôle qualité des questionnaires d'autorisation préalable (DTR) : codes LOINC et fidélité à la politique source

- **Problème.**
  - Les questionnaires DTR sont rarement liés aux codes LOINC.
  - La meilleure liaison automatique trouve le bon code du premier coup dans 18,5 % des cas seulement (R@1 = 0,185, [arXiv 2606.15449](https://arxiv.org/abs/2606.15449), 06/2026).
  - L'outil de test officiel Inferno ne vérifie pas la fidélité à la politique source.
- **Acheteur.** Assureurs santé soumis à CMS-0057-F et leurs intégrateurs.
- **Pourquoi personne ne le fait.**
  - Les générateurs revendiquent leur propre traçabilité : Cohere, GenHealth, Smile.
  - Touchstone teste seulement la conformité au format.
  - Aucun contrôle qualité indépendant n'est vendu.
- **Condition bloquante.** DTR n'est obligatoire que si la règle CMS-0062-P est finalisée. Or elle n'est pas finale, les commentaires sont clos depuis le 15/06/2026 et aucune date n'est annoncée ([WEDI](https://www.wedi.org/2026/08/26/cms-0062-information/)).
- **À valider.** Suivre la publication de la règle finale.

## Pistes écartées

| Piste | Raison |
|---|---|
| Inférence déterministe + décodage spéculatif | SGLang gère déjà le radix cache en mode déterministe ; le décodage spéculatif est en PR chez SGLang et vLLM |
| Rollback après pic de loss | Megatron rerun_state_machine ; torchtitan-npu (Huawei) PR #24 du 23/09/2026 |
| Détection du reward hacking en RL | Docent (Transluce, maintenu), rolloutscope, RewardGuard, Prime Intellect |
| Vérification des Agent Cards A2A | Corrigée par le propriétaire de la spec (PR #2099) ; sigstore-a2a ; AgentTrust (Red Hat), MolTrust |
| Garde « exactly-once » pour les appels d'outils | SafeAgent (service hébergé, maintenu), ark-trust |
| Effacement RGPD dans les index vectoriels | BigID annonce la suppression « including vector databases » (30/03/2026) |
| Certificat de parité de migration des fine-tunes | W&B (ex-OpenPipe), FutureAGI, Arthur, outils open source |
| Attribution paramétrique des sources d'entraînement (texte) | Pas d'acheteur : la licence collective CLA/ALCS/PLS répartit sans attribution ; technique immature |
