# ImmoMatch - Instructions pour l'agent Claude

Projet AS52 (Acquisition de compétences en autonomie, FISE-INFO, UTBM).
Deux étudiants, MVP en 9 sprints. Ce fichier fixe les décisions d'architecture
prises par l'équipe : l'agent doit les suivre, pas en proposer d'autres.

## Contexte du projet

ImmoMatch est une application web responsive qui classe les biens immobiliers
selon le profil de recherche de l'utilisateur et le temps de trajet vers ses
adresses du quotidien. Le cahier des charges complet est dans `docs/cahier-des-charges.md`.

## Stack

- Frontend : Angular 18 (standalone components, signals), consomme l'API REST
- Backend : Spring Boot 3, Java
- Base de données : PostgreSQL 16
- Cartographie et routage : Leaflet + OpenStreetMap + openrouteservice (quotas gratuits limités)
- Déploiement : Docker Compose (un seul déployable, pas de microservices)

## Architecture backend

Monolithe modulaire, package par fonctionnalité. Ne pas proposer une architecture
microservices, SOA ou une hexagonale généralisée : le projet est trop petit pour
ça et ce n'est pas ce qui est évalué dans cette UV.

Structure des packages :

```
com.immomatch
├── annonce
├── profil
├── favoris
├── visite
├── moderation
├── recherche
├── score
├── geo          (géocodage, calcul de trajet)
└── commun       (config, sécurité, exceptions)
```

Chaque package fonctionnel suit le schéma classique Spring :

```
controller -> service -> repository -> entité
```

- Controller : fin, juste mapping requête/réponse et délégation. Ne jamais y mettre
  de logique métier.
- Service : un service par ressource ou agrégat, pas un service par cas d'usage.
  Exemple : un seul `AnnonceService` avec les méthodes `creer`, `modifier`, `publier`,
  `archiver` plutôt que des classes séparées par opération.
- Repository : Spring Data JPA standard, une interface par agrégat racine.
- DTO obligatoires aux frontières de l'API (`PropertyRequest`, `PropertySummary`,
  `PropertyDetail`, etc.) : ne jamais exposer une entité JPA directement dans un
  controller. Mapping via MapStruct.

### Exception : le score et les intégrations géo

Les packages `score` et `geo` suivent un style hexagonal léger, parce que c'est
la seule partie du projet où ça apporte une vraie valeur (logique métier à tester
isolément, dépendance externe avec quota limité et risque de panne) :

- Le cœur du calcul de score (`ScoreService`, pondération, normalisation par
  critère) est du Java pur, sans annotation Spring, sans dépendance JPA directe.
- Il dépend d'interfaces (`GeocodingPort`, `RoutingPort`) plutôt que d'appeler
  directement le client HTTP openrouteservice.
- Les implémentations concrètes (client openrouteservice, cache `route_cache`)
  vivent dans des classes d'adaptateur séparées, injectées via Spring.

Ne pas appliquer ce style hexagonal au reste du backend (annonces, favoris,
visites, comptes) : ça reste du CRUD classique, et le sur-découper ne démontre
rien, ça ralentit juste le développement.

## Règles métier

Les règles métier (RG-A1 à RG-S3, cf. cahier des charges sections 4 à 9) doivent
être implémentées telles quelles, sans les simplifier ni les étendre sans le
demander explicitement.

## Style de code

- Code simple, niveau intermédiaire. Pas de pattern appliqué pour le plaisir du
  pattern, pas de couche d'abstraction sans au moins deux implémentations
  réelles ou un besoin de test isolé.
- Pas de caractères spéciaux exotiques dans le code (pas de tiret cadratin,
  pas de guillemets typographiques).
- Noms de variables et de méthodes en anglais (convention Java standard),
  commentaires en français si nécessaire.
- Toujours écrire les tests unitaires en même temps que le code, surtout pour
  `ScoreService` : couvrir les cas limites (critère absent du profil, budget
  dépassé, DPE non renseigné, poids exclu du dénominateur).

## Workflow attendu

- Avancer fonctionnalité par fonctionnalité (voir le backlog F01 à F26 dans
  `docs/backlog.md`), pas de génération massive en un seul prompt.
- Toujours expliquer les choix faits quand plusieurs options sont possibles,
  pas juste produire du code.
- Branches nommées `feat/xxx`, `fix/xxx` ou `docs/xxx`, avec le numéro de
  ticket dans le message de commit.
- Ne jamais commit automatiquement sans relecture humaine.

## À ne pas faire

- Ne pas proposer de microservices, de SOA, ou d'architecture événementielle
  (event sourcing, CQRS complet) : hors de propos pour ce projet.
- Ne pas ajouter de dépendance ou de librairie sans la justifier.
- Ne pas dupliquer la logique de score ou de géocodage à plusieurs endroits :
  elle doit rester centralisée dans les packages `score` et `geo`.
