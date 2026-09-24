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
