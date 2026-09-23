**ImmoMatch : Cahier des charges**

Projet AS52 · Conception et réalisation d'un site web responsive dédié à l'immobilier

**1. Cadrage du projet**

ImmoMatch est une application web responsive qui classe les biens immobiliers selon le projet réel de l'utilisateur et selon le temps de trajet vers ses adresses du quotidien.

Le produit ne cherche pas à égaler le catalogue de Leboncoin ou SeLoger. Il se différencie sur trois mécanismes, qui structurent tout le reste du document.

|                                      |                                                                       |             |
|--------------------------------------|-----------------------------------------------------------------------|-------------|
| **Différenciateur**                  | **Promesse utilisateur**                                              | **Section** |
| Recherche naturelle                  | Exprimer son besoin en une phrase à compléter, sans formulaire        | 3           |
| Carte + temps de trajet personnalisé | Savoir en combien de minutes on rejoint son travail ou son université | 5 et 6      |
| ImmoMatch Score                      | Comprendre en un coup d'œil pourquoi un bien correspond ou non        | 7           |

**Problématique**

Comment simplifier la recherche immobilière grâce à une interface intuitive, une recherche multicritère, une visualisation cartographique et une personnalisation des résultats selon les critères et les contraintes géographiques de l'utilisateur ?

**Objectifs**

1.  Réduire l'effort de saisie des critères à une phrase éditable plutôt qu'un formulaire de dix champs.

2.  Rendre la contrainte de mobilité visible sur chaque bien, et non calculable a posteriori sur un autre site.

3.  Expliquer chaque résultat par un score lisible et décomposé critère par critère.

4.  Livrer une application responsive, accessible et testée, déployable par Docker Compose.

|                                                        |                                                |
|--------------------------------------------------------|------------------------------------------------|
| **Inclus dans le MVP**                                 | **Exclu du MVP**                               |
| Recherche, annonces, page détail, favoris, comparaison | Paiement en ligne, signature de bail, mandat   |
| Carte interactive et temps de trajet multimodal        | Visite virtuelle 3D, vidéo, visite immersive   |
| ImmoMatch Score et profil de recherche                 | Messagerie temps réel agent / particulier      |
| Espaces particuliers, agent et administrateur          | Application mobile native, notifications push  |
| Authentification, rôles, modération des annonces       | Import automatique de flux d'annonces externes |

**Contraintes**

- Équipe de 2 étudiants, environ 10 à 12 semaines, temps partiel.

- Données d'annonces saisies manuellement ou générées par un jeu de données de démonstration (environ 150 à 300 biens sur 3 à 5 villes).

- Budget APIs cartographiques nul ou quasi nul, ce qui conditionne les choix de la section 5.

- Le MVP doit être démontrable de bout en bout le jour de la soutenance, même si le réseau est instable.

**2. Acteurs et rôles**

Quatre acteurs, dont un non authentifié. Un compte porte exactement un rôle, stocké en base et vérifié côté serveur à chaque appel.

|                  |                 |                    |                                                                        |
|------------------|-----------------|--------------------|------------------------------------------------------------------------|
| **Acteur**       | **Authentifié** | **Rôle technique** | **Ce qu'il vient faire**                                               |
| Visiteur         | Non             | Aucun              | Chercher, consulter, voir la carte                                     |
| Particulier      | Oui             | RÔLE_USER          | Favoris, comparaison, profil de recherche, trajets, demandes de visite |
| Agent immobilier | Oui             | RÔLE_AGENT         | Publier et gérer ses annonces, traiter les demandes de visite          |
| Administrateur   | Oui             | RÔLE_ADMIN         | Modérer, gérer les comptes, suivre les statistiques globales           |

**Matrice des droits**

|                                                       |              |                 |              |           |
|-------------------------------------------------------|--------------|-----------------|--------------|-----------|
| **Action**                                            | **Visiteur** | **Particulier** | **Agent**    | **Admin** |
| Rechercher et consulter une annonce publiée           | Oui          | Oui             | Oui          | Oui       |
| Calculer un temps de trajet                           | Oui, limite  | Oui             | Oui          | Oui       |
| Favoris, comparaison sauvegardée, profil de recherche | Non          | Oui             | Non          | Non       |
| Demander une visite                                   | Non          | Oui             | Non          | Non       |
| Créer et modifier une annonce                         | Non          | Non             | Ses annonces | Toutes    |
| Accepter ou refuser une demande de visite             | Non          | Non             | Ses annonces | Non       |
| Suspendre un compte, modérer une annonce              | Non          | Non             | Non          | Oui       |
| Statistiques globales                                 | Non          | Non             | Les siennes  | Oui       |

**Règles métier d'accès**

- RG-A1 : un agent ne voit et ne modifie que les annonces dont il est propriétaire, ou celles de son agence s'il en est le gestionnaire.

- RG-A2 : une annonce n'est visible publiquement que si son statut vaut PUBLISHED et que son agent est actif.

- RG-A3 : un particulier ne peut demander une visite que sur une annonce publiée, et une seule demande active par bien.

- RG-A4 : l'administrateur ne crée pas d'annonce ; il peut la masquer, la rejeter ou la restaurer, avec un motif obligatoire.

- RG-A5 : un visiteur non connecté dispose de 3 calculs de trajet par session, puis est invité à créer un compte. Cette limite protège le quota d'API et sert d'incitation à l'inscription.

- RG-A6 : la suppression d'un compte particulier supprime ses favoris, profils et destinations ; ses demandes de visite sont anonymisées et conservées pour l'agent.

**3. Recherche naturelle**

La recherche est une phrase éditable dont chaque variable est un jeton cliquable. Il n'y a pas de champ libre à parser : chaque jeton ouvre un petit sélecteur adapté à son type, ce qui évite tout traitement de langage naturel.

Exemple : Je cherche \[ un appartement \] à \[ Strasbourg \]

pour \[ moins de 900 EUR / mois \] avec \[ 2 chambres minimum \] \[ + \]

Un jeton non renseigné s'affiche en style pointillés avec un libellé d'invité (par exemple un bien, n'importe où). La recherche reste lançable dès le premier jeton rempli.

**Les jetons**

| **Jeton**        | **Éditeur ouvert au clic**                                              | **Valeur par défaut** | **Obligatoire** |
|------------------|-------------------------------------------------------------------------|-----------------------|-----------------|
| Transaction      | Deux boutons, louer ou acheter                                          | Louer                 | Oui             |
| Type de bien     | Liste à choix multiple (appartement, maison, studio, terrain)           | Tous                  | Non             |
| Localisation     | Champ avec autocomplétions de communes, choix multiple, rayon optionnel | Aucune                | Oui             |
| Budget           | Curseur double + saisie, unité adaptée à la transaction                 | Aucun                 | Non             |
| Chambres, pièces | Pas à pas 1 à 5+                                                        | Aucun                 | Non             |
| Surface          | Curseur double, m2                                                      | Aucune                | Non             |
| DPE              | Échelle A à G, on choisit le minimum accepté                            | Aucun                 | Non             |
| Équipements      | Chips à bascule (parking, balcon, ascenseur, jardin)                    | Aucun                 | Non             |
| Trajet           | Adresse + mode + durée maximale                                         | Aucun                 | Non             |

Le bouton « + » ajoute les jetons facultatifs non encore présents, sous forme de menu. La phrase se réécrit grammaticalement : avec un jardin, sans ascenseur, à moins de 20 minutes de mon travail.

**Desktop**

La phrase occupe une bande centrale sur la page d'accueil, puis se replié en barre collante en haut des résultats. Le clic sur un jeton ouvre un pop over ancre sous lui, largeur 280 à 360 px, ferme par Échap ou clic extérieur. La navigation clavier suit l'ordre de lecture : Tab passe de jeton en jeton, Entrée ou Espace ouvre l'éditeur, les flèches parcourent les options.

Les résultats se rafraîchissent après validation d'un jeton, avec une attente de 300 ms, sans rechargement de page. L'URL reflète l'état complet de la recherche pour permettre le partage et le retour arrière.

**Dégradation et assistance**

- Auto-complétions de localisation : appel au géocodage à partir de 3 caractères, attente de 300 ms, 5 suggestions maximum, cache local des 20 dernières recherches.

- Si le géocodage est indisponible, le jeton accepte la saisie libre et la recherche se fait sur le champ ville de la base.

- Une combinaison qui ne renvoie rien propose de relâcher le critère le plus filtrant, identifié en comptant les résultats critère par critère.

- Un lien Vue classique affiche les mêmes critères sous forme de formulaire, pour l'accessibilité et pour les utilisateurs qui préfèrent ce mode. Les deux vues partagent le même modèle d'état.

**4. Fonctionnalités du MVP**

Six blocs fonctionnels. Les critères d'acceptation détaillés sont dans le backlog, section 18.

**4.1 Recherche et résultats**

Les critères de la section 3 sont envoyés en paramètres de requête à GET /api/properties. Les résultats s'affichent en liste paginée, 10 par page en desktop, défilement infini par lots de 10 en mobile.

Tris disponibles : pertinence (IMMO Match Score si un profil est actif, sinon date), prix croissant, prix décroissant, surface, prix au m2, date de publication, temps de trajet si une destination est active.

- RG-R1 : les filtres se combinent en ET, sauf les valeurs multiples d'un même critère qui se combinent en OU (par exemple appartement OU studio).

- RG-R2 : le filtre DPE sélectionne les classes égales ou meilleures que la valeur choisie.

- RG-R3 : un bien sans DPE renseigné est conservé dans les résultats mais signalé, et pénalisé dans le score.

- RG-R4 : le tri par temps de trajet n'est proposé que si une destination est active, et ne porte que sur les biens dont le trajet a été calculé.

**4.2 Annonce**

|                             |                                      |                 |                                                  |
|-----------------------------|--------------------------------------|-----------------|--------------------------------------------------|
| **Champ**                   | **Type**                             | **Obligatoire** | **Règle**                                        |
| Titre                       | Texte 10 à 120 caractères            | Oui             |                                                  |
| Description                 | Texte 50 à 3000 caractères           | Oui             | Texte brut, pas de HTML                          |
| Transaction                 | RENT ou SALE                         | Oui             | Détermine l'unité de prix                        |
| Type de bien                | Énumération                          | Oui             |                                                  |
| Prix                        | Décimal \> 0                         | oui             | Loyer mensuel charges comprises ou prix de vente |
| Charges                     | Décimal \>= 0                        | Non             | Location uniquement                              |
| Surface                     | decimal \>= 9 m2                     | Oui             |                                                  |
| Pièces, chambres            | Entier \>= 1                         | Oui             | Chambres \< pièces                               |
| Adresse, ville, code postal | Texte                                | Oui             | Géocodée à l'enregistrement                      |
| Latitude, longitude         | Décimal                              | Oui             | Issues du géocodage, modifiables par l'agent     |
| DPE                         | A à G ou NC                          | Non             |                                                  |
| Équipements                 | Liste                                | Non             |                                                  |
| Photos                      | 1 à 12 fichiers                      | Oui, au moins 1 | Jpg, png, webp, 5 Mo maximum                     |
| Statut                      | DRAFT, PUBLISHED, ARCHIVED, REJECTED | Oui             | Voir le cycle de vie                             |

**4.3 Page détail**

La galerie photos, les infos clés, le DPE, la carte, le score expliqué, le trajet, l'agent et les boutons favori et visite: tout est présenté sur la page détail, dont la composition est précisée en section 16.

**4.4 Favoris**

Ajout et retrait depuis la carte de résultat, la page détail et la carte interactive. Réservé aux comptes particuliers ; un clic non connecté ouvre la modale de connexion et rejoue l'ajout après authentification.

- RG-F1 : un couple utilisateur / bien est unique, contrainte d'unicité en base.

- RG-F2 : un favori dont l'annonce passe en ARCHIVED reste visible, marqué comme plus disponible, et n'est plus comparable.

**4.5 Comparaison**

Deux à quatre biens, sélectionnés depuis les favoris ou les résultats. Tableau à colonnes fixes, une ligne par critère : prix, charges, surface, prix au m2, pièces, chambres, DPE, équipements, ville, temps de trajet vers la destination active, ImmoMatch Score.

- RG-C1 : la meilleure valeur de chaque ligne est mise en évidence ; pour le DPE et le temps de trajet, meilleur signifie plus faible.

- RG-C2 : comparer un bien en location et un bien en vente est autorisé mais affiche un avertissement, les prix n'étant pas comparables.

- RG-C3 : le prix au m2 est calculé, jamais saisi.

**4.6 Demande de visite**

Le particulier choisit une date et un créneau parmi les disponibilités de l'agent, ajoute un message facultatif, et envoie. L'agent accepte, refuse ou propose un autre créneau.

- RG-V1 : une seule demande active (PENDING ou ACCEPTED) par couple utilisateur / bien.

- RG-V2 : un créneau doit être au moins 24 heures dans le futur.

- RG-V3 : l'acceptation d'une demande rend le créneau indisponible pour les autres demandes du même bien.

- RG-V4 : chaque changement d'état génère une notification interne pour l'autre partie.

**5. Cartographie**

Choix retenu : Leaflet pour l'affichage, fonds de carte OpenStreetMap, et openrouteservice pour le géocodage et le routage. Ce trio est gratuit, sans carte bancaire, et suffisant pour le volume du projet.

**Comparaison des solutions**

|                        |                                                                                |                                                                                        |                                                                |
|------------------------|--------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|----------------------------------------------------------------|
| **Critère**            | **Google Maps Platform**                                                       | **Mapbox GL JS**                                                                       | **Leaflet + OSM + openrouteservice**                           |
| Gratuité réelle        | 10 000 appels par SKU et par mois depuis mars 2025, carte bancaire obligatoire | 50 000 chargements de carte et 100 000 requêtes de géocodage ou de directions par mois | quotas jour : 2 000 directions, 1 000 géocodages, 500 matrices |
| Risque de facturation  | Réel, pas de plafond dur, alertes a posteriori                                 | Réel au-delà du seuil                                                                  | Nul, l'appel est refusé une fois le quota atteint              |
| Bibliothèque front     | SDK propriétaire                                                               | Mapbox GL JS, WebGL                                                                    | Leaflet, 42 ko, API simple                                     |
| Rendu                  | Très familier                                                                  | Le plus soigné                                                                         | Correct, personnalisable par le fond de tuiles                 |
| Modes de transport     | Voiture, transports, vélo, marche                                              | Voiture, vélo, marche                                                                  | Voiture, vélo, marche ; transports non couverts                |
| Licence et attribution | Conditions Google, pas de stockage durable des résultats                       | Conditions Mapbox                                                                      | ODbL, attribution OSM obligatoire sur la carte                 |
| Intégration Angular    | Wrapper à écrire ou paquet tiers                                               | Wrapper à écrire                                                                       | Ngx-leaflet ou intégration directe, la plus simple             |

Sources : [<u>Google Maps Platform, FAQ facturation</u>](https://developers.google.com/maps/billing-and-pricing/faq), [<u>tarifs Mapbox</u>](https://www.mapbox.com/pricing), [<u>plans openrouteservice</u>](https://openrouteservice.org/plans/) et [<u>restrictions openrouteservice</u>](https://openrouteservice.org/restrictions/). Quotas relevés le 16 septembre 2026.

**Justification**

Google Maps est écarté pour une raison simple : depuis mars 2025, le crédit mensuel de 200 dollars n'existe plus, chaque SKU a son propre plafond gratuit et aucun plafond de dépense dur n'est disponible. Un bug de boucle dans un projet étudiant peut donc produire une facture. Mapbox reste un excellent second choix, avec un quota plus large que celui d'openrouteservice, mais exige aussi une carte bancaire.

Le point faible du choix retenu est l'absence de routage en transports en commun. La section 6 décrit comment ce mode est traité.

**Volumétrie estimée**

|                          |                                                     |                             |
|--------------------------|-----------------------------------------------------|-----------------------------|
| **Appel**                | **Volume attendu en démonstration**                 | **Marge sur le quota jour** |
| Chargement de tuiles     | Illimité en pratique                                | Sans objet                  |
| Géocodage d'annonce      | Une fois à la création, mis en cache en base        | Très large                  |
| Autocomplétions de ville | Environ 100 par jour de développement               | 1 000 par jour              |
| Itinéraire               | 3 modes par couple bien / destination, mis en cache | 2 000 par jour              |

Le cache décrit en section 6 est ce qui rend ces quotas confortables.

**6. Temps de trajet personnalisé**

C'est la fonctionnalité signature du produit. L'utilisateur saisit une adresse personnelle, par exemple son travail ou son université, et chaque bien affiche le temps de trajet vers cette adresse dans quatre modes.

Appartement T3 - Strasbourg 850 EUR / mois - 64 m2

Mon trajet vers \[ Université de Strasbourg \]

Voiture 18 min transports 24 min

Vélo 16 min marche 42 min recalculé il y a 2 j

**Chaine de traitement**

flowchart TD

A\[Adresse saisie\] --\> B\[Géocodage\<br/\>ORS Géocodé Search\]

B --\> C\[Destination enregistrée\<br/\>lat, lon, label\]

C --\> D{Trajet en cache ?}

D --\>\|oui\| E\[Lecture en base\]

D --\>\|non\| F\[Appel ORS Directions\<br/\>3 profils\]

F --\> G\[Écriture en cache\<br/\>route_cache\]

G --\> E

E --\> H\[Affichage durée et distance\]

Le calcul est déclenché côté backend, jamais depuis le navigateur : la clé API reste secrète et le cache est partagé par tous les utilisateurs.

**Les quatre modes**

|                      |                                 |                       |
|----------------------|---------------------------------|-----------------------|
| **Mode**             | **Profil openrouteservice**     | **Source**            |
| Voiture              | Driving-car                     | Appel réel            |
| Vélo                 | Cycling-regular                 | Appel réel            |
| Marche               | Foot-walking                    | Appel réel            |
| Transports en commun | Non couvert par le plan gratuit | Estimation documentée |

L'estimation transports applique un coefficient de 1,35 à la durée voiture, plus 6 minutes forfaitaires d'attente, uniquement dans les villes dotées d'un réseau urbain. Le chiffre est affiché avec la mention estimation et une infobulle qui explique la méthode. Cette honnêteté d'affichage est préférable à un chiffre faux présenté comme exact. En bonus, l'intégration d'un fichier GTFS local ou de l'API Navitia remplacerait cette estimation par un calcul réel.

**Points d'accès à la fonctionnalité**

|                   |                                                                        |
|-------------------|------------------------------------------------------------------------|
| **Emplacement**   | **Comportement**                                                       |
| Carte de résultat | Badge du mode préféré uniquement, calculé en lot pour la page affichée |
| Carte interactive | Durée dans l'infobulle du marqueur                                     |
| Page détail       | Bloc complet, quatre modes, distance et trace optionnel                |
| Comparaison       | Une ligne par bien, mode préféré                                       |

**Optimisation des appels**

1.  Cache en base dans route_cache, clé composée de property_id, destination_id et mode, avec durée de validité de 30 jours.

2.  Coordonnées arrondies à 4 décimales, environ 11 mètres, avant calcul de la clé, pour mutualiser les destinations proches.

3.  Calcul en lot sur la page de résultats : un seul appel Matrix pour les 12 biens affichés, au lieu de 12 appels Directions.

4.  Les modes secondaires ne sont calculés qu'à l'ouverture de la page détail, pas dans la liste.

5.  Filtré préalable par distance à vol d'oiseau : au-delà de 60 km, aucun appel n'est émis pour le vélo et la marche, un message remplace la valeur.

6.  Attente de 500 ms et bouton désactivé pendant le calcul, pour empêcher les clics répétés.

Avec ces règles, une session de démonstration typique consomme moins de 40 appels Directions, contre un quota de 2 000 par jour.

**Gestion des erreurs**

|                                            |                             |                                                                                                                |
|--------------------------------------------|-----------------------------|----------------------------------------------------------------------------------------------------------------|
| **Situation**                              | **Réponse API**             | **Comportement**                                                                                               |
| Adresse introuvable                        | 0 résultat de géocodage     | Message Adresse introuvable, suggestion de préciser la ville, aucune destination créée                         |
| Adresse ambiguë                            | Plusieurs résultats         | Liste de 3 propositions à confirmer                                                                            |
| Quota épuisé                               | 403                         | Affichage des valeurs en cache si présentes, sinon bandeau Calcul indisponible, réessayez plus tard            |
| Trop de requêtes                           | 429                         | Réessai automatique unique après 2 secondes, puis abandon silencieux                                           |
| Itinéraire impossible, par exemple une île | 404 ou 2010                 | Mention Itinéraire non disponible pour ce mode, les autres modes restent affichés                              |
| Service indisponible                       | 5xx ou délai dépassé de 5 s | La carte et la fiche restent fonctionnelles, seul le bloc trajet affiche un état dégradé avec bouton Réessayer |

Règle générale : aucune erreur de trajet ne doit bloquer l'affichage de l'annonce.

**Destinations enregistrées et confidentialité**

Un particulier connecté enregistré jusqu'à 3 destinations nommées, par exemple Travail et Université, dont une marquée par défaut. Un sélecteur permet de basculer d'une destination à l'autre sur toute l'application.

- Une adresse personnelle est une donnée à caractère personnel au sens du RGPD ; elle est liée au compte, jamais exposée dans une réponse publique.

- Le libellé est libre et affiché ; l'adresse complète n'est visible que par son propriétaire.

- La table route_cache ne stocke que des coordonnées arrondies et un identifiant de destination, pas d'adresse en clair.

- La suppression d'une destination supprime ses entrées de cache associées.

- Un visiteur non connecté peut saisir une adresse ponctuelle : elle est utilisée pour le calcul puis conservée uniquement dans le stockage de session du navigateur, jamais écrite en base.

- Une mention d'information explique l'usage de l'adresse au moment de la saisie.

**7. ImmoMatch Score**

Le score mesure l'écart entre un bien et le profil de recherche actif, sur 100. Il est calculé côté backend, à la volée, et toujours accompagné de sa décomposition.

**Formule**

Chaque critère renseigné dans le profil produit un sous-score entre 0 et 1. Le score final est la moyenne pondérée des critères renseignés, ramenée à 100.

| **Critère**            | **Poids** | **Sous-score**                                                                                                                                                                                                                          |
|------------------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Budget                 | 30        | Score plein (1) si le prix reste sous le budget ; il diminue ensuite de façon linéaire jusqu'à 0 quand le prix atteint 120 % du budget, puis reste à 0 au-delà.                                                                         |
| Localisation ou trajet | 25        | Score plein (1) si le bien est dans la ville souhaitée. Sinon, si un temps de trajet maximal est défini : le score plein est en dessous de ce seuil, puis décroissance linéaire jusqu’à 0 lorsque le trajet atteint le double du seuil. |
| Surface                | 15        | Score plein (1) si la surface atteint le minimum souhaité, puis décroissance linéaire jusqu’à 0 lorsque la surface descend à 80% de ce minimum.                                                                                         |
| Chambres et pièces     | 15        | 1 si suffisant ; 0,5 s'il manque une chambre ; 0 au-delà.                                                                                                                                                                               |
| Type de bien           | 10        | 1 si le type figure parmi les types souhaités, sinon 0                                                                                                                                                                                  |
| DPE                    | 5         | 1 si classe égale ou meilleure ; 0,6 à une classe d'écart ; 0,3 à deux ; 0 ensuite                                                                                                                                                      |

Formule retenue : score = 100 \* somme(poids_i \* sousScore_i) / somme(poids_i) pour tous les critères i renseignés dans le profil

Exemple : profil avec budget 900, surface 50, 2 chambres, Strasbourg, DPE D souhaité. Un bien à 880 EUR, 64 m2, 2 chambres, à Strasbourg, DPE E donne 30 + 25 + 15 + 15 + 5 x 0,6 soit 88 sur 90 points possibles, soit 98%. Le type de bien n'étant pas renseigné dans le profil, son poids de 10 est exclu du dénominateur.

**Critères obligatoires et facultatifs**

Chaque critère du profil porte un indicateur mandatory.

- Un critère obligatoire non satisfait exclut le bien des recommandations, quel que soit le score des autres critères.

- Il n'exclut pas le bien des résultats de recherche classique : l'utilisateur garde la main sur ses filtres.

- La transaction, achat ou location, est toujours obligatoire et n'entre pas dans le score : elle est filtrée en amont.

- Par défaut, seul le budget est obligatoire ; l'utilisateur peut cocher les autres.

**Données manquantes**

| **Cas**                                               | **Traitement**                                                                            |
|-------------------------------------------------------|-------------------------------------------------------------------------------------------|
| Critère absent du profil                              | Son poids est retiré du dénominateur, le score reste sur 100.                             |
| Donnée absente du bien, par exemple DPE non renseigné | Sous-score fixe à 0,5, et la ligne d'explication porte la mention information non fournie |
| Trajet non encore calculé                             | Le critère Localisation retombe sur la comparaison de ville seule                         |

Ce choix évite deux travers : pénaliser injustement une annonce incomplète, et la récompenser d'être incomplète.

**Explication affichée**

Chaque ligne reprend le critère, son statut et la valeur comparée. Trois statuts seulement, pour rester lisible.

|              |                         |                          |
|--------------|-------------------------|--------------------------|
| **Statut**   | **Seuil du sous-score** | **Rendu**                |
| Respecté     | \>= 0,8                 | Coché, texte vert foncé  |
| Presque      | 0,4 à 0,79              | Triangle, texte ambre    |
| Non respecté | \< 0,4                  | Croix, texte rouge foncé |

Un statut n'est jamais affiché sans sa valeur : Budget respecté, 880 EUR pour 900 EUR maximum.

**Classement et affichage**

- Le tri par pertinence ordonne par score décroissant, puis par date de publication décroissante à égalité.

- Le score n'est affiché que si un profil est actif ; sinon la pastille est masquée, pas mise à zéro.

- Bandes de couleur : 85 et plus vert, 65 à 84 ambre, moins de 65 gris. Aucun bien n'est caché à cause de son score.

- Le calcul est effectué sur la page courante de résultats, pas sur toute la base, sauf pour le tri par pertinence qui le fait en SQL sur les critères simples puis affine les 50 premiers.

**8. Profil de recherche et recommandations**

Un profil de recherche est le projet immobilier d'un utilisateur, enregistré et réutilisable. Il alimente le score de la section 7 et les recommandations.

**Contenu et cycle de vie**

Le profil reprend les mêmes critères que la recherche naturelle, plus un nom, un indicateur actif et une destination de référence avec son temps maximal. Un utilisateur peut détenir plusieurs profils, par exemple Studio étudiant et Achat famille, mais un seul actif à la fois.

- RG-P1 : le profil actif conditionne l'affichage du score et l'onglet Recommandations ; l'activation d'un profil désactive le précédent.

- RG-P2 : un profil peut être créé en un clic depuis une recherche en cours, ce qui est le parcours principal.

- RG-P3 : la suppression d'un profil ne supprime ni les favoris ni les demandes de visite associés.

- RG-P4 : chaque critère porte son indicateur mandatory, modifiable dans l'écran d’Édition.

Le filtre SQL applique d'abord les critères obligatoires et une marge de tolérance de 10 pour cent sur le budget, afin de ne pas écarter un bien juste au-dessus. Le score est ensuite calculé en mémoire sur cet ensemble réduit, ce qui reste rapide sur quelques centaines de biens et évite toute vue matérialisée.

- Les biens déjà en favori ou déjà visités restent proposés, avec un badge Déjà vu.

- Un bien refusé explicitement, via le bouton Pas intéressé, est exclu pendant 30 jours.

- Aucune recommandation n'est affichée si le profil compte moins de deux critères renseignés ; un message invite à le compléter.

- Les recommandations sont recalculées à chaque consultation, pas stockées. Un traitement planifié et un envoi par courriel sont explicitement en bonus, section 18.

**9. Espaces utilisateurs**

Trois espaces, une seule application Angular, séparation par routes et guards.

/ accueil, phrase de recherche

/recherche liste + carte

/biens/:id page détail

/connexion /inscription

/compte espace particulier

/compte/profil identité, mot de passe

/compte/favoris

/compte/comparaison

/compte/projets profils de recherche

/compte/recommandations

/compte/destinations

/compte/visites

/agent espace agent

/agent tableau de bord

/agent/annonces

/agent/annonces/nouvelle

/agent/annonces/:id/editer

/agent/visites

/agent/statistiques

/admin espace administrateur

/admin tableau de bord

/admin/utilisateurs

/admin/agents

/admin/annonces modération

/admin/statistiques

**Espace particulier**

|                 |                                                             |                                                  |
|-----------------|-------------------------------------------------------------|--------------------------------------------------|
| **Écran**       | **Contenu**                                                 | **Actions**                                      |
| Profil          | nom, courriel, mot de passe, suppression du compte          | modifier, supprimer                              |
| Favoris         | grille des biens sauvegardés, tri par date d'ajout ou score | retirer, comparer, ouvrir                        |
| Comparaison     | tableau de 2 à 4 biens                                      | ajouter, retirer, exporter en PDF, en bonus      |
| Projets         | liste des profils de recherche, un seul actif               | créer, éditer, activer, supprimer                |
| Recommandations | 10 biens ordonnés par score, avec explication               | ouvrir, mettre en favori, écarter                |
| Destinations    | jusqu'à 3 adresses nommées, une par défaut                  | ajouter, renommer, définir par défaut, supprimer |
| Visites         | demandes groupées par statut                                | annuler une demande en attente                   |

**Espace agent**

|                      |                                                                                               |                                                         |
|----------------------|-----------------------------------------------------------------------------------------------|---------------------------------------------------------|
| **Écran**            | **Contenu**                                                                                   | **Actions**                                             |
| Tableau de bord      | 4 indicateurs : annonces publiées, vues sur 30 jours, demandes en attente, taux d'acceptation | accès rapide aux demandes                               |
| Annonces             | tableau filtrable par statut, avec vues et favoris par annonce                                | publier, archiver, dupliquer, supprimer un brouillon    |
| Formulaire d'annonce | 4 étapes : bien, localisation et carte, photos, publication                                   | enregistrer en brouillon, prévisualiser, publier        |
| Photos               | téléversement multiple, glisser-déposer pour l'ordre, choix de la photo principale            | ajouter, réordonner, supprimer                          |
| Visites              | demandes en attente en premier, avec le bien et le demandeur                                  | accepter, refuser avec motif, proposer un autre créneau |
| Statistiques         | vues par annonce sur 30 jours, favoris, demandes                                              | filtrer par période                                     |

Les statistiques agent reposent sur un compteur de vues simple : une ligne par consultation, avec anti-doublon par session de 30 minutes.

**Espace administrateur**

|                 |                                                               |                                           |
|-----------------|---------------------------------------------------------------|-------------------------------------------|
| **Écran**       | **Contenu**                                                   | **Actions**                               |
| Tableau de bord | comptes, annonces par statut, visites du mois, biens signalés | accès à la file de modération             |
| Utilisateurs    | recherche, filtre par rôle et par statut                      | suspendre, réactiver, supprimer           |
| Agents          | agents et leur agence, nombre d'annonces                      | valider un nouvel agent, suspendre        |
| Modération      | annonces signalées ou nouvellement publiées                   | approuver, rejeter avec motif obligatoire |
| Statistiques    | volumes par ville, prix médian, répartition DPE               | filtrer par période                       |

• RG-S1 : un agent nouvellement inscrit est en attente de validation et ne peut pas publier avant approbation.

- RG-S2 : suspendre un agent dépublié automatiquement ses annonces, qui repassent en ARCHIVED.

- RG-S3 : toute action de modération est journalisée avec l'auteur, la date et le motif.

**10. Architecture technique**

La pile envisagée est conservée : Angular, Spring Boot, PostgreSQL, Docker Compose. Elle est adaptée au sujet et déjà connue des deux étudiants. Deux ajustements sont proposés plus bas.

Navigateur

\|

v

Angular 18 (SPA, standalone components, signals)

\| HTTP/JSON, JWT dans l'en-tête Authorization

v

Spring Boot 3 - API REST

\| \|

\| +--\> openrouteservice (géocodage, routage)

v \|

PostgreSQL 16 +--\> tuiles OSM (directement depuis le navigateur)

\|

v

Volume Docker : images des annonces

**Couches backend**

controller -\> reçoit, valide, retourne des DTO, aucune logique

service -\> règles métier, scoring, orchestration des appels externes

repository -\> Spring Data JPA, requêtes dérivées et JPQL

entity -\> mapping JPA

mapper -\> entity vers DTO, MapStruct ou méthodes simples

La recherche multicritère est le seul point où une requête dynamique est nécessaire. Elle est construite avec les Spécifications de Spring Data JPA, qui suffisent ici et évitent l'ajout de QueryDSL.

**Choix et compromis**

|                               |                                                             |                                                                                                |
|-------------------------------|-------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| **Sujet**                     | **Décision**                                                | **Raison**                                                                                     |
| Angular standalone et signals | retenu                                                      | moins de code de configuration, état de recherche naturel à exprimer en signals                |
| NgRx                          | écarté                                                      | surdimensionné pour deux étudiants ; un service avec signals suffit pour l'état de recherche   |
| Stockage des images           | volume Docker plus table photo                              | pas de compte cloud à gérer ; le chemin est dans la base, le fichier sur disque                |
| Cache                         | table route_cache en base, plus cache Spring en mémoire     | Redis ajouterait un service à maintenir pour un gain nul à cette échelle                       |
| Recherche géographique        | colonnes latitude et longitude plus filtré par bounding box | PostGIS est un apport réel mais ajoute une dépendance et une courbe d'apprentissage ; en bonus |
| Migrations                    | Flyway                                                      | schéma versionné et rejouable, indispensable à deux                                            |
| Documentation API             | springdoc-openapi                                           | génère Swagger UI sans effort, sert de livrable                                                |

**Docker Compose**

Quatre services : db PostgreSQL avec volume persistant, api Spring Boot, web Angular servi par Nginx avec proxy vers l'api, et adminer en profil développement. Un fichier .env porte les secrets, jamais versionné. Une commande docker compose up doit suffire à démarrer le projet depuis un dépôt vierge, avec un jeu de données de démonstration chargé par Flyway.

**Environnements**

|                   |                   |                                                         |
|-------------------|-------------------|---------------------------------------------------------|
| **Environnement** | **Usage**         | **Données**                                             |
| local             | développement     | jeu de démonstration                                    |
| ci                | tests automatisés | base PostgreSQL éphémère via Testcontainers             |
| demo              | soutenance        | copie figée du jeu de démonstration, déployée la veille |

**11. Modèle de données**

Quatorze tables, dont deux tables de liaison. Toutes les clés primaires sont des BIGSERIAL, toutes les dates des TIMESTAMPTZ.

**MCD textuel**

AGENCE ----1,n---- emploie ----1,1---- AGENT

AGENT ----0,n---- publie ----1,1---- BIEN

BIEN ----1,n---- possède ----1,1---- PHOTO

BIEN ----0,n---- dispose ----0,n---- ÉQUIPEMENT

UTILISATEUR ----0,n---- sauvegarde ----0,n---- BIEN (FAVORI)

UTILISATEUR ----0,n---- demande ----0,n---- BIEN (DEMANDE_VISITE)

UTILISATEUR ----0,n---- définit ----1,1---- PROFIL_RECHERCHE

UTILISATEUR ----0,n---- enregistre ----1,1---- DESTINATION

BIEN ----0,n---- trajet ----0,n---- DESTINATION (TRAJET_CACHE)

UTILISATEUR ----0,n---- reçoit ----1,1---- NOTIFICATION

ADMIN ----0,n---- modère ----1,1---- JOURNAL_MODÉRATION

BIEN ----0,n---- consulte ----1,1---- VUE

Un agent est un utilisateur de rôle RÔLE_AGENT ; l'héritage est traduit par une table agent_profile en relation 1,1 avec app_user, ce qui évite les colonnes nulles dans la table principale.

**MLD des tables centrales**

app_user

|                        |              |                                                                |
|------------------------|--------------|----------------------------------------------------------------|
| **Colonne**            | **Type**     | **Contraintes**                                                |
| id                     | BIGSERIAL    | PK                                                             |
| email                  | VARCHAR(180) | UNIQUE, NOT NULL                                               |
| password_hash          | VARCHAR(100) | NOT NULL                                                       |
| first_name, last_name  | VARCHAR(80)  | NOT NULL                                                       |
| phone                  | VARCHAR(20)  |                                                                |
| rôle                   | VARCHAR(20)  | NOT NULL, CHECK dans RÔLE_USER, RÔLE_AGENT, RÔLE_ADMIN         |
| status                 | VARCHAR(20)  | NOT NULL, défaut ACTIVE, CHECK dans ACTIVE, PENDING, SUSPENDED |
| created_at, updated_at | TIMESTAMPTZ  | NOT NULL                                                       |

property

|                        |                            |                                     |
|------------------------|----------------------------|-------------------------------------|
| **Colonne**            | **Type**                   | **Contraintes**                     |
| id                     | BIGSERIAL                  | PK                                  |
| agent_id               | BIGINT                     | FK agent_profile(user_id), NOT NULL |
| title                  | VARCHAR(120)               | NOT NULL                            |
| description            | TEXT                       | NOT NULL                            |
| transaction_type       | VARCHAR(10)                | NOT NULL, CHECK RENT ou SALE        |
| property_type          | VARCHAR(20)                | NOT NULL                            |
| price                  | NUMERIC(12,2)              | NOT NULL, CHECK \> 0                |
| charges                | NUMERIC(10,2)              | CHECK \>= 0                         |
| surface                | NUMERIC(8,2)               | NOT NULL, CHECK \> 5                |
| rooms, bedrooms        | SMALLINT                   | NOT NULL, CHECK bedrooms \< rooms   |
| floor, total_floors    | SMALLINT                   |                                     |
| address_line, city     | VARCHAR(160), VARCHAR(100) | NOT NULL                            |
| postal_code            | VARCHAR(10)                | NOT NULL                            |
| latitude, longitude    | NUMERIC(9,6), NUMERIC(9,6) | NOT NULL                            |
| energy_class           | CHAR(1)                    | CHECK dans A à G ou NULL            |
| status                 | VARCHAR(12)                | NOT NULL, défaut DRAFT              |
| published_at           | TIMESTAMPTZ                |                                     |
| created_at, updated_at | TIMESTAMPTZ                | NOT NULL                            |

search_profile

|                                                                             |              |                                           |
|-----------------------------------------------------------------------------|--------------|-------------------------------------------|
| **Colonne**                                                                 | **Type**     | **Contraintes**                           |
| id                                                                          | BIGSERIAL    | PK                                        |
| user_id                                                                     | BIGINT       | FK app_user(id) ON DELETE CASCADE         |
| name                                                                        | VARCHAR(80)  | NOT NULL                                  |
| is_active                                                                   | BOOLEAN      | NOT NULL, défaut false                    |
| transaction_type                                                            | VARCHAR(10)  | NOT NULL                                  |
| property_types                                                              | VARCHAR(120) | liste séparée par virgules                |
| cities                                                                      | VARCHAR(200) | liste séparée par virgules                |
| budget_max, surface_min                                                     | NUMERIC      |                                           |
| bedrooms_min, rooms_min                                                     | SMALLINT     |                                           |
| energy_class_min                                                            | CHAR(1)      |                                           |
| destination_id                                                              | BIGINT       | FK destination(id) ON DELETE SET NULL     |
| max_travel_minutes                                                          | SMALLINT     |                                           |
| travel_mode                                                                 | VARCHAR(12)  |                                           |
| budget_mandatory, location_mandatory, surface_mandatory, bedrooms_mandatory | BOOLEAN      | NOT NULL, défaut false sauf budget à true |
| created_at, updated_at                                                      | TIMESTAMPTZ  | NOT NULL                                  |

route_cache

|                  |             |                                            |
|------------------|-------------|--------------------------------------------|
| **Colonne**      | **Type**    | **Contraintes**                            |
| id               | BIGSERIAL   | PK                                         |
| property_id      | BIGINT      | FK property(id) ON DELETE CASCADE          |
| destination_id   | BIGINT      | FK destination(id) ON DELETE CASCADE       |
| mode             | VARCHAR(12) | NOT NULL, CHECK CAR, BIKE, WALK, TRANSIT   |
| duration_seconds | INTEGER     | NOT NULL                                   |
| distance_meters  | INTEGER     | NOT NULL                                   |
| is_estimated     | BOOLEAN     | NOT NULL, défaut false                     |
| computed_at      | TIMESTAMPTZ | NOT NULL                                   |
|                  |             | UNIQUE (property_id, destination_id, mode) |

visit_request

|                        |              |                                            |
|------------------------|--------------|--------------------------------------------|
| **Colonne**            | **Type**     | **Contraintes**                            |
| id                     | BIGSERIAL    | PK                                         |
| property_id, user_id   | BIGINT       | FK, NOT NULL                               |
| slot_start             | TIMESTAMPTZ  | NOT NULL, CHECK \> now + 24h à l'insertion |
| message                | VARCHAR(500) |                                            |
| status                 | VARCHAR(12)  | NOT NULL, défaut PENDING                   |
| agent_reply            | VARCHAR(500) | obligatoire si status = REFUSED            |
| created_at, updated_at | TIMESTAMPTZ  | NOT NULL                                   |

**Tables complémentaires**

|                  |                                                                   |                                                |
|------------------|-------------------------------------------------------------------|------------------------------------------------|
| **Table**        | **Colonnes principales**                                          | **Rôle**                                       |
| agency           | id, name, siret, city, phone                                      | agence de rattachement                         |
| agent_profile    | user_id PK et FK, agency_id, licence_number, bio, photo_url       | extension du compte agent                      |
| property_photo   | id, property_id, file_path, position, is_main                     | 1 à 12 par bien, position unique par bien      |
| feature          | id, code UNIQUE, label                                            | referentiel des équipements                    |
| property_feature | property_id, feature_id                                           | liaison, PK composite                          |
| favorite         | id, user_id, property_id, created_at                              | UNIQUE (user_id, property_id)                  |
| destination      | id, user_id, label, address_line, latitude, longitude, is_default | 3 maximum par utilisateur, contrôle applicatif |
| notification     | id, user_id, type, payload, is_read, created_at                   | notifications internes                         |
| modération_log   | id, admin_id, property_id, action, reason, created_at             | traçabilité des décisions                      |
| property_view    | id, property_id, session_hash, viewed_at                          | statistiques agent, sans donnée nominative     |

**Index**

|                           |                                       |                                          |
|---------------------------|---------------------------------------|------------------------------------------|
| **Index**                 | **Colonnes**                          | **Motif**                                |
| idx_property_search       | status, transaction_type, city, price | filtre principal de la recherche         |
| idx_property_bbox         | latitude, longitude                   | recherche par emprise de carte           |
| idx_property_agent        | agent_id, status                      | tableau de bord agent                    |
| idx_property_surface      | surface                               | tri et filtres secondaires               |
| idx_favorite_user         | user_id                               | liste des favoris                        |
| idx_visit_property_status | property_id, status                   | file des demandes de l'agent             |
| idx_route_cache_lookup    | property_id, destination_id, mode     | contrainte d'unicité, sert de couverture |
| idx_view_property_date    | property_id, viewed_at                | statistiques sur 30 jours                |

**Règles d'intégrité complémentaires**

- Un bien PUBLISHED doit avoir au moins une photo : contrôle applicatif à la publication, pas de contrainte SQL.

- Un seul search_profile actif par utilisateur : index unique partiel sur (user_id) WHERE is_active.

- Une seule photo principale par bien : index unique partiel sur (property_id) WHERE is_main.

- Une seule destination par défaut par utilisateur : même technique.

- Suppression d'un compte particulier : CASCADE sur favorite, destination, search_profile ; SET NULL puis anonymisation sur visit_request.

**12. API REST**

Préfixe /api, JSON en entrée et en sortie, JWT dans l'en-tête Authorization pour tout ce qui n'est pas public. Pagination par page et size, tri par sort, format page Spring standard : content, totalElements, totalPages, number.

**Conventions**

- Les noms de ressources sont au pluriel, les identifiants dans le chemin, les filtres en paramètres de requête.

- Aucune entité JPA n'est exposée : DTO en entrée comme en sortie.

- Les erreurs suivent un format unique : timestamp, status, error, message, path, et fieldErrors pour la validation.

- Les codes utilisés sont 200, 201, 204, 400, 401, 403, 404, 409, 422 et 503 pour une indisponibilité d'API externe.

**Endpoints**

|                                               |                    |                                                 |                                    |                    |
|-----------------------------------------------|--------------------|-------------------------------------------------|------------------------------------|--------------------|
| **Méthode et URL**                            | **Rôle**           | **Entrée**                                      | **Sortie**                         | **Codes**          |
| POST /api/auth/register                       | public             | email, mot de passe, nom, prénom, rôle souhaité | utilisateur créé                   | 201, 400, 409      |
| POST /api/auth/login                          | public             | email, mot de passe                             | accessToken, refreshToken, profil  | 200, 401           |
| POST /api/auth/refresh                        | public             | refreshToken                                    | nouveau accessToken                | 200, 401           |
| GET /api/properties                           | public             | filtres, bbox, page, size, sort, profileId      | page de PropertySummary avec score | 200, 400           |
| GET /api/properties/{id}                      | public             |                                                 | PropertyDetail                     | 200, 404           |
| POST /api/properties                          | AGENT              | PropertyRequest                                 | bien créé en DRAFT                 | 201, 400, 403      |
| PUT /api/properties/{id}                      | AGENT propriétaire | PropertyRequest                                 | bien modifié                       | 200, 403, 404      |
| PATCH /api/properties/{id}/status             | AGENT ou ADMIN     | status, reason                                  | bien mis à jour                    | 200, 403, 409, 422 |
| DELETE /api/properties/{id}                   | AGENT propriétaire |                                                 |                                    | 204, 403, 409      |
| POST /api/properties/{id}/photos              | AGENT propriétaire | multipart, 1 à 12 fichiers                      | liste des photos                   | 201, 400, 413      |
| DELETE /api/properties/{id}/photos/{photoId}  | AGENT propriétaire |                                                 |                                    | 204, 403, 404      |
| GET /api/favorites                            | USER               |                                                 | liste de PropertySummary           | 200, 401           |
| POST /api/favorites                           | USER               | propertyId                                      | favori créé                        | 201, 409           |
| DELETE /api/favorites/{propertyId}            | USER               |                                                 |                                    | 204, 404           |
| GET /api/properties/compare                   | public             | ids, 2 à 4                                      | tableau comparatif                 | 200, 400           |
| GET /api/search-profiles                      | USER               |                                                 | liste des profils                  | 200                |
| POST /api/search-profiles                     | USER               | SearchProfileRequest                            | profil créé                        | 201, 400           |
| PUT /api/search-profiles/{id}                 | USER propriétaire  | SearchProfileRequest                            | profil modifié                     | 200, 403, 404      |
| PATCH /api/search-profiles/{id}/activate      | USER propriétaire  |                                                 | profil actif                       | 200, 404           |
| GET /api/search-profiles/{id}/recommendations | USER propriétaire  | limit                                           | 10 biens avec score détaillé       | 200, 404, 422      |
| GET /api/destinations                         | USER               |                                                 | liste des destinations             | 200                |
| POST /api/destinations                        | USER               | label, address                                  | destination géocodée               | 201, 400, 409, 503 |
| DELETE /api/destinations/{id}                 | USER propriétaire  |                                                 |                                    | 204, 404           |
| GET /api/geocode                              | authentifié        | q                                               | 5 suggestions maximum              | 200, 400, 503      |
| POST /api/routes                              | public limite      | propertyId, destinationId ou adresse, modes     | durée et distance par mode         | 200, 400, 503      |
| POST /api/routes/batch                        | public limite      | propertyIds, destinationId, mode                | une entrée par bien                | 200, 400, 503      |
| POST /api/visits                              | USER               | propertyId, slotStart, message                  | demande en PENDING                 | 201, 400, 409      |
| GET /api/visits                               | USER ou AGENT      | status                                          | demandes de l'appelant             | 200                |
| PATCH /api/visits/{id}                        | AGENT du bien      | status, reply                                   | demande mise à jour                | 200, 403, 409      |
| DELETE /api/visits/{id}                       | USER auteur        |                                                 | annulation                         | 204, 403, 409      |
| GET /api/agents/me/stats                      | AGENT              | from, to                                        | indicateurs du tableau de bord     | 200                |
| GET /api/users/me                             | authentifié        |                                                 | profil courant                     | 200, 401           |
| PUT /api/users/me                             | authentifié        | nom, prénom, téléphone                          | profil mis à jour                  | 200, 400           |
| GET /api/admin/users                          | ADMIN              | rôle, status, q, page                           | page d'utilisateurs                | 200, 403           |
| PATCH /api/admin/users/{id}/status            | ADMIN              | status, reason                                  | utilisateur mis à jour             | 200, 403, 422      |
| GET /api/admin/properties                     | ADMIN              | status, page                                    | page d'annonces                    | 200, 403           |
| GET /api/admin/stats                          | ADMIN              | from, to                                        | statistiques globales              | 200, 403           |

**Exemple : recherche**

GET /api/properties?transactionType=RENT&city=Strasbourg&priceMax=900

&bedroomsMin=2&features=PARKING&profileId=7&sort=relevance&page=0&size=12

200 OK

{

"content": \[

{

"id": 142,

"title": "T3 lumineux proche Krutenau",

"price": 850,

"charges": 60,

"surface": 64,

"rooms": 3,

"bedrooms": 2,

"city": "Strasbourg",

"energyClass": "E",

"mainPhotoUrl": "/media/properties/142/1.webp",

"latitude": 48.5810,

"longitude": 7.7620,

"matchScore": 92,

"travel": { "mode": "CAR", "durationMinutes": 18, "estimated": false }

}

\],

"totalElements": 37,

"totalPages": 4,

"number": 0

}

**Exemple : calcul de trajet**

POST /api/routes

{ "propertyId": 142, "destinationId": 3, "modes": \["CAR","TRANSIT","BIKE","WALK"\] }

200 OK

{

"propertyId": 142,

"destinationLabel": "Université",

"results": \[

{ "mode": "CAR", "durationMinutes": 18, "distanceKm": 7.4, "estimated": false },

{ "mode": "TRANSIT", "durationMinutes": 24, "distanceKm": 7.4, "estimated": true },

{ "mode": "BIKE", "durationMinutes": 16, "distanceKm": 5.9, "estimated": false },

{ "mode": "WALK", "durationMinutes": 42, "distanceKm": 5.2, "estimated": false }

\],

"computedAt": "2026-09-16T10:12:00Z",

"fromCache": true

}

503 Service Unavailable

{ "status": 503, "error": "ROUTING_UNAVAILABLE",

"message": "Service d'itinéraire indisponible", "path": "/api/routes" }

**13. Sécurité, performance et accessibilité**

**Sécurité**

|                        |                                                                                                                                                                                   |
|------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Sujet**              | **Mesure**                                                                                                                                                                        |
| Mots de passe          | BCrypt, force 10, minimum 10 caractères avec au moins un chiffre ; jamais retournés par l'API                                                                                     |
| Jetons                 | JWT signe HS256, accès 15 minutes, rafraîchissement 7 jours stocké en base et révocable                                                                                           |
| Stockage côté client   | jeton d'accès en mémoire dans un service Angular, jeton de rafraîchissement en cookie httpOnly et SameSite=Strict                                                                 |
| Autorisation           | filtré JWT plus annotations PreAuthorize ; vérification de propriété dans le service, jamais seulement sur le rôle                                                                |
| Validation             | Bean Validation sur tous les DTO d'entrée, plus révalidation métier en service                                                                                                    |
| Injection SQL          | JPA et Spécifications, aucune concaténation de chaîne dans une requête                                                                                                            |
| XSS                    | Angular échappé par défaut, aucun innerHTML sur une donnée utilisateur, description stockée en texte brut                                                                         |
| CORS                   | origine unique autorisée par environnement, définie en configuration, pas d'étoile                                                                                                |
| Upload                 | extension et type MIME vérifiés, magic bytes contrôlés, 5 Mo maximum, nom de fichier régénéré en UUID, redimensionnement systématique, stockage hors du dossier servi directement |
| Secrets                | variables d'environnement et fichier .env exclu du dépôt, un .env.example versionné                                                                                               |
| Force brute            | limitation à 5 tentatives de connexion par minute et par adresse IP                                                                                                               |
| Énumération de comptes | message d'erreur identique pour identifiant inconnu et mot de passe faux                                                                                                          |
| En-têtes               | HSTS, X-Content-Type-Options, Referrer-Policy et Content-Security-Policy configurés sur Nginx                                                                                     |

**Performance**

|                        |                                                                                                                 |                                                 |
|------------------------|-----------------------------------------------------------------------------------------------------------------|-------------------------------------------------|
| **Levier**             | **Mise en œuvre**                                                                                               | **Cible**                                       |
| Pagination             | 12 résultats par page, jamais de findAll                                                                        | réponse API sous 300 ms                         |
| Index                  | voir section 11                                                                                                 | recherche filtrée sous 100 ms en base           |
| Requêtes N+1           | fetch join sur photo principale et agent dans la recherche                                                      | une requête par page de résultats               |
| Images                 | conversion en WebP, 3 tailles générées à l'upload (vignette 400, carte 800, détail 1600), attribut loading lazy | moins de 150 ko par vignette                    |
| Lazy loading Angular   | chargement diffère des routes agent et admin                                                                    | bundle initial sous 500 ko compressé            |
| Appels cartographiques | cache et lot, voir section 6                                                                                    | moins de 40 appels par session de démonstration |
| Marqueurs              | clustering au-delà de 30 marqueurs visibles                                                                     | carte fluide à 300 biens                        |
| Cache applicatif       | Caffeine via Spring Cache sur le referentiel d'équipements et les suggestions de villes                         |                                                 |

Objectifs mesurables retenus pour la soutenance : Largest Contentful Paint sous 2,5 secondes en 4G simulée, score Lighthouse performance supérieur à 80 sur mobile.

**Accessibilité**

Le niveau vise est WCAG 2.1 AA sur les parcours publics, vérifié par axe DevTools et par un test clavier complet.

- Contraste minimum de 4,5 pour 1 sur le texte, 3 pour 1 sur les composants d'interface ; la palette de la section 16 est construite pour cela.

- Tout est atteignable au clavier, y compris les jetons de recherche et la carte ; un lien d'évitement mène au contenu principal.

- Le focus est toujours visible, avec un contour de 2 px non supprimé par le CSS.

- Chaque champ possede un label associé, pas seulement un placeholder ; les erreurs sont liées par aria-describedby et annoncées en aria-live.

- Les photos d'annonce portent un texte alternatif construit à partir du titre et de la position ; les icônes décoratives sont marquées aria-hidden.

- Structure semantique : un seul h1 par page, hiérarchie de titres continue, balises header, nav, main et footer.

- La carte, difficilement accessible par nature, est doublée par la liste : tout ce qui est faisable sur la carte l'est aussi dans la liste. Ce point est documenté comme une alternative équivalente.

**14. Tests et CI/CD**

Objectif réaliste : 60 pour cent de couverture globale, mais 90 pour cent sur les deux briques critiques que sont le scoring et le service d'itinéraire. Mieux vaut peu de tests sur ce qui compte que beaucoup sur les getters.

**Stratégie**

|                     |                                             |                                                                       |                    |
|---------------------|---------------------------------------------|-----------------------------------------------------------------------|--------------------|
| **Niveau**          | **Outils**                                  | **Périmètre**                                                         | **Quantité visée** |
| Unitaire backend    | JUnit 5, Mockito, AssertJ                   | ScoringService, RouteService, règles de visite, validation            | environ 40 tests   |
| Intégration backend | Spring Boot Test, Testcontainers PostgreSQL | repositories, Spécifications de recherche, migrations Flyway          | environ 15 tests   |
| API                 | MockMvc, ou RestAssured                     | codes de retour, sécurité par rôle, format d'erreur                   | environ 20 tests   |
| Unitaire Angular    | Jasmine, Karma                              | services, pipes, logique des composants de recherche                  | environ 25 tests   |
| Composant Angular   | TestBed, HttpTestingController              | rendu de la phrase de recherche, carte de bien, tableau comparatif    | environ 10 tests   |
| Bout en bout        | Cypress ou Playwright                       | 4 parcours de la section 17                                           | 4 scenarios        |
| Sécurité            | tests dédiés plus dependency check          | accès interdit par rôle, upload de fichier non conforme, jeton expiré | environ 10 tests   |
| Accessibilité       | axe-core dans les tests Cypress             | accueil, résultats, détail                                            | 3 pages            |

Les appels externes sont toujours simulés dans les tests, par WireMock côté backend, pour ne pas consommer de quota et rester déterministes.

**Cas de test prioritaires**

1.  Un profil sans DPE renseigné ne pénalise pas le score et le poids est bien retiré du dénominateur.

2.  Un bien sans DPE reçoit 0,5 sur ce critère et la mention information non fournie.

3.  Un critère obligatoire non satisfait exclut le bien des recommandations mais pas des résultats.

4.  Deux appels successifs de trajet sur le même couple ne déclenchent qu'un seul appel externe.

5.  Une panne du service d'itinéraire renvoie 503 sans empêcher l'affichage de l'annonce.

6.  Un agent ne peut pas modifier l'annonce d'un autre agent, même avec un identifiant valide.

7.  Une demande de visite sur un créneau à moins de 24 heures est refusée.

**Pipeline**

flowchart LR

A\[Push ou PR\] --\> B\[Build\<br/\>Maven + npm\]

B --\> C\[Lint\<br/\>ESLint + Checkstyle\]

C --\> D\[Tests\<br/\>back + front\]

D --\> E\[SonarCloud\<br/\>qualité + couverture\]

E --\> F\[Images Docker\<br/\>build + push GHCR\]

F --\> G\[Déploiement\<br/\>branche main\]

Deux workflows GitHub Actions. Le premier, sur chaque pull request, s'arrête à l'analyse qualité. Le second, sur main, construit et publie les images puis déploie.

**Règles de collaboration**

- Branches par fonctionnalité, nommées feat/, fix/ ou docs/, avec le numéro de ticket.

- Pull request obligatoire, relue par l'autre étudiant, pas de commit direct sur main.

- La pipeline doit être verte pour fusionner ; une pull request en échec n'est pas fusionnée.

- Quality gate SonarCloud : aucune vulnérabilité bloquante, duplication sous 5 pour cent.

- Déploiement cible : une machine virtuelle universitaire ou un hébergeur gratuit, via docker compose pull et up. Si aucun hébergement n'est disponible, la démonstration se fait en local et la pipeline s'arrête à la publication des images.

**15. Responsive design**

L'application est conçue mobile d'abord. Le mobile n'est pas une version réduite : la carte et la liste, côte à côte en desktop, deviennent deux vues alternées.

|            |                 |                                       |                           |
|------------|-----------------|---------------------------------------|---------------------------|
| **Palier** | **Largeur**     | **Grille**                            | **Colonnes de résultats** |
| Mobile     | moins de 640 px | 4 colonnes, gouttière 16 px           | 1                         |
| Tablette   | 640 à 1023 px   | 8 colonnes, gouttière 20 px           | 2                         |
| Desktop    | 1024 à 1439 px  | 12 colonnes, gouttière 24 px          | 2 plus carte              |
| Large      | 1440 px et plus | 12 colonnes, largeur maximale 1400 px | 3 plus carte              |

**Comportement par élément**

|                       |                                                                                 |                                        |                                                                         |
|-----------------------|---------------------------------------------------------------------------------|----------------------------------------|-------------------------------------------------------------------------|
| **Élément**           | **Mobile**                                                                      | **Tablette**                           | **Desktop**                                                             |
| Navigation            | barre inférieure à 4 entrées : Rechercher, Favoris, Visites, Compte             | barre supérieure compacte              | barre supérieure complète avec libellés                                 |
| Recherche naturelle   | phrase sur 3 à 4 lignes, feuille modale par jeton                               | phrase sur 2 lignes, popover           | phrase sur une à deux lignes, popover                                   |
| Filtres secondaires   | feuille modale plein écran avec compteur de filtres actifs                      | panneau latéral escamotable            | panneau latéral fixe à gauche                                           |
| Résultats et carte    | bascule Liste et Carte, bouton flottant centré en bas                           | 60 pour cent liste, 40 pour cent carte | 55 pour cent liste, 45 pour cent carte fixe                             |
| Carte de bien         | pleine largeur, photo en 16 sur 9, informations sous la photo                   | idem sur 2 colonnes                    | photo à gauche, informations à droite en vue liste                      |
| Carte interactive     | plein écran, fiche du bien en feuille basse glissante                           | moitié d'écran                         | panneau fixe, suit le défilement                                        |
| Page détail           | galerie en carrousel avec points, blocs empilés, barre d'action collante en bas | 2 colonnes à partir des équipements    | 2 colonnes, colonne droite collante avec prix, score, trajet et boutons |
| Bloc trajet           | accordéon replié, quatre modes en grille 2 sur 2                                | idem déplié                            | déplié, quatre modes en ligne                                           |
| Comparaison           | défilement horizontal, colonne des critères figée à gauche, 2 biens visibles    | 3 biens visibles                       | 4 biens, tableau complet                                                |
| Galerie photos        | plein écran au tap, balayage latéral                                            | lightbox                               | lightbox avec vignettes latérales                                       |
| Tableau de bord agent | cartes empilées, tableaux transformés en listes de cartes                       | 2 colonnes                             | tableaux complets, 4 indicateurs en ligne                               |

**Règles transverses**

- Cible tactile de 44 px minimum, espacement de 8 px entre deux cibles adjacentes.

- Les actions principales restent dans le tiers inférieur de l'écran en mobile.

- Aucun défilement horizontal, sauf le tableau de comparaison où il est explicite et guidé par une ombre de bord.

- Les images sont servies en srcset avec trois tailles ; la taille chargée dépend du palier.

- Les tableaux agent et admin ne sont jamais compressés en mobile : ils sont reconstruits en cartes, chaque ligne devenant une carte avec ses actions.

**16. Design system et direction artistique**

Direction retenue : calme et éditoriale. Beaucoup de blanc, une seule couleur d'accent, la photographie comme seul élément vivant. L'objectif est qu'ImmoMatch ne ressemble pas à un portail d'annonces saturé.

**Couleurs**

|                     |            |                                         |
|---------------------|------------|-----------------------------------------|
| **Jeton**           | **Valeur** | **Usage**                               |
| --color-ink         | \#101720   | texte principal                         |
| --color-ink-soft    | \#5A6672   | texte secondaire, libellés              |
| --color-surface     | \#FFFFFF   | fond des cartes                         |
| --color-canvas      | \#F6F7F5   | fond de page, légèrement chaud          |
| --color-border      | \#E3E6E2   | bordures 1 px                           |
| --color-accent      | \#1F5F4B   | vert profond, actions principales       |
| --color-accent-soft | \#E7F0EB   | fonds d'état actif, jetons sélectionnés |
| --color-warning     | \#9A6B12   | statut Presque du score                 |
| --color-danger      | \#A3261F   | erreurs, critère non respecté           |
| --color-focus       | \#0B63CE   | contour de focus, distinct de l'accent  |

Le vert profond évoque la stabilité sans tomber dans le bleu bancaire ni dans le rouge des portails existants. Tous les couples texte sur fond dépassent 4,5 pour 1.

**Typographie et espacements**

|                           |                                                                            |
|---------------------------|----------------------------------------------------------------------------|
| **Jeton**                 | **Valeur**                                                                 |
| Police titres             | Fraunces ou Playfair Display, poids 600                                    |
| Police texte et interface | Inter, poids 400 et 600                                                    |
| Échelle                   | 12, 14, 16, 18, 24, 32, 44 px                                              |
| Interlignage              | 1,5 pour le texte, 1,2 pour les titres                                     |
| Échelle d'espacement      | 4, 8, 12, 16, 24, 32, 48, 64 px                                            |
| Rayons                    | 8 px sur les boutons et champs, 16 px sur les cartes, 999 px sur les chips |
| Ombres                    | une seule, 0 1px 3px rgba(16,23,32,0.08), au survol seulement              |

Une police à empattements sur les titres et une police neutre sur le reste suffisent à distinguer le produit, sans effort de mise en page supplémentaire.

**Composants**

|                    |                                                                               |                                                                   |
|--------------------|-------------------------------------------------------------------------------|-------------------------------------------------------------------|
| **Composant**      | **Variantes**                                                                 | **Règles**                                                        |
| Bouton             | principal plein accent, secondaire contour, texte seul, danger                | hauteur 44 px, 48 px en mobile, libellé verbal                    |
| Chip de critère    | inactif contour pointillé, actif fond accent-soft, renseigné texte souligné   | 32 px de haut, cible tactile élargie à 44 px                      |
| Carte de bien      | liste, grille, infobulle de carte                                             | photo 16 sur 9, prix en 18 px gras, score en pastille haut droite |
| Champ              | texte, nombre, sélection, curseur double                                      | label au-dessus, aide en dessous, erreur en rouge avec icône      |
| Badge              | DPE coloré A à G, statut d'annonce, statut de visite                          | jamais la couleur seule, toujours un libellé                      |
| Pastille de score  | verte, ambre, grise                                                           | pourcentage plus le mot correspondance en infobulle               |
| Modale et feuille  | confirmation, formulaire, galerie                                             | fermeture par Échap, focus piège, retour du focus au déclencheur  |
| État de chargement | squelette pour les listes et les cartes, spinner pour les actions ponctuelles | jamais de page blanche                                            |
| État vide          | illustration légère, une phrase, une action suggérée                          | voir section 20                                                   |
| État d'erreur      | bandeau en ligne pour les erreurs locales, page dédiée pour 404 et 500        | message en français courant, sans code technique                  |

**Wireframe : page de résultats en desktop**

+--------------------------------------------------------------+

\| ImmoMatch Rechercher Favoris Visites \[Compte\] \|

+--------------------------------------------------------------+

\| Je cherche \[un appartement\] à \[Strasbourg\] pour \[\< 900 EUR\] \|

\| avec \[2 chambres\] \[+\] Trajet vers \[Univ v\] \|

+---------------------------------+----------------------------+

\| 37 biens Trier par \[Pertinence v\] \| \|

\| \| carte \|

\| +-----------------------------+ \| \[850\] \[1100\] \|

\| \| \[photo\] T3 lumineux \| \| \[790\] \|

\| \| 850 EUR 64 m2 3p \| \| \[920\] \[680\] \|

\| \| Strasbourg DPE E \| \| \|

\| \| voiture 18 min \| 92% \| \[Rechercher ici\] \|

\| +-----------------------------+ \| \|

\| +-----------------------------+ \| \|

\| \| ... \| \| \|

+---------------------------------+----------------------------+

**Wireframe : page détail**

+--------------------------------------------------------------+

\| \< Retour aux résultats \|

+-------------------------------------+------------------------+

\| \[ galerie photos, 4 vignettes \] \| 850 EUR / mois \|

\| \| + 60 EUR de charges \|

\| T3 lumineux proche Krutenau \| \|

\| Strasbourg 64 m2 3 pieces 2 ch. \| 92 % correspondance \|

\| \| v Budget respecté \|

\| Description \| v Surface respectée \|

\| ... \| ! DPE inférieur \|

\| \| \|

\| Équipements \[parking\] \[balcon\] \| Mon trajet vers \|

\| DPE \[ E \] \| \[ Université v \] \|

\| \| voiture 18 transp 24 \|

\| +-----------------------------+ \| vélo 16 marche 42 \|

\| \| carte centrée sur le bien \| \| \|

\| +-----------------------------+ \| \[ Demander une visite\]\|

\| \| \[ Ajouter aux favoris\]\|

\| Agent : M. Dupont, Agence X \| \|

+-------------------------------------+------------------------+

**Recommandations différenciantes**

- Afficher le prix au m2 à côté du prix, calculé, sur toutes les cartes : information rarement donnée d'emblée ailleurs.

- Montrer le trajet sur la carte de résultat dès qu'une destination est active, pas seulement en page détail : c'est là que la promesse du produit devient visible.

- Ne jamais afficher une pastille de score sans son explication au survol ou au tap, sous peine de paraître arbitraire.

- Conserver les photos en proportions constantes, quitte à recadrer, pour une grille régulière.

- Limiter l'interface à une seule action principale par écran, mise en avant en couleur accent.

**17. User stories et parcours**

Les identifiants US sont repris tels quels dans le backlog de la section 18.

**Recherche et résultats**

|        |                                                                                                                                                |
|--------|------------------------------------------------------------------------------------------------------------------------------------------------|
| **ID** | **User story**                                                                                                                                 |
| US-01  | En tant que visiteur, je veux exprimer ma recherche sous forme de phrase à compléter, afin de définir mes critères sans remplir un formulaire. |
| US-02  | En tant que visiteur, je veux voir les biens correspondants en liste paginée, afin de parcourir l'offre rapidement.                            |
| US-03  | En tant que visiteur, je veux trier les résultats par prix, surface, date ou pertinence, afin d'organiser ma lecture.                          |
| US-04  | En tant que visiteur, je veux affiner par équipement et par DPE, afin de ne voir que des biens acceptables.                                    |
| US-05  | En tant que visiteur, je veux retrouver ma recherche via l'URL, afin de la partager ou de revenir dessus.                                      |

**Carte**

|        |                                                                                                                                           |
|--------|-------------------------------------------------------------------------------------------------------------------------------------------|
| **ID** | **User story**                                                                                                                            |
| US-06  | En tant que visiteur, je veux voir les biens sur une carte à côté de la liste, afin de situer l'offre géographiquement.                   |
| US-07  | En tant que visiteur, je veux survoler un résultat et voir son marqueur se mettre en évidence, afin de faire le lien entre les deux vues. |
| US-08  | En tant que visiteur, je veux relancer la recherche sur la zone affichée, afin de chercher là où je regarde.                              |
| US-09  | En tant que visiteur mobile, je veux basculer entre liste et carte, afin de profiter de tout l'écran.                                     |

**Trajet**

|        |                                                                                                                                                                       |
|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **ID** | **User story**                                                                                                                                                        |
| US-10  | En tant que visiteur, je veux saisir une adresse de destination et voir le temps de trajet depuis un bien, afin de juger sa localisation par rapport à mon quotidien. |
| US-11  | En tant que visiteur, je veux comparer voiture, transports, vélo et marche, afin de choisir selon mon mode de déplacement.                                            |
| US-12  | En tant que particulier, je veux enregistrer mes destinations habituelles, afin de ne pas ressaisir mon adresse à chaque visite.                                      |
| US-13  | En tant que particulier, je veux filtrer les biens à moins de N minutes de ma destination, afin d'écarter d'emblée les trop éloignés.                                 |

**Matching et profil**

|        |                                                                                                                                                |
|--------|------------------------------------------------------------------------------------------------------------------------------------------------|
| **ID** | **User story**                                                                                                                                 |
| US-14  | En tant que particulier, je veux enregistrer mon projet immobilier, afin de ne pas redéfinir mes critères.                                     |
| US-15  | En tant que particulier, je veux voir un pourcentage de correspondance sur chaque bien, afin de repérer les plus pertinents.                   |
| US-16  | En tant que particulier, je veux comprendre le détail du score, afin de savoir quel critère n'est pas respecté.                                |
| US-17  | En tant que particulier, je veux recevoir des recommandations basées sur mon projet, afin de découvrir des biens que je n'aurais pas cherchés. |

**Favoris, comparaison et visite**

|        |                                                                                                                    |
|--------|--------------------------------------------------------------------------------------------------------------------|
| **ID** | **User story**                                                                                                     |
| US-18  | En tant que particulier, je veux sauvegarder un bien en favori, afin de le retrouver plus tard.                    |
| US-19  | En tant que particulier, je veux comparer jusqu'à quatre biens côte à côte, afin de trancher entre mes finalistes. |
| US-20  | En tant que particulier, je veux demander une visite sur un créneau, afin d'avancer concrètement.                  |
| US-21  | En tant que particulier, je veux suivre l'état de mes demandes, afin de savoir où j'en suis.                       |

**Agent et administration**

|        |                                                                                                                     |
|--------|---------------------------------------------------------------------------------------------------------------------|
| **ID** | **User story**                                                                                                      |
| US-22  | En tant qu'agent, je veux créer une annonce guidée en quatre étapes, afin de publier sans oublier d'information.    |
| US-23  | En tant qu'agent, je veux téléverser et ordonner mes photos, afin de mettre le bien en valeur.                      |
| US-24  | En tant qu'agent, je veux désactiver une annonce louée, afin de ne plus recevoir de demandes.                       |
| US-25  | En tant qu'agent, je veux traiter les demandes de visite, afin d'organiser mon agenda.                              |
| US-26  | En tant qu'agent, je veux voir les vues et favoris de mes annonces, afin de savoir lesquelles fonctionnent.         |
| US-27  | En tant qu'administrateur, je veux modérer les annonces publiées, afin de garantir la qualité du catalogue.         |
| US-28  | En tant qu'administrateur, je veux suspendre un compte, afin de traiter un abus.                                    |
| US-29  | En tant qu'administrateur, je veux consulter les statistiques globales, afin de suivre l'activité de la plateforme. |

**Parcours principal : du besoin à la demande de visite**

flowchart TD

A\[Accueil\<br/\>phrase de recherche\] --\> B\[Résultats\<br/\>liste + carte\]

B --\> C\[Saisie destination\<br/\>Université\]

C --\> D\[Temps de trajet\<br/\>sur chaque bien\]

D --\> E\[Page détail\<br/\>score expliqué\]

E --\> F{Convaincu ?}

F --\>\|pas encore\| G\[Favori\] --\> H\[Comparaison\<br/\>2 à 4 biens\]

H --\> I\[Demande de visite\]

F --\>\|oui\| I

I --\> J\[Agent accepte\<br/\>notification\]

**Parcours secondaires**

1.  Création de projet : recherche en cours, clic sur Enregistrer ce projet, nommage, choix des critères obligatoires, activation, puis apparition des scores sur toute l'application.

2.  Publication d'annonce : connexion agent, nouvelle annonce, saisie du bien, géocodage de l'adresse avec ajustement du marqueur, ajout de 5 photos, prévisualisation, publication, apparition dans les résultats publics.

3.  Traitement d'une demande : notification de l'agent, ouverture de la demande, acceptation, notification du particulier, créneau marqué indisponible.

4.  Modération : signalement ou nouvelle annonce, ouverture par l'administrateur, rejet avec motif, l'annonce repasse chez l'agent en REJECTED, correction, republication.

**18. Backlog et planning**

Complexité en points, échelle 1, 2, 3, 5, 8. Priorité : 1 indispensable à la soutenance, 2 importante, 3 confort, B bonus.

**Backlog MVP**

|        |                                         |                     |          |          |            |                                                                                                                                  |
|--------|-----------------------------------------|---------------------|----------|----------|------------|----------------------------------------------------------------------------------------------------------------------------------|
| **ID** | **Fonctionnalité**                      | **US**              | **Prio** | **Cplx** | **Sprint** | **Critères d'acceptation**                                                                                                       |
| F01    | Socle projet, Docker, Flyway, CI        |                     | 1        | 5        | 0          | docker compose up démarre les 3 services ; la pipeline passe sur une PR vide                                                     |
| F02    | Modèle de données et migrations         |                     | 1        | 5        | 0          | les 14 tables et index sont créés ; le jeu de démonstration charge 150 biens                                                     |
| F03    | Inscription et connexion JWT            |                     | 1        | 5        | 1          | un compte créé peut se connecter ; un jeton expiré renvoie 401 ; le mot de passe est haché                                       |
| F04    | Rôles et guards Angular                 |                     | 1        | 3        | 1          | /agent est inaccessible à un particulier côté front et côté API                                                                  |
| F05    | CRUD annonce                            | US-22, US-24        | 1        | 8        | 1          | un agent créé, modifié, publié et archivé ; un autre agent reçoit 403                                                            |
| F06    | Upload et gestion des photos            | US-23               | 1        | 5        | 2          | 1 à 12 photos, ordre modifiable, photo principale, refus au-dela de 5 Mo                                                         |
| F07    | Recherche multicritère et pagination    | US-02, US-03, US-04 | 1        | 8        | 2          | tous les filtres de la section 4.1 fonctionnent et se combinent ; 12 résultats par page                                          |
| F08    | Page détail                             |                     | 1        | 3        | 2          | toutes les informations et la galerie s'affichent ; un identifiant inconnu donne une page 404                                    |
| F09    | Recherche naturelle                     | US-01, US-05        | 1        | 8        | 3          | chaque jeton s'édite et met à jour les résultats ; l'URL reflète l'état ; utilisable au clavier                                  |
| F10    | Carte interactive et liste synchronisée | US-06, US-07, US-08 | 1        | 8        | 3          | marqueurs, clustering, survol croise, recherche dans la zone                                                                     |
| F11    | Géocodage des adresses d'annonce        |                     | 1        | 3        | 3          | l'adresse saisie par l'agent produit des coordonnées ; le marqueur est ajustable                                                 |
| F12    | Calcul de trajet et cache               | US-10, US-11        | 1        | 8        | 4          | 4 modes affichés en page détail ; le second appel identique ne consomme pas de quota ; une panne renvoie 503 sans casser la page |
| F13    | Destinations enregistrées               | US-12               | 1        | 3        | 4          | 3 destinations maximum, une par défaut, sélecteur global                                                                         |
| F14    | Profil de recherche                     | US-14               | 1        | 5        | 4          | création depuis une recherche, un seul profil actif                                                                              |
| F15    | ImmoMatch Score et explication          | US-15, US-16        | 1        | 8        | 5          | score affiché sur les cartes et en détail ; explication ligne par ligne conforme à la section 7                                  |
| F16    | Favoris                                 | US-18               | 1        | 3        | 5          | ajout et retrait depuis 3 emplacements ; doublon impossible                                                                      |
| F17    | Comparaison                             | US-19               | 1        | 5        | 5          | 2 à 4 biens, meilleure valeur mise en évidence, prix au m2 calculé                                                               |
| F18    | Demande de visite                       | US-20, US-21        | 1        | 5        | 6          | créneau à plus de 24 h, une demande active par bien, suivi du statut                                                             |
| F19    | Traitement des demandes par l'agent     | US-25               | 1        | 3        | 6          | accepter, refuser avec motif ; le créneau accepté devient indisponible                                                           |
| F20    | Tableau de bord agent                   | US-26               | 2        | 5        | 6          | 4 indicateurs, vues sur 30 jours par annonce                                                                                     |
| F21    | Espace administrateur et modération     | US-27, US-28, US-29 | 2        | 5        | 7          | rejet avec motif obligatoire, suspension d'agent qui dépublié ses annonces                                                       |
| F22    | Recommandations                         | US-17               | 2        | 5        | 7          | 10 biens triés par score, critères obligatoires respectés                                                                        |
| F23    | Filtre par temps de trajet              | US-13               | 2        | 5        | 7          | filtre à N minutes appliqué aux résultats                                                                                        |
| F24    | Responsive complet et accessibilité     | US-09               | 1        | 8        | 7          | les 3 paliers de la section 15 sont conformes ; axe ne signale aucune erreur critique sur 3 pages                                |
| F25    | Gestion des erreurs et états vides      |                     | 1        | 3        | 8          | les 11 cas de la section 20 sont couverts                                                                                        |
| F26    | Tests, documentation, finalisation      |                     | 1        | 8        | 8          | couverture conforme à la section 14 ; Swagger publié ; README d'installation valide sur une machine vierge                       |

**Bonus**

|        |                                                         |          |          |                                             |
|--------|---------------------------------------------------------|----------|----------|---------------------------------------------|
| **ID** | **Fonctionnalité**                                      | **Prio** | **Cplx** | **Condition**                               |
| B01    | Transports en commun réels via GTFS ou Navitia          | B        | 8        | remplace l'estimation de la section 6       |
| B02    | Isochrones sur la carte, zone atteignable en 20 minutes | B        | 5        | endpoint ORS Isochrones, déjà dans le quota |
| B03    | Alertes par courriel sur nouveau bien correspondant     | B        | 5        | nécessite un service SMTP                   |
| B04    | Export PDF de la comparaison                            | B        | 3        |                                             |
| B05    | Recherche enregistrée avec historique                   | B        | 3        |                                             |
| B06    | Mode sombre                                             | B        | 2        | jetons de couleur déjà en variables CSS     |
| B07    | PostGIS et recherche par rayon réel                     | B        | 5        | remplace le filtre par emprise              |
| B08    | Messagerie agent et particulier                         | B        | 8        | hors périmètre raisonnable                  |

**Planning**

Neuf sprints d'une semaine, environ 12 à 15 heures par étudiant et par semaine, soit environ 250 heures au total.

|            |             |                                           |                                     |
|------------|-------------|-------------------------------------------|-------------------------------------|
| **Sprint** | **Semaine** | **Objectif**                              | **Démontrable à la fin**            |
| 0          | 1           | socle, modèle, CI                         | l'application vide tourne en Docker |
| 1          | 2           | authentification, rôles, CRUD annonce     | un agent publie une annonce         |
| 2          | 3           | photos, recherche, page détail            | on cherche et on consulte un bien   |
| 3          | 4           | recherche naturelle, carte                | la recherche signature fonctionne   |
| 4          | 5           | trajets, destinations, profil             | le temps de trajet s'affiche        |
| 5          | 6           | score, favoris, comparaison               | la promesse produit est complète    |
| 6          | 7           | visites, tableau de bord agent            | le cycle complet est jouable        |
| 7          | 8           | admin, recommandations, responsive        | tous les rôles sont couverts        |
| 8          | 9           | erreurs, tests, documentation, repetition | version de soutenance figée         |

Deux jalons de sécurité : à la fin du sprint 4, la fonctionnalité différenciante doit être démontrable ; si elle ne l'est pas, F22, F23 et le tableau de bord agent passent en bonus. A la fin du sprint 7, le périmètre est gelé et plus aucune fonctionnalité n'est ajoutée.

**Rituels**

- Planification de 30 minutes en début de semaine, avec engagement chiffré en points.

- Point de 15 minutes deux fois par semaine, en visio ou en présentiel.

- Revue et rétrospective de 45 minutes en fin de sprint, avec une seule action d'amélioration retenue.

- Tableau GitHub Projects à 4 colonnes : À faire, En cours, En revue, Terminé.

**19. Répartition du travail**

La répartition proposée dans le sujet est conservée dans son principe, avec un ajustement : chaque étudiant est propriétaire de domaines fonctionnels complets, du composant Angular jusqu'à la requête SQL. Personne n'est cantonné à une couche.

|                                       |                                     |
|---------------------------------------|-------------------------------------|
| **Étudiant A, parcours de recherche** | **Étudiant B, parcours de gestion** |
| F07 recherche multicritère            | F03 authentification et JWT         |
| F09 recherche naturelle               | F04 rôles et guards                 |
| F10 carte interactive                 | F05 CRUD annonce                    |
| F11 géocodage des annonces            | F06 photos                          |
| F12 calcul de trajet et cache         | F08 page détail                     |
| F13 destinations                      | F18 demande de visite               |
| F15 ImmoMatch Score                   | F19 traitement par l'agent          |
| F16 favoris                           | F20 tableau de bord agent           |
| F17 comparaison                       | F21 administration et modération    |
| F14 profil de recherche               | F22 recommandations, côté écran     |
| F23 filtré par trajet                 |                                     |

Travail commun, mené en binôme sur créneau partagé : F01 socle et CI, F02 modèle de données, F24 responsive et accessibilité, F25 gestion des erreurs, F26 tests et documentation, ainsi que le design system de la section 16.

**Pourquoi ce découpage**

A porte la valeur différenciante du produit, qui est aussi la partie la plus risquée ; B porte le volume fonctionnel et les trois espaces utilisateurs. Les deux charges sont équilibrées en points : environ 60 points pour A et 55 pour B, plus 30 points de travail commun.

Le seul point de contact fort est le contrat d'API des annonces : B le définit et le documente au sprint 1, A le consomme. Le DTO PropertySummary est donc figé tôt et versionné dans le dépôt.

**Coordination**

- Les DTO partagés sont écrits ensemble avant d'être implémentés, dans un fichier Markdown du dépôt, avant toute ligne de code.

- Chaque pull request est relue par l'autre ; la relecture sert aussi à transmettre la connaissance des domaines.

- Au moins une session de programmation en binome par sprint, sur la partie la plus incertaine de la semaine.

- Le sprint 8 est intégralement commun : tests, documentation, correction et répétition de la soutenance.

- En cas de retard sur un domaine, le rattrapage se fait par transfert d'un élément de backlog entier, jamais par découpage d'une fonctionnalité en cours.

**20. Gestion des erreurs et cas particuliers**

Principe unique : l'utilisateur doit toujours comprendre ce qui s'est passé et savoir quoi faire ensuite. Aucun code technique n'apparaît à l'écran, et aucune erreur secondaire ne bloque la page entière.

|                                  |                                                                                   |                                                                                        |                                                           |
|----------------------------------|-----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|-----------------------------------------------------------|
| **Cas**                          | **Ce que voit l'utilisateur**                                                     | **Action proposée**                                                                    | **Technique**                                             |
| Aucun résultat                   | Aucun bien ne correspond à ces critères, avec rappel des critères actifs          | relâcher le critère le plus filtrant, élargir la zone, enregistrer une alerte en bonus | 200 avec liste vide, jamais 404                           |
| Adresse introuvable              | Nous n'avons pas trouvé cette adresse                                             | reformuler, ajouter la ville, ou placer le point manuellement sur la carte             | 0 résultat de géocodage, aucune destination créée         |
| Adresse ambiguë                  | Plusieurs adresses correspondent                                                  | choisir parmi 3 propositions                                                           | liste retournée, choix obligatoire                        |
| API Maps indisponible            | bandeau discret, Calcul des trajets momentanément indisponible                    | réessayer, le reste de la page fonctionne                                              | 503, valeurs en cache affichées si présentes              |
| Erreur réseau du navigateur      | Connexion perdue, vos critères sont conservés                                     | bouton Réessayer, aucun rechargement force                                             | intercepteur Angular, une seule notification à la fois    |
| Annonce supprimée ou archivée    | Ce bien n'est plus disponible, avec 3 biens similaires proposés                   | retour aux résultats, voir les similaires                                              | 404 ou 410 ; le favori reste liste mais grise             |
| Session expirée                  | Votre session a expiré, reconnectez-vous                                          | modale de reconnexion qui rejoue l'action après succès                                 | 401, tentative de rafraîchissement automatique d'abord    |
| Formulaire invalide              | message sous chaque champ, résumé en haut si plus de 3 erreurs                    | correction guidée, focus sur le premier champ en erreur                                | 400 avec fieldErrors, validation aussi côté client        |
| Image invalide                   | Format non accepté ou fichier trop lourd, 5 Mo maximum                            | choisir un autre fichier ; les autres images du lot sont conservées                    | 400 ou 413, vérification avant envoi                      |
| Créneau de visite indisponible   | Ce créneau vient d'être réservé                                                   | choisir un autre créneau, la liste se rafraîchit                                       | 409, vérification en transaction                          |
| Données immobilières incomplètes | mention Non renseigné à la place de la valeur, jamais un tiret ni un zero         | le bien reste consultable                                                              | le champ manquant est traité dans le score, section 7     |
| Itinéraire impossible            | Itinéraire non disponible pour ce mode                                            | les autres modes restent affichés                                                      | code d'erreur ORS traduit en message métier               |
| Quota API épuisé                 | Calcul indisponible aujourd'hui, valeurs mises à jour le mois dernier si en cache | réessayer plus tard                                                                    | 403 ORS, journalisé pour alerter l'équipe                 |
| Erreur serveur inattendue        | page dédiée avec un identifiant d'incident court                                  | retour à l'accueil                                                                     | 500, trace complète en journal, jamais renvoyée au client |

**États vides**

|                                        |                                               |                      |
|----------------------------------------|-----------------------------------------------|----------------------|
| **Écran**                              | **Message**                                   | **Action**           |
| Favoris vide                           | Vous n'avez pas encore de favoris             | Parcourir les biens  |
| Comparaison vide                       | Sélectionnez au moins deux biens à comparer   | Ouvrir mes favoris   |
| Recommandations sans profil            | Créez un projet pour recevoir des suggestions | Créer mon projet     |
| Recommandations avec profil trop vague | Ajoutez un budget ou une ville pour affiner   | Compléter mon projet |
| Visites vide                           | Aucune demande de visite pour le moment       | Chercher un bien     |
| Annonces agent vide                    | Publiez votre première annonce                | Créer une annonce    |
| File de modération vide                | Aucune annonce en attente                     | aucune               |

**21. Livrables et soutenance**

**Livrables**

|                              |                                            |                                                                                         |                                 |
|------------------------------|--------------------------------------------|-----------------------------------------------------------------------------------------|---------------------------------|
| **Livrable**                 | **Format**                                 | **Contenu**                                                                             | **Quand**                       |
| Cahier des charges           | ce document                                | périmètre, règles métier, architecture                                                  | sprint 0, mis à jour en continu |
| Maquettes                    | Figma                                      | accueil, résultats, détail, comparaison, espace agent, en desktop et mobile             | sprint 1                        |
| UML                          | PlantUML dans le dépôt                     | cas d'utilisation, classes du domaine, sequence du calcul de trajet, états de l'annonce | sprint 2                        |
| MCD et MLD                   | section 11 plus schéma généré              | modèle relationnel complet                                                              | sprint 0                        |
| Architecture                 | schémas de la section 10                   | vue de déploiement et vue en couches                                                    | sprint 0                        |
| Code source                  | dépôt Git                                  | historique de commits équilibré entre les deux étudiants                                | continu                         |
| Documentation API            | Swagger UI via springdoc                   | tous les endpoints, essayables                                                          | continu                         |
| Documentation d'installation | README                                     | prérequis, variables d'environnement, docker compose up, comptes de démonstration       | sprint 8                        |
| Rapport de tests             | export SonarCloud plus tableau de synthèse | couverture, résultats, quality gate                                                     | sprint 8                        |
| CI/CD                        | workflows GitHub Actions                   | historique des exécutions                                                               | continu                         |
| Rapport de projet            | PDF, 25 à 35 pages                         | démarche, choix techniques justifiés, difficultés, bilan individuel                     | sprint 8                        |
| Présentation                 | 15 à 20 diapositives                       | probleme, solution, démonstration, technique, bilan                                     | sprint 8                        |
| Démonstration                | application déployée plus vidéo de secours | scenario ci-dessous                                                                     | sprint 8                        |

La vidéo de secours n'est pas un détail : elle protège contre une panne de réseau ou de quota le jour J.

**Scenario de démonstration, 12 minutes**

|             |                        |                                                                                         |                                                                          |
|-------------|------------------------|-----------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| **Temps**   | **Étape**              | **Ce qui est montré**                                                                   | **Point à souligner**                                                    |
| 0 à 1 min   | contexte               | le problème en une phrase                                                               | pas un clone de portail existant                                         |
| 1 à 3 min   | recherche naturelle    | composition de la phrase critère par critère, sur desktop                               | aucun formulaire, l'URL suit l'état                                      |
| 3 à 4 min   | résultats et carte     | survol croise liste et carte, recherche dans la zone                                    | les deux vues sont synchronisées                                         |
| 4 à 6 min   | temps de trajet        | saisie de Université de Strasbourg, quatre modes sur un bien, puis badge sur les autres | fonctionnalité centrale, avec mention honnête de l'estimation transports |
| 6 à 7 min   | ImmoMatch Score        | ouverture du détail, explication ligne par ligne, y compris un critère non respecté     | le score est explicable, pas une boîte noire                             |
| 7 à 8 min   | favoris et comparaison | ajout de deux biens, tableau comparatif, prix au m2                                     | aide réelle à la décision                                                |
| 8 à 9 min   | demande de visite      | choix d'un créneau, envoi                                                               | fin du parcours particulier                                              |
| 9 à 10 min  | espace agent           | réception de la demande, acceptation, tableau de bord                                   | le cycle est complet                                                     |
| 10 à 11 min | mobile et résilience   | même parcours sur mobile, puis coupure simulée de l'API de trajet                       | responsive pense pour le mobile, dégradation maîtrisée                   |
| 11 à 12 min | technique et bilan     | schéma d'architecture, pipeline verte, couverture de tests                              | qualité de réalisation                                                   |

**Préparation**

- Jeu de données fige la veille : 150 biens sur 3 villes, 3 comptes prêts, particulier, agent et administrateur, mots de passe notes.

- Cache de trajets préchauffé sur les biens de la démonstration, pour éviter toute latence et tout risque de quota.

- Deux répétitions chronométrées au sprint 8, dont une avec le réseau coupé.

- Répartition de la parole équilibrée : chaque étudiant présente les domaines qu'il a construits.

- Questions à anticiper : pourquoi pas Google Maps, comment se comporte le score sans DPE, que se passe-t-il à 10 000 annonces, comment sont protégées les adresses personnelles.

**22. Risques et perspectives**

**Risques**

|                                            |                 |            |                                                                                                                                                                                                      |
|--------------------------------------------|-----------------|------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Risque**                                 | **Probabilité** | **Impact** | **Réduction**                                                                                                                                                                                        |
| Complexité des APIs cartographiques        | moyenne         | élevé      | prototype jetable de Leaflet plus openrouteservice dès le sprint 0, avant tout engagement ; service d'itinéraire isolé derrière une interface Java pour pouvoir changer de fournisseur en une classe |
| Quotas ou coûts imprévus                   | faible          | élevé      | fournisseur sans facturation possible ; cache obligatoire ; compteur d'appels journalier journalisé ; cache préchauffé avant la soutenance                                                           |
| Recherche naturelle plus longue que prévue | élevée          | élevé      | vue formulaire classique livrée d'abord au sprint 2, la phrase venant par-dessus au sprint 3 ; les deux partagent le même modèle d'état, donc la vue classique reste un repli fonctionnel            |
| Temps de développement sous-estimé         | élevée          | élevé      | jalons de gel aux sprints 4 et 7 ; bonus clairement séparés ; points chiffres et vélocité mesurée dès le sprint 1                                                                                    |
| Responsive traité trop tard                | moyenne         | moyen      | mobile d'abord dès le premier composant ; vérification à chaque fin de sprint sur un vrai téléphone, pas seulement dans le navigateur                                                                |
| Upload des images                          | moyenne         | moyen      | bibliothèque éprouvée pour le redimensionnement, Thumbnailator ; limites strictes ; tests d'intégration sur fichier invalide dès le sprint 2                                                         |
| Sécurité insuffisante                      | moyenne         | élevé      | vérification de propriété systématique en service ; tests d'accès interdit dédiés ; analyse SonarCloud sur chaque pull request                                                                       |
| Matching jugé arbitraire                   | moyenne         | moyen      | formule écrite dans le cahier des charges avant le code, tests unitaires sur les cas limites, explication affichée en permanence                                                                     |
| Désynchronisation frontend et backend      | élevée          | moyen      | contrat DTO écrit et versionné avant implémentation ; Swagger comme reference unique ; génération possible des types TypeScript depuis OpenAPI                                                       |
| Indisponibilité d'un étudiant              | faible          | élevé      | revue croisée systématique, personne n'est seul à comprendre un domaine ; documentation dans le dépôt, pas dans une tête                                                                             |
| Panne le jour de la soutenance             | faible          | très élevé | version déployée plus version locale plus vidéo de secours, testées toutes les trois                                                                                                                 |

**Perspectives, hors MVP**

1.  Transports en commun réels par intégration GTFS, ce qui rendrait le quatrième mode exact et non estimé.

2.  Isochrones sur la carte : dessiner la zone atteignable en 20 minutes autour d'une destination et n'afficher que les biens à l'intérieur.

3.  Score enrichi par des données ouvertes : commerces, écoles, transports à proximité, via l'API Overpass, ce qui ferait évoluer le critère localisation d'une ville vers un véritable indice de qualité de vie.

4.  Alertes et courriels hebdomadaires sur les nouveaux biens correspondant à un projet.

5.  Estimation de prix par comparaison avec les biens similaires déjà publiés, pour signaler les annonces au-dessus du marché local.

6.  Application mobile ou PWA installable avec favoris hors ligne.

7.  Import de flux d'annonces au format standard pour alimenter le catalogue automatiquement.

8.  Messagerie intégrée entre agent et particulier, qui remplacerait l'échange par courriel autour des visites.

Ces pistes sont mentionnées en soutenance comme des évolutions maîtrisées, jamais comme des manqués du MVP.
