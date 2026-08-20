---
title: "Transparent Huge Pages : bénéfices et effets de bord"
weight: 2
---

## Contexte

Les Transparent Huge Pages (THP) promettent de réduire la pression sur le TLB en regroupant les pages mémoire de 4 Ko en blocs de 2 Mo, de façon transparente pour les applications. Le sujet revient régulièrement dans les discussions sur les bases de données, où THP a une réputation mitigée : accélérateur pour certains, cause de latences erratiques pour d'autres.

## Analyse technique

Le noyau gère THP via un démon en arrière-plan, `khugepaged`, qui scanne périodiquement la mémoire pour fusionner des pages contiguës en pages géantes lorsque c'est possible. Le mode par défaut, `madvise`, ne l'active que pour les zones mémoire explicitement marquées par l'application ; le mode `always` l'applique agressivement à toute la mémoire éligible.

Le problème classique vient du coût de la fusion et surtout de la défragmentation associée : quand `khugepaged` doit compacter la mémoire pour trouver un bloc de 2 Mo contigu, il peut déclencher des pauses notables, en particulier sous forte fragmentation mémoire, un phénomène documenté depuis longtemps avec des moteurs comme Redis ou certaines bases relationnelles.

```bash
# état courant de THP
cat /sys/kernel/mm/transparent_hugepage/enabled
# [always] madvise never

# désactiver la défragmentation agressive sans désactiver THP
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

## Observations

Sur un serveur de base de données relationnelle avec un jeu de travail important :

- en mode `always`, on observe des pics de latence P99 ponctuels corrélés aux cycles de compaction de `khugepaged` ;
- en désactivant totalement THP, la latence est plus stable mais le débit moyen en lecture séquentielle baisse légèrement ;
- le compromis `madvise` avec `defrag=never` s'est révélé être le point d'équilibre le plus satisfaisant sur ce profil de charge, laissant l'application demander explicitement les pages géantes sans subir les pauses de compaction.

## Pour aller plus loin

Il reste à évaluer le comportement des pages géantes de 1 Go (`gigantic pages`), réservées statiquement au démarrage, qui évitent complètement le problème de fragmentation mais imposent une contrainte forte sur le dimensionnement mémoire.
