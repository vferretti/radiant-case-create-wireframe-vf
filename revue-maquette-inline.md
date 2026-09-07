# Revue de `case-create-signs-inline.html` — Vincent

Fichier de travail : changements demandés et commentaires, notés au fil de la session.

> **Cible : la version B (signes en ligne), `case-create-signs-inline.html`.** Décidé le
> 2026-09-07, après avoir d'abord visé la version A (modale). Aucune note de contenu n'avait
> encore été écrite, donc rien n'est à reporter d'un fichier à l'autre.

**Pour l'instant, tout reste ici : la maquette n'est pas modifiée.** On décidera plus tard si
ces notes sont aussi intégrées dans le fichier HTML.

## Piste d'intégration (si on décide de le faire — rien n'est fait)

- **Liste séparée**, pas de fusion avec les annotations existantes (`note.1`…`note.8`).
- Réalisation envisagée : **un petit onglet** à l'intérieur du bloc `.notes-legend` existant
  (deux onglets : « Annotations » = les notes actuelles du réviseur / « Mes notes » = les notes de
  Vincent). Le toggle **Codes** continuerait d'afficher ou masquer tout le bloc.
- **Français seulement** pour les notes de Vincent — pas de clé i18n `en` à maintenir.

_Dernière mise à jour : 2026-09-07._

## Légende des statuts

- `[ ]` changement demandé — noté ici, pas appliqué à la maquette
- `[x]` appliqué à la maquette (rien pour l'instant)
- `[?]` question ouverte / décision à prendre
- `[i]` commentaire seulement (pas de changement de code)

---

## Changements et commentaires

<!-- Les entrées sont ajoutées ici au fur et à mesure. Format :

### RN — Titre court
**Statut :** [ ]
**Où :** section / sélecteur / ligne
**Demande :** ce que Vincent a demandé, dans ses mots.
**Note pour la maquette :** formulation courte, prête à devenir une annotation « Codes ».

-->

### R1 — Renommer la section 1 : « Commande et analyse » → « Cas »
**Statut :** [ ]
**Où :** section 1, `<h2 data-i18n="sec.order">` (ligne 343) ; clés i18n `sec.order` en (l. 19361)
et fr (l. 19444).
**Demande :** changer le nom de la section « Commande et analyse » pour **« Cas »**.
**Note pour la maquette :** la section 1 porte sur le cas lui-même, pas sur une commande.

**À vérifier au moment d'appliquer :**
- La clé s'appelle `sec.order` — le nom de clé peut rester tel quel (invisible pour l'utilisateur)
  ou être renommé `sec.case` pour rester lisible. Choix cosmétique.
- Équivalent anglais à décider : « Case » (parallèle direct). Actuellement « Order & analysis ».
- Même changement à faire dans `case-create-signs-modal.html` si on veut garder les deux
  versions comparables.

---

### R2 — Premier champ (Analyse) : peut-on chercher dans la liste ?
**Statut :** [x] **appliqué à la maquette le 2026-09-07**
**Où :** section 1, `<div class="ctrl select" data-sel="analysis">` (l. 350) ; menu construit par
`openMenu()` (l. 19556) et `menuItems()` (l. 19586) ; catalogue dans `ANALYSES` (l. 599).
**Question :** le premier champ est une liste déroulante — est-ce qu'on peut y chercher des
valeurs (auto-suggestion) ?

**Réponse : oui — et avec le vrai catalogue, ça devient nécessaire.**
- La maquette n'a que **4 analyses** codées en dur (`ANALYSES`, l. 599). Le vrai catalogue
  (`analysis_catalog_qlin.csv`, fourni le 2026-09-07) en compte **37**. Une liste déroulante de 37
  entrées, sans recherche, est pénible.
- Techniquement, rien à inventer : **le même fichier contient déjà le patron**, le champ de
  recherche HPO des signes cliniques (champ texte + liste filtrée en direct + surlignage). On
  réutiliserait la même mécanique pour le menu des analyses.
- Le menu actuel (`openMenu`) est un composant maison, pas un `<select>` natif — donc ajouter un
  champ de recherche en haut du menu est faisable sans rien casser ailleurs.

**Ce que le vrai catalogue impose à la recherche (précisé par Vincent, 2026-09-07) :**
- La recherche porte sur **deux champs du modèle** : `name` et `code`.
  - `name` — « 55340 - Maladies musculaires globales ». **Le numéro d'acte fait partie du nom**,
    ce n'est pas un champ distinct : chercher dans `name` couvre donc à la fois le numéro et le
    libellé.
  - `code` — le code court (MMG, RGDI, EPIL…).
- Conséquence : la recherche doit matcher **n'importe où dans la chaîne**, pas seulement au début —
  sinon taper « muscul » ne trouve rien, puisque `name` commence par le numéro d'acte.

**Ce qui a été fait (`openMenu`, l. ~19570) :**
- Un champ « Filtrer par nom ou code… » se pose au-dessus de la liste, qui devient défilante.
- Le filtre cherche dans `name` **et** `code`, **n'importe où** dans la chaîne, sans tenir compte
  des accents ni de la casse. La portion trouvée est surlignée, dans le nom comme dans le code.
- Le code court s'affiche à droite de chaque nom, en gris monospace (même vocabulaire visuel que
  les identifiants HP: de l'arbre HPO).
- « Aucun résultat » quand rien ne correspond.
- **Seuil : 8 entrées** (`MENU_SEARCH_MIN`). Au-dessous, la liste reste telle quelle — Priorité,
  Site émetteur, etc. n'ont pas de champ de recherche. Le mécanisme est générique : il suffira de
  baisser le seuil, ou d'allonger une liste, pour qu'elle en hérite.

**Vérifié dans un vrai navigateur** (19 assertions) : « muscul » trouve les 2 analyses musculaires
malgré le numéro d'acte en tête, « 55340 » trouve MMG, « rgdi » trouve RGDI, « epilepsie » sans
accent trouve « Epilepsie », le menu court (Priorité) n'a pas de champ de recherche.

**Décision restante :**
- Faut-il aussi la recherche sur **site émetteur / établissement prescripteur** ? 4 organisations
  aujourd'hui, mais la vraie liste des établissements du Québec est longue. Il suffira d'y brancher
  la vraie liste : le seuil de 8 l'activera tout seul.

---

### R3 — Remplacer les 4 analyses factices par le vrai catalogue
**Statut :** [x] **fait le 2026-09-07** — les 37 analyses sont chargées, mais sous les hypothèses
ci-dessous (points 1 à 5 toujours ouverts)
**Où :** `var ANALYSES` (l. 599) ; source : `analysis_catalog_qlin.csv` (37 analyses, tenant `qlin`).
**Demande :** la liste des analyses est dans le fichier CSV du répertoire.

**Ce que le CSV apporte, et ce qui manque :**

| Colonne CSV | Correspond à | Remarque |
|---|---|---|
| `code` | `ANALYSES[].code` | direct (MMG, RGDI, TSOL…) |
| `name` | `label_fr` | direct — inclut le numéro d'acte en préfixe |
| `analysis_type_code` | `type` (germinal/somatique) | direct ; **confirme la note 1** : un seul type par analyse dans tout le catalogue |
| `primary_condition` | `condition` | code MONDO/HPO **sans libellé** — la maquette affiche « Epilepsy (MONDO:0005027) », le CSV ne donne que le code |
| — | `label` (anglais) | **absent** — le catalogue est en français seulement |
| — | `category` (Postnatal) | **absent** — à confirmer, la maquette suppose Postnatal partout |

**Points à trancher :**
1. **Pas de nom anglais.** La maquette est bilingue ; le catalogue ne l'est pas. Options : afficher
   le nom français même en mode EN, ou faire traduire les 37 noms. À décider avec toi.
2. **Libellés des conditions manquants.** Il faudra résoudre les codes MONDO en noms lisibles
   (source externe), ou n'afficher que le code.
3. **Deux analyses sans `primary_condition`** : RAPIDE (exome rapide) et GENOR (génome entier non
   spécifique) — ce sont justement les analyses non spécifiques. Donc la condition dérivée doit
   pouvoir être **nulle** : à vérifier, la maquette dérive aujourd'hui une condition pour chaque
   analyse.
4. **Doublons apparents à clarifier** — NPC et NEUTP portent tous deux « Neutropénie congénitale » ;
   HLEB et HLH portent tous deux le numéro d'acte 55412. Erreur de saisie, ou distinction réelle ?
5. **Suggestions HPO par analyse.** Elles n'existent aujourd'hui que pour les 4 analyses factices
   (RGDI, EPI4, CARDIO, TSOL) — et **seul RGDI existe dans le vrai catalogue**. EPI4 correspond
   sans doute à EPIL (55415) ; TSOL n'a pas d'équivalent direct (plutôt EXTUM / STMO / TRATU) ;
   CARDIO n'existe pas du tout dans le catalogue. Avec 37 analyses, il faut décider : suggestions
   pour quelques analyses seulement, ou repli sur « aucune suggestion » pour les autres ?
6. Le CSV n'est **pas encore versionné** dans le dépôt (fichier non suivi). À committer si on veut
   qu'il serve de source.

**Hypothèses prises pour charger le catalogue (à confirmer ou à renverser) :**
- **Nom anglais = nom français.** En mode EN, la maquette affiche le nom français, faute de mieux.
- **Condition dérivée uniquement à partir d'un MONDO** (règle donnée par Vincent, 2026-09-07) :
  - `primary_condition` est un **MONDO** → le champ Condition principale est prérempli. **34 des 37.**
  - `primary_condition` est un **HPO** → champ **laissé vide**. Une seule analyse : **RHAB**
    (`HP:0003201`, rhabdomyolyse).
  - `primary_condition` est **absent** → champ **laissé vide**. Deux analyses : **RAPIDE** (exome
    rapide) et **GENOR** (génome entier non spécifique) — les deux analyses non spécifiques.
  - Le code brut du catalogue est conservé dans `conditionCode` dans tous les cas, HPO compris.
  - **Reste à régler : les libellés MONDO manquent.** Un seul code du catalogue a déjà un libellé
    lisible dans la maquette (`MONDO:0005027` → « Épilepsie »). Pour les 33 autres, le champ
    affiche donc **le code brut** (« MONDO:0019056 »). C'est volontairement visible : il faut une
    source de libellés MONDO, il n'y en a aucune dans le dépôt.
- **Catégorie = Postnatal** pour les 37 (absente du CSV).
- **Ordre du menu = ordre du CSV.** Pas de tri inventé. À revoir si tu préfères l'ordre
  alphabétique ou par numéro d'acte.
- **Suggestions HPO :** `EPI4` a été renommé `EPIL` (correspondance claire, panel épilepsie →
  55415). `CARDIO` et `TSOL` sont **laissées orphelines et inutilisées** plutôt que réaffectées à
  une analyse au jugé — c'est une décision clinique, pas technique.

---

## À intégrer sous le toggle « Codes » (plus tard, si on le fait)

_(Rien pour l'instant.)_
