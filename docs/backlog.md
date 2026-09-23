# ImmoMatch - Backlog

Extrait de la section 18 du [cahier des charges](./cahier-des-charges.md). Complexité en points (échelle 1, 2, 3, 5, 8). Priorité : 1 indispensable à la soutenance, 2 importante, 3 confort, B bonus.

## Backlog MVP

| ID  | Fonctionnalité                          | US                   | Prio | Cplx | Sprint | Critères d'acceptation                                                                                                           |
| --- | ---------------------------------------- | -------------------- | ---- | ---- | ------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| F01 | Socle projet, Docker, Flyway, CI         |                       | 1    | 5    | 0      | docker compose up démarre les 3 services ; la pipeline passe sur une PR vide                                                        |
| F02 | Modèle de données et migrations          |                       | 1    | 5    | 0      | les 14 tables et index sont créés ; le jeu de démonstration charge 150 biens                                                        |
| F03 | Inscription et connexion JWT             |                       | 1    | 5    | 1      | un compte créé peut se connecter ; un jeton expiré renvoie 401 ; le mot de passe est haché                                          |
| F04 | Rôles et guards Angular                  |                       | 1    | 3    | 1      | /agent est inaccessible à un particulier côté front et côté API                                                                     |
| F05 | CRUD annonce                             | US-22, US-24          | 1    | 8    | 1      | un agent créé, modifié, publié et archivé ; un autre agent reçoit 403                                                               |
| F06 | Upload et gestion des photos             | US-23                 | 1    | 5    | 2      | 1 à 12 photos, ordre modifiable, photo principale, refus au-delà de 5 Mo                                                            |
| F07 | Recherche multicritère et pagination     | US-02, US-03, US-04   | 1    | 8    | 2      | tous les filtres de la section 4.1 fonctionnent et se combinent ; 12 résultats par page                                             |
| F08 | Page détail                              |                       | 1    | 3    | 2      | toutes les informations et la galerie s'affichent ; un identifiant inconnu donne une page 404                                       |
| F09 | Recherche naturelle                      | US-01, US-05          | 1    | 8    | 3      | chaque jeton s'édite et met à jour les résultats ; l'URL reflète l'état ; utilisable au clavier                                      |
| F10 | Carte interactive et liste synchronisée  | US-06, US-07, US-08   | 1    | 8    | 3      | marqueurs, clustering, survol croisé, recherche dans la zone                                                                        |
| F11 | Géocodage des adresses d'annonce         |                       | 1    | 3    | 3      | l'adresse saisie par l'agent produit des coordonnées ; le marqueur est ajustable                                                     |
| F12 | Calcul de trajet et cache                | US-10, US-11          | 1    | 8    | 4      | 4 modes affichés en page détail ; le second appel identique ne consomme pas de quota ; une panne renvoie 503 sans casser la page     |
| F13 | Destinations enregistrées                | US-12                 | 1    | 3    | 4      | 3 destinations maximum, une par défaut, sélecteur global                                                                            |
| F14 | Profil de recherche                      | US-14                 | 1    | 5    | 4      | création depuis une recherche, un seul profil actif                                                                                 |
| F15 | ImmoMatch Score et explication           | US-15, US-16          | 1    | 8    | 5      | score affiché sur les cartes et en détail ; explication ligne par ligne conforme à la section 7                                     |
| F16 | Favoris                                  | US-18                 | 1    | 3    | 5      | ajout et retrait depuis 3 emplacements ; doublon impossible                                                                         |
| F17 | Comparaison                              | US-19                 | 1    | 5    | 5      | 2 à 4 biens, meilleure valeur mise en évidence, prix au m2 calculé                                                                  |
| F18 | Demande de visite                        | US-20, US-21          | 1    | 5    | 6      | créneau à plus de 24 h, une demande active par bien, suivi du statut                                                                |
| F19 | Traitement des demandes par l'agent      | US-25                 | 1    | 3    | 6      | accepter, refuser avec motif ; le créneau accepté devient indisponible                                                              |
| F20 | Tableau de bord agent                    | US-26                 | 2    | 5    | 6      | 4 indicateurs, vues sur 30 jours par annonce                                                                                        |
| F21 | Espace administrateur et modération      | US-27, US-28, US-29   | 2    | 5    | 7      | rejet avec motif obligatoire, suspension d'agent qui dépublie ses annonces                                                          |
| F22 | Recommandations                          | US-17                 | 2    | 5    | 7      | 10 biens triés par score, critères obligatoires respectés                                                                           |
| F23 | Filtre par temps de trajet               | US-13                 | 2    | 5    | 7      | filtre à N minutes appliqué aux résultats                                                                                           |
| F24 | Responsive complet et accessibilité      | US-09                 | 1    | 8    | 7      | les 3 paliers de la section 15 sont conformes ; axe ne signale aucune erreur critique sur 3 pages                                   |
| F25 | Gestion des erreurs et états vides       |                       | 1    | 3    | 8      | les 11 cas de la section 20 sont couverts                                                                                           |
| F26 | Tests, documentation, finalisation       |                       | 1    | 8    | 8      | couverture conforme à la section 14 ; Swagger publié ; README d'installation valide sur une machine vierge                          |

## Bonus

| ID  | Fonctionnalité                                           | Prio | Cplx | Condition                                    |
| --- | ---------------------------------------------------------- | ---- | ---- | ----------------------------------------------- |
| B01 | Transports en commun réels via GTFS ou Navitia              | B    | 8    | remplace l'estimation de la section 6            |
| B02 | Isochrones sur la carte, zone atteignable en 20 minutes     | B    | 5    | endpoint ORS Isochrones, déjà dans le quota      |
| B03 | Alertes par courriel sur nouveau bien correspondant         | B    | 5    | nécessite un service SMTP                        |
| B04 | Export PDF de la comparaison                                | B    | 3    |                                                  |
| B05 | Recherche enregistrée avec historique                       | B    | 3    |                                                  |
| B06 | Mode sombre                                                 | B    | 2    | jetons de couleur déjà en variables CSS          |
| B07 | PostGIS et recherche par rayon réel                         | B    | 5    | remplace le filtre par emprise                   |
| B08 | Messagerie agent et particulier                             | B    | 8    | hors périmètre raisonnable                       |

## Planning

Neuf sprints d'une semaine, environ 12 à 15 heures par étudiant et par semaine, soit environ 250 heures au total.

| Sprint | Semaine | Objectif                                    | Démontrable à la fin                |
| ------ | ------- | --------------------------------------------- | -------------------------------------- |
| 0      | 1       | socle, modèle, CI                             | l'application vide tourne en Docker    |
| 1      | 2       | authentification, rôles, CRUD annonce         | un agent publie une annonce            |
| 2      | 3       | photos, recherche, page détail                | on cherche et on consulte un bien      |
| 3      | 4       | recherche naturelle, carte                    | la recherche signature fonctionne      |
| 4      | 5       | trajets, destinations, profil                 | le temps de trajet s'affiche           |
| 5      | 6       | score, favoris, comparaison                   | la promesse produit est complète       |
| 6      | 7       | visites, tableau de bord agent                | le cycle complet est jouable           |
| 7      | 8       | admin, recommandations, responsive            | tous les rôles sont couverts           |
| 8      | 9       | erreurs, tests, documentation, répétition     | version de soutenance figée            |

Deux jalons de sécurité : à la fin du sprint 4, la fonctionnalité différenciante doit être démontrable ; si elle ne l'est pas, F22, F23 et le tableau de bord agent passent en bonus. À la fin du sprint 7, le périmètre est gelé et plus aucune fonctionnalité n'est ajoutée.

## Rituels

- Planification de 30 minutes en début de semaine, avec engagement chiffré en points.
- Point de 15 minutes deux fois par semaine, en visio ou en présentiel.
- Revue et rétrospective de 45 minutes en fin de sprint, avec une seule action d'amélioration retenue.
- Tableau GitHub Projects à 4 colonnes : À faire, En cours, En revue, Terminé.

## Répartition du travail

Chaque étudiant est propriétaire de domaines fonctionnels complets, du composant Angular jusqu'à la requête SQL. Personne n'est cantonné à une couche.

| Étudiant A, parcours de recherche | Étudiant B, parcours de gestion   |
| ------------------------------------ | ------------------------------------ |
| F07 recherche multicritère           | F03 authentification et JWT          |
| F09 recherche naturelle              | F04 rôles et guards                  |
| F10 carte interactive                | F05 CRUD annonce                     |
| F11 géocodage des annonces           | F06 photos                           |
| F12 calcul de trajet et cache        | F08 page détail                      |
| F13 destinations                     | F18 demande de visite                |
| F15 ImmoMatch Score                  | F19 traitement par l'agent           |
| F16 favoris                          | F20 tableau de bord agent            |
| F17 comparaison                      | F21 administration et modération     |
| F14 profil de recherche              | F22 recommandations, côté écran      |
| F23 filtre par trajet                |                                       |

Travail commun, mené en binôme sur créneau partagé : F01 socle et CI, F02 modèle de données, F24 responsive et accessibilité, F25 gestion des erreurs, F26 tests et documentation, ainsi que le design system.

**Pourquoi ce découpage** : A porte la valeur différenciante du produit, qui est aussi la partie la plus risquée ; B porte le volume fonctionnel et les trois espaces utilisateurs. Les deux charges sont équilibrées en points : environ 60 points pour A et 55 pour B, plus 30 points de travail commun.

Le seul point de contact fort est le contrat d'API des annonces : B le définit et le documente au sprint 1, A le consomme. Le DTO `PropertySummary` est donc figé tôt et versionné dans le dépôt.

### Coordination

- Les DTO partagés sont écrits ensemble avant d'être implémentés, dans un fichier Markdown du dépôt, avant toute ligne de code.
- Chaque pull request est relue par l'autre ; la relecture sert aussi à transmettre la connaissance des domaines.
- Au moins une session de programmation en binôme par sprint, sur la partie la plus incertaine de la semaine.
- Le sprint 8 est intégralement commun : tests, documentation, correction et répétition de la soutenance.
- En cas de retard sur un domaine, le rattrapage se fait par transfert d'un élément de backlog entier, jamais par découpage d'une fonctionnalité en cours.
