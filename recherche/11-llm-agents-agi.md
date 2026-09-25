# 11 — LLMs, IA agentique, AGI (au 2026-09-25)

**Limite.** Presque toutes les preuves viennent d'extraits de résultats de recherche : le proxy bloque arxiv.org et la plupart des pages sources. Environ 21 recherches pour la partie LLMs.

## Partie 1 — LLMs

**Bilan : une idée retenue, faible (MARCHÉ À VALIDER). Huit pistes rejetées avec preuve.**

### Fiche 1 — Conformité « Article 50 » clé en main pour qui fine-tune ou héberge soi-même un modèle open-weight (MARCHÉ À VALIDER)

- **Question technique d'origine.** Un filigrane statistique de texte biaise le choix de chaque token avec une clé secrète. Seul le détenteur de la clé peut ensuite détecter le filigrane. Qui tient la clé et le détecteur quand le modèle est un Llama ou un Qwen fine-tuné, servi par vLLM chez une PME ?
- **Problématique.**
  - Depuis le 2 août 2026, l'article 50(2) de l'AI Act impose aux fournisseurs de systèmes génératifs deux choses : marquer leurs sorties de façon lisible par machine, et permettre de détecter ce marquage ([arXiv 2609.09604](https://arxiv.org/html/2609.09604v1)).
  - Le délai de grâce pour le filigrane court jusqu'au 2 décembre 2026 ([ComplianceHub](https://compliancehub.wiki/eu-ai-act-december-2-2026-legacy-systems-watermarking-grace-period-expiry/)).
  - Amende : jusqu'à 15 M€ ou 3 % du chiffre d'affaires mondial ([artificialintelligenceact.eu](https://artificialintelligenceact.eu/transparency-rules-article-50/)).
  - Une organisation qui appelle une API reste déployeuse : le devoir de marquage revient au fournisseur du modèle. En revanche, celle qui **fine-tune, vend en marque blanche ou met un système sur le marché sous son nom** peut devenir fournisseur ([Stibbe](https://www.stibbe.com/publications-and-insights/water-marking-the-machine-making-ai-generated-content-detectable)).
  - Le détecteur doit en principe être gratuit. Un fournisseur de moins d'un million d'utilisateurs mensuels peut facturer les gros volumes ([arXiv 2609.09604](https://arxiv.org/html/2609.09604v1)).
  - Au 2 février 2027, la détection doit devenir interopérable. Le Code prévoit quatre options ([Wilson Sonsini](https://www.wsgr.com/en/insights/eu-commission-publishes-ai-transparency-code-of-practice.html), [Bird & Bird](https://www.twobirds.com/en/insights/2026/taking-the-eu-ai-act-to-practice-the-final-transparency-code-of-practice)) :
    1. une API standard ;
    2. un « panneau indicateur » dans le contenu, qui dit quel détecteur utiliser ;
    3. un détecteur commun à un consortium, ouvert aux PME ;
    4. une solution équivalente.
  - **Mécanisme douloureux.** Chez un auto-hébergeur, aucune passerelle n'impose le marquage. L'équipe contrôle le décodage (température, échantillonnage) ; un réglage de l'échantillonneur peut donc désactiver ou affaiblir le filigrane sans que personne le voie ([CSA](https://labs.cloudsecurityalliance.org/research/csa-research-note-eu-ai-act-article-50-transparency-20260729/), extrait). L'équipe doit aussi gérer la clé, héberger un détecteur public et le raccorder à l'interopérabilité.
- **Solution.** Un module pour vLLM et SGLang, plus un service :
  - il filigrane les sorties (schéma de la famille SynthID-Text) ;
  - un test en CI vérifie que le filigrane reste détectable après chaque changement de décodage, de quantification ou de modèle ;
  - le service gère les clés ;
  - il héberge un détecteur public gratuit, avec quotas ;
  - il se raccorde à l'option interopérable choisie, API ou consortium ;
  - il produit un dossier de preuve pour l'autorité de surveillance.
- **Comment elle y répond.** Elle transforme l'obligation (marquer, détecter, rendre interopérable, documenter) en un abonnement. La promesse ressemble à celle d'un gestionnaire de consentement pour le RGPD.
- **Concurrents.**
  - Briques gratuites :
    - PR vLLM « native SynthID-Text watermarker » ([#58426](https://github.com/vllm-project/vllm/pull/58426)) et RFC ([#53916](https://github.com/vllm-project/vllm/issues/53916)) ;
    - [MarkLLM](https://github.com/THU-BPM/MarkLLM) (open source).
  - Google étend SynthID en standard multi-éditeurs, avec Apple, NVIDIA, OpenAI, ElevenLabs et Kakao ([arXiv 2609.09604](https://arxiv.org/html/2609.09604v1)).
  - IMATAG, Digimarc, Steg.AI et Truepic font surtout de l'image et de la vidéo. Aucune offre texte clé en main trouvée pour les petits fournisseurs (3 formulations).
  - **Angle** : ni la marque ni la détection, mais l'exploitation de bout en bout (clés, non-régression, hébergement du détecteur, interopérabilité, dossier) pour la longue traîne qui ne rejoindra pas le club Google.
- **Qui paie.** Les entreprises qui fine-tunent ou hébergent un modèle open-weight et exposent un système génératif à des utilisateurs de l'UE : éditeurs SaaS européens, intégrateurs, labos de taille moyenne.
- **Revenu potentiel.** Non chiffrable avec des sources :
  - seul chiffre : environ 190 signataires du Code ([arXiv 2609.09604](https://arxiv.org/html/2609.09604v1)), dont les plus gros n'ont pas besoin du produit ;
  - le nombre d'auto-hébergeurs soumis à l'obligation n'est pas publié. **MARCHÉ À VALIDER.**
- **Preuve des critères.**
  1. Vrai problème : obligation légale datée, avec sanction.
  2. Pas un piège à goudron : l'obligation date du 2 août 2026 ; personne n'a pu échouer avant.
  3. Aigu : oui pour les juristes (sanction chiffrée), mais le contrôle par les autorités nationales reste à venir. Point faible.
  4. Marché : à valider.
  5. Angle : l'exploitation, pas l'algorithme (voir Concurrents).
  6. Pourquoi maintenant : 02/08/2026, 02/12/2026, 02/02/2027.
  7. Proxy : Usercentrics, gestion du consentement née d'une obligation européenne, a dépassé 100 M€ d'ARR avec 45 % de croissance ([Business Wire](https://www.businesswire.com/news/home/20251014687594/en/Usercentrics-Surpasses-%E2%82%AC100M-ARR-$117M-USD-as-Market-Leader-in-Data-Privacy-Paving-the-Way-for-Privacy-Led-Marketing)).
  8. Scalable : logiciel, détection bon marché (comptage statistique sur le texte).
  9. Mécanisme : le filigrane dépend des paramètres de décodage, que l'auto-hébergeur contrôle.
  10. Modèle Dropbox : les briques existent, mais leur assemblage et leur exploitation sont pénibles.
- **Risques principaux.**
  - Google ou Hugging Face offrent gratuitement un détecteur hébergé ouvert à tous.
  - Un filigrane de texte s'efface facilement par paraphrase ([Goedecke](https://www.seangoedecke.com/text-ai-watermarks/)). La valeur est donc la conformité, pas la sécurité.
  - L'autorité peut appliquer mollement la règle.

### Rejets (avec preuve)

| Piste (question d'origine) | Motif du rejet | Preuve |
|---|---|---|
| Hub neutre de détection de filigranes, tous fournisseurs | La plateforme occupe la place : Google pousse SynthID comme standard partagé avec OpenAI, NVIDIA et Apple. Les clés restent chez les fournisseurs (blocage structurel). | [arXiv 2609.09604](https://arxiv.org/html/2609.09604v1) |
| Audit des tokens de raisonnement cachés facturés | La vérification exige la coopération du fournisseur (CoIn). L'inflation contourne les audits : +1 469 % dans le cadre CoIn, +50,85 % même avec la trace visible. Vaudit audite déjà les factures IA. | [CoIn](https://arxiv.org/abs/2505.13778), [Token Inflation](https://arxiv.org/html/2605.30040v1) |
| Migration de prompts lors du retrait d'un modèle | Fonction intégrée par les plateformes : Bedrock Advanced Prompt Optimization and Migration, OpenAI Prompt Optimizer, Llama Prompt Ops, plus Arthur. | [AWS](https://aws.amazon.com/blogs/aws/amazon-bedrock-introduces-new-advanced-prompt-optimization-and-migration-tool/), [OpenAI](https://cookbook.openai.com/examples/gpt-5/prompt-optimization-cookbook), [Arthur](https://www.arthur.ai/column/model-deprecation-version-drift-agents) |
| Détection des attaques par distillation via API | Les trois labos de frontière partagent leurs données via le Frontier Model Forum (04/2026). Il reste peu d'acheteurs. | [Bloomberg](https://www.bloomberg.com/news/articles/2026-04-06/openai-anthropic-google-unite-to-combat-model-copying-in-china) |
| Entraînement de têtes de décodage spéculatif pour modèles fine-tunés | Déjà vendu par Baseten ; Red Hat Speculators est open source. | [Baseten](https://www.baseten.co/blog/how-to-train-custom-eagle-3-heads-for-speculative-decoding/), [Red Hat](https://developers.redhat.com/articles/2025/11/19/speculators-standardized-production-ready-speculative-decoding) |
| Fuite de prompts entre clients via le cache KV (canal temporel) | La correction est une option gratuite du moteur (`cache_salt`, vLLM). C'est une fonction, pas un produit. | [vLLM PR #17045](https://github.com/vllm-project/vllm/pull/17045) |
| Lignée d'un modèle fine-tuné (dérivé de DeepSeek ou Qwen ?) | Cisco Model Provenance Kit (gratuit, environ 900 modèles), HiddenLayer Model Genealogy, Cranium. | [Cisco](https://blogs.cisco.com/ai/model-provenance-kit), [HiddenLayer](https://www.hiddenlayer.com/platform/ai-supply-chain-security) |
| Effacement RGPD dans un modèle fine-tuné (désapprentissage) | Hirundo (environ 8 M$ levés) vend la fonction. Aucune sanction publique trouvée : douleur peu aiguë. | [Hirundo](https://tooldirectory.ai/tools/hirundo) |
