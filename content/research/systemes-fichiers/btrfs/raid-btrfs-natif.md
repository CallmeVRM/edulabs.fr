---
title: "RAID natif Btrfs : promesses et limites actuelles"
weight: 2
---

## Contexte

Btrfs intègre nativement la gestion de RAID au niveau du système de fichiers, sans passer par mdadm ou LVM. L'idée est séduisante : un seul outil pour gérer volumes, checksums et redondance. Mais le RAID5/6 de Btrfs traîne une réputation de fonctionnalité instable depuis des années, ce qui justifie une vérification de l'état actuel des choses.

## Analyse technique

Contrairement à un RAID matériel ou mdadm classique, le RAID Btrfs fonctionne au niveau des blocs de métadonnées et de données séparément, ce qui permet des profils mixtes (par exemple métadonnées en RAID1, données en RAID0). Le RAID1 et le RAID10 sont considérés stables depuis longtemps, avec des checksums qui permettent de détecter et corriger une corruption silencieuse en piochant la bonne copie.

Le RAID5/6, en revanche, souffre toujours du fameux "write hole" : en cas de coupure de courant pendant une écriture de parité incomplète, les blocs de parité peuvent devenir incohérents sans que Btrfs le détecte immédiatement, contrairement à ce que garantit un RAID6 mdadm bien journalisé.

```bash
# créer un volume RAID1 sur deux disques
mkfs.btrfs -d raid1 -m raid1 /dev/sdb /dev/sdc

# vérifier le profil de redondance actuel
btrfs filesystem df /mnt/data
```

## Observations

En testant une panne simulée (coupure d'alimentation pendant une écriture) sur un volume RAID6 :

- le RAID1 et le RAID10 se sont comportés de façon fiable, avec reconstruction automatique après remplacement d'un disque simulé ;
- le volume RAID6 a présenté une incohérence de parité détectée seulement lors d'un `scrub` manuel ultérieur, confirmant que le problème du write hole reste réel sur les versions testées ;
- la documentation officielle du projet continue de déconseiller RAID5/6 en production, ce que ces observations corroborent.

## Pour aller plus loin

Il vaudrait la peine de suivre l'avancement du "RAID stripe tree", la refonte annoncée pour corriger structurellement ce problème, et de refaire ce test une fois cette fonctionnalité stabilisée.
