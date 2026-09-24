# Piste YC — 5 fiches approfondies (au 2026-09-24)

Critères utilisés :
1. Problème urgent pour un acheteur précis.
2. Pourquoi maintenant, daté.
3. Faiblesse mesurée des concurrents (l'insight). Les concurrents sont permis.
4. Budget chiffré et sourcé.
5. Premier produit (MVP) faisable en quelques semaines.
6. Risque plateforme : éliminatoire si le détenteur des données ou de la distribution fait déjà la fonction.
7. Alignement avec les listes YC 2026 (RFS).

**Résultat : aucune fiche retenue.**

| Idée | 1 | 2 | 3 | 4 | 5 | 6 | Raison décisive |
|---|---|---|---|---|---|---|---|
| Conformité réseau des datacenters IA | P (réserve) | P fort : PJM −3 GW le 22/07/2026, pause ERCOT 08/2026, enregistrement NERC proposé | É | É | P technique | É | Goulot = étalonnage matériel chez les fabricants, une fois par produit ; ERCOT fournit DMView/PMView ; Schneider/ETAP GridCode intégré à NVIDIA DSX ; GridStrong vend des modèles « audit-ready » |
| Compilateur de politiques d'autorisation préalable (PA) → CRD/DTR/CQL | P faible | P CMS-0062-P 04/2026 | É | É | P technique | É | InterQual Exchange (03/2026), MCG Path, Availity ; Cohere Policy Studio convertit en CQL depuis 12/2025 ; GenHealth, Smile |
| Réduction des erreurs de paiement SNAP | P | P OBBBA | É | P partiel : 53–143 M$/an | Partiel : données bloquantes | É | Maximus (01/2026), Gainwell (12/2025), Deloitte (09/2025) ; SAS a gagné WV et NV ; pénalités 2028-2029 figées au 30/09/2026 |
| Défense contre les PR malveillantes | P partiel | P | É partiel | É | P | É partiel | Datadog Malicious PR protection (préversion depuis 10/2025) ; GitLab détecte l'injection de prompt ; seul trou mesuré : l'intention répartie sur plusieurs PR (PRWeaver, labo) |
| Exemptions « medically frail » Medicaid | P | P : règle intérimaire du 06/2026 | P fragile | É | Partiel | É | Gainwell a lancé l'outil le 31/03/2026, offert gratuitement à tous les États |

Notation : P = passe, É = échoue.

Constats utiles pour la suite :
- Terminologie des questionnaires d'autorisation préalable (bibliothèque HL7 CDS-Library) : 7,7 % seulement des items à réponse portent un code LOINC. La meilleure liaison automatique atteint un R@1 de 0,185. Inferno ne teste pas la fidélité à la politique source.
- Intention malveillante répartie sur plusieurs PR : détection de 16 à 22 % en revue de fenêtre (PRWeaver).
- NERC propose d'enregistrer les sites ≥50 MW raccordés à ≥100 kV (commentaires du 19/08 au 18/09/2026).
