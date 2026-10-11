# Rapport de projet : analyse des ventes et du budget Adventure Works

**Auteur :** DONGMO FEUDJIO Divin Virlence
**Date :** 10 octobre 2026
**Outils :** Power BI Desktop, Power Query, DAX

---

## 1. Contexte et objectifs

Ce projet consiste à analyser les ventes d'une entreprise de vente de vélos et d'accessoires (jeu de données Adventure Works) avec Power BI. Il répond à trois questions :

1. Comment évoluent le chiffre d'affaires et la marge, et quelles catégories de produits les portent ?
2. Les ventes réelles atteignent-elles les objectifs fixés par le budget ?
3. Quelles actions recommander à partir de ces constats ?

## 2. Données

Le jeu de données est composé de neuf tables :

| Table | Rôle |
|---|---|
| Sales | Table de faits : commandes, quantités, montants, remises, coût standard |
| Products | Produits |
| dimProductSubCategory, dimProductCategory | Hiérarchie des produits (sous-catégories, catégories) |
| Customers | Clients |
| Territory | Territoires (pays, région, groupe) |
| Calendar | Table de dates |
| Budget | Budget mensuel par catégorie de produits |
| BudgetPeriod | Correspondance entre les périodes du budget et le calendrier |

Les ventes couvrent les années 2016 à 2019. Le budget est exprimé par catégorie et par mois, sans découpage par pays ni par client.

## 3. Méthode

### 3.1 Import et nettoyage

Les tables ont été importées depuis un classeur Excel et nettoyées dans Power Query : vérification des types de colonnes (dates, nombres entiers et décimaux) et des colonnes de clés.

### 3.2 Modèle de données

Le modèle est un **schéma en étoile** : la table Sales est au centre, reliée aux dimensions Products, Customers, Territory et Calendar. Les produits sont reliés à leurs sous-catégories puis à leurs catégories.

Pour le budget, deux relations supplémentaires ont été ajoutées :

- Budget → BudgetPeriod → Calendar, avec un filtre dans les deux sens entre BudgetPeriod et Calendar, pour qu'un segment d'année ou de mois filtre aussi les budgets ;
- Budget → dimProductCategory, pour comparer ventes et budget avec le même axe de catégories.

La table Calendar a été marquée comme table de dates.

### 3.3 Mesures DAX

```
Chiffre d'affaires = SUM(Sales[ExtendedAmount]) - SUM(Sales[DiscountAmount])

Quantité vendue = SUM(Sales[OrderQuantity])

Coût total = SUMX(Sales, Sales[OrderQuantity] * Sales[ProductStandardCost])

Marge = [Chiffre d'affaires] - [Coût total]

Taux de marge = DIVIDE([Marge], [Chiffre d'affaires])

CA année précédente =
CALCULATE([Chiffre d'affaires], SAMEPERIODLASTYEAR(Calendar[Date]))

Évolution annuelle =
DIVIDE([Chiffre d'affaires] - [CA année précédente], [CA année précédente])

Budget total = SUM(Budget[Budget])

CA comparable =
CALCULATE(
    [Chiffre d'affaires],
    FILTER(VALUES(Calendar[Period]), NOT ISBLANK([Budget total]))
)

Écart vs budget = [CA comparable] - [Budget total]

Taux d'atteinte = DIVIDE([CA comparable], [Budget total])
```

La mesure **CA comparable** ne conserve que les mois pour lesquels un budget existe, afin que la comparaison porte sur les mêmes périodes.

### 3.4 Visuels

- **Page Ventes** : cartes de chiffres clés, courbe d'évolution annuelle avec l'année précédente, chiffre d'affaires et taux de marge par catégorie, segments par année et par pays.
- **Page Budget** : cartes (budget, réel, écart, taux d'atteinte), histogramme réel contre budget par catégorie, courbe mensuelle, tableau récapitulatif, segments par année et par catégorie.

## 4. Résultats

### 4.1 Ventes

| Indicateur | Valeur |
|---|---|
| Chiffre d'affaires total | 29,36 M |
| Marge | environ 12,08 M |
| Taux de marge | 41,15 % |

Chiffre d'affaires par catégorie :

| Catégorie | Chiffre d'affaires | Part du total |
|---|---|---|
| Bikes | 28,32 M | environ 96,5 % |
| Accessories | 700,76 K | environ 2,4 % |
| Clothing | 339,77 K | environ 1,2 % |

Observations :

- L'activité dépend presque entièrement de la catégorie **Bikes**.
- Les taux de marge sont d'environ 60 % pour **Accessories** et d'environ 41 % pour Bikes et Clothing.
- Le chiffre d'affaires croît de 2016 à 2018 puis semble plafonner autour de 10 M. Ce plafond est probablement un effet de données incomplètes : sur la page Budget, les ventes de 2019 ne couvrent que les mois de janvier à juin pour la catégorie Bikes, et celles de 2016 commencent en juillet. À six mois, le niveau de 2019 est déjà proche d'une année complète 2018, ce qui suggère une croissance sous-jacente plus forte que ce que montre la courbe annuelle. 
- Pour la France seule, le chiffre d'affaires est de 2,64 M et la marge de 1,09 M, soit environ 9 % du chiffre d'affaires total.

### 4.2 Budget contre réel

| Catégorie | Budget | Réel | Écart | Taux d'atteinte |
|---|---|---|---|---|
| Accessories | 739 000 | 700 759,96 | −38 240,04 | 94,83 % |
| Bikes | 28 750 000 | 28 318 144,65 | −431 855,35 | 98,50 % |
| Clothing | 352 000 | 339 772,61 | −12 227,39 | 96,53 % |
| **Total** | **29 841 000** | **29 358 677,22** | **−482 322,78** | **98,38 %** |

Observations :

- Les trois catégories sont **sous leur objectif**, de 1,5 % à 5,2 %.
- La catégorie **Bikes** représente environ 90 % de l'écart total, parce qu'elle pèse presque tout le chiffre d'affaires.
- **Accessories** est la catégorie la plus éloignée de son objectif en pourcentage (94,83 %).
- Le chiffre d'affaires comparable est égal au chiffre d'affaires total : le budget couvre tous les mois où des ventes ont eu lieu.

## 5. Recommandations

### 5.1 Recommandations métier

1. **Réduire la dépendance à la catégorie Bikes.** Elle représente plus de 96 % du chiffre d'affaires : une baisse de la demande sur cette seule catégorie met toute l'activité en risque. Fixer un objectif de part pour les autres catégories.
2. **Développer les accessoires.** Leur taux de marge (environ 60 %) est supérieur à celui des vélos. Les vendre en complément des vélos (vente croisée, offres groupées) peut améliorer la marge globale à faible coût. Il faut d'abord comprendre pourquoi cette catégorie est la plus éloignée de son budget (94,83 %).
3. **Analyser l'écart de la catégorie Bikes dans le temps.** Un écart de 1,5 % sur 28,7 M représente plus de 430 K. Regarder s'il est régulier ou concentré sur certains mois, certaines années ou certains pays avant d'agir.
4. **Fixer des budgets plus fins.** Le budget actuel n'est défini que par catégorie et par mois. Un budget par pays et par sous-catégorie permettrait de localiser les écarts et de piloter les équipes.
5. **Suivre la croissance annuelle.** Ajouter l'évolution par rapport à l'année précédente dans les indicateurs de pilotage, une fois les données de 2019 vérifiées.

### 5.2 Recommandations techniques

1. **Documenter les définitions.** Le chiffre d'affaires est calculé comme le montant avant remise moins les remises ; le taux de marge utilise le coût standard des produits. Confirmer avec les responsables des données que le budget est défini sur la même base, sinon une partie de l'écart provient de la définition et non des ventes.
2. **Ranger les mesures dans une table dédiée** (par exemple « Mesures ») et harmoniser leurs noms (accents, majuscules) pour faciliter la maintenance.
3. **Alléger le modèle** : masquer les colonnes de clés et supprimer les colonnes inutiles (par exemple les noms de catégories en français et en espagnol).
4. **Limiter le filtre dans les deux sens.** Il convient ici parce que le modèle est petit. Si le modèle grandit, le remplacer par une mesure DAX qui active le filtre uniquement au calcul du budget.
5. **Automatiser l'actualisation** (publication dans le service Power BI, actualisation planifiée) si le rapport est destiné à être utilisé régulièrement.


<!-- ## 7. Prochaines étapes

1. Ajouter un tableau ou un graphique du chiffre d'affaires mensuel pour confirmer la couverture des années 2016 et 2019.
2. Construire une page d'analyse par client et par produit (meilleurs clients, produits les plus rentables).
3. Analyser le budget par sous-catégorie pour localiser les écarts de la catégorie Bikes.
4. Publier le rapport et planifier l'actualisation des données. -->
