---
title: "Tableau RAID logiciel bloqué en mode dégradé"
weight: 2
---

## Symptômes

La commande `cat /proc/mdstat` affiche le tableau RAID avec un disque marqué `(F)` (failed) ou absent, et l'état général indique `degraded`. Les performances en écriture sont réduites et une alerte de supervision signale la perte de redondance. Le système continue de fonctionner normalement, mais une nouvelle panne de disque entraînerait alors une perte de données.

## Cause probable

Un des disques membres du tableau RAID a été marqué défaillant, soit à cause d'une réelle panne matérielle détectée par le contrôleur ou le noyau, soit à la suite d'un simple faux positif provoqué par une erreur de câblage, une coupure d'alimentation transitoire, ou un timeout de commande trop court pour un disque par ailleurs sain.

## Diagnostic

```bash
cat /proc/mdstat
mdadm --detail /dev/md0
smartctl -a /dev/sdb
journalctl -k | grep -i md0
```

`mdadm --detail` précise quel disque exact est en cause et depuis quand il est sorti du tableau. Croiser cette information avec les données SMART du disque permet de distinguer une panne réelle d'un incident ponctuel : un disque SMART sain retiré du tableau est un bon candidat pour une simple réintégration.

## Résolution

1. Si le disque est toujours physiquement sain (SMART correct, pas d'erreurs I/O répétées), le retirer puis le réintégrer dans le tableau : `mdadm /dev/md0 --remove /dev/sdb1` suivi de `mdadm /dev/md0 --add /dev/sdb1`.
2. Suivre la progression de la resynchronisation avec `watch cat /proc/mdstat` jusqu'à ce que l'état repasse à `clean`.
3. Si le disque présente des erreurs SMART confirmées, le remplacer physiquement avant de lancer la même procédure d'ajout avec le nouveau disque.
4. Une fois le tableau reconstruit, vérifier qu'aucune alerte de supervision RAID ne subsiste et documenter la cause retenue (panne réelle ou faux positif) pour ajuster si besoin les timeouts de commande disque.
