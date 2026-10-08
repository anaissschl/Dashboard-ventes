# 📊 Sales Analytics Dashboard — Power BI

## Présentation

Ce projet consiste à concevoir un **tableau de bord interactif sous Microsoft Power BI** pour analyser les performances commerciales, suivre l'évolution des ventes et étudier les commandes annulées.

Il couvre l'ensemble du processus décisionnel : **préparation des données, normalisation, modélisation, création de mesures DAX et visualisation interactive**.

> **Source des données :** `sales_2.csv`  
> **Outil principal :** Microsoft Power BI Desktop  
> **Langages et outils :** Power Query (M), DAX, Power BI Service

## Objectifs

- Suivre le chiffre d'affaires, le nombre de commandes et les quantités vendues.
- Analyser les ventes par période, région, catégorie et produit.
- Identifier et mesurer les commandes annulées.
- Proposer une navigation intuitive et des filtres interactifs.
- Mettre en place des fonctionnalités avancées : info-bulles, signets, sécurité RLS et affichage mobile.

## 1. Préparation et qualité des données

Le fichier `sales_2.csv` est importé dans **Power Query**. Les données sont contrôlées à l'aide des outils **Qualité de la colonne**, **Distribution des colonnes** et **Profil de la colonne** afin de détecter les valeurs manquantes, les erreurs et les incohérences.

Les types de données sont vérifiés, puis les champs sont renommés pour améliorer leur lisibilité :

| Colonne source | Colonne renommée |
|---|---|
| `OrderID` | Id commande |
| `CustomerID` | Id client |
| `CompanyName` | Nom client |
| `ProductID` | Id produit |
| `ProductName` | Nom produit |
| `Category` | Catégorie produit |
| `RegionID` | Id région |
| `RegionName` | Nom région |
| `OrderDate` | Date commande |
| `Quantity` | Quantité |
| `UnitPrice` | Prix unitaire |
| `TotalPrice` | Prix total |
| `OrderStatus` | Statut commande |

## 2. Normalisation et modélisation

La table source sert de base à un **modèle en étoile**, composé de quatre tables finales :

| Table | Rôle | Champs principaux |
|---|---|---|
| **Clients** | Dimension client | Id client, Nom client |
| **Produits** | Dimension produit | Id produit, Nom produit, Catégorie produit |
| **Régions** | Dimension géographique | Id région, Nom région |
| **Ventes** | Table de faits | Id commande, Id client, Id produit, Id région, Date commande, Quantité, Prix unitaire, Prix total, Statut commande |

Les tables de dimensions sont créées par **référence** à la requête source, avec suppression des doublons et contrôle de l'unicité des identifiants. La requête source est conservée sans être chargée dans le modèle.

### Relations

- `Clients[Id client]` **1 → \*** `Ventes[Id client]`
- `Produits[Id produit]` **1 → \*** `Ventes[Id produit]`
- `Régions[Id région]` **1 → \*** `Ventes[Id région]`

Les clés étrangères sont conservées dans **Ventes** pour assurer les relations avec les dimensions. Les libellés descriptifs redondants peuvent être retirés de cette table.

## 3. Mesures DAX

Les indicateurs sont centralisés dans une table dédiée **Mesures**.

```dax
Total ventes = SUM(Ventes[Prix total])

Nombre de commandes = DISTINCTCOUNT(Ventes[Id commande])

Quantité vendue = SUM(Ventes[Quantité])

Commande moyenne = DIVIDE([Total ventes], [Nombre de commandes])
```

### Analyse des annulations

```dax
Total commandes annulées =
CALCULATE(
    DISTINCTCOUNT(Ventes[Id commande]),
    Ventes[Statut commande] = "Cancelled"
)

Montant commandes annulées =
CALCULATE(
    SUM(Ventes[Prix total]),
    Ventes[Statut commande] = "Cancelled"
)

Pourcentage commandes annulées =
DIVIDE([Total commandes annulées], [Nombre de commandes])
```

> Le calcul des commandes annulées utilise `DISTINCTCOUNT` afin de compter les commandes uniques, même lorsqu'une commande apparaît sur plusieurs lignes. Le pourcentage doit être formaté en `%` dans Power BI.

## 4. Pages du dashboard

### Page 1 — Suivi des ventes

Cette page présente une vue d'ensemble des performances commerciales :

- **4 KPI :** chiffre d'affaires total, nombre de commandes, quantité vendue et panier moyen.
- **Courbe chronologique :** évolution des ventes et des volumes.
- **Barres horizontales :** chiffre d'affaires par région.
- **Diagramme en anneau :** répartition du chiffre d'affaires par catégorie.
- **Tableau détaillé :** commandes, clients, produits, quantités et montants.

Des filtres permettent d'affiner les analyses selon **la période, le statut des commandes et la région**.

### Page 2 — Commandes annulées

Cette page approfondit l'analyse des annulations :

- **3 KPI :** nombre de commandes annulées, montant associé et taux d'annulation.
- **Graphique temporel :** évolution du nombre et du pourcentage d'annulations.
- **Histogramme :** comparaison des annulations entre régions.
- **Treemap :** montant des annulations par catégorie et produit.
- **Graphique en ruban :** évolution trimestrielle des annulations par produit.

## 5. Design et expérience utilisateur

Le rapport adopte une interface sombre et une présentation cohérente :

| Élément | Configuration |
|---|---|
| Fond principal | `#1E2D38` |
| Fond des cartes | `#232448` |
| Hauteur de page | `2000 px` |
| Thème | JSON personnalisé, inspiré de [Loomy Lime](https://community.fabric.microsoft.com/t5/Themes-Gallery/Loomy-Lime/td-p/1130525) |
| Navigation | Menu latéral avec icônes et liens vers les pages |

Deux **info-bulles personnalisées** enrichissent l'exploration : une courbe d'évolution des ventes pour l'analyse régionale et un graphique en ruban pour l'analyse des produits par catégorie.

## 6. Fonctionnalités avancées

### Signets interactifs

Des signets facilitent la sélection de familles de produits :

- **Mobiles :** Smartphone, Headphone, SmartWatch.
- **Bureautique :** Tablet, Monitor, Laptop, Keyboard, Mouse.
- **Réinitialisation :** retour à l'état initial des filtres.

### Indicateur de part filtrée

Un visuel personnalisé **PayPal KPI Donut Chart** peut illustrer la proportion du chiffre d'affaires actuellement sélectionné par rapport au total de référence.

### Sécurité des données — RLS

Deux rôles de sécurité peuvent être configurés :

| Rôle | Restriction |
|---|---|
| Rôle 1 | Commandes des clients **AI Systems** et **TechCorp** uniquement |
| Rôle 2 | Commandes de la région **South** uniquement |

### Version mobile

Une mise en page mobile dédiée conserve les principaux KPI, graphiques et filtres, tout en simplifiant la navigation et en écartant le tableau détaillé si nécessaire.

## 7. Utilisation

1. Ouvrir **Power BI Desktop**.
2. Importer le fichier `sales_2.csv` via **Accueil → Obtenir des données → Texte/CSV**.
3. Ouvrir **Transformer les données** pour préparer les colonnes et construire les tables de référence.
4. Charger les tables finales et vérifier les relations du modèle.
5. Créer les mesures DAX et construire les deux pages du rapport.
6. Ajouter les filtres, les info-bulles et les fonctionnalités avancées souhaitées.
7. Enregistrer le rapport au format `.pbix`, puis le publier sur **Power BI Service** si nécessaire.

> Si un fichier `.pbix` final est disponible dans le dépôt, il suffit de l'ouvrir dans Power BI Desktop. La source CSV doit rester accessible pour permettre l'actualisation des données.

## Compétences mobilisées

**Data preparation · Power Query · Data quality · Data modeling · Star schema · DAX · Data visualization · Business Intelligence · Row-Level Security · Dashboard design**

---

*Projet réalisé dans le cadre d'un atelier pratique de Business Intelligence avec Microsoft Power BI.*
