# Domaine : BASES DE DONNÉES — idées de startups tirées de l'analyse des mécanismes

Date : 2026-09-25. Méthode : 5 recherches parallèles par groupe de sous-couches, puis vérifications complémentaires. Budget consommé : **167 WebSearch** (30 + 32 + 39 + 31 + 30 + 5). Pages lues directement : fichiers bruts GitHub (specs Iceberg/Delta, doc SGML et code source PostgreSQL, pgvector, hnswlib, PgBouncer, serveurs MCP Supabase/Neon/référence, Bytebase), et un Postgres 16 jetable pour reproduire certains comportements.

Conventions :
- **[OUVERT]** = page ou fichier lu en entier ; **[EXTRAIT]** = connu seulement par un extrait de résultat de recherche ; **[TEST LOCAL]** = reproduit sur un PostgreSQL 16.13 jetable dans le bac à sable (rien n'est conservé).
- **HYPOTHÈSE non sourcée** = estimation sans source.
- Aucun entretien ni contact avec des acheteurs n'est proposé.
- De nombreux domaines étaient bloqués en lecture (docs AWS, edpb.europa.eu, cnil.fr, ico.org.uk, arxiv.org, ycombinator.com, supabase.com, neon.com). Leurs contenus sont donc cités en [EXTRAIT].

**Résumé :** 6 idées sont livrées en fiches, aucune ne passe les 8 critères sans réserve.

| # | Idée | Statut |
|---|---|---|
| 1 | Preuve d'effacement vérifiable (garde-fou de restauration + scanner des copies résiduelles + reçu signé) | Candidate la plus solide. Critère 3 À TRANCHER, MARCHÉ À VALIDER |
| 2 | « Data PR » : merge des écritures d'agents IA d'une branche vers un Postgres standard, avec détection de conflits | À TRANCHER, MARCHÉ À VALIDER |
| 3 | Copilote de montée de version vérifiée pour flottes RDS/Aurora/Cloud SQL (Postgres + MySQL) | À TRANCHER, MARCHÉ À VALIDER |
| 4 | Passerelle qui applique côté serveur les `read-restrictions` Iceberg pour les clients non fiables | À TRANCHER, MARCHÉ À VALIDER |
| 5 | Surveillance du rappel réel des index vectoriels, avec correction automatique | À TRANCHER, MARCHÉ À VALIDER |
| 6 | Garde-fou qui juge le SQL d'agents sur l'effet mesuré et non sur la syntaxe | À TRANCHER, tendance défavorable |

---

## A. Fiches des idées

### Idée 1 — « Erasure Proof » : garde-fou de restauration, scanner des copies résiduelles et reçu signé (Postgres, Iceberg, Delta, index HNSW)

**Question technique d'origine.** Quand une ligne est supprimée pour une demande d'effacement, combien de copies physiques de la personne restent dans le système ? Et que se passe-t-il au premier restore ?

**Problématique (le mécanisme) :**

1. **Postgres.**
   - Un DELETE laisse une ligne morte jusqu'au VACUUM.
   - La compaction de page (`PageRepairFragmentation`/`compactify_tuples`) déplace les lignes sans remettre à zéro l'espace libéré. Le seul `memset` est sous `#ifdef CLOBBER_FREED_MEMORY`, une option de débogage (bufpage.c l. 894-896) [OUVERT] https://raw.githubusercontent.com/postgres/postgres/master/src/backend/storage/page/bufpage.c
   - Avec `full_page_writes`, le WAL contient « the entire content of each disk page … during the first modification of that page after a checkpoint » [OUVERT] https://raw.githubusercontent.com/postgres/postgres/master/doc/src/sgml/config.sgml. Les WAL archivés (pgBackRest, WAL-G, Barman) portent donc des images de pages complètes, voisins compris.
   - Toute restauration PITR à un instant antérieur au DELETE fait revenir la personne.
2. **Iceberg** (spec [OUVERT] https://raw.githubusercontent.com/apache/iceberg/main/format/spec.md) :
   - un equality delete file **stocke la valeur de la clé effacée** ;
   - « For delete files, the metrics describe the values that were deleted » : les bornes `lower_bounds`/`upper_bounds` du manifest contiennent donc l'identifiant effacé. Pour un fichier qui n'efface qu'une personne, borne basse = borne haute. Ces bornes sont tronquées à 16 caractères par défaut [EXTRAIT] https://iceberg.apache.org/docs/nightly/configuration/ ;
   - un position delete file peut porter la colonne `row` ;
   - « Writers are not required to rewrite Puffin files that contain the removed deletion vectors ».
3. **Delta** (PROTOCOL.md [OUVERT] https://raw.githubusercontent.com/delta-io/delta/master/PROTOCOL.md) :
   - les `stats` minValues/maxValues des actions `add`/`remove` restent dans `_delta_log` et les checkpoints après VACUUM ;
   - le Change Data Feed écrit les lignes modifiées ou supprimées dans `_change_data/` ;
   - avec les deletion vectors, il faut `REORG TABLE … APPLY (PURGE)` [EXTRAIT] https://docs.databricks.com/aws/en/security/privacy/gdpr-delta
4. **HNSW.**
   - hnswlib (utilisé par Chroma) : `markDeletedInternal` pose **un seul bit** (`*ll_cur |= DELETE_MARK`, l. 1579-1590). Le vecteur reste en place et `unmarkDelete` existe [OUVERT] https://raw.githubusercontent.com/nmslib/hnswlib/master/hnswlib/hnswalg.h
   - pgvector, à l'inverse, remet le vecteur à zéro au VACUUM (hnswvacuum.c l. 686-688) [OUVERT] https://raw.githubusercontent.com/pgvector/pgvector/master/src/hnswvacuum.c. Chez pgvector, le « fantôme » n'existe donc qu'avant VACUUM, et dans le WAL et les sauvegardes.
   - Les articles de la passe précédente **existent bien** :
     - « Ghost Vectors » (arXiv 2606.18497, juin 2026) : 25-46 % des PII récupérées depuis des embeddings soft-deleted [EXTRAIT] https://arxiv.org/abs/2606.18497 ;
     - « Ghost Echoes » (arXiv 2608.20352, août 2026) : dérive sémantique persistante sur ChromaDB, même après reconstruction [EXTRAIT] https://arxiv.org/abs/2608.20352
5. **Ce que demandent les régulateurs.**
   - EDPB, rapport CEF 2025 sur l'effacement, adopté le **10/02/2026**. La date du 18/02 de la passe précédente n'est pas confirmée ; c'est probablement celle de la publication. Ce rapport cite les sauvegardes parmi les défauts les plus fréquents (764 responsables contrôlés) et attend « a deletion index so restored records are re-deleted » [EXTRAIT] https://www.edpb.europa.eu/news/news/2026/edpb-identifies-challenges-hindering-full-implementation-right-erasure_en ; https://www.probackup.io/blog/gdpr-and-backups-how-to-handle-deletion-requests
   - ICO : sauvegarde « beyond use » jusqu'à son écrasement [EXTRAIT] https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/individual-rights/individual-rights/right-to-erasure/

→ **« La spec fait X, donc Y casse quand Z ».** Les formats conservent par construction la valeur effacée (delete files, bornes, journal, bit de suppression). La conformité repose donc sur des tâches de maintenance (VACUUM, expire_snapshots, rewrite manifests, REORG PURGE, reindex) et sur la discipline de restauration. Cela casse quand une maintenance n'a pas tourné, quand la rétention de sauvegarde est longue, ou dès qu'une restauration a lieu. Aujourd'hui, la réapplication se fait par script maison [EXTRAIT] https://dev.to/authagonal/your-backup-quietly-resurrects-the-users-you-deleted-g8n ; https://forums.veeam.com/veeam-backup-replication-f2/general-data-protection-regulation-gdpr-t40950.html

**Solution.**
- (a) **Journal d'effacement minimisé** : empreintes HMAC des identifiants, jamais les identifiants en clair.
- (b) **Garde-fou de restauration** : un hook post-restauration (validation AWS Backup restore testing, pgBackRest/WAL-G, clone RDS) rejoue le journal **avant** que la base restaurée accepte du trafic. Il vérifie aussi l'intégrité (comptages, contraintes).
- (c) **Scanner de résidus** en lecture seule. Il lit directement les formats ouverts : manifests, delete files et Puffin Iceberg ; `_delta_log` et `_change_data` Delta ; pages et WAL Postgres ; fichiers d'index hnswlib. Il cherche les empreintes et produit la liste des copies restantes avec leur date d'expiration prévue.
- (d) **Reçu signé par demande**, qui distingue « effacé physiquement » de « hors d'usage jusqu'au JJ/MM » (vocabulaire ICO).

**Comment la solution répond au problème.**
- Le garde-fou (b) traite l'attente commune à l'EDPB et à l'ICO : réappliquer à la restauration.
- Le scanner (c) transforme un effacement *supposé* (on a lancé DELETE) en effacement *vérifié* copie par copie.
- Le reçu (d) fournit la preuve demandée lors d'un contrôle, et pour DROP en Californie.

**Concurrents.**

| Acteur | Ce qu'il fait | Ce qui le distingue de l'idée |
|---|---|---|
| Transcend, DataGrail, Ketch, OneTrust | Orchestration logique des demandes (appels d'API aux systèmes) | Pas d'inspection des copies physiques |
| Deletable | Crypto-effacement par personne + certificat, « including in backups » [EXTRAIT] https://www.deletable.org/ | Exige d'avoir chiffré en amont ; ne traite pas l'existant |
| Ryft | Iceberg seulement [EXTRAIT] https://www.ryft.io/blog/gdpr-compliance-with-apache-iceberg-a-practical-guide ; **racheté par Cyera le 2026-04-23** (100-130 M$) [EXTRAIT] https://datalakehousehub.com/blog/table-maintenance-economics/ | Un seul format. Risque : Cyera peut étendre l'effacement dans son DSPM |
| Salesforce Own (ex-OwnBackup) | Recherche et anonymisation dans les sauvegardes, Salesforce seulement | Une seule plateforme |
| ProBackup | Purge sur ticket dans ses sauvegardes d'applications SaaS (ClickUp, Airtable…) + certificat de suppression [EXTRAIT] https://www.probackup.io/gdpr | Pas de bases de données autogérées |
| Mixpeek | Guide sur les vecteurs, pas de produit identifié | — |

- Annuaire YC : rien sur l'effacement physique multi-format (Inth fait le consentement et les demandes ; JumpWire a été racheté par WorkOS).

**Qui paie.**
- Responsable de la protection des données (DPO) et RSSI des entreprises qui ont des lakehouses ou des bases avec PITR.
- Data brokers soumis à DROP.
- Éditeurs SaaS B2B qui doivent prouver l'effacement à leurs clients.

**Revenu potentiel (bottom-up) : MARCHÉ À VALIDER.**
- Facteur sourcé : **654 data brokers** inscrits au registre californien [EXTRAIT] https://privacy.ca.gov/drop-for-data-brokers/
- Exposition sourcée : amende de 200 $ par jour et par demande non traitée ; plus de 2 000 demandes par mois en moyenne pour un broker [EXTRAIT] https://privacyrights.org/resources-tools/advocacy/deletion-obligations-under-drop-are-here-data-brokers-must-now-delete ; https://www.helpnetsecurity.com/2026/06/01/datagrail-ai-privacy-risks-report/
- Exemple d'exposition : 1 % de demandes ratées = 20 × 200 $ = 4 000 $ par jour et par broker. C'est un coût du problème, pas un prix.
- Prix de vente : aucune grille publique trouvée pour un outil voisin. Un ACV de 30 k$ serait une **HYPOTHÈSE non sourcée**. Avec cette hypothèse, le segment brokers seul vaudrait 654 × 30 k$ ≈ 19,6 M$ par an : trop petit seul.
- Le marché UE (responsables de traitement avec PITR ou lakehouse) n'est pas dénombré par une source trouvée. Les 764 organisations de l'EDPB sont un échantillon, pas un marché.

**Les 8 critères**

1. **Vrai problème : OUI.**
   - Mécanismes lus dans les specs et le code (voir plus haut).
   - L'EDPB cite les sauvegardes parmi les défauts récurrents [EXTRAIT, lien ci-dessus].
   - Des équipes écrivent des scripts de réapplication à la main [EXTRAIT, dev.to et Veeam ci-dessus].
2. **Pas un piège à goudron : pas de fermeture trouvée sur l'angle exact.**
   - Recherches : « privacy engineering startup shut down pivot », Ethyca (actif [EXTRAIT] https://www.ethyca.com/news), JumpWire et Cyral (rachetés, pas fermés).
   - Risque identifié : consolidation par les géants DSPM (Cyera a racheté Ryft).
3. **Problème aigu : À TRANCHER.**
   - Pour : DROP est récurrent (45 jours) avec 200 $ par jour et par demande, et l'EDPB mène des enquêtes formelles.
   - Contre : l'ICO accepte le « beyond use » ; la CNIL n'exigerait pas l'effacement des sauvegardes, d'après une source secondaire [EXTRAIT] https://verasafe.com/blog/do-i-need-to-erase-personal-data-from-backup-systems-under-the-gdpr/, non vérifiée sur cnil.fr (bloqué).
   - Ce qui manque : le texte exact du PDF EDPB (bloqué) et la doc CNIL officielle.
   - Conséquence : la brique « garde-fou de restauration + reçu » est aiguë partout. La brique « scanner physique » ne l'est que sous autorité stricte (Danemark, « where technically possible ») ou pour les embeddings.
4. **Marché : MARCHÉ À VALIDER** (calcul ci-dessus).
5. **Concurrence : angle unique trouvé.** Vérification *a posteriori*, sans réarchitecture, des copies laissées par plusieurs formats, avec rejeu à la restauration et reçu. Deletable fait de la prévention ; Ryft, Own et ProBackup couvrent un seul format ou une seule plateforme.
6. **Pourquoi maintenant.**
   - Rapport EDPB adopté le 10/02/2026.
   - Obligations DROP depuis le **01/08/2026**, récurrentes tous les 45 jours [EXTRAIT, privacy.ca.gov].
   - Plus de 500 000 Californiens inscrits en août 2026 [EXTRAIT] https://privacy.ca.gov/2026/08/half-a-million-californians-have-signed-up-for-drop-to-delete-their-personal-information-from-data-brokers/
   - Ghost Vectors (06/2026) et Ghost Echoes (08/2026).
   - Deletion vectors dans Iceberg v3.
7. **Proxy (entreprise comparable).**
   - Own (sauvegarde + effacement RGPD sur Salesforce), racheté 1,9 Md$ par Salesforce [EXTRAIT] https://www.cnbc.com/2024/09/05/salesforce-to-acquire-own-for-1point9-billion-in-cash.html
   - Ryft, racheté par Cyera en 2026 [EXTRAIT].
   - Transcend, environ 90 M$ levés [EXTRAIT] https://www.securityweek.com/transcend-raises-40-million-for-data-privacy-platform/
8. **Scalable : OUI.**
   - Iceberg et Delta sont des specs publiques, donc des lecteurs réutilisables d'un client à l'autre ; l'agent est en lecture seule.
   - Coût marginal : un connecteur par moteur propriétaire (HYPOTHÈSE non sourcée sur l'effort).

**Formulations du balayage de concurrence (16).**
1. « "Ghost Vectors" HNSW soft delete »
2. « "Ghost Echoes" vector database deletion »
3. « Ryft Iceberg GDPR deletion right to be forgotten »
4. « OwnBackup GDPR forget backups Salesforce »
5. « startup "proof of deletion" database backups GDPR erasure certificate verifiable deletion 2026 »
6. « Deletable crypto-shredding »
7. site:ycombinator.com/companies « data deletion GDPR erasure »
8. site:ycombinator.com/companies « privacy compliance data subject requests database »
9. « Product Hunt OR "Show HN" GDPR delete user data across databases backups vector store »
10. « "right to be forgotten" restore backup "re-apply" deletions tombstone Postgres »
11. « Mixpeek Pinecone Qdrant Weaviate delete physically compaction GDPR »
12. « crypto-shredding per-user keys Postgres "Show HN" OR startup »
13. « "suppression list" OR "erasure log" reapply deletions Rubrik Veeam Commvault »
14. « Iceberg write.metadata.metrics PII leak »
15. « Transcend DataGrail deletion backups time travel »
16. « "restore" re-apply GDPR deletions after backup restore tool "erasure" ledger startup 2026 » et « ProBackup GDPR deletion … suppression list »

Recherches GitHub (dépôts et tickets apache/iceberg, delta-io/delta) : aucun outil. Le ticket Delta #5661 « Log deleted file paths during VACUUM » montre un besoin de preuve.

**Porte d'entrée conseillée :** le garde-fou de restauration + le reçu. C'est l'attente commune de l'EDPB et de l'ICO, et elle s'accroche aux tests de restauration DORA (voir B).

---

### Idée 2 — « Data PR » : les agents écrivent sur une branche, un changeset ligne à ligne est revu puis appliqué sur Postgres standard avec détection de conflits

**Question technique d'origine.** Quand un agent IA corrige des données sur une branche Neon, Xata ou Supabase, comment l'effet validé arrive-t-il en production, et comment l'annuler seul ?

**Problématique (le mécanisme) :**
- Neon MCP : `handleCommitMigration` **rejoue le texte SQL** sur la branche parente (`handleRunSqlTransaction({ sqlStatements: splitSqlStatements(migrationSql), branchId: parentBranchId })`) puis supprime la branche temporaire [OUVERT] https://github.com/neondatabase/mcp-server-neon (mcp/tools/tools.ts).
- Pour du DML, le rejeu touche les lignes présentes *au moment du rejeu*, pas celles qui ont été validées sur la branche.
- La restauration de branche Neon est « a complete overwrite of the database timeline, not a merge or refresh » [OUVERT] https://github.com/neondatabase/website (content/docs/postgres/backup-restore/branch-restore.md). Annuler une erreur d'agent efface aussi les écritures légitimes concurrentes.
- « No platform currently offers automatic data merging back to production » [EXTRAIT] https://xata.io/blog/neon-vs-supabase-vs-xata-postgres-branching-part-1. Recherche du 25/09/2026 : toujours aucune annonce de merge de données chez Neon, Xata ou Supabase [EXTRAIT] même source et https://neon.com/faqs/postgres-platforms-database-branching-git
- Incident Replit / SaaStr (07/2025) : base de production supprimée par un agent [EXTRAIT] https://www.theregister.com/2025/07/21/replit_saastr_vibe_coding_incident/. La réponse de Replit a été une restauration *globale* en un clic [EXTRAIT] https://www.theregister.com/2025/07/22/replit_saastr_response/

**Solution.**
1. L'agent écrit toujours sur une branche (Neon/Xata) ou un clone.
2. Le décodage logique (pgoutput/wal2json) extrait le **changeset ligne à ligne** depuis le LSN de fork, avec images avant et après.
3. Un diff lisible façon PR est présenté, avec l'effet réel par table, cascades comprises.
4. L'application en prod se fait **en concurrence optimiste** : une ligne n'est appliquée que si elle est encore égale à son image « avant », sinon un conflit est signalé.
5. Le journal du changeset permet un revert ciblé.

**Comment la solution répond au problème.** On remplace « rejouer du texte SQL » par « appliquer un ensemble de lignes vérifié », et « écraser la timeline » par « annuler un seul changeset ». On reste sur Postgres standard.

**Concurrents.**

| Acteur | Ce qu'il fait | Ce qui le distingue de l'idée |
|---|---|---|
| Dolt/Doltgres | Branche et merge de données, « The Database for AI Agents » [EXTRAIT] https://www.dolthub.com/blog/2026-05-18-database-for-ai-video/ | Impose de changer de moteur |
| eterDB | **Fork** de PostgreSQL 18 : « drop-in replacement… surgical, dependency-aware undo » ; annulation *après coup* de transactions sur la base vivante, sidecars d'historique et de snapshots [EXTRAIT] https://github.com/eterdb/eterdb ; https://eterdb.com/tech ; Show HN https://news.ycombinator.com/item?id=49645654 | Undo après coup, pas revue avant merge ; impose un fork |
| Bytebase | Approbation du DML + sauvegarde préalable des lignes touchées + rollback généré [EXTRAIT] https://docs.bytebase.com/change-database/rollback-data-changes | Pas de branche ni de merge |
| Rubrik Agent Rewind (08/2025) | Retour arrière depuis des snapshots [EXTRAIT] https://www.rubrik.com/company/newsroom/press-releases/25/rubrik-unveils-agent-rewind-for-when-ai-agents-go-awry | Orienté grands comptes, restauration par snapshot |
| Neon, Xata, Supabase | Branches | Pas de merge de données |
| pavliha/grove (GitHub, 0 étoile) | « branch, diff, merge and blame your rows, in stock PostgreSQL » | Même idée, projet amateur |

**Qui paie.** Les équipes produit et plateforme qui donnent l'écriture à des agents (agents de support, d'opérations, de coding) sur une base de production, et les plateformes d'agents elles-mêmes.

**Revenu potentiel : MARCHÉ À VALIDER.**
- Facteurs sourcés :
  - plus de 80 % des bases Neon créées par des agents [EXTRAIT] https://www.infoworld.com/article/3985947/databricks-to-acquire-open-source-database-startup-neon-to-build-the-next-wave-of-ai-agents.html ;
  - plus de 60 % des nouvelles bases Supabase lancées par des outils IA [EXTRAIT] https://www.cnbc.com/2026/06/04/database-startup-supabase-raises-500-million-10point5-billion-valuation.html ;
  - prix voisin : StrongDM à 70 $ par utilisateur et par mois [EXTRAIT] https://www.strongdm.com/pricing
- Il manque le nombre d'organisations qui laissent des agents écrire sur une base de production : pas de calcul possible sans inventer.

**Les 8 critères**

1. **Vrai problème : OUI.** Mécanismes lus dans le code et la doc Neon, incident Replit, constat de Xata.
2. **Pas un piège à goudron : pas de fermeture trouvée sur l'angle.** Signal voisin négatif : Cyral, proxy d'accès aux bases, vendu à Varonis pour environ 25 M$ [EXTRAIT] https://pitchbook.com/profiles/company/399511-81. C'est une autre catégorie.
3. **Problème aigu : partiel.** Très aigu après un incident. Sinon, c'est une condition pour autoriser des agents en écriture (HYPOTHÈSE non sourcée sur la fréquence).
4. **Marché : MARCHÉ À VALIDER.**
5. **Concurrence : angle unique probable.** Merge de données avec conflits sur Postgres standard, adossé aux branches existantes.
   - eterDB (fork, undo après coup) et Dolt (autre moteur) ne l'ont pas. Vérifié pour eterDB le 25/09/2026 [EXTRAIT, GitHub eterdb].
   - **À TRANCHER :** la feuille de route Databricks Lakebase (ex-Neon), non vérifiée.
6. **Pourquoi maintenant.**
   - Neon passe à plus de 80 % de bases créées par des agents (05/2025) et Supabase à plus de 60 % (06/2026).
   - Premiers incidents d'écriture en production (07/2025).
   - Le branching copy-on-write devient une commodité.
7. **Proxy.**
   - DoltHub (même proposition sur un moteur dédié) [EXTRAIT].
   - Deploy requests de PlanetScale, mais pour le schéma seulement.
   - Pas de revenu public : HYPOTHÈSE non sourcée sur leur succès commercial.
8. **Scalable : OUI** (plan de contrôle logiciel + décodage logique). Risque : intégration native par Databricks/Neon ou Supabase.

**Formulations du balayage (12).**
1. « Postgres branch merge data changes back to parent »
2. « agent writes to database branch then merge data changes back to production row-level conflict Doltgres Xata data merge »
3. « Doltgres agents branch merge data pull request for data AI agent writes review »
4. « Show HN undo AI agent database changes row-level revert Postgres transaction »
5. « Rubrik Agent Rewind undo AI agent actions database rollback »
6. site:ycombinator.com/companies « database branching agents merge data Postgres »
7. Product Hunt « AI agent production database safe writes preview changes approve merge sandbox »
8. GitHub « database branch merge rows »
9. GitHub « postgres merge branch data agents »
10. « Bytebase AI agent MCP data change approval prior backup rollback DML review »
11. « Neon MCP prepare_database_migration temporary branch »
12. « eterDB Postgres undo transaction agents how it works extension » et « Postgres branch "merge" data changes back to production agent review diff rows conflict 2026 Neon OR Xata OR Supabase announcement »

---

### Idée 3 — Copilote de montée de version vérifiée pour flottes RDS/Aurora/Cloud SQL (PostgreSQL et MySQL), façon « Chkk pour bases de données »

**Question technique d'origine.** Pourquoi la voie quasi sans coupure (réplication logique, Blue/Green) échoue-t-elle, alors que le cloud facture désormais le fait de ne pas monter de version ?

**Problématique (le mécanisme) :**
- La doc officielle de la réplication logique exclut [OUVERT] https://raw.githubusercontent.com/postgres/postgres/master/doc/src/sgml/logical-replication.sgml :
  - le DDL ;
  - les séquences (`REFRESH SEQUENCES` seulement si l'éditeur est en PG ≥ 19, donc jamais en partant de 13 à 18) ;
  - les large objects (« There is no workaround ») ;
  - les vues matérialisées ;
  - UPDATE et DELETE en `REPLICA IDENTITY FULL` sur les types sans opclass B-tree ou Hash.
- `pg_upgrade` ne transfère pas les statistiques d'extensions ni les statistiques cumulatives. Avec la réplication logique, la cible part sans statistiques [OUVERT] https://raw.githubusercontent.com/postgres/postgres/master/doc/src/sgml/ref/pgupgrade.sgml
- Blue/Green pour une version majeure hérite de ces trous : la bascule est bloquée en cas de DDL ou de large objects [EXTRAIT] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/blue-green-deployments-replication-type.html
- Côté MySQL 8.4, `mysql_native_password` est désactivé par défaut, puis supprimé en 9.0 [EXTRAIT] https://docs.percona.com/percona-server/8.4/8.4-breaking-changes.html
- L'inaction coûte : RDS Extended Support à 0,10 $/vCPU-h en années 1-2, 0,20 $ en année 3. Il s'applique à PG 13 depuis le 01/03/2026 et à MySQL 8.0 depuis le 01/08/2026 [EXTRAIT] https://www.vantage.sh/blog/amazon-rds-extended-support ; https://ondelva.com/blog/2026/07/rds-mysql-8-0-end-of-support-2026-extended-support-cost-upgrade-8-4

**Solution.**
1. Inventaire de la flotte, avec le coût d'Extended Support par instance.
2. Vérifications préalables par paire de versions : extensions, plugins d'authentification, large objects, types sans opclass, fonctions retirées.
3. **Rejeu d'un échantillon de charge réelle** (pg_stat_statements, digests du performance_schema) sur l'environnement vert, avec comparaison des plans et des latences.
4. Comblement des trous : gel du DDL par event triggers, resynchronisation des séquences, ANALYZE par étapes avant la bascule.
5. Bascule orchestrée, avec retour arrière.

**Comment la solution répond au problème.** Chaque trou de la spec devient un contrôle automatisé. La base de connaissances par paire de versions se réutilise d'un client à l'autre.

**Concurrents.**
- **AWS Blue/Green** : ne comble pas les trous et ne rejoue pas de charge.
- **Aurora Query Plan Management** : Aurora PG seulement.
- **pt-upgrade** : MySQL, outil en ligne de commande.
- **pg_easy_replicate** (Shopify) : orchestration seule.
- **CloudNativePG** : Kubernetes seulement.
- **Xata et pgEdge** : pour leurs propres bases hébergées.
- **CloudFix** : détecte sans corriger [EXTRAIT] https://cloudfix.com/blog/cloudfix-finder-fixer-rds-optimize-eol-version/
- **Pre-Upgrade Validation d'Azure** : Azure seulement.
- **PostgresAI**, « self-driving Postgres with zero downtime upgrades » [EXTRAIT] https://postgres.ai/blog/20250725-self-driving-postgres
- **Agents DBA IA** : DeepSQL, dba.ai, Xata Agent, EDB.
- **AWS Organizations Upgrade Rollout Policy** : orchestre les montées de version *mineures* par phases [EXTRAIT] https://aws.amazon.com/about-aws/whats-new/2026/04/amazon-aurora-postgresql-17-9-16-13-15-17-14-22

**Qui paie.** Équipes plateforme ou SRE et FinOps des organisations qui ont des dizaines d'instances managées.

**Revenu potentiel : MARCHÉ À VALIDER.**
- Valeur par client, avec un prix sourcé et une flotte en HYPOTHÈSE non sourcée : 20 instances × 8 vCPU = 160 vCPU × 0,10 $ × 8 760 h = **140 160 $ par an** d'Extended Support évitable (280 320 $ en année 3).
- Exemple publié : un db.r5.xlarge coûte 292 $ par mois d'Extended Support [EXTRAIT, ondelva].
- Nombre d'organisations en Extended Support : **aucune source trouvée.**

**Les 8 critères**

1. **Vrai problème : OUI.** Wiz a construit en interne un « DB Upgrade Pilot » [EXTRAIT] https://aws.amazon.com/blogs/database/how-wiz-achieved-near-zero-downtime-for-amazon-aurora-postgresql-major-version-upgrades-at-scale-using-aurora-blue-green-deployments/ ; d'autres ont écrit leur propre procédure (Medplum, InstantDB, Gadget) [EXTRAIT] https://www.medplum.com/blog/zero-downtime-postgres-major-version-upgrade ; https://www.instantdb.com/essays/pg_upgrade
2. **Pas un piège à goudron : plutôt oui, sous réserve.**
   - Pas de fermeture trouvée sur cet angle.
   - Voisins fermés ou abandonnés : OtterTune (tuning, fermé en 06/2024 [EXTRAIT] https://news.ycombinator.com/item?id=40690380), Snaplet, Reshape.
   - Risque « one-shot », atténué par la récurrence : une version majeure PG par an, et des fins de support en cascade.
3. **Problème aigu : OUI.** Coût récurrent, visible sur la facture, qui double en année 3.
4. **Marché : MARCHÉ À VALIDER.**
5. **Concurrence : À TRANCHER.** Il manque le périmètre exact de PostgresAI (RDS managé ? rejeu de charge ? MySQL ?). L'angle multi-moteurs, managé, avec rejeu de charge et comblement des trous n'est revendiqué par aucun autre acteur trouvé.
6. **Pourquoi maintenant.**
   - Facturation PG 13 depuis le 01/03/2026 et MySQL 8.0 depuis le 01/08/2026.
   - MySQL 9.7 LTS depuis le 21/04/2026 [EXTRAIT] https://endoflife.date/mysql
7. **Proxy.** Chkk, montées de version Kubernetes par plans pré-vérifiés, seed de 5,2 M$ menée par Sequoia [EXTRAIT] https://tfir.io/chkk-emerges-from-stealth-with-5-2m-in-seed-funding-to-enhance-kubernetes-resiliency/. C'est une preuve de financement, pas encore de réussite commerciale.
8. **Scalable : OUI si c'est un produit.** Risque de glisser vers le service ou le conseil, positionnement déjà occupé par Percona et Data Egret.

**Formulations du balayage (15).**
1. « zero-downtime Postgres major version upgrade tool automated service 2026 »
2. site:ycombinator.com/companies « postgres upgrade OR migration »
3. « "extended support" RDS Aurora upgrade automation startup fleet »
4. « Show HN Postgres major upgrade logical replication orchestrator »
5. « database upgrade compatibility testing traffic replay SaaS MySQL 8.4 PostgreSQL query plan regression »
6. « AI DBA agent startup YC 2025 2026 upgrades »
7. producthunt « database version upgrade tool RDS Extended Support avoid cost »
8. « CloudFix RDS Optimize EOL Version fixer »
9. « postgres.ai zero-downtime major upgrade service »
10. « Chkk RDS Aurora upgrade »
11. « pre-flight compatibility check shadow workload replay startup launch 2026 »
12. GitHub « postgres upgrade »
13. GitHub « pg_easy_replicate »
14. « postgres.ai major upgrade RDS Aurora zero downtime service automated upgrades 2026 »
15. « Pre-Upgrade Validation Checks Azure » (Microsoft Tech Community)

---

### Idée 4 — Passerelle qui applique côté serveur les `read-restrictions` Iceberg pour les clients non fiables (DuckDB, PyIceberg, notebooks, agents), indépendante du catalogue

**Question technique d'origine.** Dans un catalogue REST Iceberg, qui applique réellement un filtre de lignes ou un masque de colonne ?

**Problématique (le mécanisme) :**
- La spec REST (PR #13879, fusionnée le **2026-09-03**) ajoute `required-row-filter` et `required-column-projections` à `LoadTableResult`, avec 9 actions de masquage. Mais c'est **le lecteur** qui les applique : « A reader that supports read restrictions must fail any read… that cannot apply a returned restriction in full » [OUVERT] https://raw.githubusercontent.com/apache/iceberg/main/open-api/rest-catalog-open-api.yaml ; https://github.com/apache/iceberg/pull/13879
- L'en-tête de capacités `X-Iceberg-Client-Capabilities` est « not a security mechanism… clients can trivially spoof its value » [OUVERT] https://github.com/apache/iceberg/pull/16394
- Au niveau du stockage, la frontière applicable reste la table entière : « more granular data-permission enforcement needs a trusted boundary above that layer » [OUVERT] https://github.com/apache/iceberg/pull/15545
- Un client qui reçoit des vended credentials lit donc tout le Parquet brut.
- Seuls Databricks (Cross-Engine ABAC, en bêta, avec une « filtering fleet » côté serveur [EXTRAIT] https://www.databricks.com/blog/introducing-cross-engine-abac) et Google (compute delegation BigLake [EXTRAIT] https://codelabs.developers.google.com/governed-lakehouse-compute-delegation) filtrent côté serveur.

**Solution.** Un service qui implémente la planification de scan et la lecture REST. Il évalue les politiques (OPA, Cedar, Ranger, ODCS) et sert des données filtrées ou masquées (Arrow Flight, ou fichiers assainis temporaires) aux clients non fiables. Il ajoute une batterie de tests de conformité pour les « Trusted Iceberg Clients ».

**Comment la solution répond au problème.** La règle ne dépend plus de la bonne volonté du client : la frontière de confiance passe au-dessus du stockage, là où la PR #15545 dit qu'elle doit se trouver. Et cela vaut pour Glue, S3 Tables, Polaris ou Lakekeeper.

**Concurrents.**
- **Plateformes** : Databricks UC Cross-Engine ABAC (bêta, propre à UC) ; Google BigLake ; Snowflake/Polaris, qui promeuvent la spec mais dépendent de clients de confiance [EXTRAIT] https://www.snowflake.com/en/blog/engineering/apache-iceberg-read-restrictions-governance0/
- **Catalogues** : Lakekeeper (OpenFGA/Cedar) ; Apache Ranger.
- **Éditeurs de gouvernance** : Immuta, Privacera, Dremio, Starburst.
- **Rien sur YC ni Product Hunt.**

**Qui paie.** Les équipes données et sécurité des organisations qui ont Iceberg hors Databricks (Glue 39,3 %, S3 Tables 25 % des utilisateurs Iceberg interrogés [EXTRAIT] https://www.ryft.io/blog/the-state-of-apache-iceberg-in-the-enterprise-2026, 252 répondants) et qui veulent ouvrir des données réglementées à des agents ou à DuckDB.

**Revenu potentiel : MARCHÉ À VALIDER.** Nombre absolu d'organisations et ACV non sourcés. Borne indirecte : Immuta, 259,5 M$ levés pour un ARR estimé à 23,5 M$ (estimation getlatka, non vérifiée) [EXTRAIT] https://getlatka.com/companies/immuta.com

**Les 8 critères**

1. **Vrai problème : OUI.** La spec le nomme elle-même (PR #15545), et Databricks a construit un produit pour ce cas.
2. **Piège à goudron : signaux de risque.**
   - Okera (proxy d'accès, 29,6 M$ levés) racheté par Databricks en 2023 pour un montant non divulgué [EXTRAIT] https://techcrunch.com/2023/05/03/databricks-acquires-ai-centric-data-governance-platform-okera/
   - Le rapport revenus / capital d'Immuta est faible (estimation non vérifiée).
   - Pas de fermeture prouvée : **À TRANCHER.**
3. **Problème aigu : À TRANCHER.** Aucune source directe ne montre des agents ou des notebooks bloqués sur des données réglementées dans Glue ou S3 Tables.
4. **Marché : MARCHÉ À VALIDER.**
5. **Concurrence : angle unique trouvé.** Filtrage côté serveur neutre vis-à-vis du catalogue, pensé d'abord pour Glue et S3 Tables, avec certification des clients. **À TRANCHER :** AWS Lake Formation va-t-il implémenter `read-restrictions` avec filtrage serveur ?
6. **Pourquoi maintenant.**
   - Spec fusionnée le 2026-09-03 ; en-tête de capacités le 2026-06-23.
   - Implémentation Java encore ouverte (#16131) [OUVERT] https://github.com/apache/iceberg/pull/16131
   - Polaris devient projet Apache de premier niveau le 2026-02-19 [EXTRAIT].
7. **Proxy.** Okera racheté par Databricks ; Ryft racheté par Cyera 100-130 M$ (2026) [EXTRAIT].
8. **Scalable : OUI, avec une réserve de marge brute.** Filtrer côté serveur veut dire relire les données. Pour limiter le coût : filtrer d'abord au niveau fichier ou row group pendant la planification.

**Formulations du balayage (12).**
1. « Iceberg fine-grained access control untrusted engine enforcement proxy row filter column masking DuckDB PyIceberg »
2. « Iceberg read restrictions spec merged September 2026 engines support »
3. « Unity Catalog external engines row filters column masks enforcement Iceberg REST »
4. « Immuta OR Privacera Iceberg REST catalog Polaris read restrictions policy enforcement multi-engine »
5. site:ycombinator.com/companies « lakehouse access control OR data governance Iceberg »
6. « "Show HN" Iceberg access control OR row-level security OR masking proxy lakehouse AI agents »
7. site:producthunt.com « Iceberg lakehouse governance OR access control OR masking »
8. GitHub « iceberg row level security masking proxy OR gateway »
9. GitHub « arrow flight policy enforcement data access agents lakehouse »
10. GitHub « iceberg rest catalog proxy »
11. « Databricks acquires Okera data access proxy policy enforcement »
12. « Immuta revenue ARR Privacera funding »

---

### Idée 5 — Surveillance du rappel réel des index vectoriels, par tranche de filtre et de suppressions, avec correction automatique

**Question technique d'origine.** Une requête vectorielle filtrée renvoie-t-elle les bons voisins, et comment le savoir en production ?

**Problématique (le mécanisme) :**
- pgvector applique le filtre **après** le parcours de l'index : « If a condition matches 10% of rows, with HNSW and the default hnsw.ef_search of 40, only 4 rows will match on average » [OUVERT] https://raw.githubusercontent.com/pgvector/pgvector/master/README.md
- Le parcours itératif (0.8.0) s'arrête à `max_scan_tuples` (20 000) ou à `work_mem × scan_mem_multiplier`. Avec un filtre très sélectif, la requête renvoie donc moins de résultats, et de moins bons, **sans erreur**.
- Les tombstones HNSW dégradent le rappel jusqu'à reconstruction.
- Corrections récentes dans pgvector : corruption et « hnsw graph not repaired » pendant le vacuum HNSW, en 0.8.3 et 0.8.4 (06/2026) [OUVERT] https://raw.githubusercontent.com/pgvector/pgvector/master/CHANGELOG.md
- Qdrant #7147 « Degraded Search Quality (HNSW Index Rebuild Required) », ouvert depuis 08/2025 [EXTRAIT] https://github.com/qdrant/qdrant/issues/7147

**Solution.** Un agent multi-moteurs :
- il rejoue un échantillon de requêtes réelles en recherche exacte ;
- il calcule le rappel par sélectivité de filtre, par locataire et par taux de suppressions ;
- il ajuste `ef_search`, `iterative_scan`, `max_scan_tuples` ou la stratégie ACORN ;
- il déclenche un REINDEX ou une compaction quand le rappel passe sous l'objectif (SLO).

**Comment la solution répond au problème.** Un échec silencieux devient une métrique avec alerte, puis une correction.

**Concurrents.**
- recallwatch (open source, local) [EXTRAIT] https://github.com/zhuhroscar-tech/recallwatch
- L'onglet ANN Recall de Qdrant (Qdrant seulement) [EXTRAIT] https://qdrant.tech/documentation/tutorials-search-engineering/retrieval-quality/
- FutureAGI et CubeAPM (pages non lues).
- L'observabilité RAG chez YC (Captain, Respan, Lemma).

**Qui paie.** Les équipes qui font de la recherche ou du RAG en production sur pgvector, Qdrant, Weaviate ou Pinecone.

**Revenu potentiel : MARCHÉ À VALIDER.**
- Seul facteur sourcé : environ 4 000 clients Pinecone, et un ARR estimé à 26,6 M$ (getlatka) [EXTRAIT] https://getlatka.com/companies/pinecone.io
- Avec une captation de 10 % de la dépense (HYPOTHÈSE non sourcée), on obtient environ 2,7 M$ : trop petit sur cette seule base.

**Les 8 critères**
1. **Vrai problème : OUI** (doc, changelog, tickets).
2. **Piège à goudron : À TRANCHER.** Risque « fonction plutôt que produit » (Qdrant l'intègre déjà).
3. **Problème aigu : À TRANCHER.** L'échec est silencieux et aucun budget n'est documenté.
4. **Marché : MARCHÉ À VALIDER.**
5. **Concurrence : angle trouvé mais fragile.** Multi-moteurs + découpage par filtre et par suppressions + correction automatique ; pages FutureAGI et CubeAPM non lues.
6. **Pourquoi maintenant.** pgvector 0.8 (10/2024) et correctifs de 06/2026 ; ACORN dans Weaviate 1.27 ; arrêt de text-embedding-004 le 2026-01-14 [EXTRAIT] https://ai.google.dev/gemini-api/docs/deprecations
7. **Proxy.** Monte Carlo, observabilité des données, série D à 1,6 Md$ (2022) [EXTRAIT] https://techcrunch.com/2022/05/24/monte-carlo-raises-135m-series-d-at-1-6b-price-showing-that-unicorn-rounds-are-still-a-thing
8. **Scalable : OUI.** Surveiller le coût de la recherche exacte de référence.

**Formulations du balayage (11).**
1. « HNSW deletes tombstones recall degradation unreachable nodes rebuild »
2. « pgvector 0.8.0 iterative index scans overfiltering »
3. « ACORN filter strategy Weaviate 1.27 »
4. « monitor ANN recall in production vector database drift alerts exact search sampling tool »
5. « Qdrant search quality measure recall exact=true »
6. producthunt « recall monitoring launch 2026 »
7. site:ycombinator.com/companies « vector search observability retrieval quality monitoring »
8. GitHub « vector index recall monitoring hnsw »
9. GitHub « filtered vector search benchmark recall »
10. Tickets pgvector « HNSW recall drops after many updates deletes vacuum »
11. Tickets Qdrant « recall degradation after deletes upserts HNSW »

---

### Idée 6 — Garde-fou « à l'effet » pour le SQL d'agents (exécution à blanc, comptage réel cascades comprises, puis approbation)

**Question technique d'origine.** Les garde-fous de SQL pour agents voient-ils ce qu'une requête va réellement faire ?

**Problématique (le mécanisme) :**
- Le détecteur de Supabase MCP se décrit lui-même ainsi : « a comment heuristic, not a SQL lexer » [OUVERT] https://github.com/supabase-community/supabase-mcp (tools/destructive-sql.ts).
  - [TEST LOCAL] : `WITH d AS (DELETE …) SELECT`, `UPDATE … WHERE true`, `DO $$ … EXECUTE`, `SELECT purge_all_users()` passent sans alerte.
- Le serveur MCP Postgres de référence enveloppe la requête dans `BEGIN TRANSACTION READ ONLY` mais accepte plusieurs instructions, donc `COMMIT; DROP …` passe. Il n'a pas de `statement_timeout` [OUVERT] https://raw.githubusercontent.com/modelcontextprotocol/servers-archived/main/src/postgres/index.ts ; https://securitylabs.datadoghq.com/articles/mcp-vulnerability-case-study-SQL-injection-in-the-postgresql-mcp-server/ ; [TEST LOCAL] reproduit.
- Bytebase estime le volume touché par `EXPLAIN` [OUVERT] https://raw.githubusercontent.com/bytebase/bytebase/main/backend/plugin/parser/pg/explain_plan.go
  - Or [TEST LOCAL] `DELETE FROM orgs WHERE id=5` : estimation de 1 ligne, effet réel de **51 001 lignes** (cascades `ON DELETE CASCADE`, invisibles dans le plan).

**Solution.** Exécuter l'instruction dans une transaction de test (sur une branche ou en prod, puis ROLLBACK). Mesurer l'effet par table (`pg_stat_xact_user_tables`, event triggers pour le DDL). Appliquer les plafonds et la politique. Soumettre l'**effet réel** à l'approbation humaine.

**Comment la solution répond au problème.** Le garde-fou juge l'effet réel, qui ne se contourne pas par la syntaxe.

**Concurrents.**
- Bytebase (approbation, estimation EXPLAIN).
- hoop.dev (« AI Agent sidecar », blocage des « massive DELETE ») [EXTRAIT] https://hoop.dev/
- Formal (YC, proxy protocolaire avec OPA) [EXTRAIT] https://www.formal.ai/
- Alter (YC, « PAM for AI agents ») [EXTRAIT] https://www.ycombinator.com/companies/alter
- Inconvo (YC), crystaldba/postgres-mcp, ForbQL, eterDB (undo après coup).

**Qui paie.** Mêmes acheteurs que pour l'idée 2.

**Revenu potentiel : MARCHÉ À VALIDER.** Même manque de chiffres ; prix voisin StrongDM à 70 $ par utilisateur et par mois [EXTRAIT].

**Les 8 critères**
1. **Vrai problème : OUI** (tests locaux, code).
2. **Piège à goudron : signal négatif.** Proxy d'accès générique : Cyral vendu environ 25 M$ [EXTRAIT].
3. **Problème aigu : après incident seulement** (HYPOTHÈSE non sourcée).
4. **Marché : MARCHÉ À VALIDER.**
5. **Concurrence : À TRANCHER.** Il faut vérifier si hoop.dev, Formal ou Bytebase font déjà une exécution à blanc avec comptage réel (docs bloquées). Risque élevé de « fonctionnalité plutôt qu'entreprise ». **Recommandation :** en faire le moteur de diff de l'idée 2 plutôt qu'un produit autonome.
6. **Pourquoi maintenant.** Élicitation MCP (`inputRequired.elicit` dans le code Supabase [OUVERT]) ; incidents d'agents en production (07/2025).
7. **Proxy.** Bytebase (open source, gouvernance « for humans and agents ») https://github.com/bytebase/bytebase
8. **Scalable : OUI**, mais faible défensibilité.

**Formulations du balayage (11).**
1. « database firewall for AI agents SQL guardrail rows affected dry run approval startup »
2. site:ycombinator.com/companies « database access AI agents guardrails »
3. « Show HN Postgres proxy for AI agents blocks destructive queries preview rows affected »
4. « Formal database proxy AI agents MCP policy enforcement 2026 »
5. « joinformal Formal AI agents database access proxy »
6. « Bytebase AI agent MCP data change approval »
7. GitHub « sql firewall agent »
8. GitHub « postgres mcp »
9. GitHub « sql blast radius dry run rows affected agent »
10. Product Hunt « AI agent production database safe writes preview approve »
11. « information flow control taint tracking MCP database »

---

## B. Idées rejetées, avec la preuve

| Idée | Critère en échec | Preuve (source) |
|---|---|---|
| Proxy de sharding Postgres | 5 Concurrence | PgDog (YC, 5,5 M$, 06/2026) [EXTRAIT] https://github.com/pgdogdev/pgdog ; Neki (PlanetScale) https://neki.dev/ ; Multigres (Supabase, 100 M$) [EXTRAIT] https://siliconangle.com/2025/10/03/postgresql-database-specialist-supabase-snags-100m-funding/ ; Citus |
| Pooler qui gère les prepared statements / évite le pinning | 5 Concurrence, 3 non aigu | PgBouncer `max_prepared_statements` [OUVERT] https://raw.githubusercontent.com/pgbouncer/pgbouncer/master/doc/config.md ; RDS Proxy multiplexe le protocole étendu depuis 11/2023 [EXTRAIT] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy-pinning.html ; PgDog |
| Orchestrateur ou linter de DDL contre la file de verrous | 5 Concurrence, signal de piège à goudron | pgroll (réessai de verrou avec backoff) [EXTRAIT] https://pgroll.com/blog/schema-changes-and-the-postgres-lock-queue ; Atlas, Bytebase, Squawk ; Reshape abandonné [EXTRAIT] https://github.com/fabianlindfors/reshape |
| Conversion PL/SQL ou T-SQL vers PG par LLM | 5 Concurrence (gratuit chez les hyperscalers) | AWS DMS Schema Conversion avec IA générative (12/2024, Sybase en 11/2025) [EXTRAIT] https://aws.amazon.com/about-aws/whats-new/2024/12/aws-dms-schema-conversion-generative-ai ; Google DMS + Gemini [EXTRAIT] https://docs.cloud.google.com/database-migration/docs/oracle-to-alloydb/code-conversion-with-gemini. Sous-angle « vérification d'équivalence » non instruit (voir C5) |
| Branching avec anonymisation des PII | 5 Concurrence, 2 piège (Snaplet fermé) | Xata (copy-on-write + anonymisation pendant la réplication) [EXTRAIT] https://xata.io/blog/xata-postgres-with-data-branching-and-pii-anonymization ; Snaplet fermé [EXTRAIT] https://postgres.fm/episodes/postgres-startup-ecosystem |
| Autopilote vacuum / wraparound | 5 Concurrence, 2 piège | pganalyze VACUUM Advisor [EXTRAIT] https://pganalyze.com/docs/vacuum-advisor/freezing ; OtterTune fermé (06/2024) [EXTRAIT] https://news.ycombinator.com/item?id=40690380 |
| CDC Postgres → Iceberg sans equality deletes (compaction ou conversion) | 5 Concurrence, 6 (résolu en open source) | Spark `convert_equality_deletes` fusionnée le 2026-04-27 (#15970), Flink `ConvertEqualityDeletes` le 2026-06-26 (#15996), Kafka Connect #18003 [OUVERT] https://github.com/apache/iceberg/pull/18003 ; vote pour interdire les equality deletes en v4 [EXTRAIT] ; Supermetal, OLake, Estuary, Streamkap, Artie ; Moonlink → Databricks, Crunchy → Snowflake, PeerDB → ClickHouse |
| Gardien de slots de réplication (WAL, failover) | 5 Absorbé par Postgres et les clouds | Synchronisation des slots en PG17, `idle_replication_slot_timeout` en PG18 [OUVERT] https://raw.githubusercontent.com/postgres/postgres/REL_18_STABLE/doc/src/sgml/config.sgml ; RDS Multi-AZ [EXTRAIT] ; Debezium `slot.failover` |
| Maintenance Iceberg en service | 5 Concurrence (absorbé par les catalogues) | S3 Tables compaction à 0,005 $/Go [EXTRAIT] https://aws.amazon.com/s3/pricing/ ; Gravitino, Polaris, Databricks Predictive Optimization ; « Table Maintenance Stopped Being a Product » [EXTRAIT] https://datalakehousehub.com/blog/table-maintenance-economics/ |
| Optimiseur de coûts Snowflake / Databricks | 5 Concurrence, 6 (natif) | Snowflake Adaptive Compute GA le 2026-06-16 [EXTRAIT] https://docs.snowflake.com/en/release-notes/2026/other/2026-06-16-adaptive-compute-ga ; Keebo, Espresso AI, SELECT, Revefi |
| Observabilité ou qualité des données | 2 Piège (consolidation) | GX Cloud retiré de la vente et racheté par FICO (2026) [EXTRAIT] https://greatexpectations.io/blog/an-update-from-great-expectations/ ; Metaplane → Datadog ; fusion dbt + Fivetran (2026-06-01) [EXTRAIT] https://www.getdbt.com/blog/dbt-labs-and-fivetran-merge-announcement |
| Preuve de tests de restauration DORA (produit seul) | 5 Concurrence | AWS Backup restore testing + `PutRestoreValidationResult` [EXTRAIT] https://docs.aws.amazon.com/aws-backup/latest/devguide/restore-testing-validation.html ; Elastio « Provable Recovery », « verifiable evidence… to regulators » [EXTRAIT] https://elastio.com/blog/elastio-launches-managed-provable-recovery-service ; Eon (4 Md$). Reste une *fonction* de l'idée 1 |
| Chiffrement au niveau champ / tokenisation PCI DSS 4.0 | 5 Concurrence, 6 (échéance passée) | Skyflow, Evervault, CipherStash, VGS, Basis Theory, MongoDB Queryable Encryption, AWS DB Encryption SDK [EXTRAIT] https://www.mongodb.com/docs/manual/core/queryable-encryption/reference/limitations/ ; exigence 3.5.1.2 en vigueur depuis le 31/03/2025 [EXTRAIT] https://www.guidepointsecurity.com/blog/pci-dss-4-0-major-future-dated-requirements/ ; JumpWire → WorkOS |
| JIT, bastion, DAM | 5 Concurrence, consolidation | Teleport, StrongDM ; Cyral → Varonis (03/2025) [EXTRAIT] https://www.varonis.com/blog/varonis-to-acquire-cyral-database-activity-monitoring |
| Découverte ou classification de données sensibles (DSPM) | 5 Concurrence | Cyera, 600 M$ levés à 12 Md$ (06/2026) [EXTRAIT] https://www.businesswire.com/news/home/20260610121080/en/ |
| Migration de modèle d'embedding sans ré-encodage (générique) | 5 Concurrence, 3 non aigu | UniVec [EXTRAIT] https://univec.ai/ ; EmbeddingAdapters [OUVERT] https://github.com/PotentiallyARobot/EmbeddingAdapters ; Show HN « Drift » https://news.ycombinator.com/item?id=48475082 ; Voyage 4, espace partagé [EXTRAIT] https://blog.voyageai.com/2026/01/15/voyage-4/ ; ré-encodage d'environ 520 $ pour 10 millions de fragments [EXTRAIT] https://www.beri.net/article/openai-vs-cohere-vs-qwen3-embedding-models-enterprise-rag-2026. Niche « vecteurs sans source » : À TRANCHER, sans chiffrage |
| Convertisseur Flux → SQL (InfluxDB 3) | 5 Concurrence (éditeur), 8 one-shot | Convertisseur IA d'InfluxData (Explorer 1.9) [EXTRAIT] https://www.influxdata.com/blog/influxdb-3-explorer-1-9/ |
| Nouveau moteur de synchronisation local-first | 2 Piège à goudron | Reflect fermé, Replicache en maintenance [EXTRAIT] https://rocicorp.dev/blog/retiring-reflect ; Triplit → Supabase ; Zero, PowerSync, Electric, Instant, LiveStore |
| Outil de migration Redis → Valkey | 3 Non aigu | Compatibilité Redis 7.2, guide officiel [EXTRAIT] https://valkey.io/topics/migration/ ; Valkey par défaut sur ElastiCache et Memorystore |
| Indexation GraphRAG moins chère | 5 Concurrence | LazyGraphRAG : environ 0,1 % du coût de GraphRAG [EXTRAIT] https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/ |
| Remplaçant de Kuzu embarqué | 5 Concurrence | LadybugDB (plus de 80 contributeurs), guide de migration ArcadeDB [EXTRAIT] https://gdotv.com/blog/kuzu-legacy-embedded-graph-database-landscape/ |
| Undo ciblé de transaction pour agents | 5 Concurrence (exactement l'angle) | eterDB (fork PG 18, « surgical, dependency-aware undo… built for agents ») [EXTRAIT] https://github.com/eterdb/eterdb ; Rubrik Agent Rewind |
| Garde-fou de coût avant exécution pour agents sur entrepôt | 5 Concurrence (exactement l'angle) | cost-guard-mcp (BigQuery/Snowflake, plafonds d'octets, de lignes et de $) [EXTRAIT] https://github.com/mcpsmiths/cost-guard-mcp |
| « Pare-feu de base » générique pour agents | 5 Concurrence, 2 signal de piège | Formal (YC), hoop.dev, Alter (YC) [EXTRAIT] ; Cyral vendu environ 25 M$ |

---

## C. Couverture par sous-couche : questions posées et réponses

### C1. PostgreSQL en production (vacuum, wraparound, montées de version)
- **PG 19 passe-t-il aux XID 64 bits ?**
  - Non : `typedef uint32 TransactionId` sur master. Seul `MultiXactOffset` devient `uint64` [OUVERT] https://raw.githubusercontent.com/postgres/postgres/master/src/include/c.h. Le gel reste obligatoire.
- **Quels seuils ?**
  - `autovacuum_freeze_max_age` = 200 M ; `vacuum_failsafe_age` = 1,6 Md.
  - Au-delà, les écritures sont refusées [OUVERT] https://raw.githubusercontent.com/postgres/postgres/master/doc/src/sgml/maintenance.sgml
- **Pourquoi la réplication logique casse-t-elle les montées de version ?**
  - DDL, séquences, large objects et vues matérialisées ne sont pas répliqués (voir Idée 3).
- **Que perd pg_upgrade ?**
  - Les statistiques d'extensions et les statistiques cumulatives (voir Idée 3).
- **Coût de l'inaction ?**
  - PG 13 en Extended Support depuis le 01/03/2026 [EXTRAIT] https://repost.aws/articles/ARRvHxJ_9sTDCGloBavca3kg/

### C2. MySQL (fin de vie de la 8.0, Extended Support RDS)
- **Dates.**
  - Fin de vie 8.0 en 04/2026 ; 8.4 LTS jusqu'en 2032 ; 9.7 LTS depuis le 21/04/2026 [EXTRAIT] https://endoflife.date/mysql
- **Coût RDS.**
  - 0,10 $/vCPU-h dès le 01/08/2026, 0,20 $ en année 3 [EXTRAIT] https://www.vantage.sh/blog/amazon-rds-extended-support
- **Ruptures.**
  - `mysql_native_password` désactivé en 8.4, supprimé en 9.0 [EXTRAIT].
- **Tests de régression.**
  - pt-upgrade et traffic mirroring [EXTRAIT] https://aws.amazon.com/blogs/database/performance-testing-mysql-migration-environments-using-query-playback-and-traffic-mirroring-part-2
  - Débouché : Idée 3.

### C3. Pooling et sharding
- **Prepared statements en mode transaction ?**
  - Oui au niveau protocole (`PGBOUNCER_{id}`), non pour `PREPARE` en SQL [OUVERT].
- **Pinning RDS Proxy ?**
  - Déclenché par SET, PREPARE en SQL, curseurs, advisory locks, et instructions de plus de 16 Ko [EXTRAIT].
- **Sharding.**
  - PgDog, Neki, Multigres, Citus (rejet en B).
- **Aurora DSQL.**
  - Contrôle de concurrence optimiste (erreur 40001), transactions de 10 000 lignes au plus [EXTRAIT] https://docs.aws.amazon.com/aurora-dsql/latest/userguide/working-with-foreign-key-constraints.html
  - Linter de compatibilité **non instruit** (niche présumée).

### C4. Migrations de schéma en ligne
- **Pourquoi un ALTER « rapide » bloque-t-il tout ?**
  - La file de verrous ACCESS EXCLUSIVE ; `lock_timeout` s'applique à chaque tentative [OUVERT] config.sgml ; [EXTRAIT] https://xata.io/blog/migrations-and-exclusive-locks
- **Outils.**
  - pgroll, Atlas, Bytebase, Squawk ; Reshape abandonné (rejet en B).
- **Merge d'une branche vers la prod.**
  - Rejeu du SQL chez Neon (voir Idée 2).

### C5. Sortie d'Oracle / SQL Server
- **Conversion par LLM déjà fournie ?**
  - Oui : AWS DMS SC avec IA générative, Google DMS + Gemini (rejet en B).
- **Limites de Babelfish.**
  - Pas de CLR, de DDL entre bases, de curseurs GLOBAL, de DBCC [EXTRAIT] https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/babelfish-compatibility.tsql.limitations-unsupported.html
- **Benchmark.**
  - PLSQLBENCH (arXiv 2608.15931) [EXTRAIT].
- **Non instruit :** vérification d'équivalence différentielle Oracle ↔ PG. **À TRANCHER** : budget épuisé pour le balayage de concurrence.

### C6. Formats de tables ouverts (Iceberg, Delta)
- **Coût d'un equality delete.**
  - Il s'applique à tous les fichiers plus anciens de la partition jusqu'à compaction [OUVERT] spec.md l. 1072-1090.
- **Deletion vectors v3.**
  - Au plus un par fichier de données ; les position deletes sont interdits en v3 [OUVERT].
- **Row lineage.**
  - Perdu avec les equality deletes (« always treated as if the existing row was completely removed ») [OUVERT] l. 469.
- **v4.**
  - Pas encore adoptée ; vote pour interdire les equality deletes [EXTRAIT].
- **Delta `catalogManaged`.**
  - Le catalogue devient la source de vérité des commits ; accès par le système de fichiers non supporté [OUVERT] PROTOCOL.md.
- **Copies PII dans les métadonnées.**
  - Voir Idée 1.

### C7. Catalogues
- **Frontière de sécurité.**
  - La table entière (#15545) [OUVERT].
- **`read-restrictions`.**
  - Appliquées par le lecteur, et l'en-tête de capacités est falsifiable [OUVERT] (voir Idée 4).
- **Maturité.**
  - Polaris projet de premier niveau (02/2026) ; Gravitino ; Lakekeeper ; Glue 39,3 %, S3 Tables 25 % [EXTRAIT].
- **Databricks Cross-Engine ABAC.**
  - En bêta [EXTRAIT].

### C8. Moteurs de requête et coûts
- **Autodimensionnement natif.**
  - Snowflake Adaptive Compute (GA le 2026-06-16) [EXTRAIT].
- **Acteurs.**
  - Espresso AI, Keebo, SELECT, Slingshot, Revefi (rejet en B).
- **DuckLake 1.0.**
  - Sorti le 2026-04-13 : métadonnées dans une base SQL, deletion vectors compatibles Iceberg [EXTRAIT] https://ducklake.select/2026/04/13/ducklake-10/
- **Élagage au-delà de min/max.**
  - Index secondaires en cours de spécification (#16961).
- **Coût des agents sur un entrepôt.**
  - Tableau de bord Cortex Analyst : environ 200 $/semaine en test, plus de 4 000 $ la première semaine en production [EXTRAIT] https://www.revefi.com/blog/wait-my-snowflake-bill-did-what-the-hidden-cost-of-agentic-ai-in-snowflake

### C9. CDC et flux
- **Pourquoi un slot logique est-il dangereux ?**
  - Rétention illimitée de WAL et de lignes de catalogue ; `max_slot_wal_keep_size` = −1 par défaut ; le borner invalide le slot [OUVERT] https://raw.githubusercontent.com/postgres/postgres/REL_17_STABLE/doc/src/sgml/logicaldecoding.sgml
- **Failover des slots en PG17.**
  - Cinq conditions à réunir [OUVERT].
- **Upsert Flink vers Iceberg.**
  - Equality deletes ; les colonnes de partition doivent faire partie de la clé [OUVERT] https://raw.githubusercontent.com/apache/iceberg/main/docs/docs/flink-writes.md
- **Consolidation.**
  - Confluent → IBM (finalisé le 2026-03-17) [EXTRAIT] ; Tableflow upsert payant.
- **Conclusion.**
  - Rejet en B (résolu en open source).

### C10. Contrats de schéma
- **Standard.**
  - ODCS v3.1.0 ; la Data Contract Specification est dépréciée (support jusqu'à fin 2026) [EXTRAIT] https://docs.datacontract.com/open-data-contract-standard
- **Acteurs.**
  - Gable, série A de 20 M$ (03/2025) [EXTRAIT].
- **Ce qui manque.**
  - Relier le contrat aux mécanismes d'exécution (registre, `read-restrictions`, CI). Prolongement possible de l'Idée 4.
  - **Non instruit en balayage de concurrence** (budget).

### C11. Qualité et lignage
- **Consolidation.**
  - GX Cloud → FICO ; Metaplane → Datadog ; dbt + Fivetran (près de 600 M$ d'ARR combiné) [EXTRAIT] https://www.businesswire.com/news/home/20260601514374/en/
- **OpenLineage.**
  - dbt-ol fonctionne en post-traitement [EXTRAIT].
- **Conclusion.**
  - Rejet en B.

### C12. Bases vectorielles (mise à jour, filtrage, changement de modèle)
- **Suppressions HNSW.**
  - Tombstones. Corruption et erreurs de réparation du graphe au vacuum de pgvector corrigées en 06/2026 [OUVERT] CHANGELOG ; bit réversible dans hnswlib [OUVERT].
- **Filtrage.**
  - Post-filtrage dans pgvector et limites du parcours itératif [OUVERT] ; ACORN dans Weaviate [EXTRAIT] https://weaviate.io/blog/speed-up-filtered-vector-search
- **Changement de modèle.**
  - Drift-Adapter : 95-99 % du rappel [EXTRAIT] https://arxiv.org/abs/2509.23471 ; vec2vec [EXTRAIT] https://arxiv.org/abs/2505.12540 ; Voyage 4 (espace partagé) ; arrêt de text-embedding-004 le 2026-01-14.
- **Débouchés.**
  - Idée 5 ; rejet de la migration générique en B.

### C13. Séries temporelles
- **InfluxDB 3 Core.**
  - Limites levées le 2025-01-27, mais la plage de temps d'une requête reste bornée [EXTRAIT] https://www.influxdata.com/blog/influxdb3-open-source-public-alpha-jan-27/
  - Flux n'est pas supporté en v3 [EXTRAIT] https://docs.influxdata.com/flux/v0/future-of-flux/
- **Cardinalité.**
  - Plus d'index des séries en v3 (Parquet + statistiques) [EXTRAIT].
- **TimescaleDB.**
  - L'éditeur s'appelle TigerData (06/2025) ; compression et agrégats continus sous licence TSL [EXTRAIT] https://www.tigerdata.com/legal/licenses
- **Conclusion.**
  - Aucune idée retenue (convertisseur Flux rejeté).

### C14. Graphes
- **Kuzu.**
  - Archivé le 2025-10-10 après son rachat (par Apple selon gdotv) ; forks Bighorn et LadybugDB [EXTRAIT].
- **GQL (ISO/IEC 39075:2024).**
  - Conformité partielle de Cypher [EXTRAIT] https://neo4j.com/docs/cypher-manual/current/appendix/gql-conformance/
- **Coût de GraphRAG.**
  - LazyGraphRAG [EXTRAIT].
- **Conclusion.**
  - Aucune idée retenue.

### C15. Local-first et synchronisation
- **Fermetures ou pivots.**
  - Reflect fermé, Replicache en maintenance ; Triplit racheté par Supabase [EXTRAIT].
- **Migrations de schéma côté client.**
  - C'est au développeur de gérer la compatibilité (PowerSync) [EXTRAIT] https://docs.powersync.com/usage/lifecycle-maintenance/implementing-schema-changes
  - Clients restés longtemps hors ligne bloqués par un changement incompatible (Triplit) [EXTRAIT] https://github.com/aspen-cloud/triplit/discussions/92
- **Conclusion.**
  - Piège à goudron (rejet en B). Le sous-angle « migrations de schéma pour clients hors ligne » n'a pas fait l'objet d'un balayage dédié.

### C16. Serverless et branching
- **Branching + anonymisation.**
  - Xata [EXTRAIT].
- **Neon.**
  - Racheté par Databricks pour environ 1 Md$ ; plus de 80 % de bases créées par des agents [EXTRAIT] https://www.cnbc.com/2025/05/14/databricks-is-buying-database-startup-neon-for-about-1-billion.html
- **Merge vers le parent et restauration.**
  - Voir Idée 2.

### C17. Bases utilisées par des agents IA (garde-fous, permissions)
- **Le read-only du serveur MCP de référence était-il contournable ?**
  - Oui (`COMMIT;`), confirmé [OUVERT] + [TEST LOCAL]. 21 000 téléchargements npm par semaine après la dépréciation [OUVERT] (Datadog).
- **Supabase MCP.**
  - « Lethal trifecta » (07/2025) [EXTRAIT] https://simonwillison.net/2025/Jul/6/supabase-mcp-lethal-trifecta/
  - Défense actuelle au niveau du prompt (`untrusted-data`) et détecteur heuristique [OUVERT].
- **Privilèges.**
  - L'agent hérite du rôle entier chez tous les serveurs examinés [OUVERT].
- **Blast radius.**
  - EXPLAIN ne voit pas les cascades [OUVERT] + [TEST LOCAL].
- **Replit.**
  - Base de production supprimée (07/2025) [EXTRAIT].
- **Débouchés.**
  - Idées 2 et 6 ; rejets en B (undo, coût, pare-feu).
- **Non vérifiés (budget) :**
  - identité déléguée utilisateur → rôle Postgres + claims RLS : Keycard, Aembit, Oso, Permit.io, Cerbos, Descope, Stytch, Astrix, Pomerium. **À TRANCHER** ;
  - taint tracking ligne à ligne contre la trifecta : Snyk/Invariant couvrent le niveau session [EXTRAIT] https://invariantlabs.ai/blog/toxic-flow-analysis. **À TRANCHER, tendance défavorable.**

### C18. Caches et changements de licence (Redis / Valkey)
- **Licence.**
  - Redis 8 ajoute l'AGPLv3 (05/2025) ; Valkey par défaut chez AWS et GCP, environ 20 % moins cher sur ElastiCache [EXTRAIT] https://tech-insider.org/valkey-vs-redis-2026/
- **Coût de migration.**
  - Faible hors modules Redis 8 (rejet en B).
- **Invalidation de cache.**
  - Non instruite spécifiquement.

### C19. Sauvegardes, PITR, tests de restauration, immutabilité (DORA)
- **Une PITR ressuscite-t-elle une personne effacée ?**
  - Oui (voir Idée 1).
- **Restore testing AWS.**
  - Validation par Lambda [EXTRAIT].
- **DORA art. 12.**
  - Tests périodiques de sauvegarde et de restauration [EXTRAIT] https://www.digital-operational-resilience-act.com/Article_12.html
- **Acteurs.**
  - Elastio, Eon (rejet en B).
- **Immutabilité contre effacement.**
  - Conflit par construction, d'où la réapplication à la restauration (Idée 1).

### C20. Accès et privilèges
- **Acteurs JIT.**
  - Teleport, StrongDM, Bytebase [EXTRAIT] https://www.strongdm.com/blog/just-in-time-access-for-developers
- **Consolidation.**
  - Cyral → Varonis ; JumpWire → WorkOS [EXTRAIT].
- **Conclusion.**
  - Rejet en B.

### C21. Données sensibles
- **Acteurs.**
  - Cyera à 12 Md$ ; Sentra.
- **Angle mort.**
  - Bornes `lower/upper_bounds` Iceberg (`truncate(16)`) et stats Delta. Garde-fou `write.metadata.metrics.column.<col>=none` à activer soi-même [EXTRAIT] https://iceberg.apache.org/docs/nightly/configuration/
  - Intégré à l'Idée 1.

### C22. Chiffrement au niveau champ (PCI DSS 4.0)
- **Exigence 3.5.1.2.**
  - Le chiffrement disque ne suffit plus depuis le 31/03/2025 [EXTRAIT].
- **Limites des solutions interrogeables.**
  - MongoDB QE : égalité **ou** plages sur un champ ; beacons AWS = HMAC tronqués, égalité seulement [EXTRAIT] https://docs.aws.amazon.com/database-encryption-sdk/latest/devguide/beacons.html
- **Conclusion.**
  - Rejet en B.

### C23. Effacement RGPD et Delete Act
- **EDPB CEF 2025.**
  - Adopté le 10/02/2026 ; 32 autorités, 764 responsables de traitement [EXTRAIT].
- **ICO.**
  - « Beyond use ».
- **CNIL.**
  - Position connue seulement par une source secondaire.
- **DROP.**
  - Depuis le 01/08/2026 : consultation au moins tous les 45 jours ; 654 brokers ; plus de 500 000 inscrits ; 200 $ par jour et par demande [EXTRAIT].
- **Articles.**
  - Ghost Vectors et Ghost Echoes confirmés [EXTRAIT].
- **Débouché.**
  - Idée 1.

### C24. Résidence
- **Résidence ou souveraineté ?**
  - Distinctes (CLOUD Act) [EXTRAIT] https://europeanstack.com/guides/eu-data-residency
- **Copies secondaires hors région (WAL archivés, réplicas, manifests).**
  - HYPOTHÈSE non sourcée ; **non instruite (budget).** À TRANCHER. Même famille de mécanisme que l'Idée 1 (copies cachées).

### C25. Audit et exfiltration
- **pgaudit.**
  - Volume énorme en `all` ; ne capture pas les valeurs renvoyées [EXTRAIT] https://github.com/pgaudit/pgaudit/blob/main/README.md
- **RDS.**
  - Pas de DAM natif pour Postgres ; GuardDuty ne profile que les connexions [EXTRAIT] https://docs.aws.amazon.com/guardduty/latest/ug/rds-protection.html
- **Marché.**
  - Consolidé (Varonis/Cyral, Cyera).
- **Conclusion.**
  - Rejet en B.

---

**Limites de ce rapport**
- Beaucoup de sources primaires réglementaires (PDF EDPB, cnil.fr, ico.org.uk) et de pages éditeurs n'ont été lues que par extrait.
- Aucun prix de vente n'a été trouvé pour les idées 1 à 6 : tous les marchés sont « MARCHÉ À VALIDER ».
- Sous-couches ou sous-angles non instruits faute de budget, signalés ci-dessus : résidence (copies hors région), vérification d'équivalence Oracle → PG, identité déléguée pour agents, invalidation de cache, linter Aurora DSQL.

STATUT: TERMINÉ
