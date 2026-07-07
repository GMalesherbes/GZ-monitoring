# Suivi de projets (outil de feuille de route autonome)

Cet outil est une version générique de la page « Triactis · Suivi implémentation IA » que tu m'as transmise. Le fonctionnement est identique (Focus, Projets, Kanban, Gantt, Bilan, filtres, tags, dépendances, avancement en pourcentage), mais il part vide, avec ta propre marque, ton propre code d'accès et ton propre espace de données. Tu peux le dupliquer pour suivre plusieurs projets en parallèle.

## Fichiers livrés

- `index.html` : l'application complète (HTML, CSS, JavaScript dans un seul fichier, sans dépendance à installer).
- `generer-code-acces.html` : petit outil local pour calculer le hash de ton propre mot de passe.
- `README.md` : ce document.

## 1. Démarrage rapide, sans rien configurer

Tu peux ouvrir `index.html` directement dans un navigateur (double-clic) dès maintenant. Le code d'accès par défaut est :

```
change-moi-2026
```

Dans ce mode, les données restent stockées dans le navigateur de la personne qui les saisit (stockage local). C'est suffisant pour un usage seul, mais rien n'est partagé entre appareils ou avec le reste de l'équipe tant que le partage en temps réel (section 4) n'est pas activé.

## 2. Changer le mot de passe

Le mot de passe n'est pas stocké en clair : le fichier ne contient qu'un hash (une empreinte) que l'on ne peut pas retransformer en mot de passe.

1. Ouvre `generer-code-acces.html` dans un navigateur.
2. Tape le code d'accès que tu veux utiliser, puis clique sur « Calculer le hash ».
3. Copie le hash obtenu.
4. Ouvre `index.html` dans un éditeur de texte, cherche la ligne suivante (au début de la section `<script>`) :
   ```
   const CODE_HASH="895350e8";
   ```
5. Remplace `895350e8` par ton nouveau hash, puis enregistre le fichier.

## 3. Héberger sur GitHub Pages

GitHub Pages permet d'obtenir une adresse publique du type `https://tonpseudo.github.io/nom-du-depot/`, exactement comme l'outil Triactis d'origine.

1. Crée un compte GitHub si tu n'en as pas encore (gratuit) sur github.com.
2. Crée un nouveau dépôt (bouton « New repository »). Tu peux le rendre public ou privé ; en privé, GitHub Pages reste accessible publiquement à l'URL générée, sauf si tu passes à un forfait payant permettant de restreindre l'accès. Le code d'accès intégré à l'application reste donc ta seule barrière si le dépôt est public.
3. Dépose `index.html` à la racine du dépôt (glisser-déposer depuis l'interface web GitHub, ou via `git push` si tu es à l'aise avec Git).
4. Va dans l'onglet « Settings » du dépôt, puis « Pages » dans la barre latérale (rubrique « Code and automation »).
5. Sous « Build and deployment », choisis la source « Deploy from a branch », sélectionne la branche `main` et le dossier `/root`, puis enregistre.
6. Au bout de quelques minutes, l'URL publique s'affiche en haut de cette même page. C'est l'adresse à partager.

Chaque mise à jour de `index.html` (nouveau mot de passe, nouvelle configuration Firebase, etc.) se fait en remplaçant le fichier dans le dépôt ; GitHub Pages republie automatiquement.

## 4. Activer le partage en temps réel (optionnel, recommandé pour une équipe)

Sans cette étape, chaque personne a sa propre copie des données dans son navigateur. Avec Firebase (offert par Google, gratuit dans les volumes d'usage d'une petite équipe), tout le monde voit et modifie les mêmes données en direct, comme dans l'outil Triactis d'origine.

### 4.1 Créer le projet Firebase

1. Va sur `console.firebase.google.com` et clique sur « Ajouter un projet ». Suis les instructions (nom du projet, Google Analytics facultatif).
2. Dans le panneau de gauche, ouvre la section base de données Firestore, puis clique sur « Créer une base de données ».
3. Choisis l'édition standard, un identifiant de base et une région (une région européenne si tes données concernent des clients européens), puis valide.
4. Pour démarrer simplement, choisis le mode test (accès ouvert temporaire, à durcir ensuite : voir 4.3).

### 4.2 Récupérer la configuration et l'intégrer

1. Dans les paramètres du projet (icône en forme d'engrenage, à côté de « Vue d'ensemble du projet »), descends jusqu'à la section des applications et clique sur l'icône « `</>` » pour enregistrer une application web.
2. Donne-lui un nom, puis copie l'objet `firebaseConfig` affiché (il ressemble à `{ apiKey: "...", authDomain: "...", projectId: "...", ... }`).
3. Deux façons de l'utiliser :
   - **Par personne** : dans l'application, clique sur le bouton « Partage », colle la configuration, puis « Activer ». Cette configuration reste alors propre au navigateur de cette personne.
   - **Pour tout le monde automatiquement (recommandé pour une équipe)** : ouvre `index.html`, repère le bloc suivant en haut du `<script>` :
     ```
     const FIREBASE_CONFIG = {
       // ...
     };
     ```
     et remplace le contenu par les valeurs copiées, par exemple :
     ```
     const FIREBASE_CONFIG = {
       apiKey: "...",
       authDomain: "...",
       projectId: "...",
       storageBucket: "...",
       messagingSenderId: "...",
       appId: "..."
     };
     ```
     Enregistre, puis republie le fichier (étape 3 si tu utilises GitHub Pages). Toute personne qui ouvre la page profite alors du partage en temps réel sans rien configurer elle-même.

### 4.3 Sécuriser les règles Firestore

Point important à ne pas sauter : la configuration Firebase (`apiKey`, `projectId`, etc.) n'est pas un secret ; elle est visible par quiconque consulte le code source de la page, que ce soit toi qui l'intègres au fichier ou une autre personne qui la colle via le bouton « Partage ». La vraie protection des données vient des règles de sécurité Firestore, pas de cette configuration.

Le mode test par défaut autorise toute personne connaissant ton `projectId` à lire et écrire pendant 30 jours, puis se ferme automatiquement. Avant que cette échéance arrive (ou idéalement tout de suite), remplace les règles par quelque chose de plus restreint. Dans la console Firebase, section Firestore, onglet « Règles » :

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

Cette règle limite l'accès au seul document `roadmaps/*` utilisé par l'application (au lieu de toute la base), mais reste ouverte à quiconque connaît ce chemin : c'est un compromis « pas de compte utilisateur à gérer » plutôt qu'une confidentialité stricte. Pour des données réellement sensibles (informations clients, dossiers en cours), mieux vaut soit garder le mode local uniquement (section 1, sans Firebase), soit ajouter une authentification Firebase (étape non couverte ici, à me demander si besoin). Dans tous les cas, ne loge pas dans cet outil des éléments relevant du post-NDA (litiges, passifs sensibles, informations d'associés).

## 5. Personnaliser l'outil

- **Projets et tags** : bouton « Gérer » dans l'application. Rien n'est codé en dur, tout se pilote depuis l'interface, et c'est synchronisé si le partage est actif.
- **Titre et sous-titre affichés en haut de page** : dans `index.html`, cherche `id="appTitle"` et `id="appSub"` (juste après `<body>`) et remplace le texte.
- **Nom affiché sur l'écran de connexion** : dans le `<script>`, les constantes `APP_TITLE`, `APP_SUBTITLE` et `APP_LOGO` (lettre du logo).

## 6. Sauvegarde et export

Le bouton « ⋯ » (menu, à côté de « Gérer ») permet d'exporter toutes les données en JSON (sauvegarde manuelle) ou d'en importer. Utile avant un changement de configuration important, ou pour transférer les données d'un espace local vers un espace partagé.

## 7. Réutiliser l'outil pour un autre projet

Le moyen le plus simple est de dupliquer `index.html` (par exemple `index-projet-b.html`) et de lui donner, dans la section de configuration du `<script>` :
- un `CODE_HASH` différent (section 2),
- un `SHARED_DOC` différent si le partage Firebase est actif (par exemple `"roadmaps/projet-b"` au lieu de `"roadmaps/mon-espace"`), pour que les deux outils ne partagent pas les mêmes données,
- un `APP_TITLE` différent (section 5).

Chaque copie peut vivre sur sa propre page GitHub Pages (un dépôt par projet, ou plusieurs fichiers `.html` dans le même dépôt).

## Ce qu'il faut garder en tête

- Le code d'accès protège l'affichage dans l'application, pas les données elles-mêmes si le partage Firebase est actif : la vraie barrière, dans ce cas, ce sont les règles Firestore (section 4.3).
- Sans Firebase, aucune donnée ne quitte le navigateur : c'est le mode le plus simple et le plus étanche, mais sans partage automatique.
- Il n'y a pas de gestion de comptes utilisateurs ni de journal de qui a modifié quoi : c'est un outil léger, pas un outil de gouvernance de données sensibles.

## Sources

- [Configuring a publishing source for your GitHub Pages site (GitHub Docs)](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
- [Get started with Firestore (Firebase)](https://firebase.google.com/docs/firestore/quickstart)
- [Add Firebase to your JavaScript project](https://firebase.google.com/docs/web/setup)
