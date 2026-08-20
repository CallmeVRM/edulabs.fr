---
title: "Snapshots Btrfs : granularité et coût réel"
weight: 1
---

## Contexte

Les snapshots Btrfs sont souvent présentés comme "quasi gratuits" grâce au copy-on-write, ce qui pousse certains à en créer à haute fréquence (toutes les cinq minutes via `snapper` par exemple). Cette note vérifie si cette gratuité tient réellement à l'échelle, et jusqu'où on peut pousser la granularité sans effet de bord.

## Analyse technique

Un snapshot Btrfs est une copie légère d'un sous-volume : il ne duplique aucune donnée à la création, seulement les métadonnées de l'arbre B qui décrivent la structure du système de fichiers. Les blocs de données restent partagés jusqu'à ce qu'une écriture ultérieure déclenche le mécanisme de copy-on-write, qui alloue alors un nouveau bloc plutôt que de modifier l'ancien.

Le coût caché apparaît à mesure que le nombre de snapshots augmente : chaque suppression de fichier ou modification doit vérifier, à travers l'arbre de références partagées, quels snapshots pointent encore vers les blocs concernés. Sur un volume avec des centaines de snapshots actifs, les opérations de nettoyage (`btrfs balance`, la suppression de sous-volumes) deviennent nettement plus lentes.

```bash
# créer un snapshot en lecture seule
btrfs subvolume snapshot -r /data /data/.snapshots/2026-08-20

# lister les sous-volumes et leur taille exclusive
btrfs subvolume list /data
btrfs qgroup show -pcre /data
```

## Observations

Sur un volume de test avec rotation automatique de snapshots horaires conservés 30 jours :

- la création reste effectivement quasi instantanée, quel que soit le nombre de snapshots déjà présents ;
- la suppression d'un ancien snapshot devient perceptiblement plus lente une fois le volume au-delà de 200 à 300 snapshots simultanés, en particulier avec les quotas (`qgroups`) activés ;
- désactiver les quotas quand ils ne sont pas strictement nécessaires réduit sensiblement ce coût, les quotas imposant un recalcul de comptabilité à chaque opération.

## Pour aller plus loin

Une comparaison avec les snapshots ZFS sur le même scénario de rotation permettrait de savoir si ce coût de nettoyage est une particularité de l'implémentation Btrfs ou un phénomène partagé par tout système copy-on-write.
