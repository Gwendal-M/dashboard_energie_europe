# Journal de construction — Dashboard énergie Europe

## Environnement

**Machine :** Windows 11 — Git Bash (MINGW64)
**Python :** 3.14.x (pythoncore-3.14-64)
**Environnement virtuel :** .venv

## Commandes de mise en place

```bash
# Création du dossier projet
mkdir dashboard_energie_europe
cd dashboard_energie_europe

# Création des sous-dossiers
mkdir data exports powerbi notebooks

# Installation des dépendances
pip install pandas duckdb openpyxl

# Vérification de l'environnement
python -c "import pandas; import duckdb; print('Ok environnement installé')"
# → Ok environnement installé

# Création du requirements.txt
echo pandas > requirements.txt
echo duckdb >> requirements.txt
echo openpyxl >> requirements.txt
```

## Versions installées

| Bibliothèque | Version |
|---|---|
| pandas | 3.0.3 |
| duckdb | 1.5.3 |
| openpyxl | 3.1.5 |
| numpy | 2.4.6 |

## Erreurs rencontrées et solutions

### ModuleNotFoundError: No module named 'pandas'
**Cause :** VS Code utilisait le mauvais interpréteur Python
le notebook tournait dans le .venv où pandas n'était pas installé.
**Solution :** Activer le .venv dans le terminal VS Code puis 
réinstaller les dépendances dedans.
```bash
.venv\Scripts\activate
pip install pandas duckdb openpyxl
```

### KeyError: 'country' lors du groupby pandas
**Cause :** Comportement du groupby + apply changé dans pandas 3.x — 
l'index était perturbé après le filtrage.
**Solution :** Boucle explicite par pays au lieu du groupby/apply,
avec reset_index() et interpolation sur colonnes numériques uniquement.

### TypeError: Cannot interpolate with str dtype
**Cause :** L'interpolation était appliquée à toutes les colonnes 
y compris la colonne texte 'country'.
**Solution :** Définir explicitement la liste des colonnes numériques
et appliquer l'interpolation uniquement sur celles-ci.

### OSError: Cannot save file into non-existent directory 'data'
**Cause :** Le notebook était ouvert depuis notebooks/ donc Python 
cherchait data/ dans notebooks/data/ au lieu de la racine.
**Solution :** Ajouter en première cellule de chaque notebook :
```python
import os
os.chdir(r'C:\Users\progw\dashboard_energie_europe')
```

### ConnectionException: Connection already closed
**Cause :** conn.close() avait été appelé avant la cellule 
d'export des dimensions.
**Solution :** Déplacer l'export des dimensions pandas dans une 
cellule séparée après conn.close() — pandas n'a pas besoin 
de DuckDB pour exporter en CSV.

### Relations many-to-many dans Power BI
**Cause :** Toutes les tables avaient plusieurs lignes par pays 
et par année — Power BI ne pouvait pas créer de relation one-to-many.
**Solution :** Créer deux tables de dimension dédiées (dim_country 
et dim_year) avec une ligne unique par entité, exportées en CSV 
depuis pandas et importées dans Power BI.

## Choix techniques documentés

**DuckDB plutôt que SQLite**
DuckDB est orienté OLAP (analytique) là où SQLite est orienté OLTP 
(transactions). Il lit directement les dataframes pandas, supporte 
les fonctions fenêtres complètes (LAG, RANK) et correspond à une 
stack analytique moderne.

**Interpolation linéaire par pays**
Les valeurs manquantes sont comblées par interpolation linéaire 
pays par pays plutôt que par une moyenne globale, pour respecter 
les dynamiques nationales propres à chaque pays.

**Schéma en étoile**
Standard de la modélisation décisionnelle BI — une table de faits 
centrale avec les mesures numériques, entourée de dimensions 
descriptives. Optimise les calculs DAX et rend le modèle maintenable.