# 🚕 Uber Pickups : Spatial Clustering & Unsupervised Machine Learning

[![Python](https://img.shields.io/badge/Python-3.11-blue)](https://python.org)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-Clustering-F7931E)](https://scikit-learn.org/)
[![Plotly](https://img.shields.io/badge/Plotly-Mapbox_Dataviz-3F4F75)](https://plotly.com/)

> **Projet de Certification RNCP - Concepteur Développeur en Science des données**
> * ✅ **Bloc 3 :** Analyse Prédictive / Machine Learning Non-Supervisé

---

## 📌 Problématique Métier
L'enjeu opérationnel d'Uber est de réduire le temps de trajet à vide (ETA) de ses chauffeurs. L'objectif de ce projet est d'identifier dynamiquement les "Hot-Zones" de commande (hubs de passagers) à New York City, selon le jour de la semaine et l'heure, pour prépositionner la flotte de véhicules.

## 🏗️ Architecture Algorithmique
L'absence de vérité terrain (pas de variable cible $y$ à prédire) impose la construction d'un pipeline d'apprentissage Non-Supervisé pour découvrir les structures latentes de la demande :

1. **Feature Engineering :** Parsing temporel pour extraire le jour et l'heure, et isolation d'une fenêtre de forte saisonnalité (Vendredi, 18h00).
2. **Standardisation :** Traitement des coordonnées géospatiales (Latitude, Longitude) via `StandardScaler` pour normaliser l'espace vectoriel, condition indispensable aux algorithmes basés sur la distance euclidienne (Norme $L2$).
3. **Modélisation Comparative :**
   * **K-Means :** Optimisation du nombre de clusters via la *Méthode du Coude* (Inertia) et validation par le *Silhouette Score* ($K=6$). Ce modèle est finalement rejeté métierement à cause de sa topologie sphérique rigide, inadaptée à la morphologie urbaine de Manhattan.
   * **DBSCAN :** Approche par densité algorithmique (`eps=0.2`, `min_samples=30`). Ce modèle est validé car il épouse les formes réelles des rues et permet l'exclusion mathématique du bruit (courses isolées, outliers), isolant ainsi de véritables hubs de rentabilité.
4. **Data Visualization :** Rendu MLOps dynamique sur cartes interactives haute fidélité via `Plotly Mapbox`.

## 🚀 Reproductibilité de l'Environnement

Pour lancer ce projet en local sur votre machine :

```bash
# 1. Cloner le repository
git clone [https://github.com/CarelleNouko/uber-pickups-clustering.git](https://github.com/CarelleNouko/uber-pickups-clustering.git)
cd uber-pickups-clustering

# 2. Créer et activer un environnement virtuel isolé
python3 -m venv venv
source venv/bin/activate  # Sur Mac/Linux
# venv\Scripts\activate   # Sur Windows

# 3. Installer les dépendances
pip install -r requirements.txt

# 4. Lancer le notebook Jupyter
jupyter notebook