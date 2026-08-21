---
title: "Sauvegarder avec Restic"
weight: 1
---

## Objectif

Ce guide met en place des sauvegardes chiffrées, dédupliquées et incrémentales avec [Restic](https://restic.net/), vers un dépôt local ou distant (SFTP, S3, Backblaze B2...).

## Prérequis

- `restic` installé (`apt install restic` ou binaire statique)
- Un espace de stockage cible (disque externe, serveur SFTP, bucket S3...)
- Un mot de passe de dépôt à conserver précieusement

{{< callout type="error" >}}
Le mot de passe du dépôt n'est stocké nulle part par Restic. En cas de perte, les données sauvegardées sont **définitivement inaccessibles**, y compris pour vous.
{{< /callout >}}

## Étapes

{{< steps >}}

### Initialiser le dépôt

```bash
export RESTIC_REPOSITORY=/mnt/backup/restic-repo
export RESTIC_PASSWORD="un-mot-de-passe-long-et-unique"
restic init
```

### Lancer une première sauvegarde

```bash
restic backup /etc /home /var/www
```

Restic ne transfère que les blocs de données nouveaux ou modifiés depuis le dernier instantané (`snapshot`), grâce à un découpage en chunks à taille variable. Une deuxième sauvegarde du même arbre est donc généralement très rapide.

### Vérifier les instantanés existants

```bash
restic snapshots
```

### Restaurer un fichier ou une arborescence

```bash
restic restore latest --target /tmp/restauration --include /etc/nginx
```

{{< /steps >}}

## Politique de rétention

Sans nettoyage, le dépôt grossit indéfiniment. La commande `forget` associée à `prune` applique une politique de rétention puis libère l'espace disque correspondant :

```bash
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

{{< callout type="info" >}}
`forget` sans `--prune` ne fait que marquer les instantanés obsolètes : les données ne sont réellement supprimées du dépôt qu'au passage de `prune`, qui peut être lancé séparément si l'opération est coûteuse en I/O.
{{< /callout >}}

## Vérification

Contrôlez régulièrement l'intégrité du dépôt, idéalement après chaque changement de support de stockage :

```bash
restic check --read-data-subset=10%
```

Une vérification complète (`--read-data-subset=100%` ou sans l'option) relit l'intégralité des données stockées et peut être longue sur un dépôt volumineux ; un sondage partiel régulier est un bon compromis pour détecter une corruption silencieuse du support.
