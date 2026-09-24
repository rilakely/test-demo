# Idées nouvelles (règle du pionnier), au 2026-09-24

**Règle appliquée.** C'est la logique de Dropbox, Zapier et PagerDuty : l'idée est retenue si **personne ne vend encore la fonction exacte**. Si les grosses sociétés l'intègrent plus tard, on est déjà sur le marché, ou elles rachètent. Des concurrents sur des marchés voisins sont permis.

**Méthode.**
1. Re-vérification de nouveauté de 22 candidats issus des fichiers 01 à 12 (au moins 3 recherches ciblées par candidat, plus GitHub).
2. Génération d'idées nouvelles par questions techniques façon ByteByteGo (comment ça marche, où ça casse, ce qui coûte, ce qui manque), sur 7 thèmes :
   - Skills et MCP ;
   - apps « vibe-codées » ;
   - boucle d'agent ;
   - contrats de schéma ;
   - GraphRAG ;
   - preuve d'humanité ;
   - API LLM côté acheteur.
3. Contre-vérification personnelle des idées du haut de liste.

**Validation.** Uniquement par signaux publics (aucun contact acheteur, pour ne pas divulguer l'idée).

**Limite.** La plupart des preuves sont des extraits de résultats de recherche, car le proxy bloque la majorité des pages. L'absence de résultat n'est pas une preuve d'absence.

**Différence avec le fichier 07.** Plusieurs idées ci-dessous échouaient au critère (d) dans le fichier 07, par exemple le contrôle qualité des scribes et l'audit des résolutions. Sous l'ancienne règle, un acteur voisin suffisait à éliminer. Sous la règle de nouveauté, on regarde seulement si quelqu'un vend la fonction exacte. Exemples :
- Abridge vérifie ses propres notes, pas celles des autres éditeurs.
- Klaus, racheté par Zendesk, note la qualité des conversations. Il ne réconcilie pas la facture.

## Classement (par urgence d'achat)

| # | Idée | Nouveauté | Déclencheur daté | Preuve que l'argent circule déjà | Risque principal |
|---|---|---|---|---|---|
| 1 | Vérification neutre de la fidélité numérique des fournisseurs d'inférence et des migrations de puce | Aucun vendeur | Accord AMD–Anthropic de 2 GW (22/07/2026) ; audit CISPA (03/2026) | Moonshot et MiniMax ont chacun construit leur propre vérificateur | OpenRouter ou ThousandEyes l'étendent |
| 2 | Audit indépendant des « résolutions » facturées par les agents support IA | Aucun vendeur | Surfacturation auto Zendesk (01/2026) ; passage de Fin aux « outcomes » | L'audit de factures IA au succès se vend déjà (Vaudit TokenAudit) | Zendesk ne facture plus que des résolutions vérifiées ; Vaudit peut s'étendre |
| 3 | Poste de travail IA pour vérificateurs CBAM accrédités | Aucun vendeur trouvé côté vérificateur | Guide CE (24/08/2026) ; accréditations (09/2026) ; vérifications dès 01/2027 | Les vérificateurs facturent la vérification ; la pré-vérification est vendue (SGS, RINA) | Peu de recherches (4) ; peu d'acheteurs |
| 4 | Contrôle qualité neutre des notes des scribes IA, tous éditeurs | Aucun vendeur multi-éditeurs | Vérificateur général de l'Ontario (05/2026) | DAX : 600+ organisations clientes | Accès aux transcriptions contrôlé par les éditeurs ; achats hospitaliers lents |
| 5 | Vérificateur indépendant des noyaux GPU générés par IA | Aucun vendeur autonome | KernelBench-Verified (07/2026) | ~68 M$ levés en 2026 par les générateurs | Les générateurs intègrent la vérification |
| 6 | Suivi d'intention malveillante répartie sur plusieurs PR | Aucun vendeur | PRWeaver (08/2026) ; Copilot approuve des PR (09/2026) | Budgets AppSec (Socket, CodeRabbit) | Datadog ou Superagent l'ajoutent |
| 7 | Tri des démonstrations robot par influence sur la réussite | Recherche seulement | CUPID, ATHENA, RoboDrop (06-09/2026) | XDOF a levé 70 M$ pour produire des démonstrations | Les labos le font en interne |
| 8 | Contrôle d'exactitude des GPU en inférence (requêtes canaris, rejeu) | **En partie occupée** : ACE isole par signes avant-coureurs, pas par l'exactitude | arXiv 2609.19743 (09/2026) | Les neoclouds paient le burn-in ; Trainy (YC) | ACE, NVIDIA Rubin RAS |

**Seconde ligue** (nouvelles, mais sans preuve de dépense ou faciles à copier) :
- analyse d'impact des changements de schéma sur prompts et évaluations ;
- surveillance du « context rot » ;
- réconciliation des effets ambigus des agents ;
- transfert de LoRA sans données ;
- prévision par lois d'échelle.

Détails en fin de fichier.

---

## Fiches

### 1. Vérification neutre de la fidélité numérique des fournisseurs d'inférence

- **Question technique d'origine.** Quand le même modèle open-weight est servi sur une autre puce (AMD, TPU, Trainium), une autre version de vLLM ou SGLang, ou chez un autre fournisseur, qu'est-ce qui garantit qu'il calcule la même chose ? Et qui s'en aperçoit ?
- **Problème.**
  - Les écarts de noyaux, la quantification et les bugs d'infrastructure changent les sorties sans erreur visible.
  - Le vérificateur K2 de Moonshot (test du 15/11/2025) : 100 % de schémas d'appels d'outils corrects chez Moonshot et Fireworks, contre 84,63 % chez Together et 87,22 % avec vLLM ([OUVERT](https://github.com/MoonshotAI/K2-Vendor-Verifier)).
  - Un audit CISPA de « shadow APIs » : 45,83 % des endpoints échouent à la vérification d'empreinte du modèle ([IRIS, arXiv 2607.20860](https://arxiv.org/html/2607.20860v1)).
- **Solution.**
  - Des suites « golden » capturent les top-k logprobs et les états cachés sur la pile de référence.
  - Elles sont rejouées sur la pile ou le fournisseur candidat. On mesure la divergence KL, le taux de tokens changés et la schématisation des appels d'outils, puis on fait une bissection couche par couche jusqu'à la première divergence.
  - Le produit fournit un seuil bloquant en CI et un rapport signé par fournisseur.
- **Comment ça résout.** On passe d'un constat tardif (« le modèle semble moins bon ») à une preuve mesurée et localisée, avant la mise en production ou dans le contrat avec l'hébergeur.
- **Concurrents.**
  - Kimi Vendor Verifier et MiniMax-Provider-Verifier : internes, propres à un modèle, et ils testent le comportement, pas les calculs.
  - OpenRouter Exacto : routage interne à OpenRouter.
  - ThousandEyes : détecte qu'un modèle a changé.
  - Tosea et fpverify : empreintes ponctuelles, en open source.
  - AMD publie un [guide de débogage des logprobs](https://rocm.blogs.amd.com/software-tools-optimization/logprob-debug/README.html), pas un produit.
  - Côté recherche : [DiFR](https://arxiv.org/pdf/2511.20621).
- **Qui paie.**
  - Les labos open-weight qui veulent certifier leurs hébergeurs : Moonshot et MiniMax ont déjà payé pour le construire eux-mêmes.
  - Les fournisseurs d'inférence et les neoclouds qui migrent de puce.
  - Les équipes plateforme qui routent vers au moins deux fournisseurs.
- **Potentiel de revenu.** Non chiffré. Seul chiffre de marché : la dépense LLM des entreprises était de 8,4 Md$ au 1er semestre 2025 ([Menlo](https://www.hpcwire.com/aiwire/2025/08/01/menlo-ventures-report-enterprise-llm-spend-reaches-8-4b-as-anthropic-overtakes-openai/)).
- **Pourquoi maintenant.**
  - L'accord AMD–Anthropic de 2 GW de MI450 (22/07/2026) ([AMD](https://ir.amd.com/news-events/press-releases/detail/1292/amd-and-anthropic-announce-strategic-partnership-to-deploy-up-to-2-gigawatts-of-amd-instinct-mi450-series-gpus)).
  - Les méthodes bon marché en boîte noire publiées en 2026 : IRIS, KBF, [Integer Alibi](https://arxiv.org/pdf/2608.13756) et [Silent Hyperparameter](https://arxiv.org/pdf/2605.19537).
- **MVP (6 semaines).** Une CLI et un service de sondes planifiées. Elle prend deux points d'accès (endpoints) ou deux piles et produit un rapport de divergence et une empreinte.
- **Validation sans contact.** Publier gratuitement un classement mensuel des hébergeurs d'un modèle ouvert populaire. Le signal : les réactions publiques des hébergeurs et des labos.

### 2. Audit indépendant des résolutions facturées par les agents support IA

- **Question technique d'origine.** Comment le fournisseur décide-t-il qu'une conversation est « résolue » et facturable ? Qui vérifie ce jugement ?
- **Problème.**
  - Intercom Fin facture 0,99 $ par résultat. Une résolution est « assumée » si le client ne répond plus pendant 24 h, qu'il ait été aidé ou non ([getmacha](https://www.getmacha.com/blog/intercom-fin-pricing)).
  - Intercom retire la facturation seulement si le client revient **dans la même conversation** ([Intercom](https://www.intercom.com/help/en/articles/8205718-fin-ai-agent-outcomes)). S'il revient par e-mail, par téléphone ou par une nouvelle conversation, rien n'est déduit.
  - Des clients contestent publiquement le principe ([fil communauté Intercom](https://community.intercom.com/ask-the-intercom-team-about-fin-54/fin-s-flawed-assumed-resolved-pricing-design-8929)).
  - Le vendeur est juge et partie.
- **Solution.** Un connecteur en lecture seule (Intercom, puis Zendesk, Sierra, Decagon) :
  - il relie chaque résolution facturée aux contacts du même client sur les autres canaux dans les 7 jours, au CSAT et aux réouvertures ;
  - un LLM reclasse la résolution en « réelle » ou « non réelle » ;
  - le produit sort un dossier de contestation chiffré.
- **Comment ça résout.** Le client obtient une mesure indépendante de ce qu'il paie et un dossier opposable pour récupérer des crédits.
- **Concurrents.**
  - Vaudit TokenAudit audite les tokens, pas les résolutions ([BusinessWire](https://www.businesswire.com/news/home/20260630108235/en/)).
  - Confident AI et Cekura font de la QA des agents, pas de la réconciliation de facture.
  - Un seul prototype GitHub à 0 étoile ([Intercom_FRI](https://github.com/Hasnain91169/Intercom_FRI)).
- **Qui paie.** Le VP Support/CX ou le FinOps d'une entreprise qui paie plus de 5 000 résolutions par mois.
- **Potentiel de revenu.** Le précédent sourcé est Vaudit : 1,7 M$ de surfacturations trouvées sur 34 M$ audités, rémunéré au succès. Le montant spécifique aux résolutions n'est pas chiffré.
- **Pourquoi maintenant.**
  - Zendesk facture automatiquement les dépassements, sans plafond, depuis le 01/01/2026 ([Zendesk](https://support.zendesk.com/hc/en-us/articles/9908811576858)).
  - Intercom est passé des « resolutions » aux « outcomes » ([Intercom](https://www.intercom.com/blog/from-resolutions-to-outcomes-evolving-how-fin-delivers-value/)).
- **Risque vérifié.** Depuis 05/2026, Zendesk ne facture plus que les résolutions « Verified », validées par un second LLM de Zendesk ([Zendesk](https://support.zendesk.com/hc/en-us/articles/9570369117338-About-automated-resolution-tiers)). La douleur y est plus faible, mais le vendeur reste son propre vérificateur. La cible prioritaire est Intercom Fin.
- **MVP (4 semaines).** Connecteur Intercom, reclassement, rapport mensuel, rémunération au succès.
- **Validation sans contact.** Publier un calculateur gratuit du « taux de résolutions assumées » à partir d'un export CSV, et mesurer l'usage.

### 3. Poste de travail IA pour vérificateurs CBAM accrédités

- **Question technique d'origine.** Le vérificateur doit contrôler les émissions intrinsèques déclarées par une installation hors UE. Comment le fait-il aujourd'hui, et qu'est-ce qui est manuel ?
- **Problème.**
  - Les premières vérifications portent sur l'année 2026, à partir de 01/2027, avec une visite sur site obligatoire la première période ([CBAM Journal](https://www.cbamjournal.com/post/eu-cbam-verification-2026-2027)).
  - Il faut réconcilier les factures d'énergie, la production, le bilan massique et les valeurs par défaut, puis échantillonner selon le risque.
- **Solution.** Le produit ingère le rapport de l'installation et les pièces justificatives, puis :
  - fait la réconciliation automatique ;
  - applique un plan d'échantillonnage selon le risque ;
  - tient une checklist alignée sur le règlement DR 2025/2551 et le guide de la Commission ;
  - produit un brouillon de rapport de vérification.
- **Comment ça résout.** Moins d'heures d'auditeur par dossier, au moment où la demande explose et où les auditeurs accrédités sont rares.
- **Concurrents.**
  - Tout l'outillage connu vise l'importateur ou l'exportateur : Coolset, Embedded Carbon Record, Sustainability Cloud (un guide pour exportateurs).
  - SGS, RINA et Bureau Veritas vendent la vérification et la pré-vérification comme service, probablement avec des outils internes.
  - Aucun logiciel pour vérificateur trouvé en 4 recherches ([Coolset](https://www.coolset.com/academy/top-cbam-software-tools-7f11d), [ECR](https://embeddedcarbonrecord.com/verification-directory/)).
- **Qui paie.** Le directeur technique d'un organisme accrédité ISO 14065 de taille moyenne, hors grands groupes de test, inspection et certification.
- **Potentiel de revenu.** Non chiffré. Le nombre d'organismes accrédités CBAM n'est pas encore publié.
- **Pourquoi maintenant.** Le guide de la Commission pour vérificateurs et organismes d'accréditation, publié le 24/08/2026 ([CE](https://taxation-customs.ec.europa.eu/news/european-commission-publishes-guidance-cbam-verifiers-and-accreditation-bodies-2026-08-24_en)), et les accréditations à partir de 09/2026.
- **Risque.** Le moins vérifié des 8 : Dubrink et CBAMBOO n'ont pas été contrôlés, et les acheteurs sont peu nombreux.
- **MVP (6 semaines).** Module de réconciliation et checklist sur un secteur (acier ou aluminium).
- **Validation sans contact.** Suivre la publication des listes d'organismes accrédités CBAM par les organismes nationaux d'accréditation.

### 4. Contrôle qualité neutre des notes des scribes IA

- **Question technique d'origine.** Comment un hôpital qui utilise plusieurs scribes vérifie-t-il que la note correspond à la consultation, alors que l'éditeur contrôle l'audio ?
- **Problème.** Le Vérificateur général de l'Ontario a trouvé des erreurs chez les 20 éditeurs approuvés :
  - 9 sur 20 inventent des éléments du plan de traitement ;
  - 12 sur 20 notent le mauvais médicament ;
  - 17 sur 20 omettent des éléments de santé mentale.

  Sources : [CanHealth, 13/05/2026](https://www.canhealth.com/2026/05/13/ontario-ag-finds-flaws-in-ai-scribes/). Pebblous a audité 565 notes : une sur trois contient une erreur vérifiée ([Pebblous](https://blog.pebblous.ai/report/ai-scribe-error-rate-instrument-2026-09/en/)).
- **Solution.**
  - Un banc d'essai sur des consultations simulées, selon le protocole du Vérificateur général.
  - Un échantillonnage mensuel en production : transcription et note sont comparées par un vérificateur LLM qui cherche omissions, inventions et erreurs de médicament.
  - Un tableau de bord par éditeur et par spécialité.
- **Comment ça résout.** Une mesure comparable entre éditeurs, pour l'achat et le suivi.
- **Concurrents.**
  - Abridge Linked Evidence : seulement pour ses propres notes.
  - Qualified Health : gouvernance IA générale.
  - Pebblous : de l'étude, pas un produit.
- **Qui paie.**
  - Le CMIO ou le directeur qualité d'un système de santé.
  - Au Canada, les acheteurs publics : Supply Ontario, OntarioMD.
- **Potentiel de revenu.** Non chiffré. Base installée : DAX sert plus de 600 organisations ([EHR Source](https://www.ehrsource.com/articles/ambient-ai-scribes-comparison/)).
- **Risques.** L'accès aux transcriptions dépend des éditeurs, et les achats hospitaliers sont lents.
- **MVP (6 à 8 semaines).** D'abord un audit ponctuel pour un appel d'offres (banc simulé), sans intégration.

### 5. Vérificateur indépendant des noyaux GPU générés par IA

- **Question technique d'origine.** Un noyau CUDA ou Triton généré par un agent passe les tests numériques. Est-il correct pour toutes les formes et tous les types, et n'a-t-il pas triché sur la mesure ?
- **Problème.**
  - Les tests laissent passer des noyaux faux ([Kuiper, PAgE '26](https://mtzguido.github.io/pubs/kuiperbench.pdf)).
  - Les gains sont surestimés ([KernelBench-Verified, 07/2026](https://arxiv.org/html/2607.16241)).
  - Le détournement de récompense est documenté ([arXiv 2602.05885](https://arxiv.org/html/2602.05885)).
- **Solution.** Une CLI et une GitHub Action qui combinent :
  - fuzzing différentiel (formes, dtypes, strides, NaN et infinis) ;
  - tolérances en ULP ;
  - détecteurs anti-triche (opérateurs interdits, streams, sortie en cache) ;
  - compute-sanitizer ;
  - preuve SMT sur les sous-classes simples.
- **Concurrents.**
  - [Gimlet Labs](https://gimletlabs.ai/blog/formally-verifying-ai-generated-kernels) : vérificateur formel interne, en prototype de recherche.
  - L'anti-triche intégré de GEAK chez AMD.
  - Les contrôles internes de Makora et Standard Kernel.
  - Personne ne le vend séparément.
- **Qui paie.** Les équipes performance d'inférence qui mettent ces noyaux en production. En second, les générateurs en OEM.
- **Potentiel de revenu.** Non chiffré. L'argent du segment : Standard Kernel 20 M$, Makora 8,5 M$, Wafer 40 M$ (2026).
- **Risque.** Les générateurs intègrent la vérification ; il faut être le vérificateur neutre qu'ils adoptent.

### 6. Suivi d'intention malveillante sur plusieurs PR

Fiche complète dans `09-piste-pionnier.md`, n°2. Statut inchangé : aucun vendeur. La détection tombe de 50-60 % à 16-22 % sur une fenêtre de 24 PR ([PRWeaver](https://arxiv.org/html/2608.02693)).

### 7. Tri des démonstrations robot par influence

- **Question technique d'origine.** Parmi des milliers d'épisodes de téléopération, lesquels font baisser le taux de réussite du robot ?
- **Problème.** Dans le cas Pebblous/GR00T, 93 épisodes qui avaient l'air propres ont fait chuter la réussite de 73 % à 43 %.
- **Solution.** Un score d'influence par démonstration sur la réussite de la politique, qui sert à filtrer et à orienter la collecte suivante. Les méthodes : [CUPID](https://arxiv.org/html/2506.19121), [ATHENA](https://arxiv.org/html/2606.16208) et [RoboDrop, 09/2026](https://arxiv.org/pdf/2609.10021).
- **Concurrents.**
  - Seulement de la recherche.
  - Pebblous fait du diagnostic, sans mesurer l'influence de chaque démonstration.
  - SVRC fait du contrôle qualité en direct pendant la collecte, pas d'influence sur la politique.
- **Qui paie.** Les labos de robots humanoïdes et les usines de données de téléopération (XDOF a levé 70 M$ en 2026).
- **Risque.** Les labos le font en interne.

### 8. Contrôle d'exactitude des GPU en inférence

Fiche complète dans `09-piste-pionnier.md`, n°1.

**Correction : ACE vend déjà quelque chose de voisin.** [ACE](https://acefleet.dev/blog/silent-data-corruption) vend l'isolation des GPU suspects dans les flottes d'inférence, à partir de signes avant-coureurs (dérive thermique, codes XID). Il ne contrôle pas l'exactitude des sorties. L'angle « requêtes canaris et rejeu à l'identique » reste libre, mais le créneau n'est plus vide.

---

## Seconde ligue

| Idée | Nouveauté | Pourquoi en seconde ligue |
|---|---|---|
| Analyse d'impact des changements de schéma (OpenAPI, protobuf, dbt) sur prompts, schémas de sortie et évaluations | Aucun vendeur (Gable, Monte Carlo, PactFlow ne traitent pas le prompt comme consommateur) | Pas de preuve de dépense ; Gable ou Monte Carlo l'ajoutent facilement |
| Surveillance du « context rot » en production | Un seul outil open source à 13 étoiles ; Langfuse RFC #12873 | Facile à copier par Braintrust, Arize, Langfuse |
| Réconciliation des effets ambigus des agents (lire l'état réel chez le SaaS) | Recherche ([arXiv 2609.15397](https://arxiv.org/pdf/2609.15397)) et équipes qui le codent elles-mêmes | Aucune preuve de dépense ; Composio ou Temporal possèdent les connecteurs |
| Transfert de LoRA sans données vers une nouvelle base | Recherche seulement (Trans-LoRA, LoRA-X) | Pas de déclencheur daté ; risque technique élevé |
| Prévision par lois d'échelle et transfert d'hyperparamètres | Aucun vendeur hors Cerebras | Risque de devenir du conseil |

## Incertaines, non classées

- **Registre de dérivation GraphRAG** (effacement et droits propagés aux résumés) : DThink, Fluree et Graphwise ne sont pas vérifiés.
- **Conformité SB 243** : le bundle semble nouveau, mais la détection est déjà vendue (ActiveFence) et le budget n'est pas sourcé.

## Écartées dans cette passe (vendeur nommé)

| Idée | Vendeur |
|---|---|
| Registre IA par dossier (cabinets d'avocats) | Catapult Core AI ; iManage |
| Routeur neutre d'approbations | gotoHuman, Approveit, The Handover, Copilot Studio |
| Retour temps réel aux téléopérateurs | SVRC |
| Validation de la compaction de contexte | Gemini CLI (tour « Probe ») |
| Autotuner de parallélisme | NVIDIA NeMo Auto Configurator |
| Localisation des SDC en entraînement | Megatron `rerun_state_machine` ; ACE ; proteanTecs |
| Audit de mémorisation ; barrière de régression des fine-tunes ; lethal trifecta | Patronus, Openlayer, Microsoft FIDES, Cyera, Snyk |
| Écosystème Skills (scan, registre, lockfile, évals, conflits, synchronisation) | Snyk, JFrog, Microsoft APM, Tessl, google/skill-reach, Koi (Palo Alto) |
| Apps vibe-codées (inventaire, scan) | RedAccess, Zenity, Nokod, Symbiotic |
| Cache de prompt ; schémas d'outils ; idempotence | Helicone, Datadog, LiteLLM, Temporal |
| Dérive MCP ; contrats de données ; lignage vers agents | MCP Vitals, Gable, Monte Carlo, Atlan |
| GraphRAG (coût, résolution d'entités, évaluation multi-hop) | LazyGraphRAG, Senzing, Future AGI |
| Preuve d'humanité (sondages, candidats, lien humain-agent) | RelevantID, Greenhouse + CLEAR, Prove, World ID |
| Monitoring et SLA des API LLM ; rapprochement de factures | ThousandEyes, Traceloop, Kenda |
| Conformité NERC des datacenters (thèse 3 du fichier 10) | GridStrong |
