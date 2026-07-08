# Suivi de projets (outil de feuille de route autonome)

Cet outil est une version personnalisée de la page « Triactis · Suivi implémentation IA » que tu m'as transmise à l'origine. Le fonctionnement est identique (Focus, Projets, Kanban, Gantt, Bilan, filtres, tags, dépendances, avancement en pourcentage), avec en plus deux ajouts pour ton usage quotidien : un accès en lecture seule pour ton n+1, et deux catégories supplémentaires (Société, Domaine) en plus des projets.

## Fichiers livrés

- `index.html` : l'application complète (HTML, CSS, JavaScript dans un seul fichier, sans dépendance à installer). C'est ce fichier qui doit être poussé sur GitHub Pages.
- `generer-code-acces.html` : petit outil local pour calculer le hash de tes codes d'accès (édition et lecture seule).
- `README.md` : ce document.

## 1. Démarrage rapide, sans rien configurer

Tu peux ouvrir `index.html` directement dans un navigateur (double-clic). Ton code d'édition actuel est celui que tu as déjà défini (hash `CODE_HASH` dans le fichier) ; il permet de tout modifier, comme avant.

## 2. Deux niveaux d'accès

Le fichier gère désormais deux codes distincts :

- **Code d'édition** (`CODE_HASH`) : c'est le tien. Accès complet, identique à avant.
- **Code de lecture seule** (`CODE_HASH_VIEW`) : à donner à ton n+1. La personne voit tout en direct (mêmes données, même synchronisation), mais aucun bouton d'ajout, de modification, de suppression ou de gestion des projets/tags n'est disponible. Par défaut, ce code est `lecture-2026` ; change-le avant de le transmettre.

Aucun mot de passe n'est stocké en clair dans le fichier : seul un hash (une empreinte à sens unique) y figure.

### Changer un code

1. Ouvre `generer-code-acces.html` dans un navigateur. Il contient deux blocs : un pour le code d'édition, un pour le code de lecture seule.
2. Tape le code souhaité dans le bloc correspondant, clique sur « Calculer le hash », puis copie le résultat.
3. Ouvre `index.html` dans un éditeur de texte, cherche `const CODE_HASH="..."` (ton code) ou `const CODE_HASH_VIEW="..."` (celui du n+1), et remplace la valeur entre guillemets par le hash copié.
4. Enregistre, puis republie le fichier (voir section 3).

Laisser `CODE_HASH_VIEW=""` désactive complètement l'accès en lecture seule.

## 3. Héberger sur GitHub Pages

GitHub Pages permet d'obtenir une adresse publique du type `https://tonpseudo.github.io/nom-du-depot/`.

1. Crée un compte GitHub si tu n'en as pas encore (gratuit) sur github.com.
2. Crée un nouveau dépôt (bouton « New repository »). Tu peux le rendre public ou privé ; en privé, GitHub Pages reste accessible publiquement à l'URL générée, sauf si tu passes à un forfait payant permettant de restreindre l'accès. Les codes d'accès intégrés à l'application restent donc ta seule barrière si le dépôt est public.
3. Dépose `index.html` à la racine du dépôt (glisser-déposer depuis l'interface web GitHub, ou via `git push` si tu es à l'aise avec Git).
4. Va dans l'onglet « Settings » du dépôt, puis « Pages » dans la barre latérale (rubrique « Code and automation »).
5. Sous « Build and deployment », choisis la source « Deploy from a branch », sélectionne la branche `main` et le dossier `/root`, puis enregistre.
6. Au bout de quelques minutes, l'URL publique s'affiche en haut de cette même page.

Chaque mise à jour de `index.html` (nouveau code, nouvelle configuration Firebase, etc.) se fait en remplaçant le fichier dans le dépôt ; GitHub Pages republie automatiquement.

## 4. Partage en temps réel (Firebase)

Ta configuration Firebase (`gz-monitoring`) est déjà intégrée dans `index.html` : dès qu'une base Firestore existe et que ses règles autorisent l'accès, toute personne qui ouvre la page profite du partage en temps réel, y compris ton n+1 en lecture seule.

### Sécuriser les règles Firestore

La configuration Firebase (`apiKey`, `projectId`, etc.) n'est pas un secret : elle est visible par quiconque consulte le code source de la page. La vraie protection vient des règles de sécurité Firestore, pas de cette configuration. Dans la console Firebase, section Firestore, onglet « Règles » :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /roadmaps/{roadmapId} {
      allow read, write: if true;
    }
  }
}
```

Cette règle limite l'accès au seul document `roadmaps/*` utilisé par l'application, mais reste ouverte à quiconque connaît ce chemin : c'est un compromis « pas de compte utilisateur à gérer » plutôt qu'une confidentialité stricte. Le code de lecture seule empêche l'édition depuis l'interface, mais ne remplace pas une vraie authentification côté Firestore. Pour des données réellement sensibles (informations clients, dossiers en cours), mieux vaut garder ce suivi pour la gestion de projet courante et ne pas y loger d'éléments relevant du post-NDA (litiges, passifs sensibles, informations d'associés).

## 5. Catégories : Projets, Sociétés, Domaines

En plus des projets, deux nouvelles catégories de tags sont pré-remplies à la première ouverture de la page après mise à jour :

- **Sociétés** : Groupe Malesherbes, Adonys, Triactis.
- **Domaines** : Développement web, Référencement web, Téléphonie d'entreprise, Intelligence artificielle, Administratif, Support informatique.

Elles apparaissent comme des groupes distincts dans le panneau de filtres (« ▾ Tags ») et dans « Gérer ». Pour classer une tâche, ajoute-lui le tag Société et/ou Domaine correspondant, comme n'importe quel tag.

Pour ajuster ces listes (ajouter, renommer, recolorer, supprimer) : bouton « Gérer » dans l'application, puis choisis la catégorie du tag dans le menu déroulant (Projet associé, ou Société / Domaine / Verticale / Transverse dans le groupe « Catégorie »). Rien n'est figé dans le code : tout se pilote depuis l'interface et se synchronise avec le reste de l'équipe si le partage est actif.

## 6. Personnaliser l'outil

- **Titre et sous-titre affichés en haut de page** : dans `index.html`, cherche `id="appTitle"` et `id="appSub"` (juste après `<body>`).
- **Nom affiché sur l'écran de connexion et titre de l'onglet du navigateur** : dans le `<script>`, les constantes `APP_TITLE`, `APP_SUBTITLE` et `APP_LOGO` (lettre du logo) ; elles s'appliquent automatiquement partout dans l'application.

## 7. Sauvegarde et export

Le bouton « ⋯ » (menu, à côté de « Gérer », masqué en lecture seule) permet d'exporter toutes les données en JSON (sauvegarde manuelle) ou d'en importer. Utile avant un changement de configuration important.

## 8. Réutiliser l'outil pour un autre projet

Le moyen le plus simple est de dupliquer `index.html` et de lui donner, dans la section de configuration du `<script>` :
- des `CODE_HASH` / `CODE_HASH_VIEW` différents (section 2),
- un `SHARED_DOC` différent si le partage Firebase est actif (par exemple `"roadmaps/projet-b"`), pour que les deux outils ne partagent pas les mêmes données,
- un `APP_TITLE` différent (section 6).

## Ce qu'il faut garder en tête

- Le code de lecture seule masque les boutons d'édition dans l'interface ; ce n'est pas une authentification côté serveur. La vraie barrière technique, si le partage Firebase est actif, ce sont les règles Firestore (section 4).
- Sans Firebase, aucune donnée ne quitte le navigateur : c'est le mode le plus simple et le plus étanche, mais sans partage automatique.
- Il n'y a pas de gestion de comptes utilisateurs ni de journal de qui a modifié quoi : c'est un outil léger, pas un outil de gouvernance de données sensibles.

## 9. Couleurs de marque et sociétés

Les couleurs de l'outil (bouton principal, logo, anneaux de progression) reprennent le violet de Groupe Malesherbes (`#552F91`) et le texte utilise `#323232`. Les tags Société ont chacun leur couleur (Groupe Malesherbes, Triactis, Adonys, Européenne d'Assurance) ; comme Triactis et Adonys partageaient la même couleur de marque, Adonys a été légèrement assombrie pour rester distinguable dans les filtres.

Un fichier `patch-couleurs-societes-*.json` est fourni pour mettre à jour les couleurs des tags Société déjà créés dans ton espace (l'auto-remplissage ne s'exécute qu'une seule fois et ne les aurait pas mis à jour tout seul). À importer une fois via « ⋯ » puis « Importer ».

Chaque tag (Société, Domaine ou autre) peut désormais aussi avoir sa couleur changée directement dans « Gérer », pas seulement son nom.

## 10. Sélection multiple

Une case à cocher apparaît sur chaque tâche dans les vues Focus et Projets. Cocher une ou plusieurs tâches fait apparaître une barre d'actions en bas de l'écran : changer le statut de toutes les tâches sélectionnées en une fois, ou les supprimer en masse. Cette fonctionnalité est désactivée en lecture seule, comme les autres actions d'édition.

## 11. Bornage temporel et statut Archivé

Une ligne « Période » dans le bandeau de filtres permet de borner l'affichage des tâches et l'avancement global : Aujourd'hui, Cette semaine, Ce mois-ci, 30 derniers jours, Année en cours, Année N-1, ou Personnalisé (deux dates). Un second choix permet de filtrer sur la date de début ou la date de fin des tâches. Par défaut, aucun bornage n'est actif et tout s'affiche.

Un statut « Archivé » est disponible (dans la fiche d'une tâche, ou en masse via la sélection multiple). Une tâche archivée reste dans les données mais disparaît de toutes les vues, de l'avancement global et des calculs, comme si elle n'existait plus. Pour la retrouver ou la désarchiver, utilise le filtre « Archivées » dans le bandeau de filtres.

## Sources

- [Configuring a publishing source for your GitHub Pages site (GitHub Docs)](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
- [Get started with Firestore (Firebase)](https://firebase.google.com/docs/firestore/quickstart)
- [Add Firebase to your JavaScript project](https://firebase.google.com/docs/web/setup)
