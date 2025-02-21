# Rapport Technique : Exploration de CouchDB et Analyses de Données

## Introduction

**Apache CouchDB** est une base de données NoSQL orientée documents qui exploite JSON pour le stockage, HTTP pour les interactions et **MapReduce** pour l'analyse de données. Sa robustesse et son système de réplication en font un choix privilégié pour les applications modernes distribuées.

Ce rapport détaille la mise en place de CouchDB, la gestion des données, et l'exploitation de MapReduce pour des analyses avancées.

---

## 1. Déploiement Initial

### 1.1 Installation via Docker (Approche conseillée)
```bash
docker run -d --name couchdb-analytics \
  -e COUCHDB_USER=analyste \
  -e COUCHDB_PASSWORD=complex123 \
  -p 5984:5984 \
  couchdb:latest
```
- `COUCHDB_USER` et `COUCHDB_PASSWORD` configurent l'accès administrateur.
- `-p 5984:5984` rend le service accessible sur le port standard.

### 1.2 Test de Connectivité
```bash
curl -X GET http://analyste:complex123@localhost:5984/
```
Un retour contenant `{"couchdb":"Welcome", "version":"X.X.X"}` confirme l'installation.

---

## 2. Opérations sur les Données

### 2.1 Initialisation d'une base
```bash
curl -X PUT http://analyste:complex123@localhost:5984/musiques
```

### 2.2 Insertion des Données

#### Insertion unitaire
```bash
curl -X POST http://analyste:complex123@localhost:5984/musiques \
  -H "Content-Type: application/json" \
  -d '{"titre":"Bohemian Rhapsody", "annee":1975}'
```

#### Insertion multiple
```bash
curl -X POST http://analyste:complex123@localhost:5984/musiques/_bulk_docs \
  -H "Content-Type: application/json" \
  -d @catalogue_musical.json
```

### 2.3 Consultation et Modification

#### Lecture d'une entrée
```bash
curl -X GET http://analyste:complex123@localhost:5984/musiques/doc_id
```

#### Mise à jour d'une entrée
```bash
curl -X PUT http://analyste:complex123@localhost:5984/musiques/doc_id \
  -H "Content-Type: application/json" \
  -d '{"_rev":"1-xxx", "titre":"Stairway to Heaven", "annee":1971}'
```

---

## 3. Applications de MapReduce

### 3.1 Vue 1 : Statistiques par Année

#### Fonction Map
```javascript
function (doc) {
  if (doc.annee && doc.titre) {
    emit(doc.annee, 1);
  }
}
```

#### Fonction Reduce
```javascript
function (keys, values, rereduce) {
  return sum(values);
}
```
Cette vue calcule la distribution des morceaux par année.

### 3.2 Vue 2 : Répertoire par Musicien

#### Fonction Map
```javascript
function (doc) {
  if (doc.musiciens) {
    doc.musiciens.forEach(musicien => {
      emit(musicien.nom, doc.titre);
    });
  }
}
```

#### Fonction Reduce
```javascript
function (keys, values) {
  return { total: values.length, morceaux: values };
}
```
Cette vue associe les morceaux à chaque musicien et calcule leur total.

### 3.3 Vue 3 : Catalogue par Compositeur

#### Fonction Map
```javascript
function (doc) {
  if (doc.compositeur) {
    emit(doc.compositeur, doc.titre);
  }
}
```

#### Fonction Reduce
```javascript
function (keys, values) {
  return values.length;
}
```
Cette vue regroupe les œuvres par compositeur avec leur décompte.

---

## 4. Analyse de Réseau avec MapReduce

### 4.1 Structure des Documents
Représentation des connexions entre artistes :
```json
{
  "artiste": "A1",
  "collaborations": [
    {"avec": "A2", "intensite": 0.5},
    {"avec": "A4", "intensite": 0.8}
  ]
}
```

### 4.2 Mesure d'Influence

#### Fonction Map
```javascript
function (doc) {
  var total = 0;
  doc.collaborations.forEach(function (collab) {
    total += Math.pow(collab.intensite, 2);
  });
  emit(doc.artiste, Math.sqrt(total));
}
```

#### Fonction Reduce
```javascript
function (keys, values, rereduce) {
  return values;
}
```
Cette vue évalue l'influence de chaque artiste dans le réseau.

### 4.3 Analyse des Relations

#### Fonction Map
```javascript
function (doc) {
  doc.collaborations.forEach(function (collab) {
    emit(collab.avec, collab.intensite * I[collab.avec]);
  });
}
```

#### Fonction Reduce
```javascript
function (keys, values, rereduce) {
  return values.reduce(function (a, b) { return a + b; }, 0);
}
```
Cette vue analyse la propagation de l'influence dans le réseau.

---

## 5. Conclusion

CouchDB se distingue comme une solution de base de données moderne particulièrement adaptée aux défis actuels du développement d'applications distribuées. Son approche native du JSON, couplée à son interface RESTful, simplifie considérablement l'intégration et la manipulation des données. 

Le moteur MapReduce intégré offre une puissance d'analyse remarquable, permettant des traitements complexes sur de grands volumes de données. La capacité de réplication multi-cluster de CouchDB en fait un choix pertinent pour les applications nécessitant une synchronisation fiable et une haute disponibilité des données, répondant ainsi aux exigences des architectures modernes distribuées.
