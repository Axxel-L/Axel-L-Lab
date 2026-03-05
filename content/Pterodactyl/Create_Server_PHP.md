---
title: "PHP - Serveurs via Pterodactyl"
---

# 🚀 Création de serveur Pterodactyl
Ce document explique le fonctionnement du script PHP `create_server.php` qui automatise la création de serveurs de jeu/vm (NodeJS ou Python) via l'API Pterodactyl. Il est utilisé dans NeoHeberg pour permettre aux utilisateurs de créer leur serveur en quelques clics.

---

## 📋 Fonctionnalités

- Vérification du solde de l'utilisateur
- Création ou récupération d'un compte Pterodactyl associé
- Sélection automatique d'une allocation libre (port dans la plage 3121-3921)
- Création du serveur via l'API Pterodactyl
- Débit du prix du plan
- Enregistrement des informations en base de données (logs, slots)
- Gestion des erreurs avec remboursement automatique en cas d'échec

---

## ⚙️ Architecture du script

Le script est un point d'entrée unique qui traite les requêtes POST provenant d'un formulaire de création. Il s'appuie sur plusieurs fonctions utilitaires.

### 🧩 Fonctions principales

| Fonction | Rôle |
|----------|------|
| `log_pterodactyl()` | Journalise les appels/réponses API dans un tableau (affichage possible en cas d'erreur) |
| `debug_log()` | Insère un message de débogage dans la table `debug_logs` (utile pour le suivi) |
| `log_server_creation()` | Enregistre une tentative de création dans la table `server_creation_logs` (succès ou échec) |
| `generateRandomPassword()` | Génère un mot de passe aléatoire pour le compte Pterodactyl |
| `callPterodactylApi()` | Effectue un appel à l'API Pterodactyl (GET, POST, PATCH, DELETE) avec gestion des erreurs |

---

## 🔄 Déroulement détaillé

> 1. Vérification de la session
- Si l'utilisateur n'est pas connecté (`$_SESSION['id']` absent), redirection vers la page de login.

> 2. Récupération des données utilisateur
- Requête en base pour obtenir `nom`, `email`, `porte_monnaie`.
- Vérification de l'existence d'un compte Pterodactyl existant via la table `pterodactyl_slots` (champ `pterodactyl_user_id`).

> 3. Traitement du formulaire POST
Le formulaire doit contenir :
- `name` : nom du serveur
- `server_type` : `nodejs` ou `python`
- `plan_id` : identifiant du plan (défini dans `plans_config.php`)
- `node_version` ou `python_version` : version souhaitée

> Validation
- Plan existe-t-il ?
- Nom non vide et ≤ 50 caractères ?
- Solde suffisant ?

> Configuration selon le type
- **NodeJS** : `nest_id = 5`, `egg_id = 18`, images Docker spécifiques, startup personnalisé.
- **Python** : `nest_id = 7`, `egg_id = 20`, images Docker spécifiques, startup défini comme `'{{STARTUP_CMD}} && {{SECOND_CMD}}'` (exécute d'abord `pip install`, puis `python main.py`).

> 4. Gestion du compte Pterodactyl
Si aucun `pterodactyl_user_id` n'existe :
- Création d'un utilisateur via `POST /users` avec un mot de passe généré aléatoirement.
- Stockage des identifiants dans la table `panel_login` (pour que l'utilisateur puisse se connecter au panel).
- Mise en session des identifiants (optionnel).

Si le compte existe déjà, on l'utilise directement.

> 5. Débit du prix
- Nouveau solde = ancien solde – prix du plan.
- Mise à jour en base (`UPDATE users`).

> 6. Récupération d'une allocation libre
- Le nœud utilisé est le nœud `1` (à adapter selon configuration).
- Appels paginés à `GET /nodes/{nodeId}/allocations` pour récupérer **toutes** les allocations.
- Filtrage des allocations **libres** (`assigned === false`) et dont le **port est compris entre 3121 et 3921** (plage réservée).
- Sélection de la première allocation libre disponible.
- Si aucune allocation trouvée, **remboursement** et message d'erreur.

> 7. Création du serveur
- Construction du payload JSON conforme à l'API Pterodactyl :
  - `name`, `user`, `egg`, `nest`, `docker_image`, `startup`, `environment`
  - `limits` : mémoire (Mo), swap (0), disque (Mo), IO, CPU
  - `feature_limits` : databases (0), allocations (1), backups (0)
  - `allocation` : l'ID de l'allocation choisie
  - `start_on_completion: true` pour démarrer automatiquement
- Envoi de la requête `POST /servers`.
- Récupération de l'ID du serveur créé.

> 8. Enregistrement en base
- Insertion dans `pterodactyl_slots` avec `user_id`, `pterodactyl_user_id`, `server_id`, `created_at`, `expires_at` (31 jours plus tard).
- Insertion d'un log de succès dans `server_creation_logs`.

> 9. Redirection avec message de succès
- Message en session : "Votre serveur NodeJS/Python 'nom' (vX) a été créé avec succès ! Expire le JJ/MM/AAAA".

> 10. Gestion des erreurs
- Si une exception survient à n'importe quelle étape **après le débit**, on **rembourse** l'utilisateur (remise à l'ancien solde).
- Toutes les erreurs sont loguées dans `server_creation_logs` avec le statut `'error'` et le message d'erreur.
- L'utilisateur est redirigé vers la page précédente avec les erreurs en session.

---

## 🗃️ Tables de base de données utilisées

> `users`
- `id` : identifiant
- `nom`, `email`
- `porte_monnaie` : solde en coins

> `pterodactyl_slots`
- `user_id` : FK vers users
- `pterodactyl_user_id` : ID du compte Pterodactyl
- `server_id` : ID du serveur créé
- `created_at`, `expires_at`

> `panel_login`
- Stockage des identifiants tiers (ici pour Pterodactyl)
- `user_id`, `type` ('pterodactyl'), `loginname`, `password`, `domain`

> `server_creation_logs`
- Log détaillé de chaque tentative (succès/échec)
- Contient toutes les informations : utilisateur, plan, ressources, ID Pterodactyl, message d'erreur, etc.

> `debug_logs` (optionnel)
- Logs de débogage pour le développement

---

## 🔐 Sécurité

- Vérification de session avant toute opération.
- Échappement des données utilisateur via requêtes préparées (PDO).
- Gestion des erreurs API avec `try/catch` et remboursement.
- Les mots de passe générés sont forts (16 caractères, caractères spéciaux).
- Les clés API Pterodactyl sont stockées dans `config.php` (constante `PTERODACTYL_API_KEY`).

---

## 🧪 Tests et débogage

- Les fonctions `debug_log()` et `log_pterodactyl()` permettent de tracer l'exécution.
- La table `server_creation_logs` conserve un historique complet.
- En cas d'échec, consultez le champ `pterodactyl_logs` de cette table pour voir les requêtes/réponses API.

---

## 🔗 Ressources

- [Documentation de l'API Pterodactyl (Application API)](https://pterodactyl.io/api/application/index.html)
- [Guide sur les eggs Pterodactyl](https://pterodactyl.io/community/customization/eggs.html)
- [Dépôt NeoHeberg (Eggs VPS)](https://github.com/neoheberg/Eggs-Pterodactyl)
