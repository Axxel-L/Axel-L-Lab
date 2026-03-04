---
title: "Egg VPS – Guide et automatisation"
---

# 🚀 Egg VPS Pterodactyl

Ce projet est né du besoin d'avoir pour NeoHeberg de proposer un environnement VPS léger et simple, avec un choix restreint de distributions (Debian, Ubuntu, Kali, Fedora, Amazon Linux), tout en gardant la puissance du travail original de **ysdragon**.

Ici, je documente les étapes de modification des scripts, la publication d'une image Docker sur **GitHub Container Registry (GHCR)** et l'automatisation du processus grâce à un simple script `npm`.

---

## 📦 Qu'est-ce qu'un container GitHub ?

GitHub Container Registry (**ghcr.io**) est un registre d'images Docker intégré à GitHub. Il permet de stocker, versionner et partager des images de conteneurs, publiquement ou privément.

**Fonctionnement :**
- On construit une image Docker localement.
- On la « tagge » avec l'URL du registre (`ghcr.io/NOM_UTILISATEUR/nom-image:tag`).
- On la pousse (push) vers GitHub.
- N'importe où ailleurs (comme sur un nœud Pterodactyl), on peut tirer (pull) cette image pour l'exécuter.

Notre egg VPS utilise cette mécanique : le serveur Pterodactyl démarre un conteneur à partir de notre image hébergée sur **ghcr.io**.

---

## 🔧 L'egg VPS

Nous avons repris l'excellent travail de [ysdragon/Pterodactyl-VPS-Egg](https://github.com/ysdragon/Pterodactyl-VPS-Egg) et nous l'avons simplifié pour **NeoHeberg**.

> 🗂️ Structure des fichiers

```
Eggs-Pterodactyl/
├── scripts/
│   ├── common.sh          (bannières colorées avec crédits)
│   ├── entrypoint.sh       (point d'entrée du conteneur)
│   ├── helper.sh           (gestion des ports et lancement de proot)
│   ├── install.sh          (menu de choix de distribution)
│   └── run.sh              (shell interactif simplifié)
├── Dockerfile              (construction de l'image)
├── package.json            (automatisation npm)
└── vps.json                (egg Pterodactyl à importer)
```

> ✨ Modifications principales

- **`install.sh`** : ne propose plus que 5 distributions (Debian, Ubuntu, Kali, Fedora, Amazon Linux). Interface plus simple.
- **`run.sh`** : épuré des fonctionnalités avancées (GUI, SSH custom, backup…). Ne garde que les commandes essentielles (`help`, `exit`, `clear`, `history`, `reinstall`, `status`).
- **`common.sh`** : bannières retravaillées avec des émojis et les crédits à `ysdragon` et `NeoHeberg`.
- **Dockerfile** : basé sur l'original, mais copie **nos** scripts.

---

## 🐳 Construction et publication manuelle de l'image

> 1. Prérequis
- Docker installé
- Être connecté à ghcr.io : `docker login ghcr.io -u NOM_UTILISATEUR` (utiliser un token avec droits `write:packages`)

> 2. Construire l'image
```bash
cd /chemin/vers/Eggs-Pterodactyl
docker build -t ghcr.io/NOM_UTILISATEUR/nom-image:latest .
```

> 3. Pousser l'image
```bash
docker push ghcr.io/NOM_UTILISATEUR/nom-image:latest
```

L'image est alors disponible sur [https://github.com/NOM_UTILISATEUR/packages?repo_name=Eggs-Pterodactyl](https://github.com/NOM_UTILISATEUR/packages?repo_name=Eggs-Pterodactyl).

---

## ⚡ Automatisation avec npm

Pour éviter de retaper les commandes à chaque modification des scripts, j'ai créé un fichier `package.json` avec une commande `build`.

> Création du package.json
```bash
npm init -y
```

> Script personnalisé
Dans `package.json`, j'ai ajouté la section `scripts` suivante :

```json
{
  "name": "eggs-pterodactyl",
  "version": "1.0.0",
  "scripts": {
    "build": "echo \"[✅] Construction du Github container ...\" && cd ./scripts/ && docker build -t ghcr.io/neoheberg/vps-neoheberg-egg:latest . && echo \"[✅] Push du Github container ...\" && docker push ghcr.io/neoheberg/vps-neoheberg-egg:latest && echo \"[✅] Construction terminée !\" && cd ../"
  }
}
```

Désormais, un simple :
```bash
npm run build
```
Exécute **automatiquement** le build et le push de l'image.  
Plus besoin de se souvenir des commandes Docker !

---

## 🖥️ Utilisation dans Pterodactyl

1. **Importer l'egg** : dans le panel Pterodactyl (admin → Nests → Import Egg), sélectionner le fichier `vps.json` modifié.
2. **Créer un serveur** : choisir l'egg "Egg VPS - NeoHeberg".
3. **Démarrer** : le serveur va pull notre image depuis `ghcr.io/neoheberg/vps-neoheberg-egg:latest` et lancer l'installation interactive.

Si l'image est publique, aucun réglage supplémentaire n'est nécessaire. Si elle reste privée, il faut ajouter les identifiants dans les paramètres du nœud (Registry Credentials).

---

## 🔗 Ressources

- [Dépôt GitHub original (ysdragon)](https://github.com/ysdragon/Pterodactyl-VPS-Egg)
- [Notre dépôt NeoHeberg](https://github.com/neoheberg/Eggs-Pterodactyl)
- [Documentation GitHub Container Registry](https://docs.github.com/fr/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Pterodactyl – Créer un egg](https://pterodactyl.io/community/customization/eggs.html)