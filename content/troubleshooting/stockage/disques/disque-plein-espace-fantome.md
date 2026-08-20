---
title: "Disque plein alors que l'espace semble disponible"
weight: 1
---

## Symptômes

Le système refuse d'écrire de nouveaux fichiers avec l'erreur « No space left on device », alors que `df -h` indique encore plusieurs gigaoctets libres sur la partition concernée. Certaines applications plantent au démarrage en tentant d'écrire leurs fichiers temporaires ou leurs journaux.

## Cause probable

Deux causes reviennent le plus souvent : l'épuisement des inodes disponibles (un très grand nombre de petits fichiers a consommé toutes les entrées d'inodes alors que l'espace en octets reste disponible), ou la présence de fichiers supprimés mais encore ouverts par un processus, qui continuent d'occuper l'espace disque tant que le processus qui les détient n'est pas arrêté.

## Diagnostic

```bash
df -h /var
df -i /var
lsof +L1 | grep deleted
du -x --max-depth=1 /var | sort -rh | head
```

`df -i` affiche le taux d'utilisation des inodes, distinct de l'espace en octets : un taux à 100 % alors que `df -h` montre de l'espace libre confirme l'épuisement des inodes. `lsof +L1` liste les fichiers supprimés mais toujours ouverts, ce qui explique un espace disque « fantôme » non restitué au système.

## Résolution

1. En cas d'épuisement des inodes, identifier les répertoires contenant un nombre anormal de petits fichiers (`find /var -xdev -printf '%h\n' | sort | uniq -c | sort -rn | head`) et nettoyer les fichiers obsolètes, en particulier dans les caches applicatifs ou les répertoires de sessions.
2. En cas de fichiers supprimés mais ouverts, identifier le processus responsable via la colonne PID de `lsof` et le redémarrer proprement, ou tronquer le fichier via `/proc/<pid>/fd/<fd>` si un redémarrage immédiat n'est pas possible.
3. Mettre en place une rotation de logs (`logrotate`) si la cause vient de journaux applicatifs qui grossissent sans être nettoyés.
4. Vérifier après action que `df -h` et `df -i` reviennent à des valeurs cohérentes avec l'usage réel constaté.
