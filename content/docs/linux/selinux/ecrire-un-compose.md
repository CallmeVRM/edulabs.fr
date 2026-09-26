---
title: "Écrire un premier fichier docker-compose.yml"
weight: 2
---

## Objectif

Décrire, dans un fichier unique, plusieurs services conteneurisés qui doivent fonctionner ensemble (par exemple une application web et sa base de données), afin de les démarrer et les arrêter d'une seule commande.

## Prérequis

- Docker Engine et le plugin Compose installés (`docker compose version` doit répondre)
- Un répertoire de projet dédié
- Connaître les images et variables d'environnement nécessaires aux services

## Étapes

1. Créer un répertoire de projet et s'y positionner.
2. Décrire les services dans un fichier `docker-compose.yml`.
3. Démarrer la pile en arrière-plan.
4. Consulter les journaux pour s'assurer que tout démarre correctement.

```bash
mkdir monapp && cd monapp
nano docker-compose.yml
docker compose up -d
docker compose logs -f
```

Exemple de fichier `docker-compose.yml` avec un service web et une base de données PostgreSQL :

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: changeme
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

Le champ `depends_on` garantit l'ordre de démarrage des conteneurs, mais ne vérifie pas qu'un service est réellement prêt à recevoir des connexions ; pour cela, il est recommandé d'ajouter des `healthcheck` dans les cas plus exigeants. Le volume nommé `db-data` assure la persistance des données de la base même si le conteneur est recréé.

## Vérification

Vérifiez que les deux conteneurs sont bien démarrés et que le service web répond :

```bash
docker compose ps
curl -I http://localhost:8080
```

`docker compose ps` doit indiquer un état `running` pour les deux services, et la requête `curl` doit renvoyer un code `200 OK`.
