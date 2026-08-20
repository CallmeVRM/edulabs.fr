---
title: "Comprendre le cache ARC de ZFS"
weight: 1
---

## Contexte

ZFS n'utilise pas le cache de pages standard du noyau Linux mais son propre mécanisme, l'ARC (Adaptive Replacement Cache). Cette particularité surprend souvent les administrateurs qui voient la commande `free` afficher peu de mémoire "disponible" alors que le système fonctionne parfaitement bien. Il paraissait utile de clarifier ce que fait réellement l'ARC.

## Analyse technique

L'ARC est un cache de blocs qui maintient deux listes séparées : une pour les blocs récemment utilisés (MRU, most recently used) et une pour les blocs fréquemment utilisés (MFU, most frequently used), avec un ajustement dynamique de la taille relative de chaque liste selon les motifs d'accès observés. Ce fonctionnement le rend en théorie plus résistant aux balayages ponctuels (un grand `find` ou une sauvegarde complète) qu'un simple cache LRU, car les blocs fréquemment accédés ne sont pas évincés par un accès massif mais ponctuel.

L'ARC peut aussi s'étendre à un second niveau, le L2ARC, stocké sur un périphérique rapide (SSD ou NVMe) pour absorber les données qui débordent de la RAM sans redescendre jusqu'au disque mécanique.

```bash
# taille courante et limites de l'ARC
cat /proc/spl/kstat/zfs/arcstats | grep -E "^(size|c_max|c_min) "

# fixer une limite haute à 8 Go
echo 8589934592 > /sys/module/zfs/parameters/zfs_arc_max
```

## Observations

Sur un serveur de fichiers avec 64 Go de RAM et une charge de lecture aléatoire mixte :

- l'ARC occupe par défaut jusqu'à environ 50 % de la RAM disponible, ce qui explique la mémoire "manquante" dans les outils standards ;
- le taux de succès du cache (`hit ratio`) mesuré sur une semaine dépasse 90 % pour les blocs de métadonnées et reste autour de 70 % pour les données, ce qui confirme l'intérêt de la séparation MRU/MFU sur cette charge ;
- limiter explicitement `zfs_arc_max` évite les situations où l'ARC entre en compétition mémoire avec d'autres services critiques du même hôte.

## Pour aller plus loin

Un test avec L2ARC sur un SSD NVMe permettrait de quantifier le gain réel une fois la RAM saturée, notamment sur un working set dépassant largement la taille de l'ARC primaire.
