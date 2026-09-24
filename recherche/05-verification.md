# Revérification du critère (d) — 60 candidats (au 2026-09-24)

**Périmètre :** tous les candidats des couches 1 à 4 qui réussissaient déjà (b) et (c). Ce sont les seuls qu'une preuve (d) erronée aurait pu éliminer à tort.

**Question posée pour chacun :** la preuve (d) citée existe-t-elle, et couvre-t-elle la fonction exacte ?

**Résultat : 26 confirmés, 27 partiels, 7 infirmés. Aucun verdict final ne change.** Chaque candidat partiel ou infirmé a été réévalué et échoue quand même, avec d'autres preuves ou sur (a).

## Ce qui était faux dans la première passe

| Couche | Candidat | Preuve citée | Problème | Nouvelle preuve |
|---|---|---|---|---|
| Données | Arbitrage des désaccords d'experts | « fait en interne » | aucune source | Label Studio Enterprise, Snorkel |
| Données | Détection de LLM dans les données « humaines » | Cleanlab → Handshake | Cleanlab ne détecte pas le texte IA | Pangram, Prolific |
| Modèle | Re-basing LoRA sans données | Datawizz | Datawizz ré-entraîne avec les données | aucune ; échec sur (a), aucun payeur |
| Modèle | Validation des juges LLM | prometheus-eval, verdict | ils construisent des juges, ils ne les valident pas | LangSmith Align Evals |
| Modèle | Moniteur par activations | « seuls les détenteurs des poids » | argument sans source | Goodfire (1,25 Md$) vend des sondes en production |
| Inférence | Vérificateur de kernels écrits par IA | « interne aux générateurs » | aucune source | Meta KernelBench-Verified (MIT) ; échec sur (a) |
| Inférence | Moniteur de context rot | « trivial » | aucune source | échec sur (d) : Arize → Dynatrace (915 M$), Langfuse → ClickHouse |
| Inférence | Traducteur CUDA → ROCm | GEAK, Modular | ce ne sont pas des traducteurs | SCALE, AMD HIPIFY |
| Données | Recherche de scénarios de conduite | « Basis » | nom du produit faux | Applied Intuition Data Explorer |

## Manques techniques réellement non couverts (preuves partielles)

Ces fonctions précises ne sont proposées par personne à ce jour. Elles échouent toutes sur (a) : aucun payeur chiffré. La plupart échouent aussi sur (d), avec un doute sur un acteur en place capable de l'ajouter.

| Candidat | Partie non couverte | (a) | (d) |
|---|---|---|---|
| Inférence déterministe | déterminisme + décodage spéculatif (vLLM #27433, SGLang #13123) | É aucun payeur | É vLLM l'a dans sa liste de travaux |
| SDC en flotte d'inférence | calcul faux sans signal d'erreur (NVSentinel ne le fait pas) | É proteanTecs = IP sur puce, adjacent | É (doute) NVIDIA possède la télémétrie |
| Engagements IA (PTU) | achat/réallocation autonomes multi-fournisseurs | É | É (doute) Finout, ProsperOps |
| Idempotence des outils d'agent | relances décidées par le modèle | P Temporal >250 M$ ARR | É Temporal/Restate peuvent l'étendre |
| Détection du reward hacking en RL | détection dédiée (Osmosis ne la fait pas) | É budgets RL = environnements | É (doute) plateformes RL, labs |
| Rollback après pic de loss | moteur de décision de rollback (NVRx ne le fait pas) | É | É (doute) NVIDIA Megatron/NVRx |
| Filtre anti-régurgitation | non-littéral (Azure, Gemini, Patronus = littéral) | É | É (doute) Copyleaks, Microsoft, Google |
| Attribution paramétrique (texte) | texte (Sureel/Musical AI = musique) | É | É (doute) ProRata adjacent |
| Migration des fine-tunes OpenAI | certificat de parité, fine-tune OpenAI comme teacher | É marché en contraction | É CoreWeave/W&B (ex-OpenPipe), Bedrock |
| Effacement RGPD vectoriel | multi-index, caches, vecteurs HNSW persistants | É | É (doute) faisable en interne |
| QA des graders RFT entreprise | Sepal faisait des données pour les labs, pas de la QA | É | É (doute) |

## Acteurs et chiffres manqués par la première passe

- **Vaudit :** audit des factures IA.
- **Pangram :** détection de texte IA, dont les données RLHF.
- **Snorkel :** validation de rubrics.
- **LivePerson Syntrix :** simulation d'utilisateurs.
- **Werk24 :** lecture de plans.
- **Zoo.dev, AdamCAD :** texte vers CAO.
- **Keystone.ai :** demande intermittente.
- **Swarm Orchestrator, Pyor :** triche aux tests.
- **Qualcomm ← Modular :** environ 3,9 Md$ (selon qz.com et Yahoo Finance) ; la couche inférence avait noté environ 3,1 Md$, montant à vérifier.
- **Palo Alto ← Protect AI :** 700 M$ selon un 10-Q de la SEC ; la couche modèle cite 635 M$ d'après un 10-K. Écart non résolu.
- **Dynatrace ← Arize :** 915 M$.
