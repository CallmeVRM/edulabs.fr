---
title: "Kernel panic juste après le chargement du noyau"
weight: 2
---

## Symptômes

Le démarrage progresse normalement jusqu'à l'affichage des premiers messages du noyau, puis s'arrête brutalement avec un message « Kernel panic - not syncing » suivi d'une trace d'appels techniques. Le système ne répond plus et nécessite un redémarrage forcé. Le problème apparaît généralement juste après l'installation d'une mise à jour de noyau.

## Cause probable

La cause la plus fréquente est un initramfs incomplet ou mal reconstruit qui ne contient pas les modules nécessaires pour monter la racine du système de fichiers (contrôleur de stockage, pilote LVM ou RAID manquant), provoquant l'erreur classique « VFS: Unable to mount root fs ». Un module tiers incompatible avec la nouvelle version du noyau, ou un paramètre de démarrage erroné pointant vers un UUID de partition invalide, peuvent produire le même résultat.

## Diagnostic

```bash
journalctl -b -1 -k | tail -50
cat /boot/grub/grub.cfg | grep -A2 "menuentry.*<ancien noyau>"
lsinitramfs /boot/initrd.img-<version> | grep -i <module>
blkid
```

Comparer les paramètres de démarrage de l'ancien noyau qui fonctionnait encore avec ceux du nouveau, dans le menu GRUB, aide souvent à repérer un UUID racine incohérent. `lsinitramfs` permet de vérifier si un module attendu (pilote de stockage notamment) est bien présent dans l'image initramfs générée.

## Résolution

1. Redémarrer en sélectionnant dans le menu GRUB l'ancienne version du noyau qui fonctionnait, afin de retrouver un système opérationnel pour investiguer.
2. Vérifier que l'UUID de la partition racine dans les paramètres de démarrage correspond bien à celui renvoyé par `blkid`.
3. Reconstruire l'initramfs du noyau en échec avec `update-initramfs -c -k <version>` (ou l'équivalent `dracut --force` selon la distribution), en s'assurant que les modules de stockage nécessaires y sont bien inclus.
4. Si le noyau récemment installé reste instable après reconstruction de l'initramfs, envisager de revenir temporairement à la version précédente en attendant un correctif amont.
