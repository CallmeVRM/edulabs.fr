---
title: "Planifier avec systemd timers"
weight: 2
---

## Objectif

Remplacer une entrée cron par un couple `service` + `timer` systemd, pour bénéficier de la journalisation dans `journald`, de la gestion des dépendances et du rattrapage automatique (`Persistent=true`) après une extinction de la machine.

## Prérequis

- Accès `root` ou `sudo` pour créer des unités dans `/etc/systemd/system/`
- Un script ou une commande de sauvegarde déjà fonctionnelle (voir [Sauvegarder avec Restic]({{< relref "sauvegarder-avec-restic" >}}))

## Étapes

{{< steps >}}

### Créer le service

`/etc/systemd/system/backup-restic.service` :

```ini
[Unit]
Description=Sauvegarde Restic

[Service]
Type=oneshot
EnvironmentFile=/etc/restic/backup.env
ExecStart=/usr/bin/restic backup /etc /home /var/www
```

Le service est de type `oneshot` : il exécute la commande puis se termine, contrairement à un service qui tournerait en continu. Les identifiants sensibles (mot de passe du dépôt, clés d'accès) sont isolés dans un fichier d'environnement dédié plutôt qu'écrits en clair dans l'unité.

### Créer le timer associé

`/etc/systemd/system/backup-restic.timer` :

```ini
[Unit]
Description=Planification de la sauvegarde Restic

[Timer]
OnCalendar=*-*-* 02:30:00
Persistent=true

[Install]
WantedBy=timers.target
```

`Persistent=true` déclenche l'exécution au prochain démarrage si la machine était éteinte à l'heure prévue, ce que `cron` ne fait pas nativement.

### Activer et démarrer le timer

```bash
systemctl daemon-reload
systemctl enable --now backup-restic.timer
```

{{< /steps >}}

## Vérification

```bash
systemctl list-timers backup-restic.timer
journalctl -u backup-restic.service --since "-7 days"
```

`list-timers` affiche la prochaine date de déclenchement calculée ainsi que la date de la dernière exécution. En cas d'échec du service, `journalctl` contient la sortie complète de `restic`, y compris les éventuels messages d'erreur réseau ou de dépôt verrouillé.

{{< callout type="warning" >}}
Un `restic backup` interrompu brutalement (coupure secteur, `OOM killer`) peut laisser un verrou (`lock`) orphelin dans le dépôt. Si une exécution suivante échoue avec `unable to create lock`, vérifiez qu'aucun processus `restic` n'est réellement actif puis levez le verrou avec `restic unlock`.
{{< /callout >}}
