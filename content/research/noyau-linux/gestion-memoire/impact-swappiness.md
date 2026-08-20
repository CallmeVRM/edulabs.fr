---
title: "Impact du paramètre swappiness sur les performances"
weight: 1
---

## Contexte

Le paramètre `vm.swappiness` est souvent modifié par réflexe ("on le met à 10, c'est mieux") sans qu'on comprenne toujours ce qu'il contrôle réellement. Cette note reprend les mécanismes sous-jacents pour évaluer dans quels cas cette optimisation courante a un effet mesurable.

## Analyse technique

`swappiness` ne détermine pas si le système va swapper, mais l'agressivité relative avec laquelle le noyau va préférer récupérer des pages de cache de fichiers (`page cache`) plutôt que des pages anonymes (mémoire de processus) lorsqu'il a besoin de libérer de la mémoire. La valeur va de 0 à 200 depuis les noyaux récents (elle était plafonnée à 100 auparavant), et sert de poids dans le calcul de pression fait par le réclamateur de pages (`kswapd` et la réclamation directe).

Une valeur basse ne désactive donc pas le swap : elle indique au noyau de sacrifier le cache de fichiers en dernier recours, ce qui peut en réalité être contre-productif si l'application dépend fortement de ce cache pour ses accès disque.

```bash
# lire la valeur courante
cat /proc/sys/vm/swappiness

# l'ajuster temporairement
sysctl vm.swappiness=10
```

## Observations

Sur une machine de base de données avec 32 Go de RAM et un jeu de données actif de 20 Go :

- passer de `swappiness=60` (valeur par défaut) à `10` réduit les événements de swap-out observés dans `/proc/vmstat` de manière significative sous pression mémoire modérée ;
- l'effet devient négligeable dès que la mémoire disponible dépasse largement le working set applicatif, ce qui est le cas la majorité du temps sur cette machine ;
- à l'inverse, sur une machine plus contrainte, une valeur trop basse a entraîné une légère dégradation car le cache de fichiers, lui, a été davantage sollicité par le réclamateur.

## Pour aller plus loin

Une comparaison avec `cgroup v2` et son propre mécanisme de pression mémoire (`memory.pressure`) permettrait de voir si le réglage global de `swappiness` reste pertinent dans un contexte fortement conteneurisé.
