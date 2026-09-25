# 10 — Calcul neuromorphique : idées de startups issues de 8 sujets de thèse

Date de rédaction : 2026-09-25. Domaine : calcul neuromorphique (sujets de thèse A à H).

**Méthode suivie.**
1. Chaque sous-couche est décortiquée à partir des solutions existantes : code, spécifications, documentation, articles.
2. Les points douloureux sont formulés comme des mécanismes (« X fait Y, donc Z casse quand… »).
3. Des idées de produit en sont tirées.
4. Chaque idée est validée sur les 8 critères YC.

**Budget de recherche.** Environ 166 recherches web au total. Quatre sous-agents en ont fait 153 (40 + 39 + 38 + 36) et 13 vérifications croisées ont été faites à la fin. S'y ajoutent la lecture directe de code sur GitHub (dépôts NIR, snnTorch, sinabs, rockpool, neurobench, aihwkit, metavision_driver, event_camera_codecs, kalibr, e2calib, v2e, expelliarmus, faery, hdf5_ecf, openeb) et de l'API GitHub.

**Conventions.**
- **[OUVERT]** : page ou code réellement lu.
- **[EXTRAIT]** : information tirée d'un extrait de résultat de recherche.
- Sauf mention contraire, la date indiquée est la date d'accès, le 2026-09-25.
- **HYPOTHÈSE non sourcée** : chiffre non appuyé par une source.
- **MARCHÉ À VALIDER** : taille de marché non démontrable avec des chiffres sourcés.
- **À TRANCHER** : point indécidable après recherche ; ce qui manque est précisé.
- Beaucoup de domaines étaient bloqués en lecture directe (arxiv.org, nature.com, sec.gov, pmc, docs.prophesee.ai, jpeg.org, keysight.com). D'où la part importante de sources [EXTRAIT].

**Constat transversal.** Le neuromorphique est bien un domaine de technologies en quête d'application, et c'est documenté :
- BrainChip : 1,22 M$ de revenus au S1 2026 pour 12,0 M$ de perte nette.
- Prophesee : redressement judiciaire en 2024, avec 4,09 M€ de chiffre d'affaires 2024.
- Rain AI : de fait arrêtée.
- GrAI Matter Labs : absorbée par Snap.
- Lava (Intel) : archivé.

Les idées qui passent ne vendent donc **pas** de puce ni de modèle SNN. Elles vendent un **outil qui supprime une friction d'intégration mesurable**, là où le mécanisme technique la rend inévitable. Presque toutes ont un marché étroit aujourd'hui ; elles sont donc marquées « MARCHÉ À VALIDER » plutôt que rejetées, comme le demandait la consigne.

---

## Sommaire des idées

| # | Idée | Sujet(s) de thèse | Statut |
|---|---|---|---|
| 1 | **EventOps** — couche de données neutre pour caméras à événements (décodage robuste, réparation du temps, index, versionnage, migration hors des SDK constructeurs) | F, H, G | Passe — MARCHÉ À VALIDER |
| 2 | **EventSync** — synchronisation au temps capteur et calibration caméra à événements + LiDAR + IMU pour intégrateurs robotiques | G | Passe — MARCHÉ À VALIDER |
| 3 | **NeuroFit** — vérificateur d'adéquation multi-puces et linter sémantique NIR/conformité, avant entraînement | C, D, F | Passe — MARCHÉ À VALIDER |
| 4 | **MemCal** — pipeline mesure → base de données → modèle compact → préréglage de simulateur, pour mémoires émergentes (RRAM/FeFET/MRAM) | A | Passe — MARCHÉ À VALIDER |
| 5 | **Gatekeeper ECG** — micrologiciel de décision de transmission et de qualité du signal pour patchs ECG/MCT, avec dossier PCCP de personnalisation bornée | C | Passe — À TRANCHER (concurrence B-Secur) + MARCHÉ À VALIDER |
| 6 | **SNN-FI** — injection de fautes sur la représentation matérielle réelle des SNN déployés (spatial/défense) | D | Passe de justesse — À TRANCHER + MARCHÉ À VALIDER |

Les rejets, avec leur preuve, sont regroupés en partie B. La couverture des sous-couches (questions techniques et réponses) est en partie C.

---

# A. Fiches des idées qui passent

## Idée 1 — EventOps : couche de données neutre pour caméras à événements

**Sujet de thèse d'origine.** F (vision neuromorphique par SNN), H (compression et données de capteurs à événements, passerelle edge/fog), G (robots à perception événementielle).

**Question technique d'origine.**
- Que se passe-t-il quand on veut lire le milieu d'un flux EVT 3.0, fusionner deux caméras, ou relire un HDF5 Metavision sur une autre machine ?
- Que devient le code d'une équipe quand le fournisseur change de SDK ?

### Problématique (mécanisme)

**1. EVT 3.0 est un format à état, avec un horodatage de 24 bits.**
- EVT 3.0 code en mots de 16 bits relatifs à un état courant (TIME_HIGH/TIME_LOW, Y, VECT_BASE_X plus masques de validité). Exemple : 32 événements voisins tiennent en 8 octets au lieu de 64. [EXTRAIT docs.prophesee.ai/stable/data/encoding_formats/evt3.html ; arXiv 2511.15556]
- L'horodatage fait 24 bits et reboucle toutes les 2^24 µs = 16,77 s.
- Le README d'`event_camera_codecs` [OUVERT, raw.githubusercontent.com/ros-event-camera/event_camera_codecs/master/README.md] l'écrit explicitement :
  - si l'on commence à décoder au milieu du flux (« an hour into it! »), le premier horodatage obtenu est toujours entre 0 et 16,77 s ;
  - le capteur a « some dubious bit errors in the time stamps » ;
  - deux caméras synchronisées matériellement peuvent se retrouver décalées de 16,7 s ;
  - le temps de départ calculé « usually disagrees » avec celui du Metavision SDK.
- Pour EVT3, le champ `time_base` du message ROS « is not used and its content is undefined ». [OUVERT, event_camera_msgs README]
- **Conséquence.**
  - Pas d'accès aléatoire sans reconstruire l'état.
  - La fusion multi-caméras casse quand un capteur reboucle avant l'autre.
  - Les erreurs de bits trompent la détection du rebouclage.
  - Chaque équipe réécrit son décodeur.
- EVT+ (arXiv 2511.15556, 2025-11-19) liste les mêmes limites : « only 16 payload types… finite timestamp resolution, and no header space for critical metadata ». [EXTRAIT]

**2. Le HDF5 de Prophesee dépend d'un filtre propriétaire.**
- Le HDF5 Metavision utilise le codec ECF, déclaré comme filtre HDF5 sous un identifiant auto-attribué 0x8ECF. [OUVERT, prophesee-ai/hdf5_ecf README]
- Un fichier n'est donc lisible dans h5py ou MATLAB qu'avec un plugin compilé et un `HDF5_PLUGIN_PATH` correct, qui diffère entre Ubuntu 22.04 et 24.04. [OUVERT, openeb README]
- Tickets associés : openeb #96 (2023-10-04) et #160, échec sous WSL2 (2025-06-25). [EXTRAIT, API GitHub]

**3. L'ERC jette des données sans trace exploitable.**
- Le contrôleur de débit (ERC) supprime des événements sous forte activité et « may compromise signal quality, potentially introducing horizontal artifacts ».
- Le seul indicateur est un taux de suppression moyenné sur 1 s. [EXTRAIT docs.prophesee.ai release notes ; arXiv 2501.18788]
- Un jeu de données enregistré en saturation contient donc des trous invisibles pour l'aval.

**4. Le logiciel constructeur est verrouillé et instable.**
- Le Metavision SDK n'est plus gratuit depuis la version 5.0 : la 4.6.2 (2024-07-02) est la dernière gratuite, et le SDK Pro est lié à l'achat d'un kit d'évaluation (EVK) depuis le 2024-10-07. [EXTRAIT docs.prophesee.ai/stable/faq.html ; centuryarks.com]
- En juillet 2026, Prophesee a annoncé la **fin de vie d'OpenEB et du Metavision SDK autonome**, remplacés par la plateforme « Hearth ».
  - Hearth assure la compatibilité « across successive generations of Prophesee sensors », donc uniquement les capteurs Prophesee.
  - [EXTRAIT image-sensors-world.blogspot.com/2026/07/prophesee-raises-20-million.html ; prophesee.ai/2026/06/15/prophesee-launches-mantara-event-based-drone-detection/, recherche du 2026-09-25]
- Le fournisseur est fragile :
  - redressement judiciaire ouvert le 2024-10-15 ;
  - effectifs passés de 120 à 55 ;
  - chiffre d'affaires 2024 de 4 085 561 € ;
  - plan arrêté le 2026-05-05 ;
  - levée de 20 M€ en juillet 2026 (Critical Path Ventures).
  - [EXTRAIT vipress.net ; pappers.fr/entreprise/prophesee-800681892 ; usinenouvelle.com ; f4news.com 2026-07-17]
- Côté iniVation, AEDAT4 repose sur des FlatBuffers compressés en LZ4/ZSTD, avec ses propres outils (dv-processing). iniVation a été rachetée par SynSense le 2024-02-01. [EXTRAIT docs.inivation.com ; startupticker.ch]
- **Conséquence** : un intégrateur qui a des caméras Sony/Prophesee et iniVation, ou qui veut rester libre de changer de fournisseur, porte seul le coût de la neutralité.

### Solution

Un SDK et un service, open-core :
1. **Décodeurs robustes** EVT 2.0/2.1/3.0, AEDAT4, HDF5-ECF, rosbag `event_camera_msgs`, puis JPEG XE (DIS atteint en avril 2026 [EXTRAIT jpeg.org/items/20260608_press.html]). Ils incluent :
   - le suivi du rebouclage 24 bits ;
   - la détection et correction des erreurs de bits d'horodatage ;
   - l'alignement multi-caméras.
2. **Index de points de reprise d'état**, pour un accès aléatoire en O(1) dans des enregistrements de plusieurs heures.
3. **Annotation automatique des fenêtres ERC-drop et de saturation**, exportée comme masque de qualité avec le jeu de données.
4. **Stockage et versionnage** des jeux de données événementiels, avec compression sans perte et conversion vers un format pivot neutre (JPEG XE quand il sera figé).
5. **Visualisation** (plugin Foxglove/Rerun) et migration assistée « Metavision/OpenEB → neutre » pour les équipes touchées par la fin de vie d'OpenEB.

### Comment la solution répond à la problématique

| Point douloureux | Réponse |
|---|---|
| Décodage à état, rebouclage, erreurs de bits | Traités une fois pour toutes dans le décodeur et l'index |
| Filtre HDF5 propriétaire | Conversion à l'ingestion |
| Données perdues par l'ERC | Rendues visibles (masque) |
| Dépendance à un SDK en fin de vie | Retirée par le format pivot |

### Concurrents

- **Prophesee Hearth / Metavision SDK Pro** : propriétaire, limité aux capteurs Prophesee, lié au kit.
- **iniVation DV / dv-processing** : lié à iniVation/SynSense.
- **Outils open source mono-fonction** [OUVERT] : faery (LGPL), expelliarmus, Tonic, event_camera_codecs, ADΔER, evt3-core (Rust).
- **Foxglove et Rerun** : plateformes généralistes de données robotiques. Aucun support natif des événements DVS trouvé dans leur documentation ou changelog.
  - Recherches « Foxglove event camera support EventArray DVS » et « Rerun.io event camera DVS » : seuls des « Events » au sens d'annotations temporelles chez Foxglove. [EXTRAIT docs.foxglove.dev/docs/data/events ; rerun.io, 2026-09-25]
  - **Risque principal** : ces plateformes peuvent l'ajouter.
- **Angle qu'aucun n'a** : neutralité multi-constructeurs, plus réparation du temps et accès aléatoire, plus masque de qualité ERC, plus migration hors d'un SDK en fin de vie.

### Qui paie

- Équipes R&D d'intégrateurs industriels et défense (cibles déclarées de Prophesee), laboratoires de robotique, équipes de perception utilisant plusieurs capteurs à événements.
- Plus tard, fabricants de capteurs alternatifs (OmniVision/CelePixel, SynSense/iniVation), qui ont intérêt à un format neutre.

### Revenu potentiel (bottom-up) — MARCHÉ À VALIDER

**Facteurs sourcés :**
- Communauté active :
  - 177 dépôts dans le topic GitHub « event-camera » ; OpenEB : 305 étoiles et 94 forks ; v2e : 489 étoiles. [OUVERT, API GitHub]
  - La liste de ressources uzh-rpg recense environ 2 218 liens. [OUVERT]
- Prix d'ancrage : Foxglove Pro coûte 20 $/mois pour 3 postes développeur et 1 To. [EXTRAIT foxglove.dev/blog/reduced-self-service-pricing]
  - Le libre-service de ce segment est donc bon marché. La valeur doit venir de contrats entreprise, dont les prix ne sont pas publiés.
- Plafond de cohérence : le leader des capteurs fait 4,09 M€ de chiffre d'affaires 2024. [EXTRAIT pappers]
  - Le parc de caméras installées reste petit.

**Calcul :**
- HYPOTHÈSE non sourcée : 150 à 300 organisations (sur la base des 94 forks d'OpenEB et des 177 dépôts, en supposant qu'une fraction publie).
- HYPOTHÈSE non sourcée : 10 000 à 20 000 $/an par organisation en contrat équipe/entreprise.
- Résultat : **1,5 à 6 M$/an** adressables aujourd'hui.

**Conclusion.** Trop petit comme marché événementiel pur. La trajectoire VC n'existe que si :
- (a) les caméras à événements percent en défense, anti-drones et industrie (signaux : Mantara, caméras IDS uEye EVS, Tobii) ;
- ou (b) EventOps devient la brique « modalité difficile » d'une plateforme de données multimodales pour l'IA physique, sur le modèle de Foxglove et Rerun.

### Preuve des 8 critères

| # | Critère | Verdict et preuve |
|---|---|---|
| 1 | Vrai problème | Oui. Mécanisme démontré dans le code et la doc : rebouclage 24 bits et erreurs de bits [OUVERT event_camera_codecs README] ; filtre 0x8ECF [OUVERT hdf5_ecf] ; tickets openeb #96/#160 [EXTRAIT] ; fin de vie OpenEB [EXTRAIT image-sensors-world 2026-07]. Le problème est subi par tout utilisateur de données enregistrées : ce n'est pas une techno en quête d'usage. |
| 2 | Pas un piège à goudron | Pas de tentative commerciale d'outil de données neutre trouvée, donc pas d'échec documenté. Piège voisin documenté : les startups de capteurs finissent absorbées (Insightness par Sony en 2019 ; CelePixel par Will Semi en 2020 ; iniVation par SynSense en 2024 ; Prophesee en redressement) [EXTRAIT crunchbase, image-sensors-world, s-ge.com]. L'outil logiciel neutre ne dépend pas de la survie d'un fabricant de capteurs ; c'est précisément ce qui le justifie. |
| 3 | Problème aigu | Moyen. Chaque équipe réécrit ses décodeurs et chemins de plugins (tickets récurrents). Le coût est ponctuel mais revient à chaque nouveau capteur ou changement de SDK. La fin de vie d'OpenEB (2026) rend le problème aigu pour la base existante. |
| 4 | Marché | MARCHÉ À VALIDER (calcul ci-dessus). |
| 5 | Concurrence | Nommée ci-dessus ; l'angle neutre, réparation du temps et masque ERC n'est présent chez aucun concurrent trouvé. |
| 6 | Pourquoi maintenant | SDK payant (2024-10-07) ; fin de vie OpenEB/Metavision au profit de Hearth (annonce de juillet 2026) ; JPEG XE au stade DIS (avril 2026) [EXTRAIT jpeg.org/items/20260608_press.html] ; EVT+ (2025-11-19) ; kit GenX320 pour Raspberry Pi 5 (2025-08-26), qui élargit la base [EXTRAIT prophesee.ai/2025/08/26/…]. |
| 7 | Proxy | Foxglove : Série B de 40 M$ menée par Bessemer, 2025-11-12 [EXTRAIT businesswire.com/news/home/20251112126106] ; Rerun : seed de 17 M$, 2025-03-20 [EXTRAIT techcrunch.com/2025/03/20/…]. Même modèle (couche de données et observabilité pour données de capteurs) dans la robotique généraliste. |
| 8 | Scalable | Logiciel et stockage : le coût marginal par client est faible. Le revenu par client croît avec le volume de données stockées (modèle Foxglove à l'usage). |

### Formulations du balayage concurrentiel (≥ 10)

1. « Prophesee EVT 3.0 format specification decoder » (ingénieur)
2. « AEDAT 4.0 file format flatbuffers »
3. GitHub topic:event-camera
4. « Rerun OR Foxglove visualize event camera DVS » (acheteur robotique)
5. « Foxglove event camera support EventArray DVS visualization »
6. « Rerun.io event camera DVS events visualization support »
7. `site:ycombinator.com/companies event camera OR neuromorphic OR "event-based vision"` : seulement Rain Neuromorphics et « Neuromorphic » (robots de labo)
8. `"event camera" OR "event-based" site:ycombinator.com/companies` : OpenVector et Conntour (« événements » au sens d'incidents sur caméras RGB, hors sujet) [EXTRAIT ycombinator.com/companies/openvector]
9. « producthunt event camera neuromorphic vision tool launch » : rien
10. « event-based vision software startup raises seed 2025 2026 » (vendeur/investisseur)
11. « Metavision SDK free license » (acheteur)
12. « Prophesee Hearth platform successor OpenEB Metavision SDK license sensors supported »
13. « Event-based Data Format Standard EVT+ »

---

## Idée 2 — EventSync : temps capteur et calibration caméra à événements + LiDAR + IMU

**Sujet de thèse d'origine.** G (perception neuromorphique pour l'autonomie longue durée : fusion événements + LiDAR + IMU, auto-supervision).

**Question technique d'origine.**
- Quelle horloge porte un message ROS 2 d'une caméra à événements ?
- Comment calibre-t-on une caméra qui ne produit pas d'images avec un LiDAR et une IMU ?

### Problématique (mécanisme)

**1. Le pilote de référence estampille à l'horloge de l'hôte.**
- Le pilote ROS 2 de référence (`metavision_driver`) ne décode pas les paquets, pour des raisons d'efficacité. L'en-tête ROS reçoit donc « l'horloge murale de l'hôte à l'arrivée du *premier* paquet SDK », pas l'horodatage matériel du capteur. [OUVERT raw.githubusercontent.com/ros-event-camera/metavision_driver/master/README.md]
- Sous charge, la gigue augmente. Le même README mesure, à saturation (~48-50 Mev/s) sur un Ryzen 8 cœurs :
  - 22 à 59 % de CPU pour le pilote seul ;
  - 80 à 90 % avec l'enregistrement rosbag.
- La synchronisation et les triggers partagent les mêmes broches, et la synchronisation stéréo exige une séquence de démarrage primaire/secondaire délicate. [OUVERT]
- Tout estimateur de fusion (VIO, LiDAR) hérite donc de la gigue de l'hôte, sauf à décoder soi-même les paquets EVT3 (voir idée 1) ou à câbler un trigger.
- La recherche GitHub « event camera imu time synchronization ros2 » renvoie **0 dépôt**. [OUVERT]

**2. Kalibr ne voit pas les événements.**
- Kalibr, l'outil standard caméra-IMU, ne mentionne pas les caméras à événements. [OUVERT github.com/ethz-asl/kalibr README]
- Le contournement académique, e2calib (UZH), reconstruit des images par réseau de neurones puis passe par Kalibr. Il fige CUDA 10.1, Python 3.7 et Metavision 2.2. [OUVERT github.com/uzh-rpg/e2calib README]
- La calibration LiDAR ↔ événements repose sur des dépôts de 1 à 16 étoiles : L2E (16), Tricalib DFKI (1, avril 2026), celex_livox_calibration (1, inactif depuis 2022). [OUVERT, recherche GitHub]
- eKalibr-Stereo (2025) ne couvre que la stéréo événementielle. [EXTRAIT arXiv 2504.04451]

**3. Les biais se règlent à la main pour un éclairage donné.**
- Cinq biais sont exposés : `bias_diff_on/off`, `fo`, `hpf`, `refr`. [OUVERT metavision_driver README]
- Le scintillement des LED à 50/60 Hz produit des rafales sur tout le capteur. [EXTRAIT arXiv 2205.08090]
- Sous ~10 lux, la bande passante du photorécepteur chute et le bruit de fond augmente. [EXTRAIT arXiv 2405.19718]
- Le seul banc de réglage reproductible, BiasBench (2025), a 3 étoiles. [OUVERT github.com/cogsys-tuebingen/biasbench]
- Des biais réglés au labo ne tiennent donc pas de jour à la nuit, ni sous LED d'entrepôt ou de serre. C'est exactement le terrain de l'autonomie longue durée (agriculture, surveillance, logistique).

### Solution

Un kit logiciel, licencié par robot ou par flotte, pour intégrateurs :
1. **Nœud ROS 2** qui décode les paquets et publie le **temps capteur**, avec estimation continue du décalage et de la dérive capteur ↔ hôte ↔ IMU ↔ LiDAR. Il comprend un décodeur robuste EVT3, partagé avec l'idée 1.
2. **Calibration** intrinsèque et extrinsèque caméra à événements + LiDAR + IMU, **sans cible** et sans pile GPU figée. Elle s'appuie sur le mouvement, comme les approches « targetless ».
3. **Profils de biais, d'ERC et d'anti-scintillement** adaptés à l'éclairage, avec auto-réglage de type BiasBench.
4. **Diagnostic de saturation** et de perte d'événements.

### Comment la solution répond à la problématique

- La gigue d'horloge est remplacée par le temps capteur modélisé.
- La calibration n'exige plus de reconstruction d'images ni de pile figée.
- Les dérives d'éclairage sont gérées par profils plutôt qu'à la main.

### Concurrents

- **Main Street Autonomy — Calibration Anywhere** : calibre sans cible les intrinsèques, extrinsèques et décalages temporels de LiDAR, radar, caméras, IMU et GNSS. Intégré à NVIDIA Isaac Perceptor. [EXTRAIT mainstreetautonomy.com/calibration ; developer.nvidia.com/blog/how-to-calibrate-sensors-with-msa-calibration-anywhere-for-nvidia-isaac-perceptor]
  - Les modalités caméra listées sont « RGB, thermal, stereo, ToF ». **Les caméras à événements ne sont pas listées.** [EXTRAIT, recherche du 2026-09-25]
  - C'est le concurrent le plus dangereux s'il ajoute cette modalité.
- **Prophesee Metavision SDK Pro / Hearth** : calibration 6DOF entre caméras, documentation de synchronisation, mono-constructeur. [EXTRAIT prophesee.ai/metavision-sdk-pro]
- **Académique** : e2calib, L2E, Tricalib, eKalibr-Stereo, BiasBench.
- **YC** : Efference (perception stéréo apprise), adjacent. [EXTRAIT ycombinator.com/companies/efference]
- **Angle qu'aucun n'a** : temps capteur, calibration à trois modalités incluant les événements, et profils d'éclairage, en un produit multi-constructeurs.

### Qui paie

OEM et intégrateurs de robots mobiles, drones et robots agricoles qui évaluent ou déploient une caméra à événements. Ce sont aussi les laboratoires industriels défense (anti-drones : Mantara de Prophesee, 2026).

### Revenu potentiel — MARCHÉ À VALIDER

- **Aucun chiffre sourcé** sur le nombre d'OEM déployant des caméras à événements, ni sur les unités vendues. Les recherches dédiées sont restées sans résultat.
- Signaux d'adoption datés :
  - caméras industrielles IDS uEye EVS à capteur IMX636 ;
  - partenariat Tobii (suivi oculaire) ;
  - Mantara (anti-drones, juin 2026).
  - [EXTRAIT prophesee.ai/2026/01/07/prophesee-recap-2025/ ; prophesee.ai/2026/06/15/…]
- Marché de la vision robotique : 3,28 Md$ en 2025 [EXTRAIT globenewswire, septembre 2025]. C'est un plafond lointain, pas un marché adressable.
- **Calcul :**
  - HYPOTHÈSE non sourcée : 50 intégrateurs × 20 000 $/an = 1 M$/an au départ.
  - HYPOTHÈSE non sourcée : licence par robot en production, ~100 à 500 $ par unité.
- **Condition de viabilité** : l'outil doit devenir un outil de calibration et de synchronisation multi-capteurs **générique**, où l'événementiel est le cas difficile qui différencie. Dans ce cas, il entre en concurrence frontale avec Main Street Autonomy.

### Preuve des 8 critères

| # | Critère | Verdict et preuve |
|---|---|---|
| 1 | Vrai problème | Oui. Horloge de l'hôte dans l'en-tête [OUVERT metavision_driver README] ; Kalibr sans événements [OUVERT] ; 0 dépôt de synchronisation événements/IMU [OUVERT]. |
| 2 | Pas un piège à goudron | Aucun échec de startup de calibration trouvé. Le proxy MSA est vivant (RBR50 2023, intégration NVIDIA 2024). Piège voisin : la valeur du neuromorphique ne va pas au silicium (Rain AI, GrAI Matter Labs). Ici, on vend l'intégration. |
| 3 | Problème aigu | Élevé pour qui déploie : une fusion avec une horloge fausse donne une odométrie fausse. Des biais non adaptés rendent le capteur inutilisable la nuit ou sous LED. Mais la population touchée est petite. |
| 4 | Marché | MARCHÉ À VALIDER. |
| 5 | Concurrence | MSA (pas d'événements listés), Prophesee (mono-constructeur), outils académiques. |
| 6 | Pourquoi maintenant | Vague d'outils académiques 2025-2026 qui montre le besoin reconnu mais non résolu (BiasBench 2025, Tricalib avril 2026, eKalibr-Stereo 2025) ; pilotes ROS 2 récents (dvsense, août 2025) ; recentrage de Prophesee sur la défense et l'industrie (2026) ; fin de vie d'OpenEB. |
| 7 | Proxy | Main Street Autonomy : calibration vendue comme produit logiciel aux entreprises de robotique, RBR50 2023, intégrée à NVIDIA Isaac Perceptor [EXTRAIT therobotreport.com/rbr50-company-2023/main-street-autonomy-simplifies-sensor-calibration/]. Son chiffre d'affaires n'a pas été trouvé. |
| 8 | Scalable | Logiciel, licence par robot. Le coût d'ajout d'un modèle de capteur est ponctuel. |

### Formulations du balayage (≥ 10)

1. Issues Kalibr « event camera calibration support »
2. GitHub « event camera lidar calibration »
3. GitHub « event camera imu time synchronization ros2 »
4. GitHub « event camera bias tuning »
5. GitHub « event camera driver ros2 »
6. « Prophesee Metavision SDK calibration module »
7. « Main Street Autonomy Calibration Anywhere »
8. « Main Street Autonomy calibration event camera support »
9. `site:ycombinator.com/companies sensor calibration robotics lidar camera`
10. `site:ycombinator.com/companies neuromorphic OR spiking OR "event camera"`
11. « producthunt … event camera robotics SDK »
12. « eKalibr-Stereo »

---

## Idée 3 — NeuroFit : vérificateur d'adéquation multi-puces et linter sémantique NIR

**Sujets de thèse d'origine.**
- C : SNN pour biomonitoring sur puce ultra-basse consommation.
- D : SNN embarqué « auto-conscient ».
- F : SNN de vision déployés sur puce.

**Question technique d'origine.**
- Un même graphe NIR donne-t-il le même résultat sur deux backends ?
- Quand découvre-t-on qu'un modèle ne rentre pas dans une puce ?

### Problématique (mécanisme)

**1. La sémantique de NIR n'est pas normative.**
- NIR définit 17 primitives en temps continu (des EDO) et laisse chaque backend discrétiser. La doc le dit : « backends are free to implement as they want, leading to varying outputs across platforms ».
- Les métadonnées de discrétisation ne doivent pas être exploitées : « no backend should rely on this metadata ». [OUVERT neuromorphs/NIR docs/source/primitives.md, commit 2026-09-18]
- Même l'amplitude d'un spike dépend du pas de temps : « the strength of the spike … depends on the discretization timestep dt ». [OUVERT paper/01_lif/debug_spike_representation]
- **Résultat mesuré** dans les fichiers `.npy` du dépôt [OUVERT] : le même graphe RNN Braille donne 95 % sur snnTorch, 85 % sur SpiNNaker2 et 48,6 % sur Lava. Sur le CNN N-MNIST : 98,47 % sur Sinabs contre 95,40 % sur la puce Speck.
- *Réserve* : ces chiffres viennent de fichiers bruts du dépôt, non recoupés avec le tableau publié (nature.com bloqué).

**2. L'export perd silencieusement le mode de reset.**
- `snntorch/export_nir.py` [OUVERT] code en dur `dt = 1e-4`, calcule `tau_mem = dt/(1-beta)` et écrit toujours `v_reset = 0`.
- Or `snn.Leaky` utilise par défaut `reset_mechanism="subtract"` (`_neurons/leaky.py` l.151 [OUVERT]), et l'import force `reset_mechanism="zero"` (`import_nir.py` [OUVERT]).
- Un modèle entraîné avec reset soustractif change donc de dynamique sans avertissement.
- De plus, un backend qui discrétise exactement ne retrouve pas le β d'origine. Calcul : β = 0,9 → τ = 1 ms → exp(−0,1) ≈ 0,905.

**3. Il n'existe aucune conformité.**
- Issue NIR #193 (août 2026), sans réponse : l'auteur de nir-rs demande des « official … fixtures for compatibility testing ».
- Issue #146 : « Are all NIR graphs valid? », sans réponse de fond.
- Issue #142 : l'ajout de `v_reset` a cassé la lecture des anciens fichiers.
- Issue #176 : les scalaires doivent être dupliqués en tableaux, soit « 1024x memory overhead ».
- [OUVERT github.com/neuromorphs/NIR/issues]
- La matrice `supported_primitives.md` [OUVERT] montre que **Delay, Scale et I ne sont supportés par aucun backend**.

**4. Les contraintes matérielles n'apparaissent qu'à la compilation.**
- Speck a des mémoires hétérogènes par cœur : poids 16K/16K/16K/32K/32K/64K/64K/16K/16K. [OUVERT sinabs/backend/dynapcnn/chips/dynapcnn.py]
- Le seul diagnostic est : « Generated config is not valid… Probably one or more layers are too large ». [OUVERT dynapcnn_network.py l.538]
- Xylo Audio 2 : fan-in maximal de 63 et 1 000 neurones. [OUVERT fiche Open Neuromorphic]
- Akida ne supporte pas le LIF. Sa quantification peut coûter « potentially several dozen percent », et le conseil officieux est de convertir **avant** d'entraîner. [OUVERT fiche Open Neuromorphic akida-brainchip]

**5. L'énergie n'est pas comparable.**
- `grep energy|joule` dans le harnais NeuroBench ne renvoie aucune ligne. [OUVERT, code cloné]
- L'article NeuroBench reconnaît qu'une méthodologie de mesure unique est « currently infeasible ». [EXTRAIT arXiv 2304.04640]

**6. Loihi est orphelin.**
- Lava et lava-dl sont archivés : « THIS PROJECT IS ARCHIVED. Intel will not provide or guarantee development of or support for this project ». Dernier push le 2026-05-13. [OUVERT github.com/lava-nc/lava]
- Le SDK successeur n'est pas publié à la date du rapport. [EXTRAIT quantaracore.in/blog/lava-archived-alternative ; intel.com, recherche du 2026-09-25]

### Solution

Un outil de développement, en CLI, en bibliothèque PyTorch et en CI :
1. **Vérification d'adéquation dès la conception**, pour chaque cible (Speck, Xylo, Pulsar, Akida, SpiNNaker2, puis le futur SDK Loihi) : mémoire par cœur, fan-in, bits, couches autorisées, primitives supportées. Elle propose un placement couche→cœur et localise précisément la couche fautive.
2. **Linter sémantique NIR** : reset perdu, `dt` implicite, Delay/Scale/I non portables, surcoût de broadcast.
3. **Diff d'équivalence** multi-backends en CI : traces de membrane, taux de spikes, précision. Les fixtures servent de suite de conformité, ce que demande l'issue #193.
4. **Estimation latence et énergie** par cible, calibrée sur des mesures publiées ou fournies par les vendeurs partenaires.
5. **Modèle économique** : licence OEM aux vendeurs de puces (sur le modèle ModelCat → NXP eIQ), puis abonnement équipe aux OEM qui évaluent plusieurs puces.

### Comment la solution répond à la problématique

- Les pertes sémantiques (points 1-2) sont détectées avant déploiement.
- La conformité (point 3) devient un artefact testable.
- Les contraintes (point 4) remontent au moment de l'entraînement, là où BrainChip conseille lui-même de les traiter.
- La comparaison d'énergie (point 5) est enfin possible avant l'achat des kits.

### Concurrents

- **SDK des vendeurs**, gratuits et mono-cible : MetaTF (BrainChip), Talamo (Innatera, lancé avec Pulsar le 2025-05-21 [EXTRAIT]), Rockpool/Sinabs + samna (SynSense), py-spinnaker2.
- **Edge Impulse** (Qualcomm depuis mars 2025) : bloc Akida uniquement. [EXTRAIT docs.edgeimpulse.com/hardware/boards/brainchip-akd1000]
- **QuantaraCore NeuroCUDA** : compilateur PyTorch→SNN sous licence MIT avec export NIR. Positionné sur « Lava archivé ». Son backend Loihi 2 est « a simulator… not a substitute ». [EXTRAIT quantaracore.in/neurocuda.html]
- **SANA-FE** (UT Austin) et modèle d'exécution Loihi 2 (arXiv 2601.10035) : estimateurs académiques mono-plateforme.
- **Angle qu'aucun n'a** : vérification d'adéquation multi-puces **avant** entraînement, linter de pertes sémantiques NIR et suite de conformité. NeuroCUDA compile, mais aucun élément trouvé ne montre qu'il vérifie les contraintes matérielles par cible ni qu'il détecte des pertes sémantiques. **Point à surveiller.**

### Qui paie

- En priorité, les **vendeurs de puces** qui veulent réduire le coût d'adoption de leurs kits : BrainChip, Innatera, SynSense, SpiNNcloud, Intel pour son futur SDK.
- Ensuite, les équipes OEM qui évaluent plusieurs puces : biomédical (thèse C), spatial et défense (Frontgrade Gaisler et Parsons sont clients d'Akida).

### Revenu potentiel — MARCHÉ À VALIDER (probablement petit)

**Facteurs sourcés :**
- Tracxn : 83 entreprises d'IA neuromorphique, dont 30 startups de puces. [EXTRAIT Tracxn 2026]
- Plus de 200 membres INRC (avril 2024). [EXTRAIT intc.com]
- Revenu BrainChip au S1 2026 : 1,22 M$. [EXTRAIT Motley Fool AU 2026-08-26]
- Prix de kits : AKD1000 PCIe 499 $, Akida Edge AI Box 799 $. [EXTRAIT CNX Software 2022-01 ; brainchip.com]

**Calcul :**
- Licence vendeur : 30 vendeurs de puces (Tracxn) × HYPOTHÈSE non sourcée 50 000 à 100 000 $/an × HYPOTHÈSE non sourcée 20 % d'adoption = **0,3 à 0,6 M$/an**.
- Équipes OEM : HYPOTHÈSE non sourcée 50 à 150 équipes × 20 000 à 50 000 $/an = **1 à 7,5 M$/an**.

**Conclusion.** Pas une taille VC aujourd'hui. Pari sur la croissance du nombre de puces commerciales (Pulsar à moins de 5 $ en volume [EXTRAIT], AKD1500 commandé par Parsons [EXTRAIT], nouveau Loihi annoncé).

### Preuve des 8 critères

| # | Critère | Verdict et preuve |
|---|---|---|
| 1 | Vrai problème | Oui, démontré dans le code : export snnTorch [OUVERT], issues NIR #142/#146/#176/#190/#193 [OUVERT], écarts de 48,6 à 95 % [OUVERT .npy], diagnostic Speck générique [OUVERT]. |
| 2 | Pas un piège à goudron | **Risque réel documenté** : OctoML (compilateur multi-cibles TVM) vendu à Nvidia en septembre 2024 pour 165 à 250 M$ contre ~900 M$ de valorisation en 2021, services fermés le 2024-10-31 [EXTRAIT GeekWire] ; Deeplite absorbé par ST (avril 2025) [EXTRAIT]. Le multi-cibles tend à finir comme fonctionnalité d'un fondeur. Mais ce sont des sorties par rachat, pas des faillites, et Edge Impulse a aussi été racheté (Qualcomm) : le schéma est « sortie stratégique précoce », pas « échec structurel ». Verdict : pas un piège à goudron au sens strict, mais un **plafond de sortie** probable. |
| 3 | Problème aigu | Élevé pour qui déploie : une perte de 46 points de précision, ou une dynamique changée en silence, rend le produit faux. La fréquence est limitée au nombre de projets de déploiement. |
| 4 | Marché | MARCHÉ À VALIDER. |
| 5 | Concurrence | Nommée ; angle vérification d'adéquation + linter + conformité non trouvé ailleurs. |
| 6 | Pourquoi maintenant | Pulsar lancé (2025-05-21) ; Akida Cloud (2025-08-05) ; AKD1500 en production avec commande Parsons (S1 2026) ; Lava archivé (2026-05-13) ; nir-rs tiers (août 2026). Pour la première fois, plusieurs puces commerciales coexistent sans chaîne commune. |
| 7 | Proxy | Edge Impulse (54 M$ levés, racheté par Qualcomm en mars 2025) [EXTRAIT siliconangle.com/2025/03/10/…] ; Eta Compute → ModelCat, outil licencié à NXP pour eIQ Model Creator [EXTRAIT EE Times ; PRNewswire]. |
| 8 | Scalable | Logiciel. Chaque nouvelle cible a un coût fixe (modèle de contraintes), puis sert tous les clients. |

### Formulations du balayage (≥ 10)

1. « neuromorphic compiler platform startup deploy SNN multiple chips Loihi Akida Xylo 2026 »
2. « QuantaraCore NeuroCUDA pricing ANN to SNN conversion »
3. « SNN energy estimation tool per neuromorphic platform energy model »
4. « bit-accurate OR hardware-in-the-loop SNN simulator sim-to-hardware gap tool »
5. « Edge Impulse BrainChip Akida deployment support »
6. « NIR validator conformance test suite cross-platform equivalence spiking »
7. `site:ycombinator.com/companies neuromorphic`
8. `site:ycombinator.com/companies spiking neural network OR event-based edge AI compiler`
9. « producthunt neuromorphic spiking neural network tool launch »
10. « neuromorphic software startup funding 2026 SNN deployment toolchain »
11. « Innatera Talamo PyTorch customers »
12. « Intel new neuromorphic SDK Loihi after Lava archived announcement 2026 »

---

## Idée 4 — MemCal : de la mesure au modèle compact et au préréglage de simulateur, pour mémoires émergentes

**Sujet de thèse d'origine.** A : fiabilité des transistors synaptiques et de la RRAM. Caractérisation, rétention, endurance, uniformité, lien entre physique et précision de l'IA.

**Question technique d'origine.**
- Comment passe-t-on de mesures d'impulsions sur une matrice RRAM à un modèle compact calibré, puis à un modèle de bruit exploitable par un simulateur de précision ?
- Qui le fait sans script maison ?

### Problématique (mécanisme)

**1. Les instruments imposent un découpage et des pilotes fragiles.**
- Le Keysight B1530A WGFMU est limité à **2 048 vecteurs** de forme d'onde et à **512 formes d'onde par séquence**. [EXTRAIT Keysight 5990-4567]
- Les longues séquences d'endurance et de relaxation doivent donc être découpées et rechargées.
- Les pilotes sont des DLL 32 bits sous Windows. Exemple réel : PythonMeasurementApp pilote `agb1500_32.dll` et `WGFMU.dll`, exige un Python 32 bits, s'exécute « en synchrone uniquement » et gère peu les erreurs. [OUVERT github.com/digiefel/PythonMeasurementApp]

**2. La relaxation RRAM borne la précision au-delà du write-verify.**
- L'essentiel de la dérive survient dans la première seconde après programmation. La perte due à la relaxation dépasse celle due à la fenêtre d'acceptation du write-verify. [EXTRAIT IEEE 10354009 ; arXiv 2301.08516]
- Il faut donc une caractérisation **résolue dans le temps** (µs → heures), en volume. C'est précisément ce que l'instrumentation rend pénible (point 1).

**3. Les simulateurs prennent des paramètres, pas des mesures.**
- aihwkit code en dur des paramètres de dérive PCM calibrés sur une puce IBM de 2019, avec `t_0 = 20 s` et σ(ν) jusqu'à 0,045. [OUVERT src/aihwkit/inference/noise/pcm.py]
- IBM a dû ajouter une compensation par colonne (v0.9.2, 2024) puis par lecture de référence (v1.1.0, 2026-02-03). [OUVERT github.com/IBM/aihwkit/releases]
- CrossSim V3.2 (2026-04-03) et NeuroSim V2.1 (≈12 h par entraînement VGG-8) prennent aussi des paramètres. [EXTRAIT cross-sim.sandia.gov ; OUVERT README NeuroSim]
- Aucun ne fournit de chaîne « mes mesures → mon préréglage ».

**4. L'extraction de modèles compacts reste manuelle.**
- Les modèles Stanford et JART VCM demandent un « réglage manuel extensif ». [EXTRAIT arXiv 2511.07926 ; emrl.de]
- IC-CAP n'avait pas de support memristor ; nSpace Labs développe un module Python pour combler ce manque. [EXTRAIT knowm.org]
- Le ML Toolkit de Keysight (MBP 2026 / IC-CAP 2025, 2026-01-15) vise GAA, GaN, SiC et chiplets, pas les memristors. [EXTRAIT keysight.com PR26-024 ; recherche du 2026-09-25]

**5. Chaque labo construit son propre schéma de données.**
- La première base relationnelle publique, 6 190 memristors, 161 006 expériences et plus de 169 M de points, date de septembre 2026 et a dû créer son schéma. [EXTRAIT arXiv 2609.01500]

### Solution

1. **Pilotes 64 bits multiplateformes** pour B1500/WGFMU, 4200A-SCS/PMU et ArC TWO, avec découpage automatique des séquences au-delà des limites.
2. **Bibliothèque de protocoles standardisés** : forming, endurance, rétention, relaxation résolue dans le temps, cartographie de variabilité sur plaquette.
3. **Base de données** au schéma relationnel ouvert, inspiré d'arXiv 2609.01500.
4. **Extraction automatique** des modèles Stanford, JART et UniMORE par surrogat ML et optimisation. Export Verilog-A.
5. **Export direct en préréglages aihwkit / CrossSim**, pour relier la physique mesurée à la précision de l'IA.
6. **Positionnement commercial recentré** sur la **qualification d'eNVM** (RRAM, FeFET, MRAM) : rétention, endurance, variabilité. C'est là qu'il y a des acheteurs, pas la synapse analogique (voir les rejets).

### Comment la solution répond à la problématique

- Les points 1 et 5 disparaissent : pilotes et schéma sont fournis.
- Le point 2 devient faisable en volume : protocoles de relaxation automatisés.
- Les points 3 et 4 sont couverts par l'extraction automatique et l'export de préréglages.

### Concurrents

- **Keysight IC-CAP / MBP 2026 ML Toolkit** : menace principale, mais memristors non ciblés.
- **nSpace Labs** : module memristor, dépendant d'IC-CAP.
- **ArC Instruments (ArC ONE/TWO)** : matériel avec interface open source, sans extraction de modèle trouvée. [EXTRAIT arc-instruments.co.uk]
- **Keithley Clarius** : tests NVRAM/ReRAM/FeRAM prêts à l'emploi, sans pont vers les simulateurs. [EXTRAIT tek.com]
- **FormFactor** : assistants de sonde autonomes.
- **Académique** : base 2609.01500, extraction CNN 2511.07926, aihwkit, CrossSim, NeuroSim, MemTorch.
- **YC** : Hilstart (test matériel au niveau cartes), Nine Fives (RF), DeepSim (thermique). **Aucun sur la caractérisation de dispositifs mémoire.** [EXTRAIT ycombinator.com/companies]
- **Angle qu'aucun n'a** : chaîne complète de l'instrument jusqu'au préréglage de simulateur IA, neutre vis-à-vis de l'instrument, spécifique aux mémoires émergentes.

### Qui paie

- Licenciés et fonderies RRAM : Weebit Nano et ses licenciés (onsemi, janvier 2025 ; Texas Instruments, décembre 2025 ; DB HiTek ; SkyWater), TSMC (RRAM 40/28/22 nm en volume, 12 nm en qualification), Infineon (AURIX TC4x en RRAM 28 nm).
- Startups mémoire : 4DS, Crossbar.
- Laboratoires universitaires et instituts (imec, CEA-Leti, Jülich) : noms cités comme exemples de labos du domaine, sans décompte sourcé.
- Sources : [EXTRAIT design-reuse.com/news/202501083 ; weebit-nano.com ; rapport annuel TSMC 2025 ; infineon.com]

### Revenu potentiel — MARCHÉ À VALIDER

- **Clients industriels nommés et sourcés : environ 11 entités** (liste ci-dessus).
- **Ancres de prix** :
  - châssis nu Keithley 4200A-SCS : 55 000 $ [EXTRAIT testequipmentdepot.com] ;
  - IC-CAP et B1500 : prix sur devis, **non trouvés**.
- **Plafond visible** : Weebit, leader de l'IP ReRAM, fait 4,4 M$ de chiffre d'affaires sur l'exercice clos au 30/06/2025. [EXTRAIT design-reuse.com/news/202529258]
- Les publications en commutation résistive ont été « multipliées par 8 en une décennie » [EXTRAIT APL Energy]. Le nombre de labos n'est pas sourcé.
- **Calcul** : 11 industriels × HYPOTHÈSE non sourcée 50 000 à 150 000 $/an + N labos (HYPOTHÈSE non sourcée, 100 à 300) × HYPOTHÈSE non sourcée 5 000 à 15 000 $/an = **environ 1 à 6 M$/an**.
- **Extension** : la même chaîne sert à la MRAM et au FeFET (TSMC MRAM 16 nm automobile qualifiée en 2025 [EXTRAIT] ; FMC a levé 100 M€ en 2025 [EXTRAIT patsnap]). Cette extension est non chiffrée.

### Preuve des 8 critères

| # | Critère | Verdict et preuve |
|---|---|---|
| 1 | Vrai problème | Oui : limites du WGFMU [EXTRAIT], pilotes 32 bits [OUVERT], paramètres codés en dur dans aihwkit [OUVERT], extraction manuelle [EXTRAIT 2511.07926], schémas maison [EXTRAIT 2609.01500]. |
| 2 | Pas un piège à goudron | Les échecs documentés concernent les **fabricants de puces analogiques RRAM** (Rain AI, quasi-arrêt, brevets rachetés par OpenAI [EXTRAIT finance.yahoo.com] ; restructuration de Mythic en 2022 [EXTRAIT eetimes.com]), pas les outils de caractérisation. Le risque est traité par le recentrage sur l'eNVM binaire, qui a des clients en production (TSMC, Infineon, TI). |
| 3 | Problème aigu | Élevé pour la qualification : rétention et endurance conditionnent la mise en production automobile (TSMC 22ULL Grade-1 en 2025). Fréquence : chaque nouveau nœud ou procédé. |
| 4 | Marché | MARCHÉ À VALIDER. |
| 5 | Concurrence | Nommée ; chaîne complète non trouvée ailleurs. |
| 6 | Pourquoi maintenant | TSMC RRAM 22 nm Grade-1 (2025), 12 nm en qualification [EXTRAIT] ; Weebit–TI (décembre 2025) ; Keysight valide la demande d'extraction par ML (2026-01-15) sans couvrir les memristors ; aihwkit 1.0 ajoute des modèles ReRAM HfOx (mai 2025) [OUVERT] ; première grande base relationnelle (septembre 2026). |
| 7 | Proxy | PDF Solutions : 219 M$ de revenus 2025, dont 205,1 M$ récurrents [EXTRAIT pdf.com, 8-K] ; Tignis (analytique IA de procédé), racheté par Cohu pour 40,1 M$ le 2025-01-07 [EXTRAIT ir.cohu.com] ; Coventor, racheté par Lam en 2017 [EXTRAIT semiengineering.com]. |
| 8 | Scalable | Logiciel. Chaque pilote d'instrument ou modèle compact est un coût fixe réutilisé par tous. |

### Formulations du balayage (≥ 10)

1. « RRAM compact model parameter extraction automated calibration machine learning measured I-V »
2. « JART VCM compact model Verilog-A variability »
3. « Keysight IC-CAP 2025 memristor RRAM Python »
4. « IC-CAP license price »
5. « Keysight machine learning toolkit device modeling PDK January 2026 »
6. « Keysight memristor ReRAM model extraction IC-CAP 2026 »
7. « nSpace Labs memristor IC-CAP add-on »
8. « ArC Instruments ArC TWO memristor characterization platform »
9. « automated memristor RRAM wafer-level characterization software startup database "emerging memory" »
10. « semiconductor lab automation software platform parameter analyzer probe station Python startup »
11. `site:ycombinator.com/companies semiconductor characterization test automation lab`
12. « WGFMU RRAM endurance instrument limitation »
13. « Keithley 4200A Clarius ReRAM PCM »

**Limite du balayage** : Product Hunt et les changelogs Siemens EDA n'ont pas été balayés pour cette idée. Aucun signal ne suggère un concurrent à cet endroit, mais c'est un trou de couverture.

---

## Idée 5 — Gatekeeper ECG : décision de transmission et qualité du signal sur patch, avec dossier PCCP

**Sujet de thèse d'origine.** C : IA « poly-neuromorphique » pour le biomonitoring, SNN et modules adaptatifs ultra-basse consommation. Le produit ne vend pas de SNN : la logique adaptative peut être dedans, mais l'argument est la réduction mesurable des transmissions et du risque.

**Question technique d'origine.**
- Où va l'énergie d'un patch ECG ?
- Une puce neuromorphique la réduit-elle ?
- Que devient un « module adaptatif » face à la FDA ?

### Problématique (mécanisme)

**1. La radio domine, pas le calcul.**
- Front-end analogique ADI MAX30001 : 85 µW en ECG. [EXTRAIT analog.com]
- Microcontrôleur Ambiq Apollo4 : ~4-5 µA/MHz. [EXTRAIT ambiq.com]
- Sur la plateforme BioGAP, la consommation est dominée par le domaine analogique et le SoC nRF52, « principalement pour streamer les données en BLE ». [EXTRAIT arXiv 2307.01619]
- Une carte SNN (Xylo IMU < 300 µW, Pulsar 400-600 µW) consomme plus que le front-end. [EXTRAIT synsense.ai ; innatera.com]
- **Donc** le gain accessible consiste à transmettre moins, par du logiciel sur le microcontrôleur existant, pas à ajouter une puce.

**2. Les faux positifs épuisent un budget de télémétrie.**
- La lettre d'avertissement FDA du 2023-05-25 sur le Zio AT documente une limite de 100 événements patient et 500 arythmies auto-détectées par période. Au-delà, rien n'était transmis. La FDA relie cette limite à des arythmies manquées, dont deux décès.
- iRhythm a obtenu deux 510(k) de correction les 21 et 30 octobre 2024.
- [EXTRAIT massdevice.com ; cardiovascularbusiness.com ; 10-Q iRhythm]
- Les artefacts de mouvement produisent 85 à 99 % de fausses alertes d'arythmie en soins intensifs. [EXTRAIT Sensors 2026, 10.3390/s26041135]
- Les patchs ont plus de bruit que les dispositifs filaires. [EXTRAIT PMC12842261]
- **Mécanisme général** : chaque faux positif coûte de l'énergie radio, du coût cellulaire, du temps de lecture au centre de monitoring et, en cas de plafond, un risque clinique.
- *Inférence non démontrée par une source* : un tri local qualité/événement réduit ces quatre coûts.

**3. L'adaptation en ligne n'est autorisable que bornée.**
- La guidance FDA sur les PCCP (Predetermined Change Control Plans), finale depuis le 2024-12-04, s'applique à toute fonction logicielle à IA. Elle exige de décrire d'avance les modifications, la méthode de validation et l'impact, et de mettre à jour l'étiquetage. [EXTRAIT federalregister.gov 2024-28361 ; ropesgray.com]
- En Europe, l'apprentissage continu n'est « généralement pas certifiable sans contrôles stricts ». [EXTRAIT hoganlovells.com]
- Un module adaptatif n'est donc vendable que dans une **enveloppe prédéfinie**, avec des critères d'acceptation vérifiables sur l'appareil.

### Solution

Une bibliothèque C embarquée, sans puce supplémentaire, pour Apollo, nRF54 et microcontrôleurs équivalents :
1. Indice de qualité de signal par segment : contact d'électrode, mouvement fusionné avec l'accéléromètre, dérive.
2. Politique « transmettre / résumer / s'abstenir » sous contrainte d'énergie et de budget d'événements.
3. Personnalisation bornée (seuils, gabarit de battement du patient), dont l'enveloppe est rédigée comme un PCCP.
4. **Livrable réglementaire inclus** : protocole, critères d'acceptation, rapport de validation, texte d'étiquetage.

### Comment la solution répond à la problématique

- Moins de transmissions inutiles, donc moins d'énergie radio (point 1).
- Moins de faux positifs consommant le budget et l'attention (point 2).
- Une adaptation livrée sous une forme que le régulateur accepte (point 3).

### Concurrents

- **B-Secur HeartKey** : bibliothèque ECG avec 510(k), qui tourne sur l'appareil ou dans le cloud. Proposée avec ADI/Maxim, ST et TI. [EXTRAIT ti.com/tool/BSECUR-3P-ALGORITHMS ; medicaldesignandoutsourcing.com]
  - HeartKey Rhythm annonce une **réduction de 40,6 % des événements de faible qualité à relire** et de 9,7 % des données jugées non diagnostiquables. [EXTRAIT prnewswire.com/news-releases/b-secur-elevates-…-302236210.html, septembre 2024]
  - C'est un **concurrent direct sur la qualité du signal**.
- **Ambiq heartKIT** : open source, 59 étoiles, lié à Ambiq. [OUVERT github.com/AmbiqAI/heartkit]
- **Edge Impulse / Qualcomm** : substitution pour les équipes qui construisent elles-mêmes.
- Algorithmes cloud (Anumana, AliveCor, Philips/Cardiologs) : non vérifiés en détail.
- **Angle revendiqué** : la **décision de transmission sous contrainte d'énergie et de budget**, plus le **dossier PCCP de personnalisation bornée**. Aucune de ces deux fonctions n'a été trouvée chez HeartKey dans les extraits lus.

**À TRANCHER.** On ne sait pas si HeartKey, qui réduit déjà les événements de faible qualité de 40,6 % et tourne sur l'appareil, offre ou peut offrir une politique de transmission configurable et un PCCP. Pour trancher, il manque la documentation technique de l'API HeartKey et la liste de ses fonctions sur l'appareil ; ni l'une ni l'autre n'était lisible dans cette session. Si HeartKey les a, l'idée est à rejeter (angle déjà pris).

### Qui paie

Fabricants de patchs ECG et de télémétrie cardiaque (MCT) hors iRhythm, qui est intégré verticalement :
- VitalConnect, SmartCardia, Wellysis, Cardiosense, HeartBeam, Fourth Frontier, Element Science [EXTRAIT fiercebiotech.com] ;
- Baxter/Bardy, Philips/BioTelemetry, Boston Scientific/Preventice [EXTRAIT mercomcapital.com ; bostonscientific.com].

### Revenu potentiel — MARCHÉ À VALIDER

- **Nombre d'acheteurs** : environ 10 à 15 fabricants nommés (comptage partiel sourcé ci-dessus).
- **Valeur de l'examen** :
  - iRhythm : 747,1 M$ de revenu 2025 pour « plus de 2 millions de patients » [EXTRAIT 10-K iRhythm], soit au plus ~374 $ par patient (calcul) ;
  - Medicare 2024 : CPT 93247 à 234,34 $ [EXTRAIT fiche Boston Scientific].
- **Calcul** :
  - 12 fabricants × HYPOTHÈSE non sourcée 100 000 à 300 000 $/an = **1,2 à 3,6 M$/an** ;
  - ou redevance HYPOTHÈSE non sourcée 1 à 5 $ par patch.
- **Petit en ECG seul.** L'extension vers l'EEG, l'EMG, le PPG et les wearables grand public autorisés est non chiffrée.
- Contexte : 116 dispositifs IA cardiovasculaires dans la liste FDA au 12/2025 [EXTRAIT intuitionlabs.ai].

### Preuve des 8 critères

| # | Critère | Verdict et preuve |
|---|---|---|
| 1 | Vrai problème | Oui : lettre FDA Zio AT et corrections 510(k) [EXTRAIT massdevice] ; faux positifs [EXTRAIT Sensors 2026] ; radio dominante [EXTRAIT arXiv 2307.01619]. |
| 2 | Pas un piège à goudron | Pièges voisins vérifiés : Proteus Digital Health (plus de 500 M$ levés, faillite en juin 2020, actifs vendus 15 M$) [EXTRAIT cnbc.com] ; Eta Compute (SNN pour capteurs), sortie du silicium [EXTRAIT eetimes.com] ; BrainChip, 1,88 M$ de revenu annuel [EXTRAIT kalkine.com.au]. Ces échecs concernent des puces ou des modèles de remboursement, pas une bibliothèque licenciée. B-Secur montre que ce dernier modèle existe. |
| 3 | Problème aigu | Élevé : risque clinique (décès cités par la FDA), coût cellulaire et énergie. |
| 4 | Marché | MARCHÉ À VALIDER. |
| 5 | Concurrence | **À TRANCHER** (voir ci-dessus). |
| 6 | Pourquoi maintenant | Guidance PCCP finale (2024-12-04) ; lettre Zio AT (2023-05-25) et corrections (octobre 2024) ; échéance Annexe I de l'AI Act pour les dispositifs médicaux reportée au 2028-08-02 [EXTRAIT morganlewis.com, juin 2026] ; microcontrôleurs à quelques µA/MHz. |
| 7 | Proxy | B-Secur : bibliothèque ECG embarquée avec 510(k), licenciée via TI, ST et ADI [EXTRAIT ti.com] ; chiffre d'affaires non trouvé. Edge Impulse (racheté par Qualcomm). |
| 8 | Scalable | Bibliothèque licenciée à la redevance. Le dossier PCCP est réutilisable d'un client à l'autre, avec une adaptation par client. |

### Formulations du balayage (≥ 10)

1. « B-Secur HeartKey ECG algorithm library licensing embedded wearables FDA cleared 2025 »
2. « B-Secur HeartKey signal quality edge processing reduce data transmission battery wearable 2025 »
3. « embedded ECG signal quality and arrhythmia firmware SDK for wearable OEMs reduce Bluetooth transmission battery 2026 launch »
4. `site:ycombinator.com/companies ECG wearable patch arrhythmia` : Makani Science, Zeit Medical (adjacents)
5. « PCCP predetermined change control plan software tool startup »
6. « producthunt.com ECG AI wearable SDK » : Open Wearables, ROOK (API de montres, hors sujet)
7. GitHub « tinyml ecg » : heartKIT
8. GitHub « ecg signal quality index »
9. GitHub « ecg arrhythmia microcontroller embedded »
10. « wearable ECG patch manufacturers list FDA cleared 2025 »
11. « BLE radio dominates power consumption wearable ECG on-device processing »

---

## Idée 6 — SNN-FI : injection de fautes sur la représentation matérielle des SNN déployés

**Sujet de thèse d'origine.** D : système nerveux embarqué à base de SNN « auto-conscient » (détection d'anomalies, auto-réparation) et matériel reconfigurable.

**Question technique d'origine.** Comment qualifier la robustesse d'un SNN tel qu'il est réellement stocké sur puce (poids quantifiés, décalages de bits) face aux SEU et aux neurones ou synapses morts ?

### Problématique (mécanisme)

- Les outils d'injection de fautes SNN sont académiques et opèrent sur des modèles PyTorch en flottant. SpikeFI est « built upon the SLAYER PyTorch framework ». [EXTRAIT arXiv 2412.06795]
- Or SLAYER est l'ancêtre de lava-dl, **archivé** depuis 2026-05-13. [OUVERT github.com/lava-nc]
- La représentation déployée diffère du modèle flottant :
  - Xylo : poids de 8 bits avec `weight_shift` par couche [OUVERT rockpool xylo_samna.py] ;
  - Akida : quantification avec pertes possibles de « several dozen percent » [OUVERT fiche ON].
- Une faute sur un bit de décalage n'a pas le même effet qu'une perturbation en flottant.
- Des clients spatiaux et défense apparaissent : Frontgrade Gaisler a licencié Akida pour des SoC « space-grade, fault-tolerant », avec 100 h de support d'intégration et 10 % de redevance [EXTRAIT Listcorp, décembre 2024] ; Parsons a commandé des AKD1500 [EXTRAIT, S1 2026].

### Solution

- Injection de fautes (SEU, collage, neurone mort) sur la représentation binaire réelle de chaque puce, via les simulateurs bit-exacts (Rockpool/Xylo, Sinabs/Speck) et les formats Akida.
- Métriques de dégradation.
- Génération automatique de moniteurs d'anomalie et de redondance ciblée (le « auto-réparation » de la thèse D).
- Rapport de qualification.

### Concurrents

- SpikeFI, SpikingJET, tests en radiation académiques (arXiv 2605.00030). [EXTRAIT]
- Aucun produit commercial trouvé.
- Hors SNN, le marché de la qualification radiation n'a **pas été recherché** ici.

### Qui paie

Intégrateurs spatiaux et défense qui embarquent des puces neuromorphiques (Frontgrade Gaisler, Parsons), et vendeurs de puces qui veulent un argument de sûreté de fonctionnement.

### Revenu potentiel — MARCHÉ À VALIDER

- Acheteurs nommés : 2 (Frontgrade Gaisler, Parsons) [EXTRAIT], plus les vendeurs de puces (30 selon Tracxn [EXTRAIT]).
- Prix : HYPOTHÈSE non sourcée.
- Aucun calcul significatif possible.

### Preuve des 8 critères

| # | Critère | Verdict et preuve |
|---|---|---|
| 1 | Vrai problème | Plausible : l'écart entre représentation flottante et représentation matérielle est démontré [OUVERT], et la dépendance à SLAYER est cassée [OUVERT]. **À TRANCHER** : aucune exigence normative publiée (ECSS, DO-254) spécifique aux SNN n'a été trouvée. Il manque la preuve qu'un acheteur spatial exige cette qualification. |
| 2 | Pas un piège à goudron | Aucun échec d'outil de ce type trouvé. |
| 3 | Problème aigu | Élevé si déployé en orbite ; la fréquence est très faible. |
| 4 | Marché | MARCHÉ À VALIDER (2 acheteurs nommés). |
| 5 | Concurrence | Académique uniquement. |
| 6 | Pourquoi maintenant | Akida dans des SoC spatiaux (décembre 2024) ; commande défense (2026) ; archivage de lava-dl/SLAYER (2026-05-13). |
| 7 | Proxy | Non trouvé dans le périmètre neuromorphique. **À TRANCHER** : chercher un proxy dans les outils d'injection de fautes et de sûreté de fonctionnement pour l'automobile et le spatial. |
| 8 | Scalable | Logiciel, mais des ventes longues (ITAR, cycles spatiaux). |

### Formulations du balayage (≥ 10)

1. « SNN fault injection framework tool reliability neuromorphic hardware open source radiation »
2. « self-repairing SNN astrocyte fault tolerance FPGA »
3. « BrainChip Akida Cloud IP licensees Renesas MegaChips Frontgrade »
4. « bit-accurate OR hardware-in-the-loop SNN simulator »
5. `site:ycombinator.com/companies neuromorphic`
6. `site:ycombinator.com/companies spiking neural network`
7. « neuromorphic software startup funding 2026 »
8. « producthunt neuromorphic »
9. « neuromorphic compiler platform multiple chips »
10. « SNN ECG EEG wearable on-chip adaptation »

---

# B. Idées rejetées, avec la preuve du rejet

| # | Idée rejetée | Sujet | Critère qui échoue | Preuve (source) |
|---|---|---|---|---|
| R1 | Nouveau capteur ou caméra à événements | F, H | 2 (piège à goudron) | Prophesee (plus de 100 M€ levés) : redressement judiciaire le 2024-10-15, 4,09 M€ de chiffre d'affaires 2024 [EXTRAIT vipress.net ; pappers.fr]. Insightness absorbée par Sony (2019), CelePixel par Will Semi (2020), iniVation par SynSense (2024) [EXTRAIT crunchbase ; image-sensors-world ; s-ge.com]. |
| R2 | Puce ou accélérateur SNN (vision, ECG, générique) | C, F | 2 et 4 | Revenus BrainChip S1 2026 : 1,22 M$, perte de 12,0 M$ [EXTRAIT Motley Fool AU 2026-08-26]. GrAI Matter Labs absorbée par Snap fin 2023 (date divergente selon les sources : 15/09 ou 31/10/2023) [EXTRAIT eetimes.com ; marketscreener]. Rain AI : série B de 150 M$ échouée, brevets vendus à OpenAI [EXTRAIT finance.yahoo.com]. Gains énergétiques SNN calculés, pas mesurés [EXTRAIT arXiv 2409.08290]. |
| R3 | Simple convertisseur de formats d'événements | H | 5 (angle déjà pris, gratuit) | faery, expelliarmus, Tonic, event_camera_codecs, ADΔER : gratuits et maintenus [OUVERT READMEs]. JPEG XE au stade DIS (avril 2026) avec logiciel de référence [EXTRAIT jpeg.org]. La fonction n'est retenue que dans EventOps (idée 1). |
| R4 | IP de codec JPEG XE / compression orientée tâche vendue aux fabricants de capteurs | H | 4 (calcul de marché) | Environ 4 clients IP possibles (Sony, Prophesee, OmniVision/Will Semi, SynSense/iniVation) [sources R1], dont un sort de redressement, avec un leader à 4,09 M€ de chiffre d'affaires. Les fabricants intègrent déjà la compression dans le silicium (IMX636 « compressive data-formatting pipeline », ISSCC 2020 [OUVERT liste uzh-rpg]). Le codec normalisé devient une commodité (logiciel de référence). Même avec l'hypothèse la plus généreuse (4 × 500 k$), le plafond est d'environ 2 M$. |
| R5 | Passerelle edge/fog « ville intelligente » à événements vendue seule | H | 1 et 4 | Le gain existe (flux DVS ~30× plus petit que la vidéo CMOS brute, ~5× après compression [EXTRAIT PMC12900004]), mais aucun acheteur identifié ni parc installé documenté. Rapports de marché contradictoires d'un facteur 500 (9 M$ à 5,13 Md$) [EXTRAIT mobilityforesights, growthmarketreports, mordorintelligence]. Technologie en quête d'application. |
| R6 | « Event Label Factory » : étiquetage et données synthétiques événementielles | F | 5 (angles pris) | Service d'annotation DVS déjà proposé par Labelvisor (« vehicle-based DVS annotation ») [EXTRAIT labelvisor.com/vehicle-based-dvs-annotation]. La génération vidéo→événements est au cœur de Hearth, « video-to-event foundation model » de Prophesee (juin 2026) [EXTRAIT blackscarab.ai ; prophesee.ai 2026-06-15]. Simulateurs gratuits EVIS et EsaacSim (2026), v2e, ESIM [EXTRAIT/OUVERT]. |
| R7 | Modèles SNN en SaaS (segmentation, mouvement humain) | F | 1 et 4 | Pas de données étiquetées (R6) ; cibles matérielles fragmentées ; communauté de quelques centaines d'étoiles (OpenEB 305, v2e 489) [OUVERT]. Écart SNN–ANN persistant en mAP50:95 (RVT 0,472 contre EAS-SNN 0,437) [EXTRAIT arXiv 2403.12574]. |
| R8 | « Edge Impulse pour le neuromorphique » généraliste | C, D | 5 | Edge Impulse supporte Akida depuis 2022 et appartient à Qualcomm (mars 2025) [EXTRAIT edgeimpulse.com ; siliconangle.com]. Talamo (Innatera) et Akida Cloud (2025-08-05) occupent le terrain de chaque vendeur [EXTRAIT]. |
| R9 | Accès cloud au matériel neuromorphique | C, D | 5 | Akida Cloud (gratuit puis à l'usage), cloud SpiNNaker2 de SpiNNcloud, INRC gratuit (plus de 200 membres) [EXTRAIT BusinessWire ; intc.com]. |
| R10 | Benchmark énergétique en service payant | C, F | 1 et 5 | NeuroBench est gratuit et communautaire. L'article déclare une mesure uniforme « currently infeasible » [EXTRAIT arXiv 2304.04640 ; OUVERT code]. Un service payant n'aurait pas d'autorité normative. La fonction d'estimation est intégrée à NeuroFit (idée 3). |
| R11 | Nouveau framework d'entraînement SNN | C, F | 5 | SpikingJelly (2 140 étoiles), snnTorch (2 057), Norse (823) : gratuits, actifs, sans monétisation connue [OUVERT API GitHub]. |
| R12 | Service de portage « post-Lava » (Loihi vers NIR et autres puces) | D | 5 et 8 | QuantaraCore publie « Intel Lava Is Archived: What to Use Instead » et se positionne exactement là [EXTRAIT quantaracore.in/blog/lava-archived-alternative]. C'est un service non récurrent : plus de 200 membres INRC, surtout académiques et à accès gratuit. Intel annonce un SDK successeur « built on open-standard AI frameworks » qui ferait disparaître le besoin [EXTRAIT]. |
| R13 | Conformité NIR vendue seule | C, D, F | 4 (calcul) | 5 plateformes matérielles déclarées dans NIR [OUVERT support.md] plus Innatera. Même avec l'hypothèse de 5 à 10 vendeurs × 50 à 100 k$, on obtient 0,25 à 1 M$. Conservée seulement comme fonction de l'idée 3. |
| R14 | Nouvelle startup d'IP ReRAM ou memristor | A | 2 et 4 | Weebit, leader : 4,4 M$ de chiffre d'affaires sur l'exercice 2025 [EXTRAIT design-reuse.com]. 4DS en revue stratégique (jetons des administrateurs à zéro, novembre 2025) ; Crossbar pivoté vers la sécurité [EXTRAIT]. |
| R15 | Puce IA analogique en mémoire à base de RRAM ou PCM | A | 2 | Rain AI de fait arrêtée [EXTRAIT]. Les survivants évitent les mémoires émergentes : Mythic et Sagence utilisent de la flash, EnCharge des capacités métalliques en CMOS standard [EXTRAIT eetimes.com ; businesswire 20250529]. |
| R16 | Outil d'entraînement conscient du matériel (HAT) calibré, vendu seul | A | 4 et 5 | Outils gratuits maintenus : aihwkit (MIT, v1.1.0 du 2026-02-03, modèles ReRAM HfOx), CrossSim 3.2 (2026-04-03), NeuroSim V1.5, MemTorch [OUVERT/EXTRAIT]. Clients potentiels : 3 à 5 startups IMC, qui n'utilisent pas la RRAM (R15) et ont leur SDK interne. Conservé seulement comme fonction d'export de MemCal (idée 4). |
| R17 | Plateforme de design inverse magnonique ou surrogat micromagnétique | B | 1 et 4 | Le YIG de qualité exige une épitaxie sur GGG à 700–850 °C, incompatible avec le CMOS [EXTRAIT PMC10533023 ; chemistryworld.com] : aucun client industriel magnonique identifié. Outils gratuits et différentiables déjà disponibles (NeuralMag, magnum.np, SpinTorch, mumax+) [EXTRAIT nature.com s41524-025-01688-1]. Un pivot MRAM ou SOT est possible, mais sans preuve de demande. |
| R18 | Outil de modélisation fractionnaire des neurones (bond graphs, port-Hamiltonien) | E | 1 | Usage industriel trouvé uniquement hors neurones : bio-impédance (modèle de Cole, ImpediMed) et batteries (circuits équivalents fractionnaires) [EXTRAIT PMC12845819 ; J. Energy Storage 2025]. Aucun usage industriel des neurones fractionnaires trouvé. Une startup viable viserait le BMS batteries, hors périmètre. |
| R19 | Micro intelligent : mot d'éveil, séparation de sources, identification du locuteur sur puce analogique ou à reservoir | D | 2 et 4 | Syntiant (S-1 du 2026-07-06) : le segment IA ne pèse que 5,1 % des 64,5 M$ du trimestre. Environ 95 % viennent des micros MEMS rachetés à Knowles [EXTRAIT renaissancecapital.com ; tomorrowaccess.com]. Aspinity sans nouvelle depuis mars 2024 [EXTRAIT businesswire 20240327]. |
| R20 | Maintenance prédictive vibratoire sur puce neuromorphique ou analogique | D | 5 | Le Machine Learning Core du ST LSM6DSOX fait déjà l'inférence pour ~13 µA sur ~563 µA [EXTRAIT st.com AN5259]. Augury domine les plateformes [EXTRAIT]. Polyn cible les pneus après plus de 4 ans [EXTRAIT edn.com ; businesswire 20260429806233]. |
| R21 | « Reservoir of reservoirs » ou matériel évolutif reconfigurable comme produit | D | 1 | Pas de silicium commercial : TDK n'a qu'un prototype (2025-10-02) [EXTRAIT tdk.com]. Le transfert de la couche de sortie d'un exemplaire à l'autre n'est pas résolu : chaque unité doit être réentraînée, et l'avantage « est substantiellement affaibli » [EXTRAIT arXiv 2607.02608]. |
| R22 | Service de recalibration de reservoirs d'un exemplaire à l'autre | D | 4 | Moins de 10 acheteurs identifiés (TDK, Polyn, Aspinity, Blumind), tous au stade prototype ou premier silicium [EXTRAIT]. |
| R23 | Contrôleur de vol neuromorphique pour drones | G | 1 | Démonstration TU Delft : 0,94 W au repos pour la plateforme contre 7 à 12 mW pour le réseau, sur Loihi, non commercial [EXTRAIT science.org/doi/10.1126/scirobotics.adi0591]. L'énergie du réseau n'est pas le poste dominant. |
| R24 | Outil PCCP générique (conformité logicielle) | C | 5 | Terrain occupé par les cabinets et guides réglementaires (Ropes & Gray, Hogan Lovells, IntuitionLabs) [EXTRAIT]. Seul l'angle « PCCP embarqué de personnalisation bornée » est conservé dans l'idée 5. |
| R25 | Puce SNN dédiée au patch ECG | C | 1 | Une carte SNN (300 à 600 µW) consomme plus que le front-end ECG (85 µW), et c'est la radio qui domine [EXTRAIT synsense.ai ; innatera.com ; analog.com ; arXiv 2307.01619]. Des bibliothèques sur microcontrôleur existent (heartKIT, HeartKey). |

---

# C. Couverture des sous-couches : questions techniques posées et réponses

## C1. Capteurs à événements (Prophesee, Sony IMX636/IMX646, iniVation, OmniVision, Samsung)

**Q : Quel débit et quelle latence pour un capteur HD ?**
- Sony IMX636 (1280×720, pixel de 4,86 µm) : 1,06 Gev/s maximum, latence ≤ 100 µs à 1 000 lux, 110 dB. [EXTRAIT framos.com ; sony-semicon.com flyer IMX636]
- L'article ISSCC 2020 inclut un « Programmable Event-Rate Controller and Compressive Data-Formatting Pipeline ». [OUVERT liste uzh-rpg]

**Q : Et le petit capteur embarqué ?**
- GenX320 (320×320) : < 150 µs, > 140 dB, 36 µW en ultra-basse consommation, ~3 mW actif, MIPI/CPI. [EXTRAIT prophesee.ai ; cnx-software.com 2025-08-27]

**Q : Que se passe-t-il en saturation ?**
- L'ERC supprime des événements et peut créer des artefacts horizontaux. Seul indicateur : taux moyen sur 1 s. [EXTRAIT docs.prophesee.ai ; arXiv 2501.18788]
- En pratique, pertes au-delà de ~10 Mev/s sur de nombreux systèmes, et 80 à 90 % de CPU avec enregistrement à saturation. [EXTRAIT ; OUVERT metavision_driver]

**Q : Le réglage est-il automatique ?**
- Non. Cinq biais, et un article entier consacré à leur réglage. [EXTRAIT arXiv 2501.18788]
- Seul banc reproductible : BiasBench (2025). [OUVERT]

**Q : Qui fabrique encore ?**
- Sony, Prophesee, iniVation (SynSense).
- OmniVision : brevet 2025 pour un capteur 3 wafers 1 MP à 4,6 Gev/s. [EXTRAIT patsnap]
- Samsung : pas d'annonce DVS récente. [EXTRAIT tangramvision.com]
- Hors Europe, le produit IMX646 n'a pas été détaillé dans les extraits : **non couvert en détail**.

**Q : Le logiciel constructeur est-il pérenne ?**
- SDK payant depuis la 5.0, lié à l'EVK depuis 2024-10-07. [EXTRAIT faq Prophesee]
- Fin de vie d'OpenEB et du SDK autonome annoncée, remplacés par Hearth (2026). [EXTRAIT image-sensors-world 2026-07]

**Q : Quelle santé pour le leader ?**
- Redressement, plan au 2026-05-05, levée de 20 M€ en juillet 2026, recentrage défense avec Mantara (anti-drones). [EXTRAIT pappers ; f4news.com ; prophesee.ai 2026-06-15]

## C2. Formats de données (EVT 2.0/2.1/3.0, AEDAT4, HDF5-ECF)

**Q : Quelle taille selon le format ?**
- Sur un même fichier : DAT 851 Mo, EVT2 ~425 Mo, EVT3 ~350 Mo, soit un facteur ~2,4 avant compression générique. [OUVERT expelliarmus README]

**Q : Comment EVT3 compresse-t-il, et à quel prix ?**
- Codage vectorisé relatif à un état : 32 événements en 8 octets.
- Contreparties : horodatage de 24 bits qui reboucle à 16,77 s, décodage impossible à partir du milieu du flux, erreurs de bits. [EXTRAIT evt3 doc ; OUVERT event_camera_codecs]

**Q : AEDAT4 ?**
- Paquets FlatBuffers compressés LZ4/ZSTD. [EXTRAIT docs.inivation.com]
- Ticket dv-python « lz4 compression will not unpack ». [EXTRAIT gitlab.com/inivation]

**Q : HDF5 Prophesee ?**
- Filtre ECF 0x8ECF, plugin et `HDF5_PLUGIN_PATH` requis, tickets #96 et #160. [OUVERT hdf5_ecf, openeb ; EXTRAIT API GitHub]

**Q : Une normalisation arrive-t-elle ?**
- JPEG XE (ISO/IEC 26112, ITU-T T.JPEG-XE) : appel à propositions clos le 2025-03-31, cinq propositions, CD en 2025, **DIS à la 111e réunion (avril 2026)**. [EXTRAIT jpeg.org/items/20250521_press.html ; jpeg.org/items/20260608_press.html]
- EVT+ proposé en 2025-11. [EXTRAIT arXiv 2511.15556]

**Q : Existe-t-il des convertisseurs ?**
- Oui, gratuits : faery, expelliarmus, Tonic, event_camera_codecs, ADΔER. [OUVERT]

## C3. Compression et transport des flux d'événements

**Q : Quels ratios sans perte ?**
- +35,27 % par rapport à LZMA sur DSEC ; ratio de 9,29 à 13,01 combiné à LZMA (Schiopu et Bilcu, CVPRW 2023). [EXTRAIT openaccess.thecvf.com]
- LLC-ARES est conçu pour l'intégration sur puce. [EXTRAIT]

**Q : Quel gain par rapport à la vidéo ?**
- Sur 24 h de surveillance intérieure : DVS brut ~30× plus petit que la vidéo CMOS, ~5× après JBIG. [EXTRAIT PMC12900004]

**Q : Le lossy dégrade-t-il la tâche, et quelle métrique utiliser ?**
- Les métriques de distorsion existantes « fail to reliably predict compression-induced degradation at the task level ».
- Cinq métriques fondées sur la classification sont proposées, avec deux pipelines comparés (JPEG 2000 sur histogrammes, G-PCC en nuage de points). [EXTRAIT arXiv 2608.28429, août 2026]

**Q : Transport réseau ?**
- Streaming scalable pour machines via MoQ (arXiv 2508.15003) et streaming basse latence (arXiv 2412.07889) : recherche uniquement. [EXTRAIT]

**Q : Le sujet est-il mûr ?**
- Section « Compression » d'environ 10 entrées sur ~2 218 liens dans la liste uzh-rpg. [OUVERT]
- Critères subjectifs de qualité : aucune norme trouvée. JPEG XE lossy est prévu après la partie sans perte.

**Q : Compression en passerelle edge/fog multi-capteurs ?**
- Aucun produit commercial trouvé. Recherche seulement : classification dans le domaine compressé (groupe de Lisbonne, IEEE Access 2025). [OUVERT liste uzh-rpg]

## C4. Jeux de données et étiquetage événementiels

**Q : Comment les grands jeux de données sont-ils étiquetés ?**
- Prophesee 1Mpx (14,6 h, 25 M de boîtes) : transfert depuis une caméra RGB via un « commercial automotive detector ». Les frames RGB ne sont pas publiées (issue #8, 2021). [EXTRAIT arXiv 2009.13436 ; OUVERT issue #8]
- eTraM (10 h, plus de 2 M de boîtes) : annotation manuelle. [EXTRAIT arXiv 2403.19976]

**Q : Les outils d'annotation gèrent-ils les événements ?**
- Pas de support natif trouvé dans CVAT, Roboflow, Labelbox ou Encord. Contournement : accumulation en frames dans Label Studio. [EXTRAIT arXiv 2403.11875]
- Existe un service DVS (Labelvisor). [EXTRAIT]

**Q : La simulation comble-t-elle le manque ?**
- Partiellement. Calibrer les statistiques du simulateur apporte +20 à 40 % en reconstruction (ECCV 2020). [EXTRAIT arXiv 2003.09078]
- Forte dégradation des modèles entraînés en synthétique pur (CARLA DVS 2025). [EXTRAIT arXiv 2506.13722]
- v2e n'a pas de préréglage IMX636. [OUVERT v2e README]
- EVIS et EsaacSim sur Isaac Sim (2026). [EXTRAIT]
- Modèle vidéo→événements Hearth chez Prophesee (2026). [EXTRAIT]

**Q : Taille de la communauté ?**
- 177 dépôts « event-camera », OpenEB 305 étoiles, v2e 489. [OUVERT API GitHub]

## C5. Chaînes d'outils SNN (Lava, snnTorch, Norse, SpikingJelly, Rockpool, Sinabs, NIR)

**Q : Comment NIR représente-t-il un réseau ?**
- 17 primitives en temps continu ; discrétisation laissée aux backends ; métadonnées non normatives. [OUVERT primitives.md]

**Q : Quels backends, et en lecture ou en écriture ?**
- 9 simulateurs et 5 plateformes matérielles. Lava-DL et SpiNNaker2 en lecture seule.
- Delay, Scale et I sans aucun support. [OUVERT support.md, supported_primitives.md]
- Akida : pas de LIF. [OUVERT fiche ON]

**Q : Que casse la conversion ?**
- Représentation des spikes dépendante de `dt`.
- Reset soustractif perdu dans snnTorch.
- Rupture de format liée à `v_reset` (#142).
- Broadcast ×1024 (#176).
- `w_in` de CubaLIF restreint (#190).
- Pas de fixtures (#193).
- [OUVERT]

**Q : Quel écart entre plateformes ?**
- De 48,6 % (Lava) à 95 % (snnTorch) sur le RNN Braille.
- 95,40 % sur Speck contre 98,47 % sur Sinabs pour le CNN N-MNIST.
- [OUVERT, .npy du dépôt NIR]

**Q : État de maintenance ?**
- Lava et lava-dl archivés (2026-05-13). [OUVERT]
- SpikingJelly (2 140 étoiles) et snnTorch (2 057) actifs.
- Rockpool sous AGPL-3.0.
- [OUVERT API GitHub]

## C6. Déploiement sur puces (Loihi 2/Hala Point, SpiNNaker2/SpiNNcloud, Akida, Speck/Xylo, Pulsar, GrAI)

**Q : Comment compile-t-on pour Speck ?**
- Sinabs + samna, 9 cœurs aux mémoires hétérogènes, erreur générique si ça ne rentre pas. [OUVERT dynapcnn.py]

**Q : Contraintes de Xylo ?**
- Poids 8 bits, 1 000 neurones, fan-in 63, `weight_shift` par couche.
- Simulation bit-exacte dans Rockpool.
- [OUVERT]

**Q : Pipeline Akida ?**
- Keras → quantizeml → CNN2SNN.
- Pertes de quantification possibles de plusieurs dizaines de pourcents.
- Apprentissage sur puce limité à la dernière couche.
- Mesure de puissance intégrée.
- [OUVERT fiche ON]

**Q : Accès et prix ?**

| Plateforme | Accès et prix |
|---|---|
| Loihi 2 | Via l'INRC uniquement (plus de 200 membres) |
| AKD1000 PCIe | 499 $ |
| Edge AI Box | 799 $ |
| Akida Cloud | Lancé le 2025-08-05 |
| SpiNNcloud | Systèmes « multi-million euro » (Leipzig) |
| Pulsar | Moins de 5 $ en volume, lancé le 2025-05-21, sans apprentissage sur puce |
| Speck / Xylo | Prix **non trouvés** |

Sources : [EXTRAIT intc.com ; CNX Software ; brainchip.com ; BusinessWire ; DCD ; innatera.com ; OUVERT fiche ON]

**Q : Santé des vendeurs ?**

| Vendeur | Situation |
|---|---|
| BrainChip | S1 2026 : 1,22 M$ de revenus, 12,0 M$ de perte |
| SynSense | Dernier tour connu : 10 M$ (2023) |
| Innatera | 21 M$ de série A (2024) |
| GrAI Matter Labs | Absorbée par Snap (2023) |

Sources : [EXTRAIT]

**Q : Intel remplace-t-il Lava ?**
- SDK successeur annoncé, « open-standard AI frameworks », non publié mi-2026. [EXTRAIT intel.com ; quantaracore.in]
- « Loihi 3 » non confirmé officiellement.

## C7. Benchmarks (NeuroBench)

**Q : Que mesure le harnais ?**
- Footprint, sparsité des connexions et des activations, mises à jour de membrane, SynOps et MACs/ACs effectifs.
- Aucune mesure d'énergie dans le code. [OUVERT, dépôt neurobench cloné]

**Q : Et la piste système ?**
- Méthodologies hétérogènes acceptées ; mesure uniforme « currently infeasible ». [EXTRAIT arXiv 2304.04640 / Nat. Commun. 2025]

**Q : Existe-t-il des estimateurs d'énergie avant déploiement ?**
- SANA-FE, modèle d'exécution Loihi 2 (arXiv 2601.10035) : académiques et mono-plateforme. [EXTRAIT]
- Projet « Energy comparisons » dans la feuille de route NIR, en cours. [OUVERT roadmap.md]

**Q : Les gains des SNN sont-ils mesurés ?**
- Le plus souvent calculés à partir de coûts « Horowitz 45 nm », sans mémoire ni mouvements de données. [EXTRAIT arXiv 2409.08290]

**Q : Comparaison avec MLPerf Tiny ?**
- **Non couverte** (budget).

## C8. Mémoires RRAM/PCM/FeFET et calcul en mémoire analogique (Weebit, TSMC, Mythic, EnCharge, Sagence, IBM)

**Q : Qui licencie la ReRAM et pour quoi faire ?**
- Weebit : onsemi (janvier 2025), TI (décembre 2025), DB HiTek (qualification JEDEC), SkyWater.
- Chiffre d'affaires FY25 : 4,4 M$.
- Usage : eNVM binaire, pas synapse. [EXTRAIT]

**Q : Où en est TSMC ?**
- RRAM 40/28/22 nm en volume, 22ULL Grade-1 automobile en 2025, 12 nm en qualification, 6 nm en développement. [EXTRAIT rapport annuel TSMC 2025]
- Infineon AURIX TC4x en RRAM 28 nm. [EXTRAIT infineon.com]

**Q : Comment la dérive PCM est-elle modélisée ?**
- Exposant ν aléatoire par dispositif (μ et σ dépendant de la conductance), `t_0 = 20 s`, bruit de programmation polynomial, bruit de lecture 1/f. [OUVERT aihwkit pcm.py]
- Calcul : pour ν = 0,05, G perd environ 34 % en un jour.

**Q : Comment HERMES tient-il sa précision ?**
- 14 nm, 64 cœurs, 4 PCM par poids, compensation globale de dérive et calibration d'ADC sur puce. La stabilité à long terme reste un sujet de recherche. [EXTRAIT nature.com s41928-023-01010-1]

**Q : Relaxation RRAM ?**
- Dominante dans la première seconde, elle borne la précision au-delà du write-verify. [EXTRAIT IEEE 10354009 ; arXiv 2301.08516]

**Q : Qui commercialise le calcul analogique ?**
- Mythic (flash, 125 M$ en décembre 2025).
- EnCharge (capacités, 100 M$ en février 2025, EN100 à plus de 200 TOPS pour 8,25 W).
- Sagence (flash, 58 M$).
- Aucun sur RRAM ou PCM. [EXTRAIT]

**Q : FeFET et ECRAM ?**
- FeFET : endurance et rétention sous cyclage sont les verrous ; FMC lève 100 M€ pour du DRAM+ et du cache.
- ECRAM : stade académique. [EXTRAIT]

## C9. Caractérisation, variabilité, dérive, modèles compacts, entraînement conscient du matériel

**Q : Quel instrument pour les impulsions ?**
- B1530A WGFMU : 100 ns à 10 s, 200 MSa/s, 2 048 vecteurs et 512 formes d'onde maximum. [EXTRAIT Keysight 5990-4567]
- Keithley 4225-PMU : 10 ns. [EXTRAIT]

**Q : Coût ?**
- 4200A-SCS nu : 55 000 $. [EXTRAIT]
- B1500 et IC-CAP : sur devis, **non trouvés**.

**Q : Automatisation ?**
- Scripts maison : DLL 32 bits, synchrones. [OUVERT PythonMeasurementApp]
- ArC TWO : 64 SMU, interface open source. [EXTRAIT]
- Plateforme universelle NVM encore en « Work-in-Progress ». [EXTRAIT arXiv 2308.02400]

**Q : Bases de données ?**
- Première base relationnelle de 6 190 memristors en septembre 2026. [EXTRAIT arXiv 2609.01500]

**Q : Modèles compacts et extraction ?**
- JART VCM (Verilog-A, lacunes d'oxygène) et variantes à variabilité (Synaptogen). [EXTRAIT emrl.de ; arXiv 2404.06344]
- Extraction manuelle ou CNN académique. [EXTRAIT arXiv 2511.07926]
- ML Toolkit de Keysight hors memristors. [EXTRAIT keysight.com]

**Q : Simulateurs de précision ?**
- aihwkit : v1.0 (2025-05, MIT, ReRAM HfOx), v1.1 (2026-02). [OUVERT]
- CrossSim 3.2 (2026-04). [EXTRAIT]
- NeuroSim V2.1 (~12 h par entraînement) et V1.5 (2025). [OUVERT/EXTRAIT]
- MemTorch (190 étoiles). [OUVERT]

**Q : Lien entre physique et précision ?**
- Encore de la recherche : isolation des « distorsions induites par le matériel » (arXiv 2605.09416) ; recalibration de type LoRA/DoRA (arXiv 2504.03763). [EXTRAIT]

## C10. Reservoir computing physique et analogique

**Q : Qui a du silicium ?**
- TDK : prototype analogique à apprentissage en temps réel (2025-10-02, CEATEC). [EXTRAIT tdk.com]
- Polyn : premier silicium NASP (octobre 2025), tape-out VibroSense (2026-04-29). [EXTRAIT edn.com ; businesswire]
- Aspinity : AML100 à moins de 20 µA. [EXTRAIT businesswire 20240327]

**Q : Où ça casse ?**
- Transfert de la couche de sortie d'un exemplaire à l'autre. [EXTRAIT arXiv 2607.02608]
- Température et variabilité. [EXTRAIT researchgate 391782672]
- Couche de sortie qui doit suivre le débit du substrat. [EXTRAIT]

**Q : Point de départ déjà gratuit ?**
- Machine Learning Core de ST : ~13 µA. [EXTRAIT AN5259]

**Q : Le « micro intelligent » est-il un marché ?**
- Syntiant : l'IA ne représente que 5,1 % du revenu trimestriel. [EXTRAIT S-1 2026]

**Q : Existe-t-il des outils multi-substrats ?**
- Aucun trouvé. [EXTRAIT]

## C11. Biomonitoring portable ultra-basse consommation (ECG, EEG, EMG, PPG)

**Q : Budget énergétique ?**
- Front-end 85 µW (MAX30001).
- ADS1298 : 0,75 mW par canal.
- Apollo4 : 4-5 µA/MHz.
- Radio BLE dominante (BioGAP : 2,2 µJ par échantillon avec calcul sur capteur contre 3,6 µJ en streaming).
- [EXTRAIT analog.com ; ti.com ; ambiq.com ; arXiv 2307.01619]

**Q : Autonomie et qualité des patchs ?**
- Zio : port moyen de 10,4 jours, 96,4 % du temps analysable. [EXTRAIT tandfonline 10.1080/03007995.2019.1610370]

**Q : Où la télémétrie casse-t-elle ?**
- Plafond d'événements du Zio AT (lettre FDA du 2023-05-25). [EXTRAIT]

**Q : Réglementation de l'adaptatif ?**
- PCCP final (2024-12-04).
- Organismes notifiés européens : l'apprentissage continu n'est « généralement pas certifiable ».
- Échéance AI Act pour les dispositifs médicaux au 2028-08-02. [EXTRAIT]

**Q : Volume d'IA cardiaque autorisée ?**
- 116 dispositifs cardiovasculaires sur 1 247 dispositifs IA (liste FDA, 12/2025). [EXTRAIT intuitionlabs.ai]
- Bibliothèque HeartKey avec 510(k). [EXTRAIT]

**Q : EEG, EMG, PPG ?**
- Couverts seulement via les AFE (MAX86178 et AFE4960 non détaillés) : **couverture partielle**. BioButton et imec non couverts (budget).

## C12. Robotique et drones à perception événementielle

**Q : Que prouve le drone neuromorphique ?**
- TU Delft (Science Robotics, 2024-05-15) : SNN à 200 Hz sur Loihi, 7 à 12 mW pour le réseau contre 0,94 W au repos pour la plateforme. [EXTRAIT]

**Q : Quelle horloge dans ROS 2 ?**
- Horloge de l'hôte à l'arrivée du premier paquet ; broches de synchronisation et de trigger partagées ; `erc_rate` à 100 Mev/s. [OUVERT metavision_driver]

**Q : Coût calcul d'un flux saturé ?**
- 22 à 59 % de CPU pour le pilote, 80 à 90 % avec enregistrement. [OUVERT]

**Q : Calibration ?**
- Kalibr sans événements. [OUVERT]
- e2calib figé sur CUDA 10.1. [OUVERT]
- Calibration LiDAR ↔ événements via des dépôts de 1 à 16 étoiles. [OUVERT]
- Main Street Autonomy sans événements listés. [EXTRAIT]

**Q : Robustesse terrain ?**
- Scintillement des LED, bruit sous ~10 lux, biais réglés à la main. [EXTRAIT arXiv 2205.08090, 2405.19718 ; OUVERT]

**Q : Nombre d'OEM de robots agricoles utilisant des caméras à événements ?**
- **Non trouvé**.

## C13. Conception de dispositifs assistée par ML (micromagnétique, TCAD) — sujets B et E

**Q : Les grands éditeurs TCAD ont-ils du ML ?**
- Silvaco FTCO (Victory Analytics et DoE, surrogats). [EXTRAIT silvaco.com/solutions/ftco]
- Synopsys Sentaurus Calibration Workbench (surrogats, recherche inverse). [EXTRAIT semiengineering.com]

**Q : Les codes micromagnétiques sont-ils différentiables ?**
- MuMax3 et OOMMF : non.
- magnum.np et NeuralMag : oui, mais mumax3 reste plus rapide. [EXTRAIT nature.com s41524-025-01688-1]
- magnum.np distribué : 7× sur 8 GPU. [EXTRAIT arXiv 2606.01114]

**Q : Design inverse magnonique ?**
- Recherche binaire directe (Wang, Chumak et Pirro, Nat. Commun. 2021). [EXTRAIT s41467-021-22897-4]
- Dispositif universel (Nature Electronics 2024). [EXTRAIT s41928-024-01333-7]
- Perspectives « AI magnonics » (arXiv 2607.07324). [EXTRAIT]

**Q : Fabrication magnonique ?**
- YIG sur GGG à 700–850 °C, champ de polarisation, amplification. [EXTRAIT PMC10533023]
- Puce magnonique intégrée en cascade (arXiv 2601.02644, janvier 2026). [EXTRAIT]

**Q : Bruit thermique, densité et énergie des primitives magnoniques ?**
- Non chiffrés dans les sources consultées : **couverture partielle**.

**Q : Usage industriel de l'ordre fractionnaire (sujet E) ?**
- Oui en bio-impédance (Cole, ImpediMed SFB7) et en batteries. Non pour les neurones, les bond graphs ou le port-Hamiltonien. [EXTRAIT PMC12845819 ; J. Electrochem. Soc.]

**Q : Identifiabilité des modèles à CPE ?**
- Études existantes (arXiv 2103.00226, 1511.01402). [EXTRAIT]

## C14. Vision par SNN (sujet F) et compression orientée tâche (sujet H)

**Q : Les SNN rivalisent-ils en détection ?**
- SpikeYOLO : 67,2 % mAP@50 sur Gen1. [EXTRAIT arXiv 2407.20708]
- RVT (ANN) 0,472 contre EAS-SNN 0,437 en mAP50:95. [EXTRAIT arXiv 2403.12574]
- DASNN-MTF (CVPR 2026) : 43,4 %. [EXTRAIT openaccess.thecvf.com]

**Q : Sur quoi s'entraînent-ils ?**
- DSEC, Gen1/1Mpx, DVS128 Gesture, N-MNIST via Tonic. [OUVERT Tonic]

**Q : Peut-on classifier dans le domaine compressé ?**
- Travaux du groupe de Lisbonne (IEEE Access 2025). [OUVERT liste uzh-rpg]

---

## Limites déclarées de ce rapport

**Sources non lues directement (domaines bloqués).**
- Chiffres du 10-K et du 10-Q d'iRhythm, tableau publié de l'article NIR (seuls les `.npy` du dépôt ont été lus), prix des kits EVK4, Speck et Xylo, prix IC-CAP et B1500, contenu d'arXiv 2510.18668.

**Non vérifiés.**
- Rachat de SandBox Semiconductor par Lam ; statut d'Adesto, General Vision, Polyn (finances) et Aspirity.
- Comparaison avec MLPerf Tiny ; nombre d'OEM robotiques utilisant des caméras à événements ; licence de samna.
- « Loihi 3 », non confirmé officiellement.

**Dates divergentes.** Rachat de GrAI Matter Labs par Snap : 15/09/2023 ou 31/10/2023 selon les sources.

**Aucune taille de marché** des six idées n'a pu être démontrée par des chiffres entièrement sourcés. Toutes portent donc « MARCHÉ À VALIDER ». Les facteurs non sourcés sont étiquetés dans chaque calcul.

STATUT: TERMINÉ
