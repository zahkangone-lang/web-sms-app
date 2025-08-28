# Guide de Déploiement de la Stack

## Description des Composants de la Stack

La stack se compose des services suivants :

- **websms-app** :

  - Application principale WebSMS, basée sur l'image Docker `arolitec/websms:latest`.
  - Expose le port 3000 pour l'accès à l'interface web.
  - Dépend des autres services pour fonctionner correctement.

- **websms-db** :

  - Base de données PostgreSQL (image `postgres:16-alpine`).
  - Stocke toutes les données persistantes de l'application.
  - Expose le port 5432.

- **websms-redis** :

  - Système de cache et de gestion des files d'attente (image `redis/redis-stack:latest`).
  - Expose le port 6379.

- **websms-minio** :

  - Stockage d'objets compatible S3 (image `minio/minio:latest`).
  - Utilisé pour stocker les fichiers et documents générés ou importés par l'application.
  - Expose les ports 9000 (API) et 9001 (console d'administration).

- **websms-typesense** :
  - Moteur de recherche full-text (image `typesense/typesense:26.0`).
  - Permet la recherche rapide et efficace dans les données de l'application.
  - Expose le port 8108.

Chaque service est isolé dans un conteneur Docker et communique via le réseau interne `websms`. Les volumes Docker assurent la persistance des données pour chaque composant critique.

## Prérequis

- Docker et Docker Compose installés sur votre machine.
- Accès aux fichiers de configuration (`compose.example.yml`, variables d’environnement, etc.).
- Accès au dépôt source de la stack.

## Gestion des fichiers .env

Chaque service de la stack peut utiliser son propre fichier d'environnement pour définir ses variables spécifiques. Voici les bonnes pratiques :

- **Fichier global** : Vous pouvez créer un fichier `.env` à la racine du projet pour les variables communes à tous les services (ex : ports, secrets partagés).
- **Fichiers dédiés** : Pour une meilleure isolation, créez des fichiers `.env.websms-app`, `.env.websms-db`, `.env.websms-redis`, `.env.websms-minio`, `.env.websms-typesense` et renseignez les variables propres à chaque service.
- **Référence dans compose.yml** : Dans le fichier `compose.yml`, utilisez la clé `env_file` pour indiquer le ou les fichiers d'environnement à charger pour chaque service.

Exemple :

```yaml
services:
  websms-app:
    env_file:
      - .env.websms-app
  websms-db:
    env_file:
      - .env.websms-db
```

## Étapes de Déploiement

1. **Cloner le dépôt**

   ```bash
   git clone <URL_DU_DEPOT>
   cd websms-js-deployment
   ```

2. **Configurer les variables d’environnement**

   - Copiez le script docker compose d’exemple :

     ```bash
     cp compose.example.yml compose.yml
     ```

   - Modifier les fichiers d’environnement nécessaires :

     - Pour chaque service, il y a une fichier d'environnement qui porte son nom (ex : `.env.websms-app`, `.env.websms-db`, etc.). Renseignez les variables spécifiques à chaque service dans le fichier approprié.
     - Si certaines variables sont communes, vous pouvez les placer dans un fichier `.env` global à la racine du projet.

   - Dans le fichier `compose.yml`, utilisez la clé `env_file` pour référencer le ou les fichiers d’environnement pour chaque service. Exemple :

     ```yaml
     services:
       websms-app:
         env_file:
           - .env.websms-app
       websms-db:
         env_file:
           - .env.websms-db
     ```

   - Adaptez le contenu de chaque fichier `.env` selon votre environnement (développement, production, etc.).

   - Ne partagez jamais vos fichiers `.env` contenant des secrets sensibles. Pour faciliter la prise en main, créez un fichier `.env.example` documentant toutes les variables nécessaires.

3. **Lancer les services avec Docker Compose**

   ```bash
   docker-compose up -d
   ```

   - Cette commande démarre tous les services en arrière-plan.

4. **Vérifier le bon fonctionnement**

   - Consultez les logs :
     ```bash
     docker-compose logs
     ```
   - Vérifiez que tous les services sont bien démarrés et accessibles.

5. **Accéder à l’application**
   - Ouvrez votre navigateur et rendez-vous à l’URL configurée (ex : http://localhost:3000).

## Astuces et Résolution de Problèmes

- Pour redémarrer la stack :
  ```bash
  docker-compose restart
  ```
- Pour arrêter tous les services :
  ```bash
  docker-compose down
  ```
- En cas d’erreur, vérifiez les logs et la configuration des variables d’environnement.

## Mise à jour

- Pour mettre à jour l'application Websms :

  ```bash
  docker-compose pull arolitec/websms:latest && docker compose up -d websms-app --force-recreate
  ```

- Pour mettre à jour toute la stack :
  ```bash
  docker compose pull && docker compose up -d --force-recreate
  ```

---

N’hésitez pas à adapter ce guide selon les spécificités de votre projet ou de votre infrastructure.
