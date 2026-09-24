# Bilan (au 2026-09-24)

**Aucun projet ne passe les 4 critères : 0 / 332 candidats, sur 5 couches et 6 domaines transverses.**

| Fichier | Périmètre | Candidats | Passent |
|---|---|---|---|
| 01-donnees.md | Données | 64 | 0 |
| 02-modele.md | Modèle | 59 | 0 |
| 03-inference.md | Inférence | 57 | 0 |
| 04-training.md | Training | 31 | 0 |
| 05-verification.md | Revérification du critère (d) sur 60 candidats | — | aucun verdict changé |
| 06-domaines-transverses.md | Neuromorphique, cryptographie, cybersécurité, sécurité des agents, architecture logicielle, datacenters | 68 | 0 |
| 07-application.md | Santé, finance/juridique, industrie/public, entreprise, grand public | 67 | 0 |

## Pourquoi tout échoue

Le critère (d) bloque presque tout, de trois façons :

1. **Rachat pour devenir une fonction.** Au moins 50 rachats de 2025-2026 relevés, par exemple :
   - Cloudflare, Nvidia, Palo Alto, CrowdStrike, Snowflake, Stripe, Dynatrace, Qualcomm, Marvell, Trimble ;
   - Cisco, Okta, Cyera, Waystar, Applied Systems, Zendesk, Deel, Postman, Anthropic, Mercor.
2. **Fonction intégrée par la plateforme qui détient les données ou les poids :**
   - Epic, Microsoft, OpenAI, Anthropic, NVIDIA (Dynamo, NVCRE, Mission Control) ;
   - Salesforce, Databricks, Snowflake.
3. **Startup financée qui vend déjà la fonction exacte**, y compris sur les niches réglementaires les plus récentes : GridStrong, Vaudit, Keycard, Smile, Warden, Cybellum, Gumshoe.

Là où le manque technique (b) et le déclencheur (c) sont réels mais la fonction non couverte, aucun budget n'est prouvé (a). Exemples :
- déterminisme avec décodage spéculatif ;
- corruption silencieuse des GPU ;
- reward hacking ;
- CQL exécutable pour les autorisations préalables ;
- confiance entre agents (A2A).

## Limites de la méthode

- Presque toutes les preuves viennent d'extraits de résultats de recherche. Le proxy bloquait la plupart des pages sources.
- La première passe avait raté des acteurs (par exemple Vaudit) et contenait 7 preuves (d) fausses sur les 60 revérifiées. La revérification n'a changé aucun verdict, mais la fiabilité d'une preuve isolée reste moyenne.
- La règle « au moindre doute, échec » élimine aussi les candidats dont le seul défaut est un doute, pas une preuve contraire.

## Réévaluation avec le critère (d) tel que formulé (acteurs en place uniquement)

Les protocoles des agents ajoutaient « ou une autre startup » au critère (d). Ce durcissement ne figure pas dans le critère d'origine. Les 332 verdicts ont été repris avec la formulation d'origine : une startup concurrente non dominante n'élimine plus un candidat.

Filtre appliqué : (a), (c) et (b) réussis, et (d) en échec uniquement à cause d'une startup non dominante. Aucun candidat ne passe ce filtre.

**Candidats qui réussissent (a).** Ils sont une quarantaine (23 dans les fichiers 04 à 07, environ 17 dans les rapports des couches 01 à 03). Tous échouent aussi sur (b) ou (d) pour l'une de ces raisons :
- un acteur en place fait déjà la fonction (NVIDIA, Microsoft, Epic, Okta, AWS, DoubleVerify, Temporal) ;
- un leader dominant la couvre (Modal, Pindrop, Horizon3, BioCatch, Profound, Exa/Parallel) ;
- la niche a été rachetée (Run:ai, Quantifi, Celestial, Koi, Seraphic, Iodine, Permiso, Tavily, Human Native, Gretel, Protect AI, Portkey, Metronome) ;
- il n'y a pas de manque technique (post-Stainless, cabinet comptable, BPO vocal, marketplace d'experts, clones d'applications pour le RL).

**Candidats éliminés sur (d) seulement par une startup non dominante.** Ils échouent tous déjà sur (a). Exemples :
- conformité réseau des datacenters (GridStrong) : aucun ARR, l'argent va au matériel et au conseil ;
- audit des factures IA (Vaudit) : environ 0,75 M$ sur l'échantillon ;
- délégation multi-hop entre agents (Keycard).
