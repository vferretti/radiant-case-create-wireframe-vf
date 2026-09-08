# Revue de `case-create-signs-inline.html`

_Version B (signes en ligne). Dernière mise à jour : 2026-09-08._

## Changements effectués

- **Menu des analyses recherchable** — champ de filtre sur le nom et le code, n'importe où dans
  la chaîne, sans accents ni casse ; apparaît sur toute liste de plus de 8 entrées.
- **Libellés des conditions MONDO** — résolus via l'API EBI OLS, ajoutés au CSV et affichés à la
  place du code brut. La liste Condition principale contient les 26 conditions réelles du catalogue.
- **Section 1 renommée** « Commande et analyse » → **« Analyse »** (EN : « Analysis »).
- **Toutes les listes déroulantes sont effaçables** — une ligne « ↺ Effacer la sélection »
  ramène le champ à l'état initial, sans valeur sélectionnée.
- **Case de consentement à la recherche retirée** ; le champ **Étude de recherche** passe sous la
  case « Cas prénatal » et reste toujours visible.
- **Liste des études** remplacée par Pragmatic, Care4Rare, RQDM.
- **Section 1 entièrement dépliée** — le bloc « Détails facultatifs » est supprimé ; médecin
  et établissement prescripteur sont remontés sur une ligne, le médecin en premier.
- **Résumé du cas : Catégorie et Priorité toujours affichées**, avec leur valeur par défaut
  (Postnatal, Routine), pour que le bloc ne saute plus.
- **Champs prénataux déplacés** — ils s'ouvrent maintenant à la fin de la section 2 et non plus
  dans la section « Cas » ; la case reste en section 1.
- **Section 2 renommée** « Patient (cas index) », qui devient **« Patient (cas index, mère) »**
  en mode prénatal (sans trait d'union : « cas index » n'en prend pas).
- **Prénatal : le sexe est prérempli à Féminin** — la patiente est la mère. Reste modifiable, et
  la valeur choisie avant est restituée si on décoche.
- **RAMQ alignée à gauche**, sous le site émetteur, au lieu de la colonne de droite.
- **Effacer l'analyse efface aussi la condition MONDO** qui en était dérivée.
- **Phénotypes suggérés : la même liste pour toutes les analyses** en attendant les vraies listes
  cliniques, sauf l'exome rapide (RAPIDE) et le génome non spécifique (GENOR), qui n'en ont aucune.
- **Aucune logique automatique sur la priorité** : ni le prénatal ni « Fœtus décédé » ne la
  changent, l'utilisateur choisit toujours.
- **« Site émetteur » renommé « Établissement du patient »** (EN : « Patient organization »),
  sans valeur par défaut.
- **Menu « MRN » retiré** (cas index et ligne famille) ; le libellé devient **« Identifiant (NDD,
  code de l'étude, …) »**.
- **Section 3 renommée « Information clinique »** (EN : « Clinical information »).
- **« Condition principale / indication » renommé « Indication principale standardisée (MONDO) »**
  et déplacé juste au-dessus de la note clinique, dans le bloc Contexte clinique. Le résumé parle
  maintenant d'« Indication principale ».
- **Contexte clinique réorganisé** : statut vital remonté à droite de la consanguinité,
  « Ascendance / origine ethnique » renommé **« Ethnicité »** (EN : « Ethnicity ») et placé sous
  la consanguinité.
- **Identifiant avant l'établissement du patient**, et **recherche simulée du patient existant** :
  elle part dès que la paire établissement + identifiant est complète, quel que soit l'ordre de
  saisie. MRN **1234** au CHU Sainte-Justine trouve un dossier et préremplit RAMQ, prénom, nom,
  sexe et date de naissance ; tout autre numéro donne « nouveau patient » et vide ce qui avait
  été prérempli.

## Commentaires

_(Aucun pour l'instant.)_
