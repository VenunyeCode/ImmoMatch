# ImmoMatch
Conception et réalisation d’un site web responsive dédié à l’immobilier

ImmoMatch est une application web de recherche immobilière qui classe les biens selon le profil de recherche de l'utilisateur et le temps de trajet réel vers ses adresses du quotidien (travail, études, famille), et non selon la seule proximité géographique. Chaque bien affiché est accompagné d'un score explicable, détaillé critère par critère.

Projet réalisé dans le cadre de l'UV **AS52, Acquisition de compétences en autonomie** (FISE-INFO, Nexa Digital School), par une équipe de deux étudiants.

## Sommaire

- [Fonctionnalités principales](#fonctionnalités-principales)
- [Stack technique](#stack-technique)
- [Architecture](#architecture)
- [Démarrage rapide](#démarrage-rapide)
- [Structure du dépôt](#structure-du-dépôt)
- [Documentation](#documentation)
- [Équipe](#équipe)

## Fonctionnalités principales

- Recherche multicritère (budget, surface, ville, nombre de pièces, DPE, etc.) avec formulaire classique et recherche en langage naturel.
- Carte interactive synchronisée avec la liste des résultats, clustering des marqueurs.
- Calcul du temps de trajet vers jusqu'à 3 destinations enregistrées, sur 4 modes (voiture, transports, vélo, marche).
- ImmoMatch Score : note globale par bien, avec explication ligne par ligne des critères respectés ou non.
- Favoris, comparaison de biens (2 à 4), demande de visite avec gestion des créneaux.
- Espace agent : publication et gestion des annonces, traitement des demandes de visite, tableau de bord.
- Espace administrateur : modération des annonces et des signalements.
- Application responsive, accessible (WCAG AA visée).

## Stack technique

| Domaine        | Choix                                                     |
| --------------- | ------------------------------------------------------------ |
| Frontend         | Angular 18 (standalone components, signals)                  |
| Backend          | Spring Boot 3, Java                                           |
| Base de données  | PostgreSQL 16                                                 |
| Cartographie     | Leaflet, OpenStreetMap                                        |
| Calcul d'itinéraires | openrouteservice                                          |
| Déploiement      | Docker Compose                                                |
| CI/CD            | GitHub Actions                                                 |
| Qualité          | SonarCloud                                                     |

## Architecture

Monolithe modulaire, découpé par fonctionnalité (`annonce`, `profil`, `favoris`, `visite`, `moderation`, `recherche`, `score`, `geo`), avec une architecture en couches classique (controller, service, repository, entité) et des DTO obligatoires aux frontières de l'API. Le calcul de score et les intégrations géographiques suivent un style hexagonal léger, isolé derrière des interfaces, afin de rester testable et de pouvoir changer de fournisseur cartographique sans impact sur le reste du code.

Le détail des choix d'architecture, avec leur justification, est dans [`docs/cahier-des-charges.md`](docs/cahier-des-charges.md) (section 10) et dans [`CLAUDE.md`](CLAUDE.md).

## Démarrage rapide

### Prérequis

- Docker et Docker Compose
- Une clé API [openrouteservice](https://openrouteservice.org/) (offre gratuite suffisante)

### Installation

```bash
git clone <url-du-depot>
cd immomatch
cp .env.example .env
# renseigner ORS_API_KEY dans .env
docker compose up
```

L'application est ensuite accessible sur `http://localhost:4200`, l'API sur `http://localhost:8080`, et la documentation Swagger sur `http://localhost:8080/swagger-ui.html`.

Un jeu de données de démonstration (150 biens sur 3 villes, comptes particulier, agent et administrateur) est chargé automatiquement au premier démarrage.

## Structure du dépôt

```
immomatch/
├── frontend/          Application Angular
├── backend/            Application Spring Boot
├── docs/
│   ├── cahier-des-charges.md
│   └── backlog.md
├── docker-compose.yml
└── CLAUDE.md
```

## Documentation

- [Cahier des charges](docs/cahier-des-charges.md) : périmètre, règles métier, architecture, maquettes, modèle de données.
- [Backlog](docs/backlog.md) : liste des fonctionnalités, planning en 9 sprints, répartition du travail.
- [CLAUDE.md](CLAUDE.md) : conventions et décisions d'architecture à respecter lors du développement assisté.

## Équipe

Projet réalisé par deux étudiants du Master Data & AI, Nexa Digital School.
