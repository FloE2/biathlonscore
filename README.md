# Biathlon score – EPS YDM

Appli de cours pour le cycle biathlon (tir carabine laser) : 2 iPad juges (5 couloirs chacun), un PC admin qui règle la séance et garde l'historique du cycle, et un écran de projection en direct.

## Les écrans

| Écran | Adresse | Rôle |
|---|---|---|
| Accueil | `…/biathlonscore/` | choisir l'écran de l'appareil |
| iPad A | `…/biathlonscore/#juge-A` | l'élève juge note les tirs des couloirs 1 à 5 du pas de tir A |
| iPad B | `…/biathlonscore/#juge-B` | idem pour le pas de tir B |
| PC admin | `…/biathlonscore/#admin` | prénom de l'admin, classe, nombre de séries, tirs autorisés, historique, élèves, export |
| Projection | `…/biathlonscore/#projection` | les 10 couloirs en direct sur grand écran |

Astuce iPad : ouvrir l'adresse dans Safari puis Partager → « Sur l'écran d'accueil » pour l'avoir en plein écran.

## Fonctionnement d'une série (iPad)

1. **+ Qui tire ?** → toucher le prénom de l'élève. Chaque élève affiche le nombre de séries déjà faites aujourd'hui (en jaune s'il n'a pas encore tiré) ; ceux qui ont le moins tiré sont en premier, ceux déjà placés sont grisés. Toucher le nom pour changer de tireur, ✕ pour le retirer.
2. **Cible touchée** → toucher la cible (disque blanc ✓). Re-toucher annule.
3. **Tir manqué** → bouton « + Manqué » (le « − » corrige).
4. Le couloir se termine **tout seul** quand les 5 cibles sont blanchies ou que le nombre de tirs autorisés est atteint. Le bouton **Tir terminé** permet aussi d'arrêter un couloir avant.
5. Le score s'affiche en gros (vert 5/5 → rouge 0/5) et part aussitôt au PC. **Corriger** rouvre la saisie et met à jour le même score. **Série suivante** enchaîne avec le même tireur jusqu'au nombre de séries fixé sur le PC, puis **Libérer** vide le couloir.

Chaque couloir avance à son rythme.

## Mise en route Firebase (une seule fois, ~10 min)

Tant que `FIREBASE_CONFIG` vaut `null` dans `index.html`, l'appli tourne en **mode démo** : tout reste dans le navigateur (pratique pour tester avec plusieurs onglets sur un même PC).

1. https://console.firebase.google.com → **Ajouter un projet** (ex. `biathlonscore`), Google Analytics inutile.
2. **Build → Realtime Database → Créer une base**, emplacement **europe-west1 (Belgique)**, démarrer en mode verrouillé.
3. Onglet **Règles** de la base : coller le contenu de `database.rules.json`, puis **Publier**.
4. **Build → Authentication → Commencer → Anonyme → Activer**.
5. **Paramètres du projet (roue dentée) → Vos applications → Web `</>`** → enregistrer l'appli, copier l'objet `firebaseConfig`.
6. Dans `index.html`, remplacer `const FIREBASE_CONFIG = null;` par `const FIREBASE_CONFIG = { …la config copiée… };` (avec la ligne `databaseURL`).

## Première séance

1. PC admin → onglet **Élèves** → importer les fichiers Excel des classes (colonnes NOM et Prénom ; la classe est devinée depuis le nom du fichier, ex. `3A.xlsx`, sinon la saisir). Ajouter / modifier / supprimer un élève au même endroit.
2. Onglet **Séance** : choisir son prénom, la classe, le nombre de séries et le nombre de tirs autorisés (5 par défaut).
3. Ouvrir `#juge-A` et `#juge-B` sur les iPad, `#projection` sur le vidéoprojecteur.

## Fin de séance

Onglet **Séance** du PC → **Terminer la séance** : libère les 2 iPad. Les scores sont enregistrés au fil de l'eau ; chaque séance est rangée par date dans l'historique du cycle (fiche élève, exports). La carte montre aussi les élèves qui n'ont pas encore tiré.

## Données

Firebase Realtime Database (Europe) :
- `config` : admin, classe, nbSeries, tirsMax
- `eleves/{id}` : nom, prénom, classe
- `postes/{A|B}/lanes/{0-4}` : état en direct de chaque couloir
- `tirs/{id}` : une ligne par série terminée (élève, classe, date, couloir, série, touchées, manqués, tirs autorisés, admin)

Aucune liste d'élèves n'est stockée dans ce dépôt public : elles ne sont que dans la base Firebase.
Exports CSV (onglet **Export**) : tous les tirs, et un bilan par élève (moyenne, % de cibles blanchies, précision, progression 1re → dernière séance).
