# Rapport sur CouchDB
### 1. Introduction
CouchDB est une base de données NoSQL open-source développée par Apache Software Foundation. Elle est conçue pour stocker, répliquer et synchroniser des données de manière fiable. CouchDB repose sur une architecture orientée documents (Document-Oriented Database) et utilise JSON pour stocker les données.
---
### 2. Base de Données Documentaire
Contrairement aux bases de données relationnelles traditionnelles (SQL), CouchDB adopte une structure flexible, basée sur des documents JSON. Chaque document est une structure autonome avec des paires **clef-valeur** et un identifiant unique (*id*).
- **Structure JSON** : Les données sont stockées dans un format JSON lisible par les humains.
- **Schema flexible** : Contrairement à une base relationnelle, chaque document peut avoir une structure différente.
- **Documents autonomes** : Les documents ne dépendent pas les uns des autres, ce qui facilite les opérations sur des données distribuées.
#### Exemple d'un document JSON dans CouchDB :
```json
{
  "_id": "001",
  "name": "Alice",
  "age": 25,
  "city": "Paris"
}
```
Chaque document a :
- `_id` : Identifiant unique obligatoire.
- Des paires clef-valeur pour stocker les informations.
---
### 3. Fonctionnement
CouchDB repose sur les principes suivants :
- **RESTful API** : CouchDB expose ses opérations via des requêtes HTTP (GET, POST, PUT, DELETE).
- **Réplication** : Supporte la réplication maître-maître permettant de synchroniser plusieurs instances de bases de données.
- **MapReduce** : CouchDB utilise MapReduce pour effectuer des requêtes complexes et créer des vues indexées.
- **ACID** : CouchDB garantit des propriétés ACID à l'échelle du document.
- **Conflits et résolutions** : CouchDB gère automatiquement les conflits de synchronisation grâce à sa réplication.
#### Vue d'ensemble de l'architecture :
CouchDB fonctionne comme un serveur HTTP où chaque base de données est un ensemble de documents stockés sous forme JSON. Les opérations se font à travers des requêtes HTTP REST, ce qui simplifie l'interaction avec CouchDB sans client spécifique.
---
### 4. MapReduce dans CouchDB
CouchDB utilise MapReduce pour créer des vues. **Map** transforme les données en clé-valeur, tandis que **Reduce** agrège les résultats.
---
#### 4.1 MapReduce en Général  
MapReduce est un paradigme de programmation permettant de traiter de grandes quantités de données de manière distribuée. Il divise le traitement en deux phases principales : **Map** et **Reduce**.
1. **Fonction Map** :  
   - La fonction Map prend en entrée un ensemble de données.  
   - Elle traite chaque élément indépendamment pour produire des paires **clé/valeur**.  
   - Chaque nœud traite une partie des données en parallèle.  
2. **Fonction Reduce** :  
   - La fonction Reduce combine les résultats intermédiaires générés par la fonction Map.  
   - Elle regroupe les données par clé et applique une opération d'agrégation (comme un total, une moyenne, etc.).  
---
#### 4.2 Exemple Général de MapReduce  
Considérons un ensemble de documents représentant des ventes de produits :  
```json
[
  {"produit": "A", "quantite": 10},
  {"produit": "B", "quantite": 5},
  {"produit": "A", "quantite": 7}
]
```
##### 4.2.1 Fonction Map  
La fonction Map extrait les ventes par produit :  
```javascript
function(doc) {
  emit(doc.produit, doc.quantite);
}
```
**Résultat de Map** :  
```
A: 10  
B: 5  
A: 7  
```
##### 4.2.2 Fonction Reduce  
La fonction Reduce additionne les quantités par produit :  
```javascript
function(keys, values) {
  return sum(values);
}
```
**Résultat Final** :  
```
A: 17  
B: 5  
```
