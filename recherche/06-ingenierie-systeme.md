# Domaine 06 : ingénierie système. Idées de startups candidates YC

Date : 2026-09-25. Rapport rédigé en français, à partir de l'analyse des mécanismes des solutions existantes (specs, documentation, code) sur les neuf sous-couches du domaine.

**Méthode.** Quatre groupes de recherche ont travaillé en parallèle :
- G1 : embarqué/firmware et conception matérielle ;
- G2 : exigences/MBSE et sûreté/certification ;
- G3 : SDV et simulation/jumeaux numériques ;
- G4 : robotique/drones, spatial et défense.

Pour chaque sous-couche, nous avons d'abord démonté les solutions existantes, puis tiré le point douloureux du mécanisme, et enfin validé chaque idée sur les 8 critères YC.

**Budget.** Environ 167 WebSearch utilisées (38 + 45 + 40 + 44), plus des lectures directes de specs et de code sur GitHub / raw.githubusercontent.com (clones `git clone --depth 1` ou `curl`).

**Légende des sources.**
- [OUVERT] : page ou fichier lu en entier.
- [EXTRAIT] : extrait d'un résultat de recherche.
- Toutes les sources ont été consultées le 2026-09-25 ; la date de la source est indiquée quand elle est connue.
- Toute estimation non sourcée est étiquetée « HYPOTHÈSE non sourcée ».

**Limite transversale.** Le proxy réseau bloque de nombreux sites officiels (unece.org, ecfr.gov, fmi-standard.org, arxiv.org, docs.github.com, kernel.org, etc.). Les textes réglementaires (CRA, 22 CFR 120.54, UNECE R152/R156/R171, GB 44496, décision FCC) sont donc cités d'après des extraits de recherche. Les specs techniques (FMI 3.0, SSP 2.0, Uptane, VSS, API SysML v2, doc Git, politique GitHub, QoS ROS 2, rosbag2, F´, ESP-IDF, Zephyr) ont été lues à la source [OUVERT].

## Synthèse

| # | Idée | Sous-couche(s) | Statut |
|---|---|---|---|
| A1 | Agent de backport de correctifs CVE dans les forks « vendeur » (noyaux BSP Linux, puis SDK MCU), avec preuve compilation/démarrage/tests sur la config du produit, VEX et dossier CRA | Embarqué/firmware (+ CRA) | **PASSE**, MARCHÉ À VALIDER |
| A2 | ProofGate : du LLR au contrat ACSL puis au code C généré par LLM et prouvé (Frama-C/WP), empaqueté comme crédit DO-333 / ISO 26262 | Sûreté/certification, exigences | **PASSE**, MARCHÉ À VALIDER (risque de piège à goudron « formel = niche » identifié, non prouvé) |
| A3 | « Credibility CI » : preuves de crédibilité reproductibles et indépendantes du simulateur (FMI/SSP) pour l'homologation virtuelle (R152, R171, 2022/1426, UNR ADS) | Simulation/jumeaux, SDV | **PASSE**, MARCHÉ À VALIDER (automobile seul ≈ 17,5 M$, extension multi-secteurs nécessaire) |
| A4 | ExportGuard for Code : frontière ITAR/EAR par chemin de code, appliquée sur Git, la CI, les agents IA et la publication | Défense (souveraineté), spatial | **PASSE**, MARCHÉ À VALIDER |
| A5 | Base « CVE → fichier/symbole » pour composants MCU (Zephyr, mbedTLS, lwIP…) et VEX automatique depuis Kconfig et carte de liens | Embarqué/firmware | **À TRANCHER** |
| A6 | Ledger d'indépendance et de confiance outil pour code et tests écrits par IA (DO-178C / ISO 26262) | Sûreté/certification | **À TRANCHER** |
| A7 | Moteur d'analyse d'impact « type-approval » des mises à jour OTA (RXSWIN, R156 + GB 44496) | SDV | **À TRANCHER** |
| A8 | HBOM de conformité pour l'origine des composants de drones (Blue UAS, seuil de 65 % de valeur US) | Robotique/drones, défense | **À TRANCHER** |

**Correspondance des numéros.** Dans le corps des fiches et des sections C, « idée 1 / idée 2 » renvoie à la numérotation interne de chaque groupe :
- G1 : idée 1 = A1, idée 2 = A5 ;
- G2 : idée 1 = A2, idée 2 = A6 ;
- G3 : idée 1 = A3, idée 2 = A7 ;
- G4 : A1 = A4, A2 = A8.

De même, « tableau B » renvoie à la sous-section B du groupe concerné (B1 à B4).

Aucune de ces idées n'a de chiffre de marché publié. Les marchés sont estimés en bottom-up, avec des facteurs en partie hypothétiques et étiquetés. Aucune idée n'a été rejetée pour cause de marché douteux.

---

## A. Fiches des idées qui passent (dont « MARCHÉ À VALIDER » et « À TRANCHER »)

### A1. Agent de backport de correctifs de sécurité pour les forks « vendeur » (noyaux BSP Linux, puis SDK MCU), avec preuve de compilation et de démarrage sur la config du produit, et VEX généré (G1, PASSE, MARCHÉ À VALIDER)

**Question technique d'origine.** Quand un CVE du noyau est corrigé dans une branche stable upstream, comment arrive-t-il dans un produit qui tourne sur un noyau BSP de fondeur (par exemple `linux-imx` 5.10/6.6 ou Rockchip 5.10), qui a divergé d'upstream, pendant les 5 ans ou plus exigés par le CRA ? Qui fait ce travail, et à quel coût ?

**Problème (mécanisme).**
1. **Volume.** Depuis février 2024, le noyau Linux est sa propre CNA. Il attribue un CVE à chaque correctif stable qui remplit la définition : 4 325 CVE en 2024 et 5 708 en 2025, soit environ 50 par semaine. En 2026, on a vu des lots de 440 CVE en 24 h, gonflés par les outils d'audit par IA (Sashiko, etc.). Sources [EXTRAIT] : https://www.noze.it/en/insights/432-linux-kernel-cves-two-days/ ; https://jerrygamblin.com/2026/01/01/2025-cve-data-review/ ; https://securityonline.info/linux-kernel-cves/ ; https://xenospectrum.com/en/linux-kernel-cve-batch-ai-review/
2. **Fork divergent.** « Les CVE corrigés dans le noyau LTS upstream ne sont pas intégrés automatiquement dans le fork du vendeur. Le vendeur doit évaluer chaque patch, le backporter s'il est pertinent, puis publier un BSP mis à jour. » De plus, une version affichée comme « 5.10.110-rk3588 » ne deviendra jamais 5.10.268 par une mise à jour upstream. La comparaison de versions ne dit donc pas quels correctifs sont présents. Sources [EXTRAIT] : https://promwad.com/news/embedded-linux-bsp-maintenance-kernel-patching-strategy ; https://www.techveda.live/bsp-tracker/ ; https://www.onekey.com/resource/why-version-matching-is-not-enough-part-i-linux-kernel
3. **Durée de support upstream plus courte que la durée légale.** La durée par défaut des nouveaux LTS a été ramenée de 6 à 2 ans en 2023. Les dates révisées le 25/02/2026 sont les suivantes : 5.10 → décembre 2026, 6.6 → décembre 2027, 6.12 et 6.18 → décembre 2028. Source [EXTRAIT] : https://www.phoronix.com/news/Linux-6.18-LTS-6.12-6.6-Extend ; https://fossforce.com/2026/03/greg-kroah-hartman-stretches-support-periods-for-key-linux-lts-kernels/
   - Or le CRA, art. 13(8), impose une période de support d'au moins 5 ans en règle générale. L'art. 13(9) exige que chaque mise à jour de sécurité reste disponible 10 ans. Source [EXTRAIT] : https://www.european-cyber-resilience-act.com/Cyber_Resilience_Act_Article_13.html ; https://fluchsfriction.medium.com/cyber-resilience-act-faq-support-period-c9713f1ce7ed
   - Conséquence : un produit sorti en 2026 sur un BSP 6.6 doit être patché jusqu'en 2031 au moins, alors qu'upstream s'arrête en 2027.
4. **Même mécanisme côté MCU.** Chaque release d'ESP-IDF est supportée 30 mois (12 mois « Service » puis 18 mois « Maintenance »), et « pas de correctif pour les releases EOL ». C'est plus court que les 5 ans du CRA. Source [OUVERT] : https://raw.githubusercontent.com/espressif/esp-idf/master/SUPPORT_POLICY.md. Zephyr LTS annonce « au moins 5 ans » ([OUVERT] https://raw.githubusercontent.com/zephyrproject-rtos/zephyr/main/doc/project/release_process.rst), mais les SDK dérivés (nRF Connect SDK, forks vendeurs) suivent leur propre rythme (HYPOTHÈSE non sourcée pour NCS).
5. **Les outils automatiques cassent sur les cas difficiles.**
   - PortGPT (IEEE S&P 2026, agent LLM) réussit 89 % sur les jeux de données établis, mais seulement 62 % sur 146 cas complexes, et 9 sur 18 patches sur la branche 6.1 stable. Source [EXTRAIT] : https://arxiv.org/abs/2510.22396 ; https://www.helpnetsecurity.com/2025/11/05/portgpt-ai-backport-security-patches-automatically/ ; code : https://github.com/OS3Lab/patch-backporting
   - Le benchmark unifié d'août 2026 (1 234 cas) fait passer le meilleur taux de 85,2 % (patches de type I) à 24,0 % (type IV). Il identifie quatre causes : API cible inconnue, décalage sémantique entre versions, dépendances non locales, localisation du patch. Source [EXTRAIT] : https://arxiv.org/html/2608.17671
   - Donc : **« le noyau émet environ 50 CVE par semaine sur des fichiers que le fork vendeur a modifiés. Le backport naïf (cherry-pick ou LLM seul) échoue sur les patches structurels. Quand le LTS upstream s'arrête (5.10 en décembre 2026), plus personne ne fait ce travail, alors que le CRA l'exige pendant au moins 5 ans. »**
6. **L'alternative actuelle est un service humain cher.** Lynx (ex-Timesys) vend un service de « Linux OS/BSP Maintenance », présenté comme coûtant « moins de la moitié d'un ingénieur junior ». Vigiles filtre les CVE selon le `.config` noyau et suit les backports dans les LTS. Source [EXTRAIT] : https://www.timesys.com/security/vigiles-embedded-device-security-maintenance-process/ ; https://www.lynx.com/solutions/linux-lts-bsp-maintenance ; https://www.timesys.com/solutions/linux-os-bsp-maintenance/

**Solution.** Un service logiciel (agent) branché sur le dépôt noyau ou SDK du client. Pour chaque CVE ou correctif stable, il :
1. filtre selon le `.config` et les fichiers réellement compilés (les métadonnées de fichiers du CNA noyau le permettent) ;
2. localise le code dans le fork divergent ;
3. génère le backport avec un agent outillé (historique git, renommages, API cible) ;
4. compile pour la config exacte du produit, démarre sous QEMU ou sur une carte de ferme (labgrid ou LAVA), puis lance les tests de régression (kselftest, LTP, tests du client) ;
5. produit une série de patches relisible, un statut VEX par CVE (`fixed` ou `not_affected`, avec justification) et un dossier de preuve CRA (dates, SBOM delta) ;
6. escalade à un humain uniquement les patches de type III/IV, file d'attente priorisée par KEV ou « activement exploité » (délai de 24 h de l'art. 14).

Extension en phase 2 : forks ESP-IDF, NCS, Zephyr et STM32Cube dont la release est en fin de vie.

**Comment elle répond.**
- Elle remplace un ingénieur ou un service humain par un pipeline qui passe à l'échelle du volume de CVE.
- La preuve compilation + démarrage + tests traite les causes d'échec identifiées dans le benchmark 2608.17671.
- Le VEX et le dossier de preuve répondent directement aux obligations de l'art. 13 (correctifs sur la période de support) et de l'art. 14 (déclaration sous 24 h / 72 h).

**Concurrents.**

| Concurrent | Ce qu'il fait | Ce qui manque par rapport à l'angle |
|---|---|---|
| Lynx/Timesys (Vigiles + service de maintenance BSP) | Filtrage des CVE et service humain | Pas d'agent de backport automatisé annoncé dans les sources lues |
| Seal Security (13 M$ Series A, juillet 2025) | Backporte des correctifs dans des paquets applicatifs et OS « sealed » ([EXTRAIT] https://www.prnewswire.com/news-releases/seal-security-raises-13m-series-a-to-secure-open-source-code-at-scale-302515952.html ; https://www.seal.security/product) | Aucune preuve trouvée qu'il traite des forks noyau BSP propriétaires ; il travaille sur des paquets publics |
| Canonical (SRU noyau toutes les 2 semaines), CIP SLTS, TuxCare, CIQ | Maintenance de leurs propres noyaux | Pas des forks BSP du client |
| Foundries.io (Qualcomm), Toradex | Stratégie « upstream-first », qui contourne le problème en abandonnant les forks | Ne marche que si le SoC est bien supporté en mainline |
| ONEKEY, RunSafe | Détection et SBOM | Ne produisent pas le correctif |
| PortGPT, FixMorph, TSBPort | Recherche académique | Pas de produit |

Angle qu'aucun n'a d'après le balayage : backport automatisé **dans le fork privé du client**, validé sur **sa** config et **son** matériel, avec VEX et preuve CRA en sortie.

**Qui paie.** Les fabricants OEM et ODM de produits embarqués Linux vendus dans l'UE : industriel, énergie, médical, passerelles IoT. Payent aussi les fabricants de SoM et de cartes (Toradex, Advantech, etc.) qui doivent maintenir leurs BSP pour leurs clients, ainsi que les intégrateurs et bureaux d'études qui revendent la maintenance.

**Revenu potentiel (bottom-up) : MARCHÉ À VALIDER.**
- Valeur par client, ancre sourcée : Lynx positionne le service humain de maintenance d'un BSP à « moins de la moitié du coût d'un ingénieur junior » par an ([EXTRAIT] timesys.com, lien ci-dessus). HYPOTHÈSE non sourcée : un ingénieur junior embarqué coûte environ 60–80 k€ chargé en Europe de l'Ouest, soit environ 30–40 k€ par BSP et par an pour le service humain. Un agent vendu environ 50 % de ce prix donne environ 15–20 k€ par fork et par an.
- Nombre de clients : Embedded Linux est utilisé par 44 à 46 % des développeurs et systèmes embarqués (enquête Eclipse 2024, [EXTRAIT] https://commandlinux.com/statistics/linux-embedded-systems-market-size/). Je n'ai trouvé aucune source donnant le nombre d'entreprises qui livrent un produit embarqué Linux dans l'UE : l'étude d'impact du CRA n'a pas été trouvée.
- Calcul illustratif, HYPOTHÈSE non sourcée : 5 000 fabricants × 2 forks en moyenne × 15 k€ = 150 M€/an de SAM UE. À 20 000 fabricants dans le monde, on obtient environ 600 M€.
- Facteur manquant pour trancher : le nombre de fabricants ou de lignes produits embarqués Linux concernés par le CRA.

**Preuve des 8 critères.**

1. **Vrai problème : OUI.** Mécanisme fork divergent + 50 CVE par semaine + LTS de 2 à 4 ans face aux 5 ans du CRA (sources ci-dessus). Un marché de service existe déjà (Lynx). Il y a même des particuliers qui backportent à la main vers des noyaux vendeurs EOL : https://github.com/JakeStone594/f_hid-4.14-backports (août 2026) [OUVERT via recherche GitHub].
2. **Pas un piège à goudron : pas de preuve de piège.**
   - Aucune fermeture ni aucun pivot trouvé sur cette idée précise (recherche « embedded Linux long-term maintenance startup shut down OR pivot », sans résultat pertinent).
   - Risque identifié : dérive vers une activité de service à faible marge, puisque tous les acteurs en place sont des sociétés de services (Lynx, Pengutronix, Witekio).
   - Signal positif : Foundries.io, positionné sur la mise à jour sécurisée d'embarqué Linux, a été racheté par Qualcomm en mars 2024 ([EXTRAIT] https://www.eenewseurope.com/en/qualcomm-buys-foundries-io/).
3. **Aigu : OUI.** Fréquence d'environ 50 CVE par semaine, avec des pics de 400 en 24 h. Gravité : obligation légale de déclaration en vigueur depuis le 11/09/2026 ([EXTRAIT] https://www.crowell.com/en/insights/client-alerts/its-live-the-cyber-resilience-act-reporting-is-mandatory-as-of-today-11-september-2026). Échéance technique : le 5.10 atteint sa fin de vie en décembre 2026 alors que beaucoup de BSP en dépendent (exemple Rockchip 5.10 cité par techveda).
4. **Marché : MARCHÉ À VALIDER** (voir le calcul ; le nombre de clients n'est pas sourcé).
5. **Concurrence :** voir le tableau ci-dessus. Aucun acteur ne combine « fork privé du client + agent + preuve sur matériel + VEX ». Risque : Seal Security pourrait étendre son offre aux noyaux. Ce n'est pas un motif de rejet selon les consignes.
6. **Pourquoi maintenant : changements datés.**
   - Obligations de déclaration du CRA : 11/09/2026.
   - Obligations complètes du CRA (Annexe I, correctifs sur la période de support) : 11/12/2027.
   - Fin de vie du LTS 5.10 : décembre 2026.
   - Le noyau devient CNA en février 2024, puis afflux de CVE trouvés par IA en 2026.
   - Les agents LLM de backport atteignent un niveau publiable (PortGPT, S&P 2026).
7. **Proxy : Seal Security.** Même modèle (« backporter le correctif au lieu de forcer une montée de version ») pour les bibliothèques applicatives : 20 M$ levés, Series A de 13 M$ en juillet 2025 (lien ci-dessus). Proxy secondaire : Foundries.io racheté par Qualcomm.
8. **Scalable : OUI, sous condition.** Les correctifs upstream sont communs à tous les clients : le travail de backport vers un même BSP de fondeur (par exemple `lf-6.6.y` de NXP) se mutualise entre tous les OEM qui l'utilisent, ce qui donne un effet réseau par arbre BSP. Le coût marginal se limite au calcul et à la ferme de test. Le risque de dérive vers le service est à surveiller (critère 2).

**Formulations du balayage concurrence (14).**
1. « BSP kernel CVE patching service AI agent embedded Linux vendor kernel automated backport product launch 2026 »
2. « LLM automated backporting Linux kernel security patches older vendor kernel tool startup PortGPT OR FixMorph »
3. « Seal Security backported patches Linux kernel embedded Yocto vendor kernel firmware »
4. « site:ycombinator.com/companies backport security patches legacy versions »
5. « Lynx Timesys Vigiles kernel configuration filtering CVE patch backport BSP maintenance service pricing »
6. « AI-assisted kernel CVE backporting embedded vendor kernels TuxCare OR CIQ OR Canonical OR Linutronix OR Bootlin service 2026 »
7. « vendor BSP kernel linux-imx backport CVE fixes stable patches not applied embedded long-term maintenance CRA »
8. GitHub : « backport kernel llm »
9. GitHub : « automated backport patches agent cve »
10. GitHub : « backport vendor kernel CVE »
11. GitHub : « PortGPT » (a trouvé une org « Port-Pilot/PortGPT » vide, créée en juillet 2026, sans produit visible)
12. « "Port-Pilot" OR "PortPilot" AI patch backporting startup »
13. « embedded Linux long-term maintenance startup shut down OR pivot »
14. « Qualcomm acquires Foundries.io »

Limite : Product Hunt, G2 et Show HN n'ont pas été interrogés séparément, faute de budget. G2 est seulement apparu pour Seal Security.


---

### A2. ProofGate : du LLR au contrat ACSL puis au code C prouvé, empaqueté pour la certification (G2, PASSE, MARCHÉ À VALIDER)

**Question technique d'origine.** Si un LLM écrit du code C de niveau DAL A/B ou ASIL C/D, quel objectif DO-178C ou ISO 26262 devient le goulot ? Peut-on déplacer la confiance du générateur (non qualifiable) vers un vérificateur déterministe qualifiable à bas coût ?

**Problématique (mécanisme).**
1. En DO-178C (§12.2.1), un outil doit être qualifié quand il élimine, réduit ou automatise un processus *sans que sa sortie soit vérifiée* selon la section 6. Un générateur de code de niveau A relèverait du critère 1, donc de TQL-1. Aucune méthode acceptée ne permet de qualifier un LLM non déterministe.
   - Sources : TASKING DO-330 https://www.tasking.com/do-330/ [EXTRAIT] ; einfochips https://www.einfochips.com/blog/ensuring-airborne-software-integrity-an-overview-to-avionics-tool-qualification/ [EXTRAIT].
   - Constat répété : « no accepted methodologies for qualifying LLM-based code generators under ISO 26262, DO-178C, IEC 62304 » (résultats de recherche : EE Times https://www.eetimes.com/vibe-coding-in-safety-critical-software-promise-pitfalls-and-a-path-forward/ et arXiv 2506.04038 https://arxiv.org/pdf/2506.04038 [EXTRAIT]).
2. La seule voie praticable est donc de **vérifier toute la sortie**. Les objectifs de revue du code source (DO-178C Table A-5 : conformité aux LLR, traçabilité, conformité aux standards, exactitude) restent des revues humaines.
   - Le LLM produit du code plus vite que les humains ne peuvent le relire. AdaCore le formule ainsi : « AI generates artifacts so quickly that this can easily overwhelm human reviewers » (https://www.unmannedsystemstechnology.com/2026/09/adacore-demonstrates-deterministic-guardrails-for-ai-high-integrity-software-development/ [EXTRAIT], septembre 2026).
   - Jama : « AI-generated code manufactures implementation decisions at high volume and records none of them » (https://www.jamasoftware.com/blog/ai-generated-code-risks/ [EXTRAIT]).
   - **Le goulot passe de l'écriture à la revue.**
3. En ISO 26262-8 §11, l'outil reçoit TD1 (TCL1, pas de qualification) seulement si sa sortie est vérifiée selon ISO 26262. Un IDE est plutôt classé TD2 (https://www.embitel.com/blog/embedded-blog/why-is-software-tool-qualification-indispensable-in-iso-26262-based-software-development [EXTRAIT]). La méthode TI/TD est décrite dans https://github.com/github/codeql-coding-standards/blob/main/docs/iso_26262_tool_qualification.md [OUVERT, v0.8.0 du 2025-08-29]. On retrouve la même logique : la confiance vient de la vérification aval.
4. **Levier existant et sous-exploité : DO-333.** Le supplément « méthodes formelles » permet à une preuve de remplacer des revues et des tests de conformité aux LLR, avec des activités alternatives pour la couverture structurelle.
   - Airbus remplace des tests unitaires par des « unit proofs » (Caveat, ancêtre de Frama-C) depuis l'A380, puis sur l'A400M et l'A350. Cet exemple figure dans DO-333. Économie annoncée de 30 à 50 %.
   - Sources : https://militaryembedded.com/avionics/safety-certification/formal-program-verification-avionics-certification [EXTRAIT] ; https://www.di.ens.fr/~delmas/papers/fm09.pdf [EXTRAIT] ; https://arxiv.org/html/1508.03894v1 (étude ACSL + Frama-C sur les LLR d'un projet DO-178C) [EXTRAIT].
   - **Pourquoi c'est resté une niche :** écrire les spécifications formelles et faire aboutir les preuves demande des experts rares. C'est précisément la partie que les LLM automatisent en 2025-2026 (voir « pourquoi maintenant »).
5. Asymétrie de qualification que la solution exploite : le **générateur** (LLM) relèverait de TQL-1 s'il n'était pas vérifié. Le **vérificateur** (prouveur déterministe utilisé pour automatiser une vérification) relève du critère 3, donc TQL-5 (ou du critère 2 et TQL-4 s'il sert à réduire d'autres vérifications).
   - Source : DO-178C Tableau 12-1, connaissance de la norme. Le texte RTCA est payant et n'a pas été ouvert ; les TQL 1 à 5 sont confirmés par https://www.tasking.com/do-330/ [EXTRAIT].
   - **Conclusion :** la confiance doit porter sur contrat + prouveur, pas sur le LLM.

**Solution.** Plugin CI et IDE pour C (MISRA) :
- (a) importe les LLR depuis DOORS Next, Polarion, Codebeamer ou Jama (ReqIF ou API) ;
- (b) un LLM propose un **contrat ACSL** par fonction (pré/postconditions, frame, invariants) et le lie au LLR ;
- (c) **le contrat est la seule chose relue par l'humain**, à côté du texte du LLR, avec une vue de diff sémantique ;
- (d) un LLM génère le code et les annotations de boucle ; Frama-C/WP (open source) prouve ; en cas d'échec, boucle de réparation ;
- (e) produit le paquet de certification : matrice LLR, contrat, preuve, code ; données de qualification TQL-5 du prouveur pour la configuration utilisée ; arguments DO-333 FM.6.3 et ISO 26262-6 (vérification formelle, Tables 7/9) ;
- (f) le code non prouvable est signalé et redirigé vers la revue humaine classique et les tests.

**Comment elle répond.** La revue humaine passe d'environ N lignes de code à environ quelques lignes de contrat par fonction. Le LLM sort de l'argument de sûreté : seule la sortie prouvée est créditée, et le prouveur est qualifiable à bas coût. Le crédit DO-333 réduit les tests unitaires et une partie de l'effort de couverture structurelle, comme chez Airbus.

**Concurrents, et l'angle qu'aucun n'a :**
- **AdaCore GNAT Foundry: Intersection** (démonstrateur open source, septembre 2026) : garde-fous déterministes par SPARK autour d'artefacts IA. **Ada/SPARK seulement** ; la majorité du code automobile et médical est en C. https://www.unmannedsystemstechnology.com/2026/09/adacore-demonstrates-deterministic-guardrails-for-ai-high-integrity-software-development/ [EXTRAIT]
- **TrustInSoft** (spin-off de l'écosystème Frama-C) : analyse « sound » pour prouver l'absence d'erreurs à l'exécution (UB, mémoire), plus Rust depuis la 2025.10, plus un échafaudage de tests généré par IA.
  - **Angle différent :** TrustInSoft prouve l'absence de comportements indéfinis, pas la conformité fonctionnelle au LLR avec un package DO-333.
  - Sources : https://www.comparethecloud.net/news/trustinsoft-adds-ai-generated-test-scaffolding-and-rust-support-to-its-formal-verification-platform [EXTRAIT] ; https://www.trust-in-soft.com/resources/blogs/formal-methods-ensuring-the-safety-of-ai-generated-code [EXTRAIT].
- **Parasoft** (GoogleTest certifié TÜV + IA agentique), **LDRA**, **VectorCAST**, **Rapita**, **Polyspace Code Prover** : tests, couverture, analyse statique. Pas de chaîne LLR → contrat → preuve orientée génération IA. https://www.parasoft.com/learning-center/do-178c/overview/ [EXTRAIT]
- **Ketryx** (Series B de 39 M$, septembre 2025, maintenant DO-178C) : traçabilité et analyse d'impact par IA. **Ne produit pas de preuve de code.** https://www.ketryx.com/industries/aerospace [EXTRAIT] ; https://www.ketryx.com/press-release/series-b [EXTRAIT]
- **Recherche académique sans produit** : spec2code (LLM + ACSL + Frama-C, cas Scania), AutoACSL (juin 2026, 98 % de génération de spécifications, 96 % de preuves complètes), VeCoGen. https://arxiv.org/html/2411.13269 ; https://arxiv.org/abs/2606.20969 ; https://pith.science/paper/2411.19275 ; https://arxiv.org/html/2605.21532 [EXTRAIT]
- **Aucune startup trouvée** sur l'angle exact (C + contrat lié au LLR + package de crédit DO-333/ISO 26262 + flux IA).

**Qui paie.** Responsables logiciel et vérification chez les Tier-1 automobiles (ASIL C/D), les avionneurs et équipementiers (DAL A-C), les fabricants de dispositifs médicaux de classe C (IEC 62304), le ferroviaire (EN 50716) et le nucléaire. Budget « outils de vérification » qui achète aujourd'hui LDRA, VectorCAST, Polyspace et Rapita.

**Revenu potentiel (bottom-up) : MARCHÉ À VALIDER.**
- Calcul : nombre d'équipes logicielles certifiées × prix annuel.
- HYPOTHÈSE non sourcée : 5 000 équipes logicielles développant du C sous DO-178C, ISO 26262 ASIL C/D ou IEC 62304 classe C dans le monde. Aucune source trouvée pour ce chiffre.
- HYPOTHÈSE non sourcée : 60 k$ par équipe et par an, positionné sous le coût d'une licence d'outil de vérification premium (les prix Polyspace et VectorCAST ne sont pas publics : https://www.peerspot.com/products/polyspace-code-prover-reviews [EXTRAIT]).
- Résultat : 5 000 × 60 k$ = **300 M$/an de marché adressable**. Ce chiffre reste une hypothèse.
- Points d'ancrage sourcés :
  - bas : Ferrocene (chaîne Rust qualifiée) à 240 € par siège et par an, minimum 10 sièges (https://ferrous-systems.com/blog/officially-qualified-ferrocene/ [EXTRAIT]) ;
  - bas et signal de risque : TrustInSoft, outil formel pour le C, a un chiffre d'affaires estimé entre 1 et 10 M$ après 13 ans (https://www.owler.com/company/trust-in-soft, https://craft.co/trustinsoft [EXTRAIT]) ;
  - haut, dépense en validation autonome : Applied Intuition à 830 M$ d'ARR en 2025 (https://sacra.com/c/applied-intuition/ [EXTRAIT]).
- **À valider :** le nombre d'équipes et le prix réellement supportable.

**Preuve des 8 critères :**
1. **Vrai problème : OUI.** Le goulot de revue du code IA est reconnu par AdaCore (09/2026) et Jama. L'absence de méthode de qualification des LLM est documentée (sources ci-dessus).
2. **Pas un piège à goudron : RISQUE IDENTIFIÉ, pas prouvé.** Les éditeurs d'outils formels restent petits (TrustInSoft, 1 à 10 M$), parce que les spécifications et preuves coûtaient cher en experts. Le mécanisme change : la génération ACSL par LLM atteint 96 % de preuves complètes (AutoACSL, 06/2026). Aucune fermeture ni aucun pivot trouvé sur cet angle exact.
3. **Aigu : OUI.** Chaque ligne IA en DAL A/B ou ASIL D doit être revue et tracée. Le crédit formel réduit de 30 à 50 % le coût de vérification (source militaryembedded ci-dessus).
4. **Marché : MARCHÉ À VALIDER** (calcul ci-dessus).
5. **Concurrence : angle libre en C.** AdaCore couvre Ada/SPARK, TrustInSoft couvre l'absence d'UB, pas le LLR ni le DO-333.
6. **Pourquoi maintenant :**
   - AutoACSL 06/2026 ;
   - GNAT Foundry 09/2026, qui montre que le marché se prépare ;
   - ED-324/ARP6983 (ML en aéronautique) visé pour juin 2026 (https://na.eventscloud.com/file_uploads/115fca49330a77ce92d7fe04e9874faf_Day1-Jahn-202508ED-324ARP6983presFAAAI-MLTechExchangeMeeting_8-5-25-Read-Only.pdf [EXTRAIT]) ;
   - ISO/PAS 8800 publiée en 12/2024 (https://www.iso.org/standard/83303.html [EXTRAIT]) ;
   - MC/DC natif dans Clang 17+ (https://clang.llvm.org/docs/SourceBasedCodeCoverage.html [EXTRAIT]).
7. **Proxy :** Airbus « unit proof » (preuve à la place des tests unitaires) en production depuis l'A380. Ferrocene : un open source durci et vendu sous forme qualifiée crée un produit payant.
8. **Scalable : OUI.** Logiciel en CI, prouveur open source ; seul le kit de qualification est à maintenir par version.

**Formulations du balayage concurrence (≥10, toutes exécutées) :**
1. « LLM generate ACSL contracts Frama-C WP verified C code startup OR company embedded safety 2026 »
2. « TrustInSoft AdaCore AI-generated code formal verification safety-critical 2025 announcement »
3. « startup AI coding agent for safety-critical embedded software MISRA DO-178C ISO 26262 launch 2026 »
4. « startup AI agents DO-178C certification evidence automation aerospace software 2025 raises »
5. « site:ycombinator.com/companies safety certification ISO 26262 OR DO-178C OR IEC 62304 » (aucun résultat YC)
6. « DO-178C LLM generated code tool qualification DO-330 AI coding assistant certification »
7. « generative AI coding assistants safety-critical software guidance 2026 automotive aerospace "AI-generated code" ISO 26262 »
8. « AdaCore GNAT Foundry Intersection deterministic guardrails AI SPARK 2026 »
9. « Airbus unit proof Frama-C Caveat DO-178 replace unit testing DO-333 »
10. « Ketryx aerospace DO-178C AI agents code review traceability features 2026 »
11. « ISO 26262 tool confidence level LLM code generator TI TD »
12. « site:ycombinator.com/companies requirements engineering hardware systems engineering »

Limite : Product Hunt, G2 et Show HN n'ont pas été interrogés séparément, faute de budget de recherche. C'est à compléter.


---

### A3. « Credibility CI » : compilateur de preuves de crédibilité pour la simulation d'homologation, indépendant des éditeurs (FMI/SSP) (G3, PASSE, MARCHÉ À VALIDER)

**Question technique d'origine.** Un OEM peut maintenant remplacer une partie des essais physiques AEB (R152) ou DCAS (R171) par des essais virtuels. Qu'est-ce qui garantit que la même chaîne de co-simulation (modèle véhicule en FMU, capteurs, scénario) redonnera le même résultat le jour de l'audit par le service technique ? Et qui démontre, avec quelles métriques, que cette simulation correspond aux essais physiques effectués ?

**Problème (mécanisme, sources).**
1. **Les specs FMI et SSP ne définissent pas l'exécution.** Dans FMI 3.0, l'algorithme de co-simulation appartient à l'importeur : « the co-simulation algorithm of the importer must raise an event (if supported) […] In the case of Co-Simulation without EventMode […] detecting discrete changes to continuous input variables […] requires heuristics » (`docs/4_1_co-simulation_math.adoc`). L'intégrateur est « responsible for advancing time, setting states, handling events » (`docs/1___overview.adoc` l.101). Le retour arrière d'état n'est possible que si le flag optionnel `canGetAndSetFMUState` vaut true (`docs/2_4_common_schema.adoc` l.309-315). Côté SSP 2.0 : « The core SSP standard does not include these execution-specific settings, but layered standards will be defined to include those settings » (`docs/1___overview.adoc` l.90).
   **Conséquence :** un même paquet SSP/FMU peut donner des résultats différents selon l'outil, le pas de communication ou le solveur. Rien dans l'artefact normalisé ne fige ces réglages.
   Sources : [OUVERT] https://github.com/modelica/fmi-standard (clone, docs/) ; [OUVERT] https://github.com/modelica/ssp-standard (clone, dernier commit du 2026-05-05).
2. **Les binaires dépendent de la plateforme.** Une FMU doit contenir « at least one implementation », source ou binaire, par plateforme (`x86_64-windows`, `x86_64-linux`…) et doit documenter ses dépendances externes (`docs/2_5_fmu_distribution.adoc` l.4-6, 55-75). Quand un fournisseur livre une FMU uniquement en binaire Windows, elle ne peut pas être rejouée sur une ferme Linux ou dans un conteneur, ce qui casse la reproductibilité exigée pour l'audit. [OUVERT] même dépôt.
3. **Il n'existe plus de preuve de conformité à jour.** Le FMI Cross-Check est « temporarily archived while we're reworking the FMI Cross-Check process » (dernier commit « Add archived note » du 2022-02-10). Les résultats d'import s'y limitaient à des fichiers d'auto-déclaration `passed/failed` (FMI-CROSS-CHECK-RULES.md l.10, 205), et le dépôt ne contient aucun dossier FMI 3.0. [OUVERT] https://github.com/modelica/fmi-cross-check
4. **La réglementation exige maintenant une crédibilité démontrée.**
   - (EU) 2022/1426 fixe des principes d'évaluation de crédibilité de la chaîne virtuelle, mais « leaves room for interpretation regarding the process implementation » [EXTRAIT, résultat sur 2022/1426, IntechOpen / ASAM-AVL] https://www.asam.net/index.php?eID=dumpFile&t=f&f=8877&token=df370ee4d0c080aa1a15029b4268e2e4a7ccdccc
   - R171 DCAS contient un « Simulation Credibility Framework » [EXTRAIT] https://www.morai.ai/post/credibility-framework-simulation-autonomous-vehicles
   - L'amendement R152 (AEBS) autorise les tests virtuels, mais « at least 30% of the required tests must still be conducted physically, with at least one test for each scenario variant » [EXTRAIT] https://www.foretellix.com/aebs-regulation/ et https://www.dspace.com/en/pub/home/applicationfields/stories/digital-homologation.cfm
   **Conséquence :** chaque homologation produit des paires essai virtuel / essai physique. Ces paires doivent être corrélées de façon traçable, et cela doit être refait à chaque changement de modèle ou d'outil.
5. **Les standards de traçabilité existent, les produits manquent.** Le Credible Simulation Process (SET Level) et SSP Traceability existent. Le seul kit open source (Virtual Vehicle, projet ITEA UPSIM) n'a qu'un commit (2024-12-13) et « will provide metrics » au futur. [OUVERT] https://github.com/virtual-vehicle/Credibility-Assessment-Framework

**Solution.** Un service ou appliance, en SaaS ou sur site, qui :
- (a) ingère un système SSP et ses FMU ou des configurations CarMaker / dSPACE ASM, recompile les FMU livrées en source ou les encapsule (remoting) pour les exécuter dans des conteneurs Linux figés ;
- (b) fige et signe l'ensemble des réglages d'exécution (algorithme maître, pas de communication, solveurs, graines) avec l'élément `MetaData` et les signatures de SSP 2.0 ;
- (c) rejoue automatiquement les scénarios des essais physiques obligatoires (≥30 % en R152) à partir des logs de piste, calcule les métriques de validation (écarts, incertitude, sensibilité) et les niveaux de crédibilité du CSP SET Level ;
- (d) tourne en CI à chaque nouvelle version de modèle, de FMU ou d'outil pour détecter les régressions de crédibilité ;
- (e) génère le dossier « simulation handbook / credibility assessment » structuré selon les annexes 2022/1426, R171, R152 et la future UNR ADS.

**Comment elle répond au problème.**
- Le trou « exécution non spécifiée » est comblé par le conteneur et les réglages figés et signés.
- Le trou « binaire Windows » est comblé par la recompilation depuis les sources ou par le remoting.
- L'absence de cross-check est compensée par des tests de régression propres au client.
- L'exigence de corrélation est couverte par la mise en correspondance automatique des essais physiques et virtuels.
- Le dossier réglementaire est généré au lieu d'être rédigé à la main.

**Concurrents nommés, et angle qu'aucun n'a.**
- **Foretellix** travaille sur un « unified framework for simulation trustworthiness », mais sa « toolchain qualification ensures that every component of the internal Foretellix toolchain works correctly » : c'est centré sur son propre outil Foretify et sur la couverture de scénarios [EXTRAIT] https://www.foretellix.com/unified-approach-to-simulation-trustworthiness-in-av-development/
- **dSPACE** vend du conseil (« simulation handbook ») lié à son écosystème SIMPHERA/ASM [EXTRAIT] https://www.dspace.com/en/pub/home/applicationfields/stories/digital-homologation.cfm
- **TÜV SÜD VIVALDI** est un service d'évaluation des outils de simulation, pas un outil de production de preuves en continu [EXTRAIT] https://www.tuvsud.com/en/industries/automotive/autonomous-vehicle-testing-and-homologation-services/simulation-validation-for-autonomous-vehicles
- **MORAI** mène un projet de recherche avec le KIT (mai 2025) [EXTRAIT] URL morai ci-dessus.
- **Applied Intuition** fait de la simulation de capteurs et de la V&V, avec un outil certifié ISO 26262 [EXTRAIT] https://www.appliedintuition.com/press-releases/iso-26262
- **DNV Simulation Trust Center** propose une collaboration FMU dans le cloud et la recommandation DNV-RP-0513 d'assurance des modèles, orientées maritime et énergie [EXTRAIT] https://www.dnv.com/services/simulation-trust-center-collaboration-platform-207515/ et https://www.dnv.com/digital-trust/recommended-practices/simulation-models-assurance-dnv-rp-0513/
- **Ansys Minerva** propose un workflow de crédibilité ASME V&V40 pour le médical [EXTRAIT] https://ansys.synopsys.com/blog/ansys-minerva-streamlines-credibility-assessment-for-healthcare-in-silico-testing
- **uofa** (GitHub, 2 étoiles) : « Machine-verifiable credibility evidence for regulated computational simulation », un signal très précoce [OUVERT via recherche GitHub] https://github.com/cloudronin/uofa

**Angle libre :** aucun acteur trouvé ne combine (i) l'indépendance vis-à-vis du simulateur, au niveau FMI/SSP, (ii) l'exécution reproductible figée et signée, (iii) la corrélation automatique avec les essais physiques obligatoires de R152/R171 et (iv) la génération du dossier en CI. Les outils de scénarios (Foretellix, Applied) qualifient leur propre chaîne, et les SPDM (Minerva) ne ciblent pas l'homologation automobile.

**Qui paie.** Les directions homologation et V&V ADAS des OEM (budget type-approval), les Tier-1 ADAS qui livrent la fonction avec son dossier, les développeurs ADS qui visent la nouvelle UNR ADS, et en second temps les services techniques (UTAC, TÜV, IDIADA) en licence « auditeur ».

**Revenu potentiel (bottom-up) : MARCHÉ À VALIDER.**
- Nombre de clients :
  - 20 groupes OEM majeurs. Applied Intuition vend à « 18 of the top 20 automotive OEMs », ce qui montre que ces 20 acheteurs de simulation existent [EXTRAIT] https://sacra.com/research/applied-intuition-at-830m-year/
  - L'EEA indique que dix pools de constructeurs font environ 97 % des immatriculations dans l'EEE en 2024 [EXTRAIT] https://theicct.org/publication/co2-emissions-from-new-passenger-cars-in-europe-car-manufacturers-performance-in-2024-dec25/
  - Environ 15 Tier-1 et développeurs ADS : HYPOTHÈSE non sourcée.
  - Total : environ 35 comptes.
- Valeur par client : 300 k$ à 1 M$ par an (HYPOTHÈSE non sourcée). Repère : l'ARR de Foretellix, spécialiste V&V, serait de 26,2 M$ [EXTRAIT, fiabilité faible] https://getlatka.com/companies/foretellix.com
- Calcul automobile : 35 × 0,5 M$ ≈ **17,5 M$ ARR adressable**, avec une fourchette de 10 à 35 M$.
- Extensions possibles :
  - chaque nouvelle réglementation qui ouvre l'essai virtuel (R152 puis d'autres) ;
  - la tarification par dossier d'homologation (nombre de types × variantes, non sourcé) ;
  - le médical (ASME V&V40 reconnu par la FDA [EXTRAIT] URL Minerva ci-dessus) ;
  - le spatial (NASA-STD-7009B, 2024 [EXTRAIT] https://standards.nasa.gov/standard/NASA/NASA-STD-7009) ;
  - le maritime (DNV-RP-0513).
- Verdict : le marché automobile seul est petit pour une ambition YC. Le potentiel dépend de l'extension multi-industries ou de la tarification par homologation, d'où « MARCHÉ À VALIDER ».

**Preuve des 8 critères.**
1. **Vrai problème : OUI.** Le mécanisme (exécution non normalisée, binaires par plateforme, cross-check archivé) est démontré plus haut par le texte des specs [OUVERT]. La réglementation laisse l'implémentation ouverte (2022/1426, extrait ASAM/AVL).
2. **Pas un piège à goudron : pas de fermeture trouvée sur cet angle.** Le seul projet voisin (Credibility-Assessment-Framework) est un projet de recherche resté en l'état (1 commit, 2024-12), pas une startup morte. Le risque de piège documenté est l'OTA (Airbiquity, voir B), qui ne concerne pas cet angle. Recherches « simulation credibility … startup », « virtual homologation startup 2025 2026 seed » : aucune startup dédiée et aucun post-mortem.
3. **Problème aigu : PROBABLE.** Chaque homologation R152/R171 virtuelle exige la corrélation et le dossier. Chaque mise à jour de modèle ou d'outil invalide potentiellement la preuve. La fréquence réelle des ré-homologations reste à valider (pas de chiffre public).
4. **Marché :** voir le calcul ci-dessus, MARCHÉ À VALIDER.
5. **Concurrence :** voir la liste ci-dessus. Aucun acteur n'a exactement l'angle. Le risque principal est que Foretellix ou dSPACE étendent leur offre, ce qui n'est pas un motif de rejet selon les consignes.
6. **Pourquoi maintenant, avec des dates :**
   - WP.29 intègre l'approche d'homologation virtuelle dans R171 fin 2024 [EXTRAIT] dSPACE ;
   - l'amendement R152 est approuvé en 2025, annonce officielle attendue en 2026 [EXTRAIT] dSPACE / Foretellix ;
   - la GTR et l'UNR ADS sont adoptées par le GRVA en janvier 2026 puis par WP.29 en juin 2026 [EXTRAIT] https://news.un.org/en/story/2026/06/1167797 et https://www.globalpolicywatch.com/2026/05/un-regulation-and-gtr-on-automated-driving-systems-current-state-of-play/ ;
   - SSP 2.0 intègre `MetaData` et les signatures (SSP Traceability) dans le cœur du standard [OUVERT] ssp-standard ;
   - NAFEMS 2026 organise un événement « SSP Traceability standard … building block for simulation governance » [EXTRAIT] https://www.nafems.org/events/nafems/2026/the-ssp-traceability-standard-a-data-standard-supporting-the-credible-simulation-process-for-system-simulation-as-a-building-block-for-simulation-governance/
7. **Proxy :**
   - Applied Intuition, outils de simulation et V&V vendus aux OEM : 830 M$ d'ARR en 2025, valorisation 15 Md$ [EXTRAIT] https://sacra.com/research/applied-intuition-at-830m-year/ et https://www.appliedintuition.com/press-releases/series-f
   - Foretellix, V&V vendu aux OEM : 135 M$ levés [EXTRAIT] https://www.foretellix.com/foretellix-raises-85-million-in-series-c-closing/
   - Dans un autre secteur, Ansys a jugé le workflow de crédibilité V&V40 assez rentable pour le produire.
8. **Scalable : OUI.** Logiciel et CI en conteneurs, avec des formats ouverts (FMI/SSP) supportés par « more than 280 simulation tools » [EXTRAIT] https://www.nafems.org/events/nafems/2026/the-ssp-standard-system-structure-and-parameterization-the-companion-standard-to-fmi-for-system-simulation-and-collaborative-product-development/ Les mêmes preuves se réutilisent entre réglementations.

**Formulations du balayage concurrentiel (≥10).**
1. « simulation credibility assessment software tool automated evidence ADS homologation startup »
2. « "simulation credibility" OR "toolchain credibility" product launch 2025 2026 sim-to-real correlation automated driving »
3. « Mobex simulation credibility assessment platform »
4. « Applied Intuition sensor simulation validation correlation real-world "credibility" toolchain qualification R157 R171 »
5. « Foretellix OR dSPACE OR AVL "credibility assessment" virtual toolchain feature release 2026 »
6. « Foretellix "simulation trustworthiness" toolchain qualification … »
7. « FMU continuous integration regression testing co-simulation SSP cloud platform model credibility traceability product »
8. « SPDM "model credibility" NASA-STD-7009 OR "ASME V&V 40" … Minerva OR Teamcenter OR SIMULIA »
9. « DNV "Simulation Trust Center" FMU features credibility … »
10. « site:ycombinator.com/companies digital twin OR co-simulation OR Modelica OR FMU » : aucune entreprise YC trouvée.
11. « "Show HN" OR "Product Hunt" simulation validation sim-to-real correlation virtual homologation evidence FMU » : rien de pertinent.
12. « virtual homologation startup 2025 2026 seed "virtual validation" credibility dossier … »
13. GitHub : « simulation credibility » (trouvé cloudronin/uofa) ; « ssp traceability fmu » (0 résultat).
14. « UN R171 DCAS virtual testing simulation credibility annex … » ; « UN R152 amendment virtual testing AEB … »


---

### A4. « ExportGuard for Code » : frontière ITAR/EAR par chemin de code, appliquée sur Git, la CI, les agents IA et la publication (G4, PASSE, MARCHÉ À VALIDER)

**Question technique d'origine.** Une startup défense ou spatiale a un monorepo : 80 % de code commercial ou EAR99, 20 % de code de guidage, de charge utile ou de liaison classé USML ou 9E515. Peut-on donner à un ingénieur non-« US person » l'accès au premier sans lui livrer le second ? Que fait réellement Git pour l'autorisation en lecture ? Pourquoi la dérogation ITAR « chiffrement de bout en bout » ne règle-t-elle pas le cas d'une forge de code ? Et que se passe-t-il quand un agent IA de code lit tout le dépôt ?

**Problématique (mécanisme, avec sources).**
1. **Git n'a pas de contrôle d'accès en lecture plus fin que le dépôt.** Le protocole de récupération peut divulguer des objets qui n'étaient pas censés être partagés. La documentation officielle de Git le dit en toutes lettres : *« The fetch and push protocols are not designed to prevent one side from stealing data from the other repository that was not intended to be shared. If you have private data that you need to protect from a malicious peer, your best option is to store it in another repository… namespaces on a server are not effective for read access control »*. Elle décrit ensuite deux attaques : les lignes « have » et les deltas qui révèlent des morceaux d'un objet X.
   Source : https://raw.githubusercontent.com/git/git/master/Documentation/transfer-data-leaks.adoc [OUVERT], inclus dans `gitnamespaces`.
   → Conséquence : la seule frontière fiable est **un dépôt séparé**. Les partial clone, sparse checkout et namespaces ne sont pas des frontières de sécurité.
2. **GitHub.com ne sait pas restreindre par pays ni par nationalité.** *« The cloud-hosted service offering available at GitHub.com has not been designed to host data subject to the ITAR and does not currently offer the ability to restrict repository access by country… we recommend you consider GitHub Enterprise Server »*.
   Source : https://raw.githubusercontent.com/github/docs/main/content/site-policy/other-site-policies/github-and-trade-controls.md [OUVERT].
   Même avec la résidence de données US (GHE.com, depuis le 12 mai 2025) et Copilot en résidence US avec modèles FedRAMP Moderate (13 avr. 2026), le contrôle reste par dépôt ou organisation. Sources : https://github.com/newsroom/press-releases/data-residency-in-us [EXTRAIT], https://github.blog/changelog/2026-04-13-copilot-data-residency-in-us-eu-and-fedramp-compliance-now-available/ [EXTRAIT].
3. **La dérogation de chiffrement ITAR est incompatible avec une forge de code.** Selon 22 CFR 120.54(a)(5), stocker ou envoyer des données techniques non classifiées n'est pas une exportation si elles sont chiffrées de bout en bout. Condition : aucun intermédiaire, y compris le fournisseur cloud, n'a accès au texte clair. Le chiffrement doit être FIPS 140-2 ou au moins de force AES-128, et les données ne doivent pas être stockées dans un pays du § 126.1.
   Sources : https://www.ecfr.gov/current/title-22/chapter-I/subchapter-M/part-120/subpart-C/section-120.54 [EXTRAIT], https://www.virtru.com/blog/compliance/itar/data-sharing [EXTRAIT].
   Or un serveur Git ou une CI doit lire le texte clair pour faire des diffs, fusionner, indexer, lancer des builds ou faire tourner Copilot. La dérogation ne s'applique donc pas, et on retombe sur une enclave auto-hébergée entière (GHES, GitLab Dedicated for Government).
4. **L'« export réputé » (deemed export) s'applique au sein même de l'entreprise.** Donner accès à du code contrôlé à un étranger présent aux États-Unis, par exemple un salarié H-1B, équivaut à une exportation vers son pays. Les licences individuelles prennent des mois.
   Sources : https://www.buchalter.com/blogs/foreign-talent-domestic-risk-deemed-export-issues-in-tech-hiring/ [EXTRAIT], https://www.f1jobs.io/resources/blog/software-engineer-defense-tech-primeair-anduril-visa [EXTRAIT], https://itarconsultant.us/blog/itar-technical-data-definition-classification-examples/ [EXTRAIT] : « granting unauthorized repository access » figure parmi les violations fréquentes.
5. **Les agents IA élargissent la surface d'exposition.** Donner à un outil IA un accès large en lecture à un dépôt qui contient des fichiers USML crée un risque de divulgation non autorisée.
   Source : https://www.kiteworks.com/regulatory-compliance/itar-ai-agents-compliance-gap/ [EXTRAIT]. Un agent de code lit par construction tout le dépôt.
   → **Donc** : puisque l'unité d'autorisation est le dépôt (points 1 et 2) et que le chiffrement de bout en bout est impossible pour une forge (point 3), les équipes mettent **tout** dans une enclave. Cela bloque les ingénieurs non-US persons, le cloud commercial, les agents IA du marché et la publication open source des parties non contrôlées. Sinon, elles découpent à la main en dépôts séparés, et du code contrôlé finit par fuir par copier-coller, cherry-pick, historique Git ou prompt IA. Rien ne vérifie en continu que les objets Git contrôlés ne sont pas atteignables depuis un dépôt non contrôlé.

**Solution.** Une couche « frontière d'export » pour le cycle de développement logiciel, en cinq briques :
1. **Manifeste `EXPORT.yaml`** (sur le modèle de CODEOWNERS) : chemin → juridiction (USML cat./ECCN/EAR99), justification, approbateur (Empowered Official), date de revue. Des suggestions de classification sont proposées (dépendances, seuils de paramètres, similarité avec du code déjà classé), mais la décision reste humaine.
2. **Moteur de découpage et synchronisation.** Le code contrôlé vit dans un dépôt enclave (GHES ou GitLab en GovCloud). Le reste vit dans un dépôt « sûr » (GitHub.com, cloud). La synchronisation est bidirectionnelle et transformée, dans l'esprit de Copybara. Un invariant est vérifié : aucun blob ni commit contrôlé n'est atteignable dans le dépôt sûr, historique compris.
3. **Détection de fuite par empreintes.** Des empreintes de type winnowing du code contrôlé sont vérifiées dans les hooks pre-receive du serveur non contrôlé, les PR, les pastes et les prompts ou contextes envoyés aux agents IA (proxy ou MCP).
4. **Routage identité et CI.** L'attribut « US person » vient du SIRH ou de l'IdP. Les jobs qui touchent des chemins contrôlés tournent sur des runners administrés par des US persons, dans des régions US. Les agents IA ne reçoivent que les chemins autorisés pour l'identité qui les invoque.
5. **Journal de preuves** pour le Technology Control Plan et, le cas échéant, une divulgation volontaire à la DDTC.

**Comment elle répond au problème.** Elle transforme une frontière grossière (toute l'organisation dans une enclave) en frontière par chemin, sans demander à Git ce qu'il ne sait pas faire : on garde l'isolement par dépôt, mais on l'automatise et on le vérifie en continu. Le code non contrôlé redevient utilisable par des non-US persons, par la CI cloud et par les agents IA.

**Concurrents (nommés) et angle manquant.**
| Acteur | Ce qu'il fait | Ce qui manque pour l'angle A1 |
|---|---|---|
| GitHub Enterprise Server / GHE.com data residency | Enclave auto-hébergée ; résidence de données US | Contrôle par dépôt ou organisation, pas par pays ni par chemin (politique [OUVERT]) |
| GitLab Dedicated for Government | Instance isolée pour le gouvernement US | Même granularité (https://gitlab.com/gitlab-com/support/support-training/-/issues/4670 [EXTRAIT]) |
| Perforce P4 (Helix Core) | Table de protections par chemin de dépôt, utilisée pour l'ITAR/EAR | Hors Git ; pas de classification, pas de contrôle des agents IA ni de sync vers le cloud (https://laoutaris.org/blog/helix-core/ [EXTRAIT], https://help.perforce.com/helix-core/server-apps/cmdref/current/Content/CmdRef/p4_protect.html [EXTRAIT]) |
| NextLabs (Export Control for Technical Data) | ABAC par citoyenneté et localisation pour SAP, PLM, SharePoint | Pas de Git ni de CI (https://www.nextlabs.com/blogs/controlling-the-transfer-of-itar-related-technical-data/ [EXTRAIT]) |
| archTIS NC Protect | ABAC par nationalité, classification ITAR/EAR automatique des documents M365/SharePoint | Documents Microsoft uniquement (https://www.archtis.com/itar-and-ear-compliance-in-microsoft-365-and-sharepoint/ [EXTRAIT]) |
| Kiteworks | Réseau privé de données, tagging ITAR, MCP pour LLM | Fichiers et e-mails, pas l'objet Git (https://www.kiteworks.com/platform/compliance/itar-compliance/ [EXTRAIT]) |
| PreVeil, Virtru | Partage chiffré de bout en bout (dérogation 120.54) | Incompatible avec une forge (point 3) |
| TrueFoundry, Fasoo, Cyberhaven, Concentric | Passerelle IA ou DLP « ITAR » : scan des prompts, vérification US person | Travaillent au niveau du prompt ou du fichier, pas du graphe d'objets Git ni de la classification par chemin (https://www.truefoundry.com/blog/what-is-itar [EXTRAIT], https://en.fasoo.ai/blog/an-itar-compliant-approach-to-ai-usage [EXTRAIT]) |
| Tabnine (on-prem) | Assistant de code déployé en local pour la défense | Ne gère pas la frontière entre dépôts (https://www.startuphub.ai/insights/ai-coding-agents-daily-2026 [EXTRAIT]) |
| GitGuardian | Détection de secrets dans Git (hooks, scan) | Détecte des secrets, pas du code classé ; c'est le proxy technique le plus proche |
| eCustoms, Descartes, Thomson Reuters ONESOURCE | Classification USML/ECCN de produits | Classement d'articles pour la douane, pas de code (https://ecustoms.com/compliance_solutions/itar_software/ [EXTRAIT]) |

**Angle qu'aucun n'a** : une classification d'export **par chemin de code**, appliquée en même temps (i) sur l'atteignabilité des objets Git entre dépôts, (ii) sur le routage de la CI et (iii) sur le contexte envoyé aux agents IA, avec un journal de preuves.

**Qui paie.** Le directeur conformité export ou l'Empowered Official, avec le VP Engineering ou le RSSI, dans les entreprises enregistrées à la DDTC qui écrivent du logiciel : startups défense ou spatial, maîtres d'œuvre et sous-traitants, universités sous contrat. Le budget vient des lignes conformité export et outillage développeurs.

**Revenu potentiel (bottom-up).** MARCHÉ À VALIDER.
- Environ **14 000 entités enregistrées à la DDTC** (https://www.hklaw.com/en/insights/publications/2024/12/state-department-finalizes-itar-registration-fee-increases, chiffre cité dans les résultats : « roughly 14,000 registered entities » [EXTRAIT]).
- Part de ces entités qui développent du logiciel ou du code source contrôlé : **HYPOTHÈSE non sourcée, 20 %**, soit 2 800 organisations.
- Nombre moyen d'ingénieurs logiciel concernés par organisation : **HYPOTHÈSE non sourcée, 50**.
- Prix : **HYPOTHÈSE non sourcée, 30 $/ingénieur/mois**, soit 360 $/an.
- Calcul : 2 800 × 50 × 360 $ = **50,4 M$/an** (SAM US ITAR seul).
- Hausses possibles, non chiffrées : entreprises sous EAR seul (ECCN 9E515, 3E, 5D) et hors DDTC ; universités ; grands comptes (maîtres d'œuvre avec des milliers d'ingénieurs).
- Signal de dynamique : **966 opérations VC défense en 2025 pour 49,1 Md$**, un record (https://pitchbook.com/news/reports/q4-2025-defense-tech-vc-trends [EXTRAIT]). Les jeunes entreprises défense sont nombreuses et mélangent code commercial et code contrôlé.

**Preuve des 8 critères.**
1. **Vrai problème** : Git ne permet pas de contrôle d'accès en lecture plus fin que le dépôt (doc Git [OUVERT]) ; GitHub.com ne restreint pas par pays (politique GitHub [OUVERT]) ; l'export réputé couvre l'accès au code (Buchalter [EXTRAIT]).
2. **Pas un piège à goudron** : aucune fermeture ni pivot trouvé sur « frontière d'export pour le code » (balayage ci-dessous). Les acteurs adjacents qui font de l'ABAC ITAR pour les documents sont vivants et vendent (NextLabs, archTIS sur Carahsoft : https://www.carahsoft.com/archtis/solutions [EXTRAIT]).
   Risque identifié : l'alternative « tout en enclave » est considérée comme suffisante par une partie des acheteurs. C'est un risque d'adoption, pas un post-mortem documenté.
3. **Aigu** : la sanction civile ITAR peut atteindre environ 1 M$ par violation (Kiteworks [EXTRAIT]). L'exposition se produit à chaque clone, PR, job CI ou appel d'agent IA. Une licence individuelle d'export réputé prend des mois (Buchalter [EXTRAIT]).
4. **Marché** : environ 50 M$/an de SAM US avec des hypothèses marquées ; MARCHÉ À VALIDER (voir le calcul ci-dessus).
5. **Concurrence** : tableau ci-dessus. Aucun acteur ne fait de classification par chemin appliquée à Git, à la CI et aux agents IA.
6. **Pourquoi maintenant** :
   - Copilot en résidence US + FedRAMP (13 avr. 2026), ce qui fait entrer l'IA dans les organisations réglementées sans régler l'ITAR ;
   - GHE.com résidence US (12 mai 2025), toujours « not designed for ITAR » ;
   - agents IA qui lisent tout le dépôt (Kiteworks, 2026 [EXTRAIT]) ;
   - record de VC défense en 2025 (PitchBook [EXTRAIT]) ;
   - hausse des frais DDTC au 9 janv. 2025 (Holland & Knight [EXTRAIT]), signe que l'administration renforce ce régime.
7. **Proxy** :
   - **Ketryx** : surcouche conformité sur GitHub et Jira pour le logiciel médical réglementé, Série B de 39 M$ en sept. 2025 (https://www.ketryx.com/press-release/series-b [EXTRAIT]) ;
   - **GitGuardian** : détection dans Git, Série B de 44 M$ en déc. 2021 puis Série C de 50 M$ en févr. 2026 (https://blog.gitguardian.com/gitguardian-closes-44-million-series-b/ [EXTRAIT], https://siliconangle.com/2026/02/11/gitguardian-raises-50m-expand-non-human-identity-ai-agent-security/ [EXTRAIT]) ;
   - **archTIS/NextLabs** : ABAC ITAR pour les documents.
8. **Scalable** : logiciel (appliance auto-hébergée + SaaS de contrôle), tarif par ingénieur. Même moteur pour l'EAR, pour d'autres pays (contrôles à l'export UE à double usage, HYPOTHÈSE d'extension) et pour les politiques internes (propriété intellectuelle).

**Formulations du balayage concurrence (15, EN/FR, vocabulaires vendeur, ingénieur et acheteur).**
1. « ITAR export controlled source code GitHub access control foreign person developers tool »
2. « ITAR technical data AI coding assistant Copilot export control risk defense contractors source code LLM »
3. « NextLabs ITAR attribute based access control git repository source code nationality »
4. « "export control" classification source code repository tool ECCN jurisdiction tagging startup »
5. « site:ycombinator.com/companies ITAR export control compliance » (seul résultat pertinent : Exosat, qui conçoit ses satellites *sans* contenu ITAR, un angle opposé : https://www.ycombinator.com/companies/industry/Satellites [EXTRAIT])
6. « monorepo ITAR controlled modules segregate code "US persons" CI runners GitLab Dedicated for Government defense startup »
7. « ITAR compliant GitHub alternative for defense startups code hosting 2025 launch » (résultats : GForge, Assembla, auto-hébergement)
8. « GitHub Enterprise Cloud data residency US ITAR FedRAMP GHE.com Copilot export controlled » (changelog de l'acteur en place)
9. « Perforce Helix Core ITAR export controlled file-level permissions protections table defense »
10. « "deemed export" defense tech startup foreign national engineers source code access technology control plan software »
11. « export control DLP for source code "ITAR" pre-receive hook scan commits controlled technical data open source release review »
12. « defense tech startup launches ITAR compliant developer platform AI coding agents US persons 2026 »
13. « "Show HN" OR "Product Hunt" ITAR compliance developer tool export controlled code » (rien de pertinent)
14. « ITAR "source code" jurisdiction classification per repository… "technology control plan" automated enforcement git CI runners US-person »
15. « archTIS NC Protect ITAR… OR Kiteworks AI data gateway ITAR code repositories »

Non couvert faute de budget : G2 n'a pas été interrogé spécifiquement. À faire avant décision.


---

### A5. À TRANCHER : base de données « CVE → fichier / symbole » pour les composants MCU et VEX automatique à partir du Kconfig et de la carte de liens (G1)

**Question.** Le SBOM produit par `west spdx` permet-il de dire si un CVE Zephyr est présent dans *ce* binaire ?

**Mécanisme.**
- `west spdx` enregistre les fichiers sources réellement compilés ([OUVERT] https://raw.githubusercontent.com/zephyrproject-rtos/zephyr/main/doc/develop/west/zephyr-cmds.rst).
- En revanche, le suivi des vulnérabilités ne se fait qu'au niveau du **module**, via des CPE/PURL déclarés dans `zephyr/module.yml` ([OUVERT] https://raw.githubusercontent.com/zephyrproject-rtos/zephyr/main/doc/develop/modules.rst).
- Il n'existe aucune métadonnée « CVE → fichiers » équivalente à celle du CNA noyau Linux. L'issue #85570, qui demande un équivalent de `cve-check` Yocto, est toujours ouverte et marquée « Stale » depuis février 2025 ([EXTRAIT/OUVERT via WebFetch] https://github.com/zephyrproject-rtos/zephyr/issues/85570).
- Résultat : faux positifs massifs (plus de 50 % selon RunSafe [EXTRAIT] https://runsafesecurity.com/blog/sbom-vulnerability-management-embedded/).

**Pourquoi « à trancher » et non retenue.**
- RunSafe Identify fait déjà un SBOM au moment de la compilation, au niveau fichier, pour le C/C++ embarqué, avec VEX automatique et « reachability » ([EXTRAIT] https://runsafesecurity.com/platform/identify/ ; https://www.tipranks.com/news/private-companies/runsafe-security-highlights-reachability-focused-upgrade-to-vulnerability-management-platform).
- ONEKEY identifie les composants et les fonctions dans les binaires RTOS (FreeRTOS, ThreadX, Zephyr, µC/OS) par signatures ([EXTRAIT] https://www.onekey.com/resource/how-we-taught-our-platform-to-understand-rtos-firmware).
- Ce qui manque pour trancher : savoir si RunSafe exploite le Kconfig Zephyr et une base CVE → fonction pour les piles MCU. Leurs pages étaient bloquées par le proxy. Si oui → rejet. Sinon → niche de données (flux vendu aux scanners).


---

### A6. À TRANCHER : ledger d'indépendance et de confiance outil pour artefacts écrits par IA (G2)

**Question technique d'origine.** Si le même modèle écrit le code *et* les tests, l'exigence d'« indépendance » (DO-178C, objectifs « with independence » des Tables A-6/A-7 en DAL A/B) et l'argument TD1 d'ISO 26262-8 tiennent-ils encore ?

**Problématique (mécanisme).**
- Selon Rapita, l'indépendance signifie que le vérificateur n'est pas l'auteur. Un outil peut en tenir lieu, mais il doit alors être qualifié (https://www.rapitasystems.com/do178c-testing [EXTRAIT]).
- L'argument TD1 d'un générateur repose sur des tests menés selon ISO 26262 (source embitel ci-dessus). Si le code et les tests sont générés par le même modèle, dans le même contexte, c'est un mode commun que ni Git ni les outils ALM ne voient : Git ne distingue pas le code humain du code IA.
- Le 2026-02, Cursor a publié la spécification ouverte **Agent Trace** (attribution IA au niveau fichier et ligne, avec le modèle utilisé). Elle ne juge pas la qualité et ne traite aucune règle de certification (https://www.infoq.com/news/2026/02/agent-trace-cursor/ [EXTRAIT] ; https://github.com/cursor/agent-trace [EXTRAIT, README inaccessible par curl]).
- Aucune source trouvée ne traite explicitement l'indépendance code/tests générés par un même LLM ; la recherche dédiée n'a rien donné. C'est un vide documentaire.

**Solution.**
1. Collecte des traces Agent Trace et Git, puis graphe « qui ou quel modèle a produit quel artefact ».
2. Moteur de règles : indépendance par niveau DAL ou ASIL (auteur du code ≠ auteur des tests ≠ relecteur, humain ou famille de modèle différente).
3. Argument TI/TD/TCL généré par outil IA.
4. **Preuve empirique TD** : injection de mutations de type LLM (Mull ou Dextool, open source, Dextool utilisé chez Saab Aeronautics : https://itea4.org/project/exploitable-result/221/dextool-mutate-a-mutation-testing-tool-for-c-c.html [EXTRAIT]) pour mesurer le taux de détection de la chaîne de vérification. ISO 26262-6 recommande déjà l'injection de fautes, y compris par mutation de code, en ASIL C/D (https://www.embitel.com/blog/embedded-blog/fault-injection-testing-of-safety-critical-automotive-software [EXTRAIT]).

**Concurrents :**
- Validas (analyse de chaîne d'outils TCA, QKits, dont llvm-cov : https://www.validas.de/solutions/qkits/qkits-tool-qualification/llvm-cov [EXTRAIT]) ;
- Ketryx (traçabilité et impact) ;
- Jama (« product context layer ») ;
- Parasoft ;
- Cursor (format de trace seulement).

Aucun ne combine attribution par modèle, règles d'indépendance et preuve de détection par mutation.

**Revenu :**
- HYPOTHÈSE non sourcée : même base d'équipes que l'idée 1 × 20 à 40 k$/an. **MARCHÉ À VALIDER.**

**Ce qui manque pour trancher :**
- une prise de position d'autorité ou d'assesseur (FAA/EASA CAST, TÜV) exigeant de démontrer l'indépendance des artefacts IA ; aucune trouvée ;
- sans cette exigence, l'acheteur n'a pas de déclencheur réglementaire. Le risque est un « nice to have ».
- Critère 3 (acuité) et critère 6 (pourquoi maintenant : Agent Trace 02/2026) partiellement établis. Critère 7, proxy : outils de provenance et SBOM portés par une obligation réglementaire (non sourcé ici).


---

### A7. À TRANCHER : moteur d'analyse d'impact « type-approval » pour les mises à jour SDV (R156 + GB 44496) (G3)

**Question technique d'origine.** Quand un paquet OTA touche un calculateur central qui héberge des fonctions relevant de plusieurs réglementations (freinage R13, direction R79, AEB R152…), comment le SUMS décide-t-il de façon traçable quels RXSWIN changent et si l'homologation est affectée ? Et comment le prouver à la fois en UNECE et en Chine ?

**Problème (mécanisme).**
- **Uptane délègue la décision.** La résolution de dépendances et de conflits est explicitement hors du standard : « The exact process by which this determination takes place is out of scope for this Standard. However, the Director SHALL take into account dependencies and conflicts between images » (uptane-standard.md, l.599). La compromission de la chaîne d'approvisionnement ou du build est aussi hors périmètre (l.275-281). [OUVERT] https://github.com/uptane/uptane-standard (commit du 2026-07-23)
- **R156 impose l'évaluation.** Pour chaque mise à jour, il faut évaluer et prouver si l'état homologué change, et gérer les RXSWIN [EXTRAIT] https://www.itemis.com/en/glossary/unece-r156/
- **GB 44496-2024 ajoute des exigences** d'évaluation d'impact et de gestion d'urgence, ainsi que l'archivage des « OTA software version compilation rule files ». Application aux nouveaux types au 2026-01-01 et à tous les types au 2028-01-01 [EXTRAIT] https://www.atic-ts.com/comparison-of-differences-between-gb-and-unr156-for-automotive-software-updates/ et https://www.codeofchina.com/standard/GB44496-2024.html
- **Conséquence :** avec l'OTA fréquent et la centralisation des calculateurs, l'analyse d'impact devient un goulot manuel en double juridiction. La fréquence et le coût réels ne sont pas sourcés publiquement.

**Solution.** Un graphe « fonction ↔ exigence réglementaire ↔ composant logiciel ↔ artefact binaire », alimenté par les diffs de build (SBOM, ARXML, sorties de CI). L'outil calcule automatiquement les RXSWIN et identifiants GB impactés, bloque les campagnes non conformes et génère les preuves pour les deux juridictions.

**Concurrents.**
- Sibros « R156 Readiness Tracker », qui « automates evidence collection and ECU readiness monitoring » [EXTRAIT] https://sibros.tech/resource-center/ota-processes-compliant-with-wp-29-r156-explained-with-real-world-examples
- msg, qui détermine les RxSWIN par projet, les écrit dans la passerelle en production et contrôle le produit final [EXTRAIT] https://www.msg.group/en/automotive/software-identifikation-using-rxswin-for-homologation
- Mender, qui produit des rapports d'homologation UNECE et des SBOM [EXTRAIT] https://mender.io/industries/automotive
- Codebeamer et Polarion pour l'analyse d'impact générique en ALM [EXTRAIT] https://www.spkaa.com/blog/choosing-the-right-alm-solution-codebeamer-vs-polarion
- Sonatus et Excelfore côté orchestration.

**Pourquoi « À TRANCHER ».** L'angle « déduction automatique de l'impact réglementaire à partir du diff de code, en double UNECE/GB » n'est revendiqué par aucun acteur trouvé. Mais Sibros (preuves) et msg (détermination des RXSWIN) couvrent une partie du besoin.
Ce qui manque pour trancher :
1. la documentation produit détaillée du Readiness Tracker de Sibros (fetch bloqué) ;
2. une source sur le volume de mises à jour évaluées par OEM et par an ;
3. une source sur le nombre de constructeurs soumis à la fois à GB 44496 et à R156.

Marché : environ 20 à 40 OEM (HYPOTHÈSE) × 200 à 500 k$ (HYPOTHÈSE), soit 4 à 20 M$, donc MARCHÉ À VALIDER.
Piège potentiel : l'OTA générique est un piège documenté (Airbiquity, voir B). Le positionnement doit donc rester sur la conformité, pas sur la livraison.
Pourquoi maintenant : GB 44496 obligatoire pour les nouveaux types depuis le 2026-01-01 ; R156 appliquée à tous les véhicules neufs dans l'UE depuis juillet 2024 [EXTRAIT] https://diadrom.com/insights/un-r156-sums-requirements
Proxy : Sibros est financé par du capital-risque sur le créneau OTA et conformité [EXTRAIT, montant peu fiable] PitchBook / EE Times.
Formulations du balayage (10) : « RXSWIN management software tool SUMS compliance platform OEM » ; « "type approval" impact analysis software update automated tool SDV OTA "RXSWIN" Sibros OR Excelfore OR Siemens OR PTC » ; « homologation-relevant software change detection platform "software update management system" SaaS » ; « msg RxSWIN homologation software identification solution product » ; « PREEvision OR Polarion OR codebeamer "RXSWIN" … » ; « Sibros OR Sonatus "R156" compliance feature … » ; « OTA "dependency resolution" ECU software compatibility matrix … » ; « GB 44496-2024 … differences UN R156 … » ; « GB 44496 vs UN R156 differences impact assessment … » ; « site:ycombinator.com/companies automotive software AUTOSAR ».


---

### A8. À TRANCHER : preuve d'origine des composants de drones (Blue UAS / « domestic end product ») (G4)
- **Mécanisme** :
  - En décembre 2025, la FCC a ajouté les drones et composants étrangers à la Covered List : plus d'autorisation d'équipement nouvelle.
  - Deux exemptions ont été prolongées en juillet 2026 : produits sur la **DCMA Blue UAS Cleared List**, et produits assemblés aux États-Unis avec **≥ 65 % de valeur de composants US**. Ces exemptions valent jusqu'au 1er janv. 2027 (https://www.hklaw.com/en/insights/publications/2026/01/fcc-exempts-certain-drones-from-covered-list [EXTRAIT], https://insideunmannedsystems.com/fcc-carves-out-blue-uas-and-buy-american-drones-from-foreign-drone-ban/ [EXTRAIT]).
  - La liste Blue UAS est tenue par la DCMA depuis le 1er janv. 2026 (https://flightbrief.news/topics/blue-uas/ [EXTRAIT]).
  - La vérification passe par un démontage par un tiers, circuit intégré par circuit intégré, et une nomenclature documentée. Le seuil passe à 75 % en 2029 (https://insideunmannedsystems.com/rebuilding-the-supply-chain/ [EXTRAIT]).
  - → Le calcul de valeur % par composant critique (NDAA §848 : contrôleur de vol, radio, caméra, nacelle, GCS, logiciel) est manuel et doit être refait à chaque changement de nomenclature.
- **Idée** : un « HBOM de conformité » qui relie la nomenclature de conception (ERP/PLM), l'origine de chaque pièce, le calcul du pourcentage « domestic end product » et le dossier de soumission Blue UAS/FCC, avec alertes quand un changement de fournisseur fait passer sous le seuil.
- **Ce qui manque pour trancher** :
  - le nombre de fabricants et d'intégrateurs de drones et composants aux États-Unis (non sourcé, aucune recherche dédiée faute de budget) ;
  - un balayage de la concurrence (données d'origine des circuits intégrés : SiliconExpert, Z2Data, Exiger, Assent, non vérifiés dans cette session) ;
  - l'incertitude réglementaire après le 1er janv. 2027.
  - Risque de marché faible : si le nombre de fabricants est de l'ordre de quelques centaines (HYPOTHÈSE non sourcée), le marché logiciel pur reste modeste.


---

## B. Idées rejetées, avec la preuve du rejet (critère + source)

Numéros de critère : (1) vrai problème, (2) piège à goudron, (3) acuité, (4) marché, (5) concurrence, (6) pourquoi maintenant, (7) proxy, (8) scalabilité. Les lignes marquées « À TRANCHER » ne sont pas rejetées : il leur manque l'élément indiqué.

### B1. Embarqué/firmware et conception matérielle (G1)

| Idée | Critère qui échoue | Preuve (source) |
|---|---|---|
| MCP / ferme de cartes hébergée pour que les agents IA flashent et testent du firmware sur matériel réel | (5) Concurrence : angle exact déjà pris | firmware-test-farm-mcp, endpoint hébergé « flash, run, structured pass/fail » [EXTRAIT] https://github.com/oxyvore-cyber/firmware-test-farm-mcp ; embedded-debugger-mcp (probe-rs/OpenOCD, 24 outils) ; Embedder (YC) « reads datasheets…, flashes the board, runs the tests » [EXTRAIT] https://www.ycombinator.com/companies/embedder/jobs/8xz70XR-founding-firmware-engineer ; Anthropic MHS (27/08/2026) [EXTRAIT] recherche « MCP server hardware-in-the-loop » |
| Assistant IA de code embarqué ancré sur datasheets et registres | (5) Concurrence | Embedder (YC), même lien ; Simantic (YC) pour la simulation matérielle destinée à l'IA [EXTRAIT] https://www.ycombinator.com/companies/industry/hardware |
| Génération de modèles de périphériques (Renode/QEMU) pour CI sans carte | (5) Concurrence | Simantic (YC), « hardware simulation developed for the adoption of AI in hardware engineering » [EXTRAIT], même lien |
| Ferme de cartes / HIL CI générique (labgrid/LAVA en SaaS) | (5) Concurrence + open source dominant | labgrid + Linux Automation GmbH (matériel dédié), Linaro Automation Appliance, Jumpstarter (Red Hat) [EXTRAIT] https://pengutronix.de/en/software/labgrid.html ; https://www.linaro.org/blog/building-reliable-real-hardware-ci-lessons-from-15-years-of-test-labs |
| SBOM firmware sensible à la config + VEX (générique, Linux) | (5) Concurrence | Yocto : VEX natif + filtrage des fichiers noyau (−70 à 80 % de faux positifs) [EXTRAIT] https://docs.yoctoproject.org/next/security-manual/vulnerabilities.html ; Vigiles filtre selon le `.config` ; RunSafe Identify (VEX + reachability) |
| Outil de déclaration CRA sous 24 h (plateforme ENISA SRP) | (5) Concurrence (marché saturé de conformité) | CVD Portal, CRA Evidence, Zealience, sbomify, Element, etc. [EXTRAIT] https://cvdportal.com/cra-reporting-obligations ; https://zealience.com/resource-hub/cyber-resilience-act-article-14-reporting/ ; https://craevidence.com/blog/how-to-generate-firmware-sbom |
| Revue IA de schémas PCB | (2) Piège documenté (pivot) + (5) | JITX (YC S24) a commencé par la revue de schémas puis a pivoté vers le « design as code » [EXTRAIT] https://www.protoflow.ai/compare/best-ai-pcb-design-software-2026 ; Flux Copilot, CircuitMind |
| Routage / placement PCB autonome | (5) Concurrence financée | Quilter (Series B de 25 M$, octobre 2025, 40 M$ au total), DeepPCB (InstaDeep), Flux, Cadence, Siemens [EXTRAIT] https://www.quilter.ai/blog/the-2026-guide-to-autonomous-pcb-design-quilter-vs-deeppcb-vs-flux-ai |
| Génération de symboles et footprints depuis la datasheet | (5) Concurrence | SnapMagic InstaBuild (OCR du tableau de brochage), ProtoFlow Part Generator (IPC-7351B, exports KiCad/Altium/Allegro) [EXTRAIT] https://www.snapeda.com/instabuild/ ; https://www.protoflow.ai/generate-parts |
| Agents de vérification RTL (UVM, assertions formelles, testbenches) | (5) Concurrence dense et financée | ChipAgents (étude avec STMicro : UVM ×400, assertions ×240), Bronco AI, Cognichip [EXTRAIT] https://chipagents.ai/blogs/ai-agents-uvm ; https://bronco.ai/ |
| Crates Rust embarqué certifiés (HAL, runtime) pour la sûreté | (5) Concurrence + (4) marché étroit (non chiffré) | Ferrocene : core certifié SIL 2 (décembre 2025), ASIL B en version 26.02, 5 169 fonctions ; HighTec : compilateur Rust ASIL D + PXROS-HR ; Veecle (runtime sur PXROS) ; Bluewind (pilotes AURIX) ; OxidOS [EXTRAIT] https://ferrous-systems.com/blog/ferrocene-26-02-0/ ; https://github.com/veecle/veecle-pxros ; https://hightec-rt.com/products/rust-development-platform |

### B2. Exigences/MBSE et sûreté/certification (G2)

| Idée | Critère qui échoue | Preuve (URL) |
|---|---|---|
| « GitHub pour SysML v2 » : dépôt Git, PR, diff et merge sémantiques de modèles | (5) Concurrence : angle déjà pris | SysGit : plateforme Git-centrée SysML v2 textuel et graphique avec branches et PR (https://www.sysgit.io/ [EXTRAIT]). LemonTree for SysML v2 : diff et merge cohérents .sysml/.json, v5.0 prévue fin juin 2026 (https://www.lieberlieber.com/en/lieberlieber-lemontree-for-sysmlv2/ [EXTRAIT]). Syside, avec Git natif (https://sensmetry.com/advent-of-sysml-v2-lesson-6-version-control-with-git/ [EXTRAIT]). |
| Serveur conforme à l'API Systems Modeling (combler l'absence de diff/merge dans l'implémentation pilote) | (5) Concurrence | La spec OpenAPI 1.0 définit `/commits/{compareCommitId}/diff` et `/branches/{targetBranchId}/merge` (le 409 signifie conflit laissé au client) ; les routes du pilote ne les implémentent pas [OUVERT, voir C]. Mais OpenMBEE Flexo SysMLv2 implémente l'API (https://www.openmbee.org/flexo.html, https://github.com/Open-MBEE/flexo-mms-sysmlv2 [EXTRAIT]), et LemonTree couvre le merge. |
| Nouvel outil RM/MBSE « AI-native » généraliste | (5) Concurrence saturée | Flow Engineering (23 M$ Series A Sequoia, 10/2025 : https://www.flowengineering.com/blog/flow-raises-23m-from-sequoia-to-accelerate-the-future-of-hardware-development [EXTRAIT]), Dalus (YC, MBSE SysML v2, analyse de dangers : https://www.ycombinator.com/companies/dalus [EXTRAIT]), Trace.Space (4 M$ seed, Cherry : https://www.eu-startups.com/2025/02/from-days-to-minutes-trace-space-raises-e3-8-million-for-requirements-management-platform/ [EXTRAIT]), Valispace (racheté par Altium pour 19,97 M$, puis groupe Renesas : https://www.marketscreener.com/quote/stock/ALTIUM-10353100/news/Altium-Limited-acquired-Valispace-GmbH-forr-19-97-million-46034563/ [EXTRAIT]), Artifact (YC, https://www.ycombinator.com/companies/artifact-2 [EXTRAIT]). |
| Hub d'échange ReqIF OEM/fournisseur sans perte (ID stables en aller-retour) | (5) Concurrence + (2) piège probable | Le problème est réel : réexporter crée de nouveaux ID, donc des doublons côté client (https://codebeamer.com/cb/wiki/37458975, https://www.reqview.com/blog/import-export-reqif/ [EXTRAIT]). Mais EVOCEAN propose déjà une « ReqIF exchange platform » (https://reqif.evocean.com/learn/reqif-polarion-integration [EXTRAIT]), et chaque ALM (Codebeamer, Polarion) gère la valeur des champs en aller-retour. Marché de niche d'intégration. |
| Safety case « vivant » / continu (GSN + SPI) pour les véhicules autonomes | (5) Concurrence | Edge Case Research : nLoop (safety case vivant) et Guardian (« Digital Safety Twin », suit l'évolution du safety case) (https://www.fleetequipmentmag.com/edge-case-guardian-autonomous-system-safety/ [EXTRAIT]). Applied Intuition a racheté la gestion de safety case d'Embark (https://research.contrary.com/company/applied-intuition [EXTRAIT]). UL 4600 éd. 3 impose les SPI (https://www.ul.com/news/ul-4600-edition-3-updates-incorporate-autonomous-trucking [EXTRAIT]). |
| Plateforme de conformité IEC 62304 / DO-178C par traçabilité automatique | (5) Concurrence | Ketryx (plus de 55 M$ levés, médical, automobile et aéronautique) : https://www.ketryx.com/press-release/series-b, https://www.ketryx.com/industries/aerospace [EXTRAIT]. |
| Rapports de conformité hardware et robotique (ISO 12100/10218) générés par IA | (5) Concurrence | Saphira AI (YC S24) : https://www.ycombinator.com/companies/saphira-ai [EXTRAIT]. |
| Kits de qualification pour chaînes d'outils open source (llvm-cov MC/DC, clang-tidy) | (5) Concurrence | Validas QKit llvm-cov et clang-tidy (https://www.validas.de/solutions/qkits/qkits-tool-qualification/llvm-cov ; https://discourse.llvm.org/t/safety-tool-qualification-of-llvm-features-coverage-and-clang-tidy/85276 [EXTRAIT]) ; Ferrocene pour Rust. |
| Outil STPA assisté par LLM | À TRANCHER : (4) marché | Normes : SAE J3187 (2022, puis 2023 « any industry », annexe MBSE 2023 : https://www.sae.org/standards/j3187-3_202309-system-theoretic-process-analysis-stpa-recommended-practices-evaluations-safety-critical-systems-industry-appendix-stpa-model-based-systems-engineering-mbse [EXTRAIT]). Recherche active (https://arxiv.org/html/2503.12043 [EXTRAIT]). Aucune startup commerciale trouvée. L'outillage STPA existe chez Astah System Safety (https://scsc.uk/tools [EXTRAIT]) et Ansys medini (non vérifié ici). **Manque** : nombre de praticiens STPA et prix payé, aucune donnée trouvée. |
| Outil GSN / assurance case « as code » | (5) + (2) | L'outillage open source existe : gsn2x (YAML vers SVG, extension VS Code) [OUVERT https://github.com/jonasthewolf/gsn2x] ; Assurance Forge (SACM, alpha) [OUVERT https://github.com/lasrod/assurance-forge]. Côté commercial : ASCE, Socrates, AdvoCATE (https://criticalsystemslabs.com/socrates-assurance/ [EXTRAIT]). L'outil stand-alone Astah GSN a été abandonné (https://scsc.uk/tools [EXTRAIT]), ce qui signale un marché faible. |
| Migration DOORS Classic vers moderne | (6) Pourquoi maintenant faible | IBM supportera DOORS 9.7 au-delà de la période standard, avec au moins 12 mois de préavis ; aucune date annoncée (https://www.ibm.com/support/pages/ibm-engineering-requirements-management-doors97x [EXTRAIT]). Services de migration déjà nombreux (Jama, MG Tech). |

### B3. SDV et simulation/jumeaux numériques (G3)

| Idée | Critère qui fait échouer | Preuve / source |
|---|---|---|
| Diff/merge sémantique ARXML pour Git | (5) Concurrence ayant exactement l'angle | dSPACE AUTOSAR Compare s'intègre comme outil de diff/merge Git, avec un merge à trois sessions LOCAL/BASE/REMOTE [EXTRAIT] https://www.dspace.com/en/pub/home/news/engineers-insights/dspace_autosar_compare_git.cfm ; ARForge v1.4.0 fait du diff de modèle intégré à Git [EXTRAIT] https://dev.to/bzivkovic86/arforge-v140-git-integrated-model-diffing-for-autosar-classic-projects-4kgf |
| Assistant IA de configuration BSW (EB tresos / DaVinci) | (5) Concurrent existant | AutoC propose un assistant IA pour la configuration BSW EB tresos et la synchronisation ECUC EB↔DaVinci [EXTRAIT] https://www.autoc-tool.com/en/guide/eb-tresos. À TRANCHER si l'on vise l'Adaptive (non exploré faute de budget). |
| TARA automatisée par IA (ISO/SAE 21434) | (5) Marché saturé | ThreatZ/VxLabs revendique « TARA time -85 % » [EXTRAIT] https://threatz.io/automotive-tara/ ; PlaxidityX Security AutoDesigner (IA) [EXTRAIT] https://plaxidityx.com/products/security-autodesigner/ ; Panasonic VERZEUSE (2024) [EXTRAIT] https://news.panasonic.com/global/press/en241024-4 ; ainsi que Cybellum, VicOne et itemis |
| Mise en correspondance R155 ↔ GB 44495 pour exportateurs | (5) Déjà couvert | « A single ThreatZ project can generate compliance evidence mapped to R155, ISO/SAE 21434, GB 44495 » [EXTRAIT] https://vxlabs.ai/gb-44495/ ; VicOne xZETA [EXTRAIT] https://vicone.com/products/xzeta/ |
| Plateforme OTA générique (façon Uptane) | (2) Piège à goudron et (5) saturation | Karma n'a racheté que les « assets and key personnel » d'Airbiquity, pionnier de l'OTA, en février 2024 [EXTRAIT] https://karmaautomotive.com/karma-news/karma-automotive-acquires-assets-and-key-personnel-from-connected-vehicle-pioneer-airbiquity/ ; les places sont déjà prises par Sibros, Excelfore, Mender et Sonatus [EXTRAIT] |
| Traqueur de preuves R156 / SUMS | (5) Concurrent ayant l'angle | Sibros R156 Readiness Tracker (URL en A, idée 2) |
| Substituts IA / CFD neuronale | (5) Saturé et très financé | PhysicsX : 300 M$ en série C, valorisation 2,4 Md$ (juin 2026) [EXTRAIT] https://sifted.eu/articles/physicsx-funding-round-temasek-valuation ; Navier AI (YC W24, 5,6 M$) [EXTRAIT] https://www.ycombinator.com/launches/KRn-navier-ai-real-time-cfd-simulations ; Godela et SuperRadiant (YC) [EXTRAIT] recherche site:ycombinator.com |
| Service payant de conformité FMI 3.0 / SSP (remplaçant du Cross-Check) | (4) Marché trop petit (calcul) | Environ 280 outils SSP [EXTRAIT NAFEMS] × environ 10 k$/an (HYPOTHÈSE non sourcée) ≈ 2,8 M$. Le Reference-FMUs et le FMU Compliance Checker sont gratuits. La partie « tests de régression FMU » est intégrée à l'idée 1. |
| Compilateur / runner OpenSCENARIO 2.x | (5) et (4) | Couvert par des projets de recherche open source (OSC2Runner, compilateur OSC 2.1 pour CARLA) [EXTRAIT] https://arxiv.org/html/2606.26533v1 et https://arxiv.org/html/2604.16452 ; Foretellix (auteur du langage M-SDL à l'origine d'OSC 2.0) et Applied le supportent [EXTRAIT] https://www.appliedintuition.com/blog/asam-openscenario-v2 ; pas de marché payant séparé démontré |
| Mapping automatique VSS ↔ signaux CAN/DBC | (4) Marché douteux, À TRANCHER | VSS ne définit que la sémantique (722 nœuds de branche/signal dans 68 fichiers .vspec en v6.1) ; le mapping se fait au cas par cas via des outils ouverts (dbc2vss de Kuksa). Aucune source sur une disposition à payer. [OUVERT] clone VSS |

### B4. Robotique/drones, spatial et défense (G4)

| # | Idée | Critère qui rejette | Preuve (URL) |
|---|---|---|---|
| 1 | Plateforme de données et observabilité robotique (recherche dans les rosbag/MCAP, relecture, diagnostic de flotte) | (5) Concurrence dense, même angle | Foxglove : Série B de 40 M$, plus de 58 M$ levés, format MCAP (https://www.therobotreport.com/foxglove-raises-40m-scale-data-platform-roboticists/ [EXTRAIT]) ; Roboto AI : recherche en langage naturel dans les logs, tous formats (https://www.roboto.ai/ [EXTRAIT]) ; Rerun, Formant (paysage : https://segments.ai/blog/software-tools-for-robotics-landscape/ [EXTRAIT]) |
| 2 | Logiciel d'évaluation des risques ISO 10218-2:2025 et d'évaluation des menaces cyber pour intégrateurs | (5) | Safexpert (processus CE complet, évaluation des risques : https://www.ibf-solutions.com/en/software-risk-assessment [EXTRAIT]) ; Safetics (guide ISO 10218-2:2025 incluant l'évaluation des menaces cyber : https://en.doc.safetics.io/insight-10218-2/ [EXTRAIT]) ; Nemko Digital, SyncSoft AI sur le Règlement Machines 2027 (https://digital.nemko.com/regulations/eu-machinery-regulation [EXTRAIT]). Sous-angle « dérive des configurations de sécurité des contrôleurs robots » : **À TRANCHER** (concurrents de versionnage de programmes d'automates non vérifiés ici) |
| 3 | Outil d'automatisation SORA 2.5 (autorisation opérationnelle UE) | (5) + gratuit public | L'outil **eSORA** de l'EASA automatise SORA 2.5 et compile les preuves (https://www.easa.europa.eu/en/newsroom-and-events/press-releases/easa-presses-accelerator-support-drone-operations-eu [EXTRAIT]) ; AirHub, UAV-Planner (https://www.airhub.app/resources/news/sora-2-5-key-changes, https://www.uav-planner.com/en/blog/sora-drone-risk-assessment-guide [EXTRAIT]) |
| 4 | Plateforme de conformité opérateur FAA Part 108 (BVLOS) | **À TRANCHER** (6) : texte final inconnu | Règle finale non publiée, à l'OIRA depuis le 10 juil. 2026 ; publication espérée fin 2026 (https://www.theflightbrief.com/articles/faa-part-108-bvlos-update-september-2026 [EXTRAIT], https://airdata.com/blog/2026/part-108 [EXTRAIT]). Les exigences de données et d'enregistrement du NPRM restent imprécises (https://www.faa.gov/newsroom/BVLOS_NPRM_website_version.pdf [EXTRAIT]). Il manque le texte final. |
| 5 | SaaS d'évaluation de conjonction et d'évitement de collision pour petits opérateurs | (5) | Kayhan Pathfinder : évaluation, planification de manœuvre et coordination entre opérateurs (https://spacenews.com/kayhan-coordinated-collision-avoidance/ [EXTRAIT]) ; Neuraspace, OKAPI:Orbits (https://infineospace.substack.com/p/satellite-collision-avoidance-innovation [EXTRAIT]) ; offre publique TraCSS (70 utilisateurs pilotes, 11 345 satellites : https://www.satellitetoday.com/government-military/2026/08/26/tracss-traffic-coordination-system-remains-in-pilot-mode-due-to-budget-uncertainty/ [EXTRAIT]) |
| 6 | Framework ou OS de vol générique + plateforme d'opérations « tout-en-un » pour petits satellites | (2) Piège documenté | Kubos (KubOS + Major Tom) : actifs rachetés par Xplore en 2022 ; le dépôt KubOS a disparu de GitHub (https://www.geekwire.com/2022/xplore-acquires-assets-of-kubos-flight-software-company-as-it-ramps-up-for-first-space-mission/ [EXTRAIT], https://www.spacebandits.io/startups/kubos [EXTRAIT]). Concurrence gratuite : F´ (NASA/JPL, open source) et cFS |
| 7 | Traçabilité et conformité NPR 7150.2 / ECSS-E-ST-40C « dans GitHub » | (5) | LDRA TBmanager (traçabilité bidirectionnelle NPR 7150.2D : https://ldra.com/npr7150-2d/ [EXTRAIT]) ; Trace.Space (connecté à Jira, Git et CI : https://www.trace.space/industries/software [EXTRAIT]) ; MathWorks ; NASA ASEP « Process Compliance Copilot » pour la classe D sur GitHub (https://ntrs.nasa.gov/api/citations/20250006564/downloads/DASC25_ASEP_Process_Compliance_Copilot_revf.pdf [EXTRAIT]) ; `nasa/coda` NPR-7150.2D.yml (https://github.com/nasa/coda/blob/int/NASA-NPR-7150.2D.yml [EXTRAIT]) |
| 8 | Outil de conformité CMMC Niveau 2 pour PME | (3)+(5)+(6) | Phase 2 suspendue le 13 juil. 2026, revue de 60 jours, rapport attendu fin sept.–oct. 2026 (https://federalnewsnetwork.com/cybersecurity/2026/07/pentagon-suspends-cmmc-phase-two-requirements-launches-review-of-program/ [EXTRAIT], https://www.insidegovernmentcontracts.com/2026/09/cmmc-reform-task-force-updates-september-2026/ [EXTRAIT]). Marché saturé : Vanta, Drata, Secureframe, PreVeil, Summit 7, Totem (https://secureframe.com/blog/cmmc-phase-2-preparation [EXTRAIT]) |
| 9 | Outil SBOM / évaluation tierce pour SWFT (Software Fast Track DoD) | (5) | Anchore, Sonatype, NetRise, ReversingLabs positionnés explicitement sur SWFT (https://anchore.com/blog/dod-swft-initiative-and-promise-of-cato-fulfilled/, https://www.sonatype.com/blog/understanding-swft-the-latest-effort-to-modernize-dod-software-procurement, https://www.netrise.io/blog/swft-atos-why-sbom-validation-needs-binary-analysis [EXTRAIT]) |
| 10 | Plateforme cATO / ATO héritée pour éditeurs voulant vendre au DoD | (5) | Second Front Game Warden : FedRAMP High, IL5, ATO en environ 90 jours (https://www.secondfront.com/products/game-warden/ [EXTRAIT]) ; Defense Unicorns UDS, UDS Registry, UDS Fleet (https://defenseunicorns.com/platform/uds-platform/ [EXTRAIT]) |
| 11 | Passerelle IA « ITAR » générique (scan de prompts, vérification US person) | (5) | Kiteworks (MCP, récupération tenant compte de la classification), TrueFoundry, Fasoo, Cyberhaven, Concentric, Tabnine on-prem (URL en A1). C'est pour cela que A1 se positionne sur le graphe Git et la classification par chemin |
| 12 | Outillage de diagnostic et scalabilité DDS/QoS ROS 2 | **À TRANCHER** (4) + mécanisme résorbé | Le trafic de découverte DDS en n² et les tempêtes de paquets sur Wi-Fi sont traités par rmw_zenoh, visé en Tier 1 pour Kilted : routeur TCP, 97–99 % de trafic de découverte en moins rapporté (https://zenoh.io/blog/2021-03-23-discovery/, https://github.com/ros2/rmw_zenoh/issues/265 [EXTRAIT]). Il manque une estimation du marché payant (Foxglove et les outils ROS existants couvrent une partie de l'introspection) |

---

## C. Questions techniques posées et réponses, par sous-couche

### C1. Logiciel embarqué et firmware (RTOS, Rust embarqué, toolchains, tests sur matériel/HIL, assistants IA, CRA)

#### RTOS (Zephyr, FreeRTOS, ThreadX/Eclipse, SafeRTOS)

- **Q : Quelle garantie de maintenance donnent les RTOS et SDK face aux 5 ans du CRA ?**
  - Zephyr LTS : « au moins 5 ans » ([OUVERT] release_process.rst).
  - ESP-IDF : 30 mois par release, sans correctif après la fin de vie ([OUVERT] SUPPORT_POLICY.md).
  - Il y a donc un écart structurel pour les SDK vendeurs, ce qui nourrit l'idée 1, phase 2.
- **Q : Comment Zephyr trace-t-il les vulnérabilités ?**
  - Au niveau module via CPE/PURL dans `module.yml`. SBOM `west spdx` au niveau fichier (SPDX 2.3 par défaut, 3.0 possible, profil Build).
  - Pas de `cve-check` intégré : l'issue #85570 reste ouverte et « Stale » ([OUVERT] zephyr-cmds.rst, modules.rst ; issue via WebFetch).
- **Q : Comment les scanners binaires voient-ils un firmware RTOS sans symboles ?**
  - ONEKEY identifie l'architecture, l'adresse de chargement et les composants par signatures de fonctions de releases précompilées ([EXTRAIT] onekey.com, lien plus haut).
- Non approfondi, faute de budget : SafeRTOS, ThreadX sous Eclipse (licence, certification).

#### Rust embarqué (Ferrocene, embassy, probe-rs)

- **Q : Qu'est-ce qui est qualifié ou certifié ?**
  - Compilateur Ferrocene : ISO 26262 ASIL D, IEC 61508 SIL 3, IEC 62304 classe C.
  - Sous-ensemble de `core` : SIL 2 (décembre 2025), puis ASIL B (26.02) ; le nombre de fonctions certifiées passe de 2 903 à 5 169.
  - Embassy et les HAL ne sont pas certifiés d'après les sources trouvées ([EXTRAIT] ferrous-systems.com ; theregister.com 2025/12/04).
- **Q : Qui comble le trou HAL/runtime ?** HighTec, Veecle, Bluewind, Infineon (écosystème AURIX), OxidOS : l'espace est occupé (voir tableau B).
- **Q : probe-rs sert-il de base aux agents ?** Oui : embedded-debugger-mcp expose probe-rs/OpenOCD, avec 24 outils, flash et RTT ([EXTRAIT] mdskills.ai).

#### Toolchains (compilateurs qualifiés, CMake/west, Yocto)

- **Q : Comment Yocto réduit-il les faux positifs CVE ?**
  - Justifications VEX via `CVE_STATUS` (par exemple `not-applicable-config` → `vulnerableCodeNotPresent`).
  - Filtrage du noyau selon les fichiers compilés, grâce aux métadonnées du CNA noyau : −70 à 80 % de faux positifs ([EXTRAIT] docs.yoctoproject.org ; slides FOSDEM 2025).
  - Bug connu : le VEX embarqué ne supprime rien dans sbomify, dont le matcher ne compare que les PURL ([EXTRAIT] https://github.com/sbomify/sbomify/issues/1555).
- **Q : Les LTS Yocto et noyau couvrent-ils la période CRA ?**
  - Noyau : 5.10 jusqu'en décembre 2026, 6.6 jusqu'en décembre 2027, 6.12 et 6.18 jusqu'en décembre 2028. Insuffisant face à 5 ans ou plus (idée 1).
  - Durée des LTS Yocto : non vérifiée faute de budget.

#### Tests sur matériel / HIL (dSPACE, NI, fermes de cartes, Renode, QEMU)

- **Q : Quelle infrastructure open source et commerciale existe pour la CI sur carte ?** labgrid (Pengutronix) avec le matériel de Linux Automation GmbH, LAVA et Linaro Automation Appliance, Jumpstarter ([EXTRAIT] liens tableau B). Linaro publie un retour d'expérience sur 15 ans de laboratoires de test.
- **Q : Les agents IA ont-ils accès au matériel ?** Oui, déjà : firmware-test-farm-mcp (hébergé), embedded-mcp (TI CC26xx, nRF52/91), Embedder (YC), MHS d'Anthropic (08/2026). Idée rejetée.
- Non approfondi, faute de budget : dSPACE et NI (coût des bancs HIL), Renode et QEMU (couverture des périphériques). C'est la principale zone non couverte de ce rapport.

#### Assistants IA de code sur code embarqué

- **Q : Où cassent-ils ?** Absence de vérité terrain matérielle (registres, timing). Les solutions en cours ferment la boucle avec le matériel via MCP ou fermes (C4) ou avec la datasheet (Embedder).
- **Q : Constat côté noyau Linux ?** Environ 1 commit sur 5 porte désormais la mention `Assisted-by:`, et l'IA accélère la découverte de CVE ([EXTRAIT] https://www.linuxcompatible.org/story/linux-kernel-727-released-737-commits-security-fixes-and-aiassisted-development/). Cela augmente la charge de backport en aval (idée 1).

#### EU Cyber Resilience Act (firmware / SBOM)

- **Q : Qu'est-ce qui s'applique depuis le 11/09/2026 ?**
  - Art. 14 : alerte précoce sous 24 h pour une vulnérabilité activement exploitée, notification sous 72 h, rapport final sous 14 jours (1 mois pour un incident grave).
  - Dépôt via la Single Reporting Platform de l'ENISA, adressée au CSIRT coordinateur ([EXTRAIT] https://www.crowell.com/en/insights/client-alerts/its-live-the-cyber-resilience-act-reporting-is-mandatory-as-of-today-11-september-2026 ; https://www.hlc.com/en/publications/eu-cyber-resilience-act-vulnerability-and-incident-reporting-obligations-now-apply).
- **Q : Quelle durée de correctifs ?** Art. 13(8) : au moins 5 ans. Art. 13(9) : chaque mise à jour reste disponible 10 ans ([EXTRAIT] article 13).
- **Q : Pourquoi le SBOM firmware ne suffit-il pas ?**
  - L'inventaire établi à la compilation ne correspond pas au produit livré : liaison statique, code vendorisé, forks patchés.
  - Plus de 50 % de faux positifs ([EXTRAIT] runsafesecurity.com).
  - Pour dire « présent / absent », il faut la lignée du fork et la config (techveda, ONEKEY). Cela alimente les idées 1 et 2.

### C2. Ingénierie des exigences et MBSE (SysML v2, API, traçabilité, DOORS/Polarion/Jama/Codebeamer, ReqIF)

**Q1. Où en est SysML v2 ?**
- L'OMG a approuvé l'adoption finale de SysML 2.0, KerML 1.0 et Systems Modeling API & Services 1.0 en juillet 2025 (adoption formelle au 30/06/2025, publication en 09/2025).
- Sources : https://www.omg.org/news/releases/pr2025/07-21-25.htm [EXTRAIT] ; https://github.com/Systems-Modeling/SysML-v2-Release [EXTRAIT].

**Q2. Que fait vraiment l'API REST ? Le versionnement est-il complet ?**
- Le modèle est de type Git : Project, Branch, Tag, Commit, Change, Element, Relationship, Query.
- La spécification OpenAPI 1.0 (`public/docs/openapi.json`, 23 chemins) définit `GET /projects/{projectId}/commits/{compareCommitId}/diff` et `POST /projects/{projectId}/branches/{targetBranchId}/merge`, avec les réponses 201/409/404. **Le 409 signifie qu'en cas de conflit, la résolution revient au client.**
- **Les routes de l'implémentation de référence (`conf/routes`) n'implémentent ni diff ni merge.** Elles n'exposent que CRUD projets/branches/tags, commits/changes, éléments/relations et requêtes.
- Sources : https://raw.githubusercontent.com/Systems-Modeling/SysML-v2-API-Services/master/conf/routes [OUVERT] ; https://raw.githubusercontent.com/Systems-Modeling/SysML-v2-API-Services/master/public/docs/openapi.json [OUVERT].
- **Conséquence :** la fusion sémantique de modèles est laissée aux éditeurs. LemonTree, SysGit et Flexo occupent cet espace (tableau B).

**Q3. La notation textuelle rend-elle Git suffisant ?**
- Sensmetry affirme que branches, PR, diffs et blame fonctionnent tels quels.
- LieberLieber objecte qu'un merge textuel peut produire un modèle invalide, et que seul un merge conscient du modèle garantit la cohérence.
- Sources : https://sensmetry.com/advent-of-sysml-v2-lesson-6-version-control-with-git/ ; https://www.lieberlieber.com/en/lieberlieber-lemontree-for-sysmlv2/ [EXTRAIT].

**Q4. Qui outille SysML v2 ?**
- Syside (Sensmetry : éditeur gratuit, diagrammes et automatisation payants), SysON (Obeo), SysGit, Dalus (YC), OpenMBEE Flexo, Cameo 2026x via plugin (exécution de modèles, vérification d'exigences), Ansys SAM 2026 R1.
- Sources : https://github.com/jgsystemsconsulting/awesome-sysml-v2 ; https://github.com/Open-MBEE/OpenSysML/pull/470 ; https://www.ansys.com/blog/advancing-mbse-ansys-sam-2026-r1 [EXTRAIT].

**Q5. Consolidation du marché RM ?**
- Valispace racheté par Altium le 21/12/2023 (15,56 M$ cash + 4,41 M$ différés), Altium elle-même rachetée par Renesas.
- Flow Engineering : rentable à 7 personnes en 02/2025, Series A de 23 M$ chez Sequoia en 10/2025.
- Trace.Space : 4 M$ en seed.
- « Sphinx » : les résultats renvoient à des homonymes (fintech, data) ; pas de startup SE trouvée sous ce nom.
- Sources : https://www.marketscreener.com/... (voir B) ; https://research.contrary.com/company/flow-engineering [EXTRAIT] ; https://www.se-trends.de/en/renesas-buys-altium-buys-valispace/ [EXTRAIT].

**Q6. Où casse l'échange ReqIF ?**
- Réexporter génère de nouveaux ID, que l'outil du client réimporte en doublons. Les types d'attributs et de liens personnalisés ne sont pas interprétés de la même façon selon l'outil.
- Sources : https://www.reqview.com/blog/import-export-reqif/ ; https://codebeamer.com/cb/wiki/37458975 [EXTRAIT].

**Q7. DOORS Classic ?**
- La version 9.6.x est en fin de support en 09/2025. La 9.7.x bénéficie d'un support étendu, sans date de fin, avec au moins 12 mois de préavis promis.
- Sources : https://www.ibm.com/support/pages/ibm-engineering-requirements-management-doors97x ; https://www.jamasoftware.com/blog/ibm-doors-software/ [EXTRAIT].

### C3. Sûreté et certification (ISO 26262, SOTIF, DO-178C/DO-330/DO-333, IEC 62304, IEC 61508, safety cases, qualification d'outils, code généré par IA)

**Q8. Quand faut-il qualifier un outil (DO-330) ?**
- Quand il élimine, réduit ou automatise un processus sans que sa sortie soit vérifiée.
- TQL-1 correspond à un outil qui génère du logiciel de niveau A. DO-330 est applicable hors aéronautique.
- Source : https://www.tasking.com/do-330/ [EXTRAIT].
- Déduction : un LLM non vérifié relèverait de TQL-1, ce qui est irréaliste. Toute la valeur se déplace donc vers la vérification de la sortie.

**Q9. Comment ISO 26262 traite-t-il les outils ?**
- TI et TD déterminent le TCL. TCL1 dispense de qualification. Les méthodes de qualification sont 1a à 1d ; exemple CodeQL : TI2/TD2, donc TCL2, qualifié par 1b et 1c.
- Sources : https://github.com/github/codeql-coding-standards/blob/main/docs/iso_26262_tool_qualification.md [OUVERT] ; https://www.renesas.com/en/blogs/renesas-fusa-support-automotive-4-confidence-use-software-tools-aiml-development [EXTRAIT].

**Q10. Quel cadre pour l'IA embarquée ?**
- EASA AI Concept Paper Issue 02 (niveaux 1 et 2, learning assurance, human-AI teaming), puis RMT.0742 vers des AMC : https://www.easa.europa.eu/en/newsroom-and-events/news/easa-publishes-artificial-intelligence-concept-paper-issue-2-guidance [EXTRAIT].
- ED-324/ARP6983 (G-34/WG-114) : publication visée en juin 2026, limitée aux modèles figés en apprentissage supervisé (source eventscloud ci-dessus) [EXTRAIT].
- ISO/PAS 8800:2024 : IA embarquée dans le véhicule, étend ISO 26262 et 21448 ; l'IA hors véhicule est hors périmètre, ce qui exclut les outils de développement IA : https://www.iso.org/standard/83303.html [EXTRAIT].
- **Aucun de ces textes ne traite le code *écrit* par IA ; c'est le vide exploité par l'idée 1.**

**Q11. Les méthodes formelles donnent-elles du crédit de certification ?**
- Oui, via DO-333 (supplément de DO-178C) : unit proof chez Airbus (A380, A400M, A350), avec des activités alternatives pour la couverture structurelle (sources de l'idée 1).

**Q12. La génération de spécifications et de preuves par LLM est-elle mûre ?**
- AutoACSL (06/2026) : 98 % de génération de spécifications, 96 % de preuves complètes.
- spec2code : code automobile vérifié par Frama-C sur 2 des 3 modules.
- Sources : https://arxiv.org/abs/2606.20969 ; https://arxiv.org/html/2411.13269 [EXTRAIT].

**Q13. Mutation et injection de fautes ?**
- Recommandées dans ISO 26262 (fortement en ASIL C/D).
- Outils C/C++ open source : Mull (LLVM) et Dextool (Saab).
- Sources : https://www.embitel.com/blog/embedded-blog/fault-injection-testing-of-safety-critical-automotive-software ; https://itea4.org/project/exploitable-result/221/dextool-mutate-a-mutation-testing-tool-for-c-c.html [EXTRAIT].

**Q14. Couverture MC/DC open source ?**
- Clang 17+ avec `-fcoverage-mcdc`.
- Kit de qualification Validas pour llvm-cov.
- Un outil open source de couverture pour DO-178C a reçu le prix du meilleur article à DASC 2025.
- Sources : https://clang.llvm.org/docs/SourceBasedCodeCoverage.html ; https://github.com/xlab-uiuc/linux-mcdc [EXTRAIT].

**Q15. Qui fait les safety cases ?**
- Pour les véhicules autonomes : Edge Case (nLoop, Guardian) et Applied Intuition.
- Côté outillage GSN : ASCE (v5.1, 10/2022), Socrates, AdvoCATE (NASA), Astah System Safety.
- En open source : gsn2x, Assurance Forge (alpha).
- Voir le tableau B pour les sources.

**Q16. Chaînes d'outils qualifiées open source : modèle économique ?**
- Ferrocene : ISO 26262 ASIL D, IEC 61508 SIL 3 et IEC 62304 classe C (01/2025), à 240 € par siège et par an.
- Sources : https://ferrous-systems.com/blog/ferrocene-achieves-iec-62304-qualification/ ; https://ferrous-systems.com/blog/officially-qualified-ferrocene/ [EXTRAIT].

**Q17. Qu'est-ce qui a déjà été financé chez YC dans ces sous-couches ?**
- Dalus (MBSE), Artifact (IDE hardware), Saphira AI (certification hardware, S24).
- La recherche `site:ycombinator.com/companies` avec ISO 26262, DO-178C ou IEC 62304 n'a renvoyé aucune entreprise YC.

### C4. Véhicules définis par logiciel (AUTOSAR, S-CORE, VSS, OTA/Uptane, R155/R156, Chine)

**AUTOSAR Classic / Adaptive, ARXML, DaVinci, EB tresos**
- Q : Pourquoi les diffs ARXML sont-ils inexploitables dans Git ? R : Chaque outil sérialise les balises XML dans un ordre différent, ce qui crée un « wall of XML noise ». Des outils de diff sémantique élément par élément existent déjà (dSPACE AUTOSAR Compare, ARForge) [EXTRAIT] URLs en B.
- Q : Comment une configuration MCAL faite sous EB tresos rejoint-elle une pile BSW DaVinci ? R : La guide d'intégration TI indique que les modules MCAL configurés sous EB tresos doivent être importés dans DaVinci Configurator pour que les modules Vector puissent les référencer. On a donc une double chaîne d'outils et une synchronisation ECUC [EXTRAIT] https://software-dl.ti.com/mcu-plus-sdk/esd/PLATFORM_SW_MCAL/AM263x/latest/src/Integration.html ; AutoC propose une synchronisation EB↔DaVinci [EXTRAIT].
- Q : Y a-t-il des startups YC sur AUTOSAR ? R : La recherche site:ycombinator.com n'en fait remonter aucune explicitement (Olympian Motors, Carma…) [EXTRAIT] https://www.ycombinator.com/companies/industry/automotive

**Eclipse SDV / S-CORE**
- Q : Quel niveau de sûreté vise S-CORE et comment gère-t-il l'open source ? R : Les exigences portent `:safety: ASIL_B`, avec une règle « no_mixed_asil » (module ASIL ou QM dans son ensemble) et une checklist « If OSS software components is used, is it planned to be qualified? ». Le dépôt fonctionne sous Bazel, sous Linux uniquement, et prépare une « Reference Integration V1.0 » ; il est actif (commit du 2026-09-23). [OUVERT] https://github.com/eclipse-score/score (docs/safety, docs/score_releases)
- Implication : la qualification des composants open source en ASIL-B est un besoin, mais il est déjà servi par des consultants et des éditeurs de chaînes qualifiées. Non approfondi (budget).

**COVESA VSS**
- Q : VSS garantit-il la stabilité des signaux ? R : Non. Les versions majeures suppriment des signaux (« Signals deprecated in 4.X versions removed » en 5.0, branche OBD dépréciée). La v6.1 ajoute des enums et des unités QUDT, et change le format GraphQL dans une version mineure. [OUVERT] https://github.com/COVESA/vehicle_signal_specification (CHANGELOG.md, commit du 2026-09-17)
- Q : VSS relie-t-il les signaux aux bus physiques ? R : Non, VSS ne décrit que l'arbre sémantique (68 fichiers .vspec). Le mapping est fait par des « providers » (Kuksa) [OUVERT].

**OTA : Uptane, Sibros, Excelfore, Airbiquity**
- Q : Qu'est-ce qu'Uptane laisse aux implémenteurs ? R : La résolution des dépendances et des conflits (le Director « SHALL take into account » ceux-ci, mais le processus est « out of scope »), la sécurité de la chaîne de build et UDS/OBD. La compatibilité matérielle passe par des identifiants matériels dans les Targets metadata. [OUVERT] uptane-standard.md l.275-281, 468, 599, 618
- Q : Comment l'industrie gère-t-elle les variantes ? R : Par une base de compatibilité HW/SW par variante, dans le PLM ou un outil de gestion de flotte, interrogée avant de cibler un VIN [EXTRAIT] https://eureka.patsnap.com/blog/tech-solutions/reduce-version-dependency-errors-ota-validation/
- Q : Viabilité des acteurs indépendants ? R : Airbiquity a été vendu par actifs à Karma en février 2024 [EXTRAIT].

**UNECE R155 / ISO/SAE 21434 / R156 / Chine**
- Q : Qu'est-ce que le RXSWIN ? R : C'est l'identifiant de la configuration logicielle pertinente pour l'homologation. L'OEM en assure l'intégrité et la numérotation, et doit prouver pour chaque mise à jour si l'état homologué change [EXTRAIT] itemis, certx.com.
- Q : En quoi la Chine diffère-t-elle ? R :
  - GB 44495-2024 (cybersécurité) : nouveaux types au 2026-01, tous les types au 2028-01, sans certificat CSMS/SUMS [EXTRAIT] https://vxlabs.ai/gb-44495/
  - GB 44496-2024 (mises à jour) : ajoute l'évaluation d'impact, la gestion d'urgence, la confirmation de l'utilisateur et l'archivage des règles de compilation de version [EXTRAIT] atic-ts.
- Q : Les outils TARA sont-ils automatisés ? R : Oui, et le marché est saturé (ThreatZ, PlaxidityX, Panasonic, Cybellum, VicOne) [EXTRAIT].

### C5. Robotique et drones (ROS 2, observabilité, ISO 10218, Part 108, SORA 2.5, Remote ID, Blue UAS)

**Q1. Pourquoi des topics ROS 2 « ne se voient pas » sans erreur ?**
Le profil `rmw_qos_profile_sensor_data` est en `BEST_EFFORT`, `KEEP_LAST 5`, `VOLATILE` (https://raw.githubusercontent.com/ros2/rmw/rolling/rmw/include/rmw/qos_profiles.h [OUVERT]). Un abonné en `RELIABLE` est incompatible avec un éditeur en `BEST_EFFORT` : la connexion ne se fait pas, et seul un événement ou avertissement d'incompatibilité QoS le signale. Il n'y a pas de vérification statique du graphe au lancement.

**Q2. Pourquoi une bag ROS 2 découpée ou prise en snapshot est-elle parfois « inutilisable » ?**
Les messages `transient_local` (/map, /tf_static) n'apparaissent pas dans le nouveau fichier. rosbag2 a ajouté `--repeat-transient-local` pour les réinjecter au découpage ou au snapshot. Un override QoS l'emporte sur ce mécanisme.
La compression rosbag2 (`--compression-mode file`) rend les fichiers non indexables, alors que la compression interne MCAP reste indexable.
Le mode snapshot garde un buffer en mémoire (`--max-cache-duration`) et écrit sur appel du service `~/snapshot`.
Source : https://raw.githubusercontent.com/ros2/rosbag2/rolling/README.md [OUVERT].

**Q3. Pourquoi DDS casse à l'échelle ou sur Wi-Fi ?**
DDS maintient un graphe complet de participants : trafic de découverte en n² et tempêtes de paquets à l'arrivée de nouveaux participants, aggravées sur Wi-Fi. Zenoh passe par un routeur TCP. rmw_zenoh vise le Tier 1 pour Kilted (https://zenoh.io/blog/2021-03-23-discovery/, https://docs.ros.org/en/rolling/Installation/RMW-Implementations/Non-DDS-Implementations/Working-with-Zenoh.html, https://github.com/ros2/rmw_zenoh/issues/265 [EXTRAIT]).

**Q4. Qui couvre déjà les données et l'observabilité robotique ?**
- Foxglove : MCAP, Série B de 40 M$.
- Roboto AI : fondateurs ex-Amazon Robotics, recherche IA dans les logs.
- Rerun, Formant.
Sources : https://foxglove.dev/product/mcap, https://www.roboto.ai/, https://segments.ai/blog/software-tools-for-robotics-landscape/ [EXTRAIT].
Il n'a pas été vérifié ici si Formant a été racheté.

**Q5. Qu'est-ce que l'ISO 10218-2:2025 change pour l'intégrateur ?**
- Les exigences de sécurité fonctionnelle deviennent explicites.
- Nouvelles classes de robots.
- **Évaluation des menaces cyber obligatoire**. Si ces menaces créent un risque de sécurité, il faut empêcher l'accès non autorisé au matériel, au logiciel, à la configuration et aux programmes.
Sources : https://www.automate.org/robotics/blogs/updated-iso-10218-faq, https://en.doc.safetics.io/insight-10218-2/, https://www.controldesign.com/industry-news/news/55268769/iso-10218-update-makes-functional-safety-requirements-more-explicit [EXTRAIT].
Dans l'UE, le Règlement Machines 2023/1230 s'applique au 20 janv. 2027, avec une protection contre la corruption et le prEN 50742 en préparation (https://digital.nemko.com/regulations/eu-machinery-regulation [EXTRAIT]).
Non approfondi faute de budget : ISO 13482, ISO 3691-4, UL 3100 (aucune recherche dédiée).

**Q6. Simulation et sim-to-real (Gazebo, Isaac Sim, MuJoCo)** : aucune recherche dédiée faute de budget. Aucune affirmation chiffrée.

**Q7. Quel est l'état de Part 108 ?**
- NPRM publié le 7 août 2025 (90 FR 38212, dossier FAA-2025-1908).
- Environ 3 100 commentaires ; réouverture partielle le 28 janv. 2026 sur la priorité de passage et la conspicuité électronique.
- Texte envoyé à l'OIRA le 10 juil. 2026 ; non publié au 18 sept. 2026.
- Structure : permis ou certificat, SMS pour les certificats, Operations Supervisor et Flight Coordinator, évaluations de sécurité TSA.
Sources : https://www.theflightbrief.com/articles/faa-part-108-bvlos-update-september-2026, https://pilotinstitute.com/part-108-explained/, https://www.skydio.com/blog/drones-faa-bvlos-waivers-new-rules [EXTRAIT].

**Q8. Qu'apporte SORA 2.5 dans l'UE ?**
Décision ED 2025/018/R du 29 sept. 2025. Moins de preuves demandées en SAIL II. Outil eSORA de l'EASA avec une fonction transfrontalière (https://aero.bureauveritas.com/newsroom/easa-adopts-sora-25-new-simplifications-specific-category-drones [EXTRAIT]).

**Q9. Comment PX4 et ArduPilot gèrent-ils le Remote ID ?**
ArduRemoteID est un émetteur OpenDroneID MAVLink/DroneCAN sur ESP32-S3/C3. Il vise le moyen de conformité ASTM F3586-22. La responsabilité de conformité et la déclaration de conformité (DoC) à la FAA restent au fabricant (https://raw.githubusercontent.com/ArduPilot/ArduRemoteID/master/README.md [OUVERT]).

**Q10. Qu'est-ce qui change pour Blue UAS et le NDAA ?**
DCMA gère la liste depuis le 1er janv. 2026. La FCC Covered List vise les drones étrangers depuis décembre 2025, avec des exemptions Blue UAS ou ≥ 65 % US jusqu'au 1er janv. 2027 (voir A2).

### C6. Simulation et jumeaux numériques (FMI/FMU/SSP, Modelica, crédibilité, simulateurs, substituts IA)

**FMI 2.0/3.0, FMU, SSP (dépôts modelica)**
- Q : Qui garantit le déterminisme d'une co-simulation ? R : Personne dans le standard. L'algorithme maître et la détection d'événements appartiennent à l'importeur, et le retour arrière d'état est optionnel (`canGetAndSetFMUState`). [OUVERT] fmi-standard docs/4_1, 1___overview, 2_4
- Q : Une FMU est-elle portable ? R : Seulement si elle contient des sources ou des binaires pour la plateforme cible. Les dépendances externes doivent être documentées dans `externalDependencies`. [OUVERT] docs/2_5
- Q : Existe-t-il une conformité active ? R : Non. Le Cross-Check est archivé depuis le 2022-02-10, repose sur l'auto-déclaration et ne couvre pas FMI 3.0. [OUVERT] fmi-cross-check
- Q : Que fixe SSP 2.0 ? R : FMI 3.0 (horloges, tableaux, paramètres structurels), l'intégration de `MetaData` et des signatures issues de SSP Traceability, et les modèles Modelica comme composants. Il ne fixe pas les réglages d'exécution. [OUVERT] ssp-standard

**Modelica et outils (Simulink, Dymola, OpenModelica, Twin Builder)**
- Non approfondi individuellement (budget). Le point commun avec FMI : le déploiement de modèles hors de l'outil auteur passe par l'export FMU binaire, d'où le problème de portabilité ci-dessus.

**Crédibilité (SET Level, SSP Traceability, NASA-STD-7009)**
- Q : Existe-t-il une implémentation outillée du Credible Simulation Process ? R : Seulement un kit de recherche (ITEA UPSIM), avec des niveaux de crédibilité 0 à 3 et un « Capability Assessment » (futur « M&S SPICE »), 1 commit en 2024-12 [OUVERT].
- NASA-STD-7009B date de 2024 [EXTRAIT]. ASME V&V40 est reconnu par la FDA, et Ansys Minerva l'outille [EXTRAIT].
- DNV publie la recommandation RP-0513 et propose le Simulation Trust Center [EXTRAIT].

**Simulateurs (CARLA, CarMaker, Applied, Foretellix, OpenSCENARIO)**
- CARLA 0.10.0 passe à UE 5.5 (2024-12-19), avec coexistence des branches UE4.26 et UE5 « for the foreseeable future » [EXTRAIT] https://carla.org/2024/12/19/release-0.10.0/
- OpenSCENARIO : écart entre XML et DSL 2.x. ScenarioRunner ne supporte que des brouillons DSL dépréciés [EXTRAIT] arXiv 2604.16452.
- Applied Intuition : 830 M$ d'ARR, 18 des 20 premiers OEM [EXTRAIT].
- Foretellix : 135 M$ levés, cadre « simulation trustworthiness » [EXTRAIT].
- Réglementation : R171 virtuel (fin 2024), R152 virtuel (≥30 % d'essais physiques), UNR/GTR ADS (WP.29, juin 2026) [EXTRAIT].
- IPG CarMaker : non approfondi.

**Substituts IA (PhysicsX, Navier…)**
- PhysicsX : 2,4 Md$ (juin 2026) ; Navier AI (YC W24) ; Godela et SuperRadiant (YC) [EXTRAIT].
- Seedtable recense 68 startups de simulation, cumulant 2,8 Md$ levés (janvier 2026) [EXTRAIT] https://www.seedtable.com/best-simulation-startups
- Conclusion : segment saturé, voir B.

### C7. Conception matérielle (PCB, FPGA, chaîne EDA, vérification)

#### Conception PCB (Altium, KiCad, Allegro, Flux, JITX, Quilter, CircuitMind, Diode)

- **Q : Quelles étapes sont déjà automatisées ?**
  - Placement et routage : Quilter (simulation physique, carte SBC DDR4/PCIe qui démarre), DeepPCB, Flux.
  - Du cahier des charges au schéma : CircuitMind.
  - Design as code : JITX, après un pivot depuis la revue de schémas.
  - Bibliothèques : SnapMagic, ProtoFlow.
  - Sources [EXTRAIT] : quilter.ai, protoflow.ai.
- **Q : Reste-t-il un trou ?** Aucun trou non occupé n'a été démontré avec le budget disponible.

#### FPGA et chaîne EDA / vérification (Vivado/Quartus, HLS, UVM, formel, cocotb, ChipAgents, Cognichip)

- **Q : Où en est l'automatisation de la vérification ?**
  - ChipAgents (avec STMicro) : lecture de specs ×15, assertions formelles ×240, environnement UVM ×400.
  - Bronco AI : debug de simulation et fermeture de couverture.
  - Sur un benchmark ouvert, le meilleur outil tombe de 85,2 % sur les patches simples à 24,0 % sur les plus complexes : cette donnée concerne le backport (idée 1), pas la vérification RTL. La recherche est active côté vérification aussi (AgentDV, arXiv 2608.27148).
  - Sources [EXTRAIT] : chipagents.ai, bronco.ai.
- Non approfondi, faute de budget : reproductibilité et CI Vivado/Quartus, HLS, cocotb spécifique FPGA.

### C8. Spatial (logiciel de vol, constellations, collision, NPR 7150.2 / ECSS)

**Q1. Que fournit F´ et où s'arrête-t-il ?**
F´ est un framework par composants en C++ : files de messages, threads, modélisation (FPP) et génération de code, composants réutilisables, outils de test unitaire et d'intégration. Il cible les CubeSats et SmallSats. Rust est optionnel pour certains composants (https://raw.githubusercontent.com/nasa/fprime/devel/README.md [OUVERT]).
Il ne fournit ni opérations de flotte, ni conformité NPR ou ECSS, ni évitement de collision. L'API GitHub étant en limite de débit, les métadonnées F´ et cFS (étoiles, release) ne sont pas vérifiées.

**Q2. Comment fonctionne le flux CDM et pourquoi il y a des fausses alertes ?**
Les CDM du 18e/19e SDS arrivent via Space-Track. Le Pc dépend du réalisme des covariances : une covariance sous-estimée donne une fausse précision ; une covariance surestimée dilue le Pc et peut masquer un risque réel (https://link.springer.com/article/10.1007/s40295-025-00549-9, https://nodis3.gsfc.nasa.gov/OCE_docs/OCE_51.pdf [EXTRAIT]).

**Q3. Où en est TraCSS ?**
Criblage toutes les 4 h avec les éphémérides des opérateurs. CDM diffusés via Space-Track. 70 utilisateurs pilotes, 11 345 satellites, encore en mode pilote en août 2026. Le budget demandé pour l'exercice 2027 est réduit d'environ 80 % par rapport aux 52,5 M$ de 2026 (https://www.satellitetoday.com/government-military/2026/08/26/tracss-traffic-coordination-system-remains-in-pilot-mode-due-to-budget-uncertainty/, https://spacenews.com/noaa-budget-proposal-seeks-to-cancel-tracss/ [EXTRAIT]).
Cela pourrait profiter aux offres commerciales, mais celles-ci sont déjà nombreuses (rejet 5).

**Q4. Que sont devenues les startups de logiciel de vol et d'opérations ?**
Kubos a été racheté par Xplore en 2022 (rejet 6). Sedaro, Slingshot, Antaris, Epsilon3 : pas de recherche dédiée faute de budget.

**Q5. Qu'impliquent NPR 7150.2D et ECSS-E-ST-40C ?**
- NPR 7150.2D impose une matrice de conformité (SWE-125) (https://swehb.nasa.gov/display/7150/SWE-125+-+Requirements+Compliance+Matrix [EXTRAIT]).
- ECSS-E-ST-40C Rev.1 date du 30 avril 2025 (https://ecss.nl/standard/ecss-e-st-40c-rev-1-software-30-april-2025/ [EXTRAIT]).
- Outillage existant : LDRA, MathWorks, Trace.Space, ainsi que l'ASEP compliance copilot de la NASA (rejet 7).

### C9. Défense (souveraineté ITAR/EAR, CMMC, SWFT/cATO, plateformes)

**Q1. Où en est vraiment CMMC ?**
- Phase 1 (auto-évaluations) en vigueur depuis le 10 nov. 2025.
- La Phase 2 (évaluations C3PAO), prévue le 10 nov. 2026, a été suspendue le 13 juil. 2026. Un Reform Task Force de 60 jours rend son rapport vers le 13 sept. ou fin sept.–oct. 2026. La DFARS 252.204-7012 et NIST 800-171 Rev. 2 restent en vigueur.
- Coût estimé par le gouvernement pour une petite entité : environ 101 752 $.
Sources : https://federalnewsnetwork.com/cybersecurity/2026/07/pentagon-suspends-cmmc-phase-two-requirements-launches-review-of-program/, https://www.lw.com/en/insights/what-defense-contractors-should-know-about-dod-suspension-of-cmmc-phase-2, https://godlan.com/cmmc-assessment-cost/ [EXTRAIT].

**Q2. Pourquoi l'ITAR bloque le cloud et la forge de code ?**
Voir A1 : la dérogation de chiffrement de bout en bout 120.54(a)(5) exclut tout intermédiaire en clair ; Git ne contrôle la lecture qu'au niveau du dépôt [OUVERT] ; GitHub.com ne restreint pas par pays [OUVERT] ; l'export réputé couvre les salariés étrangers.

**Q3. Que change SWFT par rapport au RMF/ATO ?**
Pilote lancé en mai 2025, devenu exigence évolutive en janv. 2026. Il demande un SBOM éditeur et un SBOM d'un évaluateur tiers, avec 12 facteurs de risque dont la santé financière (https://defensescoop.com/2025/06/09/katie-arrington-swft-software-fast-track/, https://anchore.com/blog/dod-swft-initiative-and-promise-of-cato-fulfilled/ [EXTRAIT]). L'industrie signale un manque de processus d'attestation standardisé (https://federalnewsnetwork.com/defense-main/2025/12/industry-flags-dods-lack-of-standardized-software-attestation-processes/ [EXTRAIT]).

**Q4. Plateformes logicielles défense : que proposent-elles ?**
- Anduril Lattice SDK : applications tierces sur Lattice Mesh sans autorisation d'Anduril ; 10 partenaires initiaux dont Apex et Saronic (https://breakingdefense.com/2024/12/decentralizing-battle-data-cdao-anduril-open-tactical-mesh-to-third-party-developers/ [EXTRAIT], https://github.com/anduril/lattice-sdk-cpp [EXTRAIT]).
- Defense Unicorns UDS : livraison en environnement déconnecté, UDS Registry, UDS Fleet.
- Second Front Game Warden : ATO héritée IL5 / FedRAMP High.
- Istari, Rune, Govini : pas de recherche dédiée faute de budget.

**Q5. Qu'est-ce qui existe pour contrôler les données techniques ITAR ?**
NextLabs (SAP, PLM), archTIS (M365), Kiteworks (fichiers, MCP), Perforce (protections par chemin), PreVeil et Virtru (chiffrement de bout en bout). Aucun ne couvre le graphe Git, la CI et les agents IA (A1).

---

## Limites et zones peu couvertes

Signalées honnêtement ; aucune idée n'a été rejetée sur ces zones faute de recherche.

- **Embarqué et matériel :**
  - bancs HIL dSPACE/NI, couverture des périphériques Renode/QEMU ;
  - SafeRTOS, ThreadX (Eclipse) ;
  - durée des LTS Yocto ;
  - CI Vivado/Quartus, HLS, cocotb côté FPGA.
- **Robotique :**
  - ISO 13482, ISO 3691-4, UL 3100 ;
  - simulation et sim-to-real (Isaac Sim, MuJoCo, Gazebo), Nav2.
- **Spatial :** Sedaro, Slingshot, Epsilon3, Antaris.
- **Défense :** Istari, Rune, Govini.
- **Simulation :** Modelica, Dymola, Twin Builder et IPG CarMaker, abordés seulement à travers FMI.
- **Balayage concurrence :** Product Hunt, G2 et Show HN n'ont été interrogés que par des requêtes combinées, pas un par un pour chaque idée retenue. C'est à compléter avant toute décision, notamment G2 pour A4, et Product Hunt/G2 pour A1 et A2.
- **Marchés :** les quatre idées qui passent reposent sur au moins un facteur marqué « HYPOTHÈSE non sourcée ». Ce facteur est indiqué dans chaque fiche :
  - A1 : nombre de fabricants livrant de l'embarqué Linux dans l'UE ;
  - A2 : nombre d'équipes C certifiées et prix ;
  - A3 : nombre de Tier-1 et prix ;
  - A4 : part des entités DDTC qui écrivent du code contrôlé, taille des équipes et prix.

STATUT: TERMINÉ
