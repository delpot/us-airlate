# Notes sur les données

## Sommaire

1. [Colonnes](#1-colonnes)
2. [Variables à créer](#2-variables-à-créer)
3. [Types de jeux de données](#3-types-de-jeux-de-données)
4. [Questions et vues](#4-questions-et-vues)
5. [Sources à ajouter](#5-sources-à-ajouter)
6. [Pièges](#6-pièges)

## 1. Colonnes

Une ligne = un vol prévu à une date donnée, y compris les vols annulés et déroutés.

`*` = définition à vérifier.

| Colonne 2025 | Signification | Type | À traiter |
|---|---|---|---|
| `Year` | Année | Temporel | |
| `Quarter` | Trimestre | Ordinal | |
| `Month` | Mois | Ordinal, cyclique | |
| `DayofMonth` | Jour du mois | Ordinal | |
| `DayOfWeek` | Jour de la semaine, de 1 (lundi) à 7 (dimanche) | Ordinal, cyclique | |
| `FlightDate` | Date du vol | Temporel | Texte → date |
| `Reporting_Airline` | Code de la compagnie qui opère le vol | Nominal | Code → nom |
| `Flight_Number_Reporting_Airline` | Numéro de vol | Nominal | |
| `Origin` | Code de l'aéroport de départ | Nominal | Ajouter les coordonnées |
| `OriginCityName` | Ville de départ | Nominal | |
| `OriginState` | État de départ | Nominal | |
| `Dest` | Code de l'aéroport d'arrivée | Nominal | Ajouter les coordonnées |
| `DestCityName` | Ville d'arrivée | Nominal | |
| `DestState` | État d'arrivée | Nominal | |
| `CRSDepTime` | Heure de départ prévue, locale | Temporel, cyclique | `hhmm` → heure |
| `DepTime` | Heure de départ réelle, locale | Temporel, cyclique | `hhmm` → heure |
| `DepDelay` | Retard au départ en minutes, négatif si en avance | Quantitatif | |
| `DepDelayMinutes` | Retard au départ, avances ramenées à 0 | Quantitatif | |
| `DepDel15` | 1 si retard au départ d'au moins 15 min | Booléen | 0/1 |
| `TaxiOut` | Minutes de roulage au départ | Quantitatif | |
| `WheelsOff` | Heure de décollage, locale | Temporel | `hhmm` → heure |
| `WheelsOn` | Heure d'atterrissage, locale | Temporel | `hhmm` → heure |
| `TaxiIn` | Minutes de roulage à l'arrivée | Quantitatif | |
| `CRSArrTime` | Heure d'arrivée prévue, locale | Temporel, cyclique | `hhmm` → heure |
| `ArrTime` | Heure d'arrivée réelle, locale | Temporel, cyclique | `hhmm` → heure |
| `ArrDelay` | Retard à l'arrivée en minutes, négatif si en avance | Quantitatif | |
| `ArrDelayMinutes` | Retard à l'arrivée, avances ramenées à 0 | Quantitatif | |
| `ArrDel15` | 1 si retard à l'arrivée d'au moins 15 min | Booléen | 0/1 |
| `Cancelled` | 1 si vol annulé | Booléen | 0/1 |
| `CancellationCode` | Motif : A compagnie, B météo, C système aérien, D sécurité `*` | Nominal | Code → libellé |
| `Diverted` | 1 si vol dérouté | Booléen | 0/1 |
| `CRSElapsedTime` | Durée prévue en minutes | Quantitatif | |
| `ActualElapsedTime` | Durée réelle en minutes | Quantitatif | |
| `AirTime` | Temps de vol en minutes | Quantitatif | |
| `Distance` | Distance entre les aéroports, en miles | Quantitatif | |
| `CarrierDelay` | Minutes dues à la compagnie | Quantitatif | |
| `WeatherDelay` | Minutes dues à la météo extrême | Quantitatif | |
| `NASDelay` | Minutes dues au système aérien : trafic, contrôle aérien, opérations d'aéroport, météo non extrême `*` | Quantitatif | |
| `SecurityDelay` | Minutes dues à la sécurité | Quantitatif | |
| `LateAircraftDelay` | Minutes dues à l'arrivée tardive de l'avion | Quantitatif | |

## 2. Variables à créer

| Variable | Type | Construction |
|---|---|---|
| Cause du retard | Nominal | 5 colonnes de causes → une colonne `cause` + une colonne `minutes` |
| Tranche de retard | Ordinal | Annulé, à l'heure (moins de 15 min), 15-60 min, plus d'une heure |
| Nombre de vols | Quantitatif | Compter les lignes par groupe |
| Taux de retard | Quantitatif | Part des vols avec `ArrDel15` = 1 |
| Taux d'annulation | Quantitatif | Part des vols avec `Cancelled` = 1 |
| Route | Nominal | `Origin` + `Dest` |
| Heure de départ | Ordinal, cyclique | Heure extraite de `CRSDepTime` |

## 3. Types de jeux de données

| Type | Statut | Contenu | Usage |
|---|---|---|---|
| Table | Présent | Une ligne par vol | Base de toutes les analyses |
| Table agrégée | À calculer | Compagnie × mois, aéroport × heure | Heatmaps, classements |
| Série temporelle | À calculer | Taux de retard par jour, semaine, mois | Évolution, saisonnalité |
| Réseau | À calculer | Nœuds : aéroports. Liens : routes pondérées | Hubs, points faibles |
| Spatial (points) | À ajouter | Latitude et longitude des aéroports | Carte des aéroports et des routes |
| Géométrie (polygones) | À ajouter | Contours des États | Fond de carte, carte par État |
| Hiérarchie | Optionnel | Année → mois → jour, État → aéroport | Zoom du général au détail |
| Réseau de rotations | Nécessite `Tail_Number` | Enchaînement des vols d'un même avion | Propagation des retards |
| Champ (grille) | Pas utile | Grille météo | Hors sujet |

## 4. Questions et vues

Vues proposées, à valider en groupe.

### Voyageur

| Question | Possible ? | Vue possible | Données |
|---|---|---|---|
| Quelle compagnie choisir ? | Oui | Barres classées | `Reporting_Airline`, `ArrDel15`, `Cancelled` |
| Quel mois ou quelle saison ? | Oui | Courbe temporelle | `Month`, `FlightDate`, `ArrDel15` |
| Quel jour et quelle heure ? | Oui | Heatmap jour × heure | `DayOfWeek`, `CRSDepTime`, `ArrDel15` |
| Quel aéroport ? | Oui | Carte à points, barres classées | `Origin`, `Dest`, coordonnées |
| Quelle route ? | Oui | Fiche de route, distribution des retards | `Origin`, `Dest`, `ArrDelay` |
| Ma correspondance est-elle sûre ? | Non | | Itinéraires absents |

### Analyste ops

| Question | Possible ? | Vue possible | Données |
|---|---|---|---|
| Quels aéroports accumulent du retard ? | Oui | Carte à points | `Origin`, `Dest`, retards, coordonnées |
| Quelles routes sont les plus fragiles ? | Oui | Réseau sur carte | `Origin`, `Dest`, `ArrDelay`, `Cancelled` |
| À quel moment le retard monte-t-il ? | Oui | Courbe par heure | `CRSDepTime`, `DepDelay`, `ArrDelay` |
| Où y a-t-il congestion au sol ? | Oui | Heatmap aéroport × heure | `Origin`, `CRSDepTime`, `TaxiOut` |
| Le temps prévu est-il réaliste ? | Oui | Nuage de points | `CRSElapsedTime`, `ActualElapsedTime` |
| Pourquoi le retard apparaît-il ? | En partie | Barres empilées par cause | 5 colonnes de causes |
| Comment se propage-t-il ? | Non | Réseau de rotations | `Tail_Number` absent |

## 5. Sources à ajouter

| Priorité | Source | Apport | Jointure |
|---|---|---|---|
| Nécessaire | Table des codes compagnies du BTS | Noms des compagnies | `Reporting_Airline` |
| Nécessaire pour la carte | OurAirports ou équivalent | Coordonnées des aéroports | `Origin`, `Dest` |
| Nécessaire pour la carte | `us-atlas` | Contours des États | Noms ou codes d'États |
| Très utile pour l'ops | TranStats | `Tail_Number` | Date, compagnie, numéro de vol, aéroports |
| Avec `Tail_Number` | Fuseaux horaires des aéroports | Comparer des heures locales | `Origin`, `Dest` |
| Optionnel | Table Marketing Carrier du BTS | Marque du billet | Vol |
| Optionnel | Météo horaire de la NOAA | Effet réel de la météo | Aéroport et heure |

Format et licence de ces sources : à vérifier.

## 6. Pièges

| Piège | Conséquence | Réponse |
|---|---|---|
| `Reporting_Airline` n'est pas forcément la marque du billet | Compagnies régionales comptées à part (`OO`, `9E`) | Le préciser, ou ajouter Marketing Carrier |
| Un vol annulé n'a pas de retard | Une compagnie qui annule beaucoup paraît meilleure | Toujours montrer aussi le taux d'annulation |
| Causes renseignées seulement pour les retards d'au moins 15 min à l'arrivée | Pas de cause pour les petits retards | Le préciser dans les vues |
| Météo non extrême comptée dans `NASDelay` `*` | `WeatherDelay` sous-estime la météo | Ne pas conclure sur la météo avec cette seule colonne |
| Causes déclarées par les compagnies | Fiabilité limitée | Rester prudent dans les conclusions |
| Heures locales | Heures non comparables entre fuseaux | Ajouter les fuseaux pour suivre un avion |
| Moyennes tirées par des retards extrêmes | Moyenne trompeuse | Regarder aussi la médiane et le taux de retard |
| Vols intérieurs des compagnies déclarantes uniquement | Pas tout le trafic aérien | Le préciser dans le projet |
| `geoAlbersUsa` ne gère pas Porto Rico ni les autres territoires | Aéroports absents de la carte | Vérifier leur présence, prévoir un traitement à part |
| Trop de lignes pour le navigateur | D3 ne peut pas charger des millions de vols | Exporter de petits fichiers agrégés dans `web/` |

