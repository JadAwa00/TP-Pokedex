# 🚀 TP - 02. NodeJS - Serveur API - Pokedex (+ TP - 02.Bis)

Serveur API REST réalisé en **NodeJS** + **Express** qui fournit les données du Pokédex.

Deux serveurs, un fichier par TP :

| Fichier | TP | Port | Contenu |
|---|---|---|---|
| `index.js` | 🚀 TP - 02 | `5001` | Serveur API : routes REST + fichiers statiques |
| `index-v2.js` | 🚀 TP - 02.Bis | `5002` | Idem + limitation du nombre de requêtes par IP |

## Arborescence

```
BACKEND
├── DATA
│   ├── items.json
│   ├── moves.json
│   ├── pokedex.json
│   └── types.json
├── FILES
│   ├── images
│   ├── sprites
│   └── thumbnails
├── index.js        # TP - 02
├── index-v2.js     # TP - 02.Bis
└── README.md
```

## Installation

```bash
# Dépendances
npm install express
```

## Lancement

```bash
# TP - 02 (port 5001)
node index.js
# ou, avec rechargement à chaque modification du fichier
node --watch index.js

# TP - 02.Bis (port 5002)
node index-v2.js
# ou
node --watch index-v2.js
```

- TP - 02 : `http://127.0.0.1:5001`
- TP - 02.Bis : `http://127.0.0.1:5002`

Les deux serveurs peuvent tourner en même temps (ports distincts).

## Routes

Toutes les routes sont préfixées par `/api/v1`. Elles sont communes aux deux serveurs.

| Route | Paramètre | Nom de la fonction | Description |
|---|---|---|---|
| `/api/v1/pokemons` | — | `findAll` | Renvoie TOUS les pokemons (809) |
| `/api/v1/pokemons/random` | — | `findOneByRandom` | Renvoie UN pokemon au hasard |
| `/api/v1/pokemons/{id}` | `id: integer` | `findOneById` | Renvoie UN pokemon à partir de son id. Si non trouvé renvoie **404** |
| `/api/v1/pokemons/{name}` | `name: string` | `findOneByName` | Renvoie UN pokemon à partir de son nom (anglais, français, japonais ou chinois). Si non trouvé renvoie **404** |

### Bonus — fichiers statiques

| Route | Dossier |
|---|---|
| `/images/*` | `FILES/images` |
| `/sprites/*` | `FILES/sprites` |
| `/thumbnails/*` | `FILES/thumbnails` |

Les fichiers sont nommés avec l'id sur 3 chiffres : `/images/001.png`.
Le dossier `sprites` est fourni vide sur le NAS.

##  TP - 02.Bis — Limitation des requêtes par IP

`index-v2.js` ajoute la **Mission 1** du TP - 02.Bis : limiter le nombre de requêtes par IP sur une période donnée, l'infrastructure ne pouvant absorber toutes les requêtes.

### Fonction `limitRequests`

```js
function limitRequests(req, maxRequests = 10, timeWindow = 60000) // -> boolean
```

Algorithme (cf. TP) :

1. récupérer l'adresse IP de la requête (`req.ip`) ;
2. définir un booléen d'autorisation ;
3. récupérer le moment actuel (`Date.now()`) ;
4. créer le tableau de comptage dans `req.app.locals.requestCounts` s'il n'existe pas ;
5. y associer un tableau de timestamps pour cette IP ;
6. supprimer les timestamps trop anciens (hors fenêtre) ;
7. si le nombre d'entrées ≥ la limite → requête refusée ;
8. sinon ajouter le timestamp de la requête.

### Paramètres utilisés

```js
const RATE_LIMIT_MAX = 10;       // 10 requêtes...
const RATE_LIMIT_WINDOW = 60000; // ...par IP et par minute (ms)
```

> L'exemple du TP appelle `limitRequests(request, 1, 60000)`, soit **1 seule requête par minute**. Les constantes ci-dessus permettent de tester toutes les routes tranquillement : changez `RATE_LIMIT_MAX` à `1` pour reproduire l'exemple exact du TP.

Un middleware applique la limite à **toutes les routes `/api/v1/*`** (les fichiers statiques ne sont pas concernés). Au-delà de la limite, la réponse est **429**.

### En-têtes renvoyés

| En-tête | Signification |
|---|---|
| `RateLimit-Limit` | nombre maximum de requêtes autorisées |
| `RateLimit-Remaining` | requêtes restantes dans la fenêtre |
| `RateLimit-Reset` | moment de réinitialisation (timestamp epoch en secondes) |
| `Retry-After` | secondes à attendre avant de réessayer (uniquement en 429) |


## Exemples d'appels

```bash
# Tous les pokemons
curl http://127.0.0.1:5001/api/v1/pokemons

# Un pokemon au hasard
curl http://127.0.0.1:5001/api/v1/pokemons/random

# Un pokemon par son id
curl http://127.0.0.1:5001/api/v1/pokemons/25

# Un pokemon par son nom
curl http://127.0.0.1:5001/api/v1/pokemons/pikachu

# Id ou nom inexistant -> 404
curl -i http://127.0.0.1:5001/api/v1/pokemons/999999
curl -i http://127.0.0.1:5001/api/v1/pokemons/zzzz
```

On peut aussi vérifier les routes directement dans le navigateur ou avec **Postman**.

## Versioning

Projet versionné avec **git**, dépôt distant **GitHub** :
commits et pushes réguliers.
