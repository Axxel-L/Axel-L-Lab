---
title: "PostgreSQL – Guide"
---

# 🐘 PostgreSQL : cours et expérimentations

Bienvenue dans mon dossier dédié à **PostgreSQL**, le système de gestion de bases de données relationnelles open source le plus avancé au monde.

Ici, je compile mes découvertes, commandes utiles, erreurs courantes et corrections. L'objectif est de créer à la fois un pense-bête personnel et une ressource pédagogique.

---

## 📚 Qu'est-ce que PostgreSQL ?

PostgreSQL (souvent abrégé **Postgres**) est un SGBD relationnel-objet, connu pour sa robustesse, sa conformité aux standards SQL, son extensibilité et sa richesse fonctionnelle.  
Il supporte des types de données avancés (JSON, XML, tableaux…), l'indexation avancée, la réplication, et bien plus.

---

## 🚀 Premiers pas

> Installation (Ubuntu / Debian)

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Vérifier le service :

```bash
sudo systemctl status postgresql
```

> Accéder à la console PostgreSQL

Par défaut, un utilisateur système `postgres` est créé. Pour lancer `psql` :

```bash
sudo -u postgres psql
```

Vous êtes maintenant dans le terminal interactif de PostgreSQL.

---

## 🧪 Commandes de base dans psql

> Gestion des bases

```sql
-- Lister les bases existantes
\l

-- Créer une base
CREATE DATABASE ma_base;

-- Se connecter à une base
\c ma_base

-- Supprimer une base (attention !)
DROP DATABASE ma_base;
```

> Gestion des utilisateurs

```sql
-- Lister les rôles/utilisateurs
\du

-- Créer un utilisateur avec mot de passe
CREATE USER mon_user WITH PASSWORD 'mon_mdp';

-- Donner tous les privilèges sur une base à un utilisateur
GRANT ALL PRIVILEGES ON DATABASE ma_base TO mon_user;
```

> Manipulation des tables

```sql
-- Créer une table
CREATE TABLE etudiants (
    id SERIAL PRIMARY KEY,
    nom VARCHAR(100),
    age INTEGER
);

-- Voir les tables de la base
\dt

-- Décrire une table
\d etudiants

-- Insérer des données
INSERT INTO etudiants (nom, age) VALUES ('Alice', 22);

-- Interroger
SELECT * FROM etudiants;

-- Mettre à jour
UPDATE etudiants SET age = 23 WHERE nom = 'Alice';

-- Supprimer
DELETE FROM etudiants WHERE nom = 'Alice';
```

> Quitter psql

```sql
\q
```

---

## 🔧 Commandes utiles dans le terminal (hors psql)

Créer une base directement depuis le shell :

```bash
createdb -U postgres ma_base
```

Supprimer une base :

```bash
dropdb -U postgres ma_base
```

Exécuter un fichier SQL :

```bash
psql -U postgres -d ma_base -f mon_fichier.sql
```

Sauvegarder une base (dump) :

```bash
pg_dump -U postgres ma_base > sauvegarde.sql
```

Restaurer une base depuis un dump :

```bash
psql -U postgres -d ma_base < sauvegarde.sql
```

---

## ⚠️ Erreurs fréquentes et solutions

> `Peer authentication failed`

**Problème** : vous essayez de vous connecter avec `psql` sans mot de passe et PostgreSQL refuse.  
**Solution** : modifiez le fichier `pg_hba.conf` pour autoriser l'authentification par mot de passe, ou utilisez `sudo -u postgres psql`.

> `database "ma_base" does not exist`

**Problème** : la base n'existe pas.  
**Solution** : créez-la d'abord avec `CREATE DATABASE` ou `createdb`.

> Port déjà utilisé

**Problème** : PostgreSQL n'arrive pas à démarrer car le port 5432 est occupé.  
**Solution** : changez de port dans `postgresql.conf` ou stoppez l'autre processus.

---

## 🧠 Concepts importants à retenir

- **Schéma** : espace de nom contenant des tables, vues, etc. Par défaut `public`.
- **Rôle** : équivalent à un utilisateur ou un groupe.
- **Tablespace** : emplacement physique des données.
- **MVCC** (Multi-Version Concurrency Control) : gestion des accès concurrents sans verrous lecture.
- **VACUUM** : nettoyage des versions obsolètes des lignes.

---

## 🔗 Ressources

- [Documentation officielle PostgreSQL](https://www.postgresql.org/docs/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [SQL en ligne (bac à sable)](https://www.db-fiddle.com/)