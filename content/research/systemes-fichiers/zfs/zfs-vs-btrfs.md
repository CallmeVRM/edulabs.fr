---
title: "ZFS et Btrfs : comparatif pour un usage serveur"
weight: 2
---

## Contexte

Le choix entre ZFS et Btrfs revient régulièrement lors du dimensionnement d'un nouveau serveur de stockage. Les deux visent des objectifs proches (intégrité des données, snapshots, gestion de volumes) mais avec des historiques et des contraintes d'intégration très différents sous Linux, ce qui justifie de remettre les critères de choix à plat.

## Analyse technique

ZFS, sous licence CDDL jugée incompatible avec la GPL par la Software Foundation, ne peut pas être intégré au noyau mainline et doit être installé comme module hors arbre via OpenZFS, avec un risque de rupture à chaque montée de version noyau majeure. En contrepartie, il bénéficie de plus de vingt ans de maturité industrielle, notamment sur Solaris puis FreeBSD, et de fonctionnalités comme les RAID-Z sans le problème de write hole qui touche Btrfs, grâce à une conception qui évite l'écriture partielle de bandes.

Btrfs, intégré au noyau, profite d'une installation plus simple et d'un développement suivant le rythme du noyau lui-même, mais certaines fonctionnalités (RAID5/6 notamment) restent en retrait par rapport à leur équivalent ZFS en termes de fiabilité éprouvée.

```bash
# comparaison rapide des commandes équivalentes
zpool status tank          # ZFS : état du pool
btrfs filesystem show      # Btrfs : état du système de fichiers

zfs list -o space          # ZFS : usage par dataset
btrfs filesystem usage /mnt # Btrfs : usage global
```

## Observations

En confrontant les deux sur un même serveur de sauvegarde (deux volumes équivalents, un par technologie) :

- ZFS s'est montré plus prévisible sous forte charge d'écriture séquentielle continue, avec une dégradation plus progressive à l'approche du plein ;
- Btrfs s'est révélé plus simple à administrer au quotidien, notamment pour redimensionner des sous-volumes à la volée sans étape d'export/import ;
- les deux ont détecté et corrigé sans intervention une corruption silencieuse injectée artificiellement sur un des disques, confirmant que le mécanisme de checksum fonctionne comme attendu dans les deux cas.

## Pour aller plus loin

Le choix dépend beaucoup du besoin en RAID parité : pour ce cas précis, ZFS reste la valeur la plus sûre si RAID5/6 (RAID-Z) est requis, une question à retester régulièrement à mesure que Btrfs progresse.
