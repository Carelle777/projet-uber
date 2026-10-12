# 🚕 Uber Pickups : où placer les chauffeurs à New York ?

Projet de certification Jedha Fullstack (RNCP38777) — **Carelle Nouko**

## 1. Le problème

Quand un client attend trop longtemps sa voiture (plus de 5 minutes), il peut annuler et aller chez un concurrent.
Les chauffeurs, eux, ne savent pas toujours où se placer pour trouver des clients.

**Mon objectif :** à partir des courses Uber de New York (avril 2014), trouver les **« hot-zones »**, c'est-à-dire
les endroits où il y a beaucoup de demandes, pour dire aux chauffeurs où se placer.

## 2. Les données

- **Source :** les courses Uber de New York, avril 2014 (fichier `uber-raw-data-apr14.csv`, fourni par Jedha,
  aussi disponible sur Kaggle sous « Uber Pickups in New York City »).
- **Une ligne = une course**, avec sa date et heure, sa latitude et sa longitude.
- Le fichier est trop gros pour GitHub (26 Mo) : il n'est **pas** dans ce dépôt. Il faut le placer dans le dossier `data/`.
- **Total du mois :** 564 516 courses.

**Ma méthode « Start Small, Grow Big » :** je teste d'abord sur **un seul créneau**, le **jeudi à 18h** (6 637 courses),
parce que les zones chaudes changent selon le jour et l'heure. Ensuite j'étends à toute la semaine.

## 3. Ce que fait le notebook (`notebooks/01-uber.ipynb`)

| Étape | Ce que je fais | Pourquoi |
|---|---|---|
| 1 | Je charge les données, je crée le jour et l'heure, je garde le jeudi 18h | Tester sur un petit morceau d'abord |
| 2 | Méthode du coude : je teste K-Means avec 2 à 10 zones | Choisir combien de zones créer |
| 3 | Carte K-Means avec 5 zones | Voir les zones sur New York |
| 4 | DBSCAN : zones de densité | Écarter les courses isolées |
| 5 | Je calcule le centre GPS de chaque zone DBSCAN | Donner des adresses précises aux chauffeurs |
| 6 | Volume de courses par jour | Voir quels jours sont les plus chargés |
| 7 | DBSCAN pour chacun des 7 jours à 18h | Décrire les zones pour toute la semaine |
| 8 | Score de silhouette pour K = 4, 5, 6, 7 | Vérifier le choix de K avec un 2e outil |
| 9 | Test de sensibilité de DBSCAN | Vérifier que mes réglages ne sont pas un hasard |

Avant tout calcul, je **remets la latitude et la longitude à la même échelle** (`StandardScaler`),
pour que les distances soient calculées de façon équitable.

## 4. Les deux méthodes utilisées

- **K-Means :** je dis à l'algorithme « fais-moi 5 zones », et il place 5 centres. Problème : il rattache
  **toutes** les courses à une zone, même celles qui sont isolées, et cela agrandit les zones.
- **DBSCAN :** il cherche les endroits **denses**. Une zone existe si au moins 15 courses sont proches les unes
  des autres (`min_samples=15`, `eps=0.2`). Les courses isolées sont mises de côté (le « bruit », cluster `-1`).

## 5. Résultats (jeudi 18h, avril 2014)

**Choix de K pour K-Means.** L'inertie baisse à chaque zone ajoutée, sans coude net :

| K | 2 | 3 | 4 | **5** | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|
| Inertie | 9 297 | 6 364 | 4 707 | **3 757** | 3 096 | 2 522 | 2 099 | 1 752 | 1 478 |

Comme le coude n'est pas net, j'ai vérifié avec le **score de silhouette** (plus il est haut, mieux les zones sont séparées) :

| K | 4 | **5** | 6 | 7 |
|---|---|---|---|---|
| Silhouette | 0,528 | **0,533** | 0,521 | 0,524 |

K=5 a le meilleur score, mais l'écart avec les autres est **très petit** : K=5 est un choix raisonnable, pas un choix « prouvé ».

**DBSCAN (eps=0,2 ; min_samples=15) :** **8 zones** et **266 courses isolées** (environ 4 % des 6 637 courses).

Les 8 centres GPS à envoyer aux chauffeurs (jeudi 18h) :

| Zone | Latitude | Longitude |
|---|---|---|
| 0 | 40,7186 | -73,9561 |
| 1 | 40,7464 | -73,9867 |
| 2 | 40,7729 | -73,8858 |
| 3 | 40,7707 | -73,8664 |
| 4 | 40,6450 | -73,7857 |
| 5 | 40,6929 | -74,1787 |
| 6 | 40,6877 | -73,9838 |
| 7 | 40,7217 | -74,0367 |

**Test de sensibilité :** quand je change `eps`, le résultat change doucement, sans rupture :

| eps | 0,1 | 0,15 | **0,2** | 0,25 | 0,3 |
|---|---|---|---|---|---|
| Zones | 11 | 9 | **8** | 6 | 5 |
| Courses isolées | 529 | 358 | **266** | 227 | 182 |

`eps=0,2` est donc un choix raisonnable, mais des valeurs proches resteraient défendables.

## 6. Et les autres jours ? (à 18h)

| Jour | Courses | Zones | Courses isolées | % isolées |
|---|---|---|---|---|
| Lundi | 4 725 | 7 | 256 | 5,4 % |
| Mardi | 7 089 | 7 | 299 | 4,2 % |
| Mercredi | 8 334 | 10 | 325 | 3,9 % |
| Jeudi | 6 637 | 8 | 266 | 4,0 % |
| Vendredi | 7 258 | 7 | 368 | 5,1 % |
| Samedi | 6 165 | 6 | 300 | 4,9 % |
| Dimanche | 2 795 | 6 | 170 | 6,1 % |

Les jours les plus chargés (courses par jour, en moyenne sur le mois) sont le **vendredi (≈ 22 600), le mercredi (≈ 21 700)
et le jeudi (≈ 21 300)**. Avril 2014 a 5 mardis et 5 mercredis, donc j'ai ramené les totaux à un jour moyen pour comparer
équitablement. Le dimanche est le jour le plus calme (≈ 12 800).

## 7. Ce que je recommande

1. Envoyer aux chauffeurs les 8 coordonnées GPS pour le jeudi à 18h.
2. Refaire le même calcul pour **chaque jour et chaque heure** : les zones changent selon le créneau.
3. Placer en priorité des chauffeurs du **mercredi au vendredi**, les jours les plus chargés.

## 8. Mes limites

- Je n'ai étudié en détail que **le jeudi 18h**. Pour les autres jours, je compte seulement les zones.
- Les données **ne contiennent pas le temps d'attente** : je montre où est la demande, mais je ne peux **pas prouver**
  que l'attente passera sous 5 minutes. C'est une hypothèse à tester (par exemple avec un test A/B).
- **K-Means et DBSCAN ne répondent pas à la même question**, donc je ne peux pas dire avec un seul chiffre que
  l'un est meilleur que l'autre.
- Le choix de K=5 et celui d'`eps=0,2` sont des choix raisonnables, pas des vérités.
- Données d'**avril 2014 seulement** : c'est une démonstration de méthode, pas un produit prêt à déployer.

## 9. Lancer le projet

```bash
git clone https://github.com/Carelle777/projet-uber.git
cd projet-uber
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
# placer uber-raw-data-apr14.csv dans le dossier data/
# puis ouvrir notebooks/01-uber.ipynb et cliquer sur « Run All »
```

## 10. Structure du projet

```
projet-uber/
├── README.md
├── requirements.txt
├── .gitignore
├── data/                      (le CSV est à placer ici, il n'est pas sur GitHub)
└── notebooks/
    └── 01-uber.ipynb
```

## 11. Petit lexique

- **Hot-zone :** un endroit où il y a beaucoup de demandes de courses.
- **Inertie :** la distance totale entre chaque course et le centre de sa zone. Plus elle est basse, plus les zones sont « serrées ».
- **Silhouette :** une note de qualité des zones (plus elle est haute, mieux les zones sont séparées).
- **Bruit :** les courses isolées que DBSCAN refuse de mettre dans une zone.
- **Centroïde :** le point au centre d'une zone (la moyenne des GPS de ses courses).
- **eps / min_samples :** les deux réglages de DBSCAN (jusqu'où chercher des voisins / combien de voisins il faut pour faire une zone).
- **StandardScaler :** met latitude et longitude à la même échelle avant de calculer des distances.

## 12. Outils

Python · pandas · scikit-learn (KMeans, DBSCAN, StandardScaler, silhouette) · Plotly · Jupyter