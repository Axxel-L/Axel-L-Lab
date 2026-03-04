---
title: "PostgreSQL - Docker"
---

# 🐳 Docker : PostgreSQL

Ce dossier regroupe mes expérimentations avec **Docker** et **Docker Compose**, en particulier pour faire tourner des bases de données PostgreSQL de façon reproductible.

L’objectif est de maîtriser l’utilisation de conteneurs pour isoler, déployer et partager facilement des environnements de développement ou de test.

---

## 📦 Qu'est-ce que Docker ?

Docker est une plateforme de conteneurisation qui permet d’empaqueter une application et ses dépendances dans une image, puis d’exécuter cette image sous forme de conteneur isolé sur un hôte.

**Concepts clés :**

- **Image** : modèle en lecture seule (ex. `postgres:18`).
- **Conteneur** : instance exécutable d’une image.
- **Volume** : espace persistant pour les données (indépendant du cycle de vie du conteneur).
- **Réseau** : permet aux conteneurs de communiquer entre eux.
- **Docker Compose** : outil pour définir et exécuter des applications multi-conteneurs avec un fichier YAML.

---

## 🧪 Installation rapide de Docker

Sur Ubuntu / Debian :

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2
sudo systemctl enable --now docker
sudo usermod -aG docker $USER   # déconnecter/reconnecter pour appliquer
```

Vérifier :

```bash
docker --version
docker compose version
```

---

## 🐘 Lancer PostgreSQL

> Démarrer un conteneur simple

```bash
docker run --name ma-pg -e POSTGRES_PASSWORD=monpass -d postgres:18
```

- `--name` : nom du conteneur.
- `-e` : variable d’environnement pour définir le mot de passe de l’utilisateur `postgres`.
- `-d` : mode détaché (arrière-plan).
- `postgres:18` : l’image utilisée (ici version 18, tag à adapter si besoin).

> Se connecter à PostgreSQL dans le conteneur

```bash
docker exec -it ma-pg psql -U postgres
```

> Arrêter et supprimer le conteneur

```bash
docker stop ma-pg
docker rm ma-pg
```

> Persister les données avec un volume

```bash
docker run --name ma-pg \
  -e POSTGRES_PASSWORD=monpass \
  -v pgdata:/var/lib/postgresql/data \
  -d postgres:18
```

Le volume nommé `pgdata` sera créé automatiquement et persistera même après la suppression du conteneur.

---

## 🐙 Utiliser Docker Compose pour PostgreSQL

Docker Compose permet de définir toute la configuration dans un fichier `docker-compose.yml` et de la démarrer d’une commande.

> Exemple minimal avec PostgreSQL 18

Créez un répertoire pour votre projet :

```bash
mkdir mon-projet-pg
cd mon-projet-pg
```

Créez le fichier `docker-compose.yml` :

```yaml
services:
  postgres:
    image: postgres:18
    container_name: postgres-lab
    environment:
      POSTGRES_PASSWORD: monmotdepasse
      POSTGRES_USER: monuser          # optionnel, par défaut "postgres"
      POSTGRES_DB: mabase              # optionnel, crée une base par défaut
    ports:
      - "5432:5432"                    # expose le port 5432 sur l'hôte
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

> Lancer l’environnement

```bash
docker compose up -d
```

- `-d` : détaché.

Vérifier que le conteneur tourne :

```bash
docker compose ps
```

> Arrêter et nettoyer

```bash
docker compose down      # arrête les conteneurs
docker compose down -v   # arrête et supprime aussi le volume (attention : perte des données)
```

> Accéder au conteneur

```bash
docker compose exec postgres psql -U monuser -d mabase
```

> Visualiser les logs

```bash
docker compose logs -f postgres
```

## ⚠️ Erreurs fréquentes et solutions

> `port is already allocated`

**Problème** : le port 5432 est déjà utilisé sur l’hôte (par une autre instance PostgreSQL ou un autre conteneur).  
**Solution** : changez le port côté hôte, par exemple `"5433:5432"`.

> Connexion refusée depuis l’hôte

**Problème** : impossible de se connecter avec un client local.  
**Solution** : vérifiez que PostgreSQL écoute bien sur toutes les interfaces (par défaut, c’est le cas dans l’image officielle). Si vous avez monté une config personnalisée, assurez-vous que `listen_addresses = '*'`.

> Authentification échouée

**Problème** : `FATAL: password authentication failed for user "..."`.  
**Solution** : vérifiez les variables d’environnement `POSTGRES_PASSWORD`, `POSTGRES_USER`. Si vous avez monté un volume existant, le mot de passe initial est figé, il faut utiliser celui défini à la création du volume.

> Les données ne persistent pas

**Problème** : après `docker compose down`, les données disparaissent.  
**Solution** : vous avez probablement omis le volume ou utilisé un volume anonyme. Déclarez toujours un volume nommé dans la section `volumes:`.

---

## 🧠 Concepts à retenir

- **Image taguée** : `postgres:18` fait référence à la version majeure 18 (à vérifier selon disponibilité).
- **Variables d’environnement** : permettent de configurer le conteneur sans modifier l’image.
- **Volumes** : la seule façon de conserver des données au-delà de la vie du conteneur.
- **Docker Compose** : idéal pour définir des environnements complexes et reproductibles.

---

## 🔗 Ressources

- [Documentation Docker](https://docs.docker.com/)
- [Image officielle PostgreSQL sur Docker Hub](https://hub.docker.com/_/postgres)
- [Docker Compose reference](https://docs.docker.com/compose/compose-file/)