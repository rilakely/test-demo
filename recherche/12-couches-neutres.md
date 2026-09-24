# Idées « couche neutre » sur le modèle Dropbox / Zapier / PagerDuty (au 2026-09-24)

**Le schéma :** chaque plateforme propose la fonction, mais seulement dans son propre écosystème. Les clients utilisent plusieurs plateformes concurrentes. Il leur faut donc une couche neutre, qu'aucune plateforme ne construira bien parce qu'elle devrait prendre en charge ses rivaux.

**Précédents YC :**
- Dropbox (S07) : Steve Jobs a dit « a feature, not a product » ([TechCrunch 2012](https://techcrunch.com/2012/01/23/dld-2012-drew-houston-yes-steve-jobs-called-dropbox-a-feature/amp)).
- Zapier (S12) : ~310 M$ d'ARR en 2023.
- PagerDuty (S10) : IPO en 2019, 492,5 M$ de chiffre d'affaires sur l'exercice 2026.

## Idées retenues (classées)

### 1. Registre neutre de l'usage de l'IA pour les petits et moyens cabinets d'avocats
- **Plusieurs outils en même temps :** 69 % des professionnels du droit utilisent des IA généralistes (ChatGPT, Gemini, Claude), contre 31 % un an plus tôt. Seuls 46 % des cabinets en ont adopté une au niveau du cabinet (rapport 8am, 03/2026).
- **Chaque plateforme ne trace que son outil :** Claude Compliance API (05/2026, Claude Enterprise seulement), Microsoft Purview (écosystème Microsoft seulement). Aucune ne produit un livrable par dossier ou par acte de procédure.
- **La douleur :**
  - Plus de 1 600 décisions de justice pour hallucinations d'IA.
  - 143 ordonnances de juges fédéraux sur l'IA ; environ 61 exigent de déclarer l'usage et de certifier la vérification des citations.
  - Les assureurs en responsabilité civile professionnelle exigent l'inventaire des outils (09/2026).
  - Exclusions d'IA générative par Verisk (01/2026).
- **Concurrents :**
  - PortEden : journalisation orientée sécurité, pas de rattachement au dossier.
  - Tracelaw : vérifie le document final, à 99–500 $/mois.
  - Harmonic : cible les grandes entreprises et la prévention des fuites de données.
  - Aucun vendeur de la fonction exacte trouvé.
- **Premier produit (6-8 semaines) :**
  - Capture sur tous les assistants, par extension de navigateur, import des exports et API de conformité.
  - Rapport de provenance pour chaque acte déposé.
  - Classeur prêt pour le renouvellement d'assurance.
- **Risques :**
  - Secret professionnel : le journal pourrait devoir être communiqué à la partie adverse, d'où un stockage local et chiffré.
  - Les cabinets se regroupent sur un seul outil intégré (Clio Work, Smokeball Archie).
  - Absorption par Clio ou 8am.

### 2. Routeur neutre des approbations humaines pour agents d'entreprise (« PagerDuty des approbations »)
- **Plusieurs plateformes en même temps :** l'entreprise médiane utilise 3 plateformes d'agents. Copilot Studio est présent dans 70 % des cas, OpenAI Agents SDK dans 68 %, Claude dans 47 % (VentureBeat Pulse, 08/2026).
- **Chaque plateforme garde ses approbations chez elle :** Copilot → Teams/Outlook, Agentforce → Omni-Channel Salesforce, OpenAI SDK → aux applications de les gérer.
- **Pourquoi maintenant :** le rapport FINRA 2026 (09/12/2025) demande des procédures de supervision humaine et la traçabilité des actions des agents.
- **Concurrents :**
  - gotoHuman et Praesidia.
  - HumanLayer (YC F24) a pivoté vers les outils de code : signal négatif sur la demande côté développeurs.
  - Microsoft Agent 365 et ServiceNow AI Control Tower couvrent le registre, pas le routage neutre des approbations.
- **Risques :**
  - Les actions natives d'Agentforce ne passent pas par MCP : la barrière reste consultative.
  - Teams Approvals peut devenir « suffisant ».

### 3. Budget unique sur plusieurs agents de code (encombré)
- **La douleur :**
  - Uber a épuisé son budget IA 2026 en 4 mois, puis a plafonné à 1 500 $ par mois et **par outil** ([Bloomberg](https://www.bloomberg.com/news/articles/2026-06-02/uber-caps-usage-of-ai-tools-like-claude-code-to-cut-costs)).
  - Gartner (06/2026) : 23 % des organisations dépensent 200 à 500 $ par développeur et par mois.
  - Copilot passe entièrement à l'usage le 01/06/2026.
- **Déjà occupé :**
  - devbudget (open source, « one budget and one policy file » pour Claude Code, Cursor, Gemini CLI et Codex CLI).
  - Airia (passerelle).
  - Des startups YC : Mentlio, Carrot Labs, Allowance.
  - Des outils FinOps : CloudZero, Finout, Flexera.

## Piste NERC réorientée
La conformité récurrente est un marché petit et tardif : ~50–230 M$/an, à partir de 2027-2028. La variante qui a du sens vise à **accélérer la mise sous tension** des datacenters :
- produire des modèles électriques conformes aux exigences d'ERCOT (PGRR144) et de NERC (CLO-001) ;
- piloter les dossiers ERCOT et PJM jusqu'à leur acceptation.

Un retard coûte ~4,5 M$ par semaine pour 100 MW, au loyer de colocation CBRE (valeur plafond). Conditions : un fondateur ingénieur réseau, un accès aux données d'essai des fabricants d'onduleurs, et des développeurs de datacenters pilotes.

## Écartées
Mémoire portable grand public (3 % de payants, extension Mem0 archivée) ; présence marchande sur les agents d'achat (Shopify Agentic Storefronts, Profound) ; inventaire d'agents (Agent 365, ServiceNow, MuleSoft) ; comptabilité des coûts d'agents (Flexera) ; synchronisation des connaissances (Glean, 300 M$ d'ARR) ; provenance du code IA (Entire, 60 M$ en seed ; standard Agent Trace) ; règles partagées entre agents de code (AGENTS.md à la Linux Foundation) ; front desk IA pour TPE (Yext ; multi-canal non prouvé).
