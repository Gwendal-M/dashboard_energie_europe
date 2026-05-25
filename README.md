# Dashboard de pilotage — Mix énergétique européen

Projet de data analyse construit de A à Z pour explorer la transition 
énergétique de 22 pays européens entre 2000 et 2025.

## Question métier

Quels pays européens sont les plus avancés dans leur transition 
énergétique, et quelle est la tendance sur 25 ans ?

## Architecture du projet

```
dashboard-energie-europe/
├── data/
│   ├── energy_europe_clean.csv     ← données nettoyées
│   └── energy_europe.duckdb        ← base analytique locale
├── notebooks/
│   ├── 01_nettoyage.ipynb          ← collecte, filtrage, nettoyage
│   └── 02_modelisation.ipynb       ← modélisation DuckDB, vues SQL, exports
├── exports/
│   ├── export_faits.csv            ← table de faits
│   ├── export_classement.csv       ← vue classement par pays
│   ├── export_evolution.csv        ← vue variation annuelle
│   ├── export_region.csv           ← vue résumé par région
│   ├── dim_country.csv             ← dimension pays
│   └── dim_year.csv                ← dimension année
├── powerbi/
│   └── dashboard_energie.pbix      ← dashboard Power BI
├── JOURNAL.md                      ← journal de construction et erreurs
├── README.md
└── requirements.txt
```

## Stack technique

| Outil | Usage |
|---|---|
| Python 3.14 / pandas 3.0 | Collecte, nettoyage, transformation |
| DuckDB 1.5 | Base analytique locale, modélisation en étoile |
| SQL | Vues analytiques, fonctions fenêtres (LAG, RANK) |
| Power BI Desktop | Dashboard 3 pages, mesures DAX |

## Source de données

**Our World in Data — Energy dataset**
https://github.com/owid/energy-data

Données issues de BP Statistical Review et Ember Climate.
Couvre 300+ pays, 100+ indicateurs énergétiques, de 1900 à 2025.
Pour ce projet : 22 pays européens, 10 indicateurs, 2000-2025.

## Modèle de données — schéma en étoile
fact_energy (table de faits)
```
├── dim_country   (pays, région)
└── dim_year      (année, décennie, période)
```
La table de faits contient une ligne par pays et par année avec
les mesures numériques. Les dimensions portent les attributs
descriptifs. Ce schéma est le standard de la modélisation
décisionnelle BI — il optimise les calculs DAX et rend le
modèle lisible et maintenable.

## Description des KPI

### Mix renouvelable moyen EU (%)
Moyenne du pourcentage de renouvelables dans le mix électrique
sur les pays et années sélectionnés. Calculé comme :

pct_renouvelable = renewables_electricity / electricity_generation × 100

Les renouvelables incluent solaire, éolien et hydraulique.
Ce KPI mesure l'avancement global de la transition sur la
sélection active dans le slicer.

### Mix nucléaire moyen EU (%)
Même logique avec la production nucléaire. Le nucléaire est
traité séparément des renouvelables car son statut dans la
transition énergétique est débattu — il est décarboné mais
pas renouvelable. Ce choix permet de distinguer les pays qui
décarbonent via le nucléaire (France) de ceux qui le font
via les renouvelables (Norvège, Danemark).

### Intensité carbone moyenne (gCO₂/kWh)
Grammes de CO₂ émis par kWh d'électricité produit.
C'est l'indicateur le plus direct de la décarbonation du mix
indépendant de la source (renouvelable ou nucléaire), il mesure
l'impact climatique réel. Une intensité carbone qui baisse
signifie que le mix devient plus propre, quelle que soit la
stratégie choisie par le pays.

## Statut de transition — choix analytiques

Le statut de transition est une classification en 3 niveaux
définie par des seuils analytiques assumés :

| Statut | Condition | Couleur |
|---|---|---|
| Objectif atteint | > 50% renouvelables | Vert |
| En progression | Entre 30% et 50% | Orange |
| Recul | < 30% renouvelables | Rouge |

**Pourquoi ces seuils ?**

Le seuil de 50% est aligné sur les objectifs de la directive
européenne sur les énergies renouvelables (RED III) qui fixe
un objectif de 42.5% à l'échelle de l'UE d'ici 2030.
J'ai choisi 50% comme seuil "objectif atteint" pour identifier
les pays qui ont déjà dépassé cet objectif collectif.

Le seuil de 30% correspond à la moyenne européenne au début
des années 2010 — un pays en dessous de ce niveau accuse un
retard structurel par rapport à la trajectoire historique
du continent.

En entreprise, ces seuils seraient définis avec les experts
métier et les parties prenantes. L'important est que la logique
soit documentée et explicable.

## Pages du dashboard

**Page 1 — Vue d'ensemble — Transition énergétique européenne**
Vue globale sur les 22 pays avec classement renouvelables par pays,
3 KPI, slicer année 2000-2025.

**Page 2 — Analyse par pays**
Sélection d'un pays via slicer, évolution du mix par source
en aires empilées 2000-2025, courbe d'intensité carbone.

**Page 3 — Alertes**
Tableau de tous les pays avec statut de transition en
mise en forme conditionnelle vert/orange/rouge,

## Lancer le projet
# Si Power BI affiche une erreur de source de données :
# Transformer les données → Source → remplacer le chemin
# par le chemin local vers le dossier exports/
```bash
# Cloner le repo
git clone https://github.com/Gwendal-M/dashboard-energie-europe

# Installer les dépendances
pip install -r requirements.txt

# Lancer les notebooks dans l'ordre
# 1. notebooks/01_nettoyage.ipynb
# 2. notebooks/02_modelisation.ipynb

# Ouvrir le dashboard
# powerbi/dashboard_energie.pbix (Power BI Desktop requis)
```
