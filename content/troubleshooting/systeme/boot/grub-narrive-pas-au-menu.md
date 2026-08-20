---
title: "GRUB ne présente plus son menu au démarrage"
weight: 1
---

## Symptômes

Au démarrage, l'écran affiche uniquement une invite `grub>` minimaliste au lieu du menu habituel avec la liste des noyaux disponibles, ou reste bloqué sur un écran noir avec un simple curseur clignotant. Aucune touche ne semble déclencher l'apparition du menu attendu.

## Cause probable

Le fichier de configuration `grub.cfg` est absent, corrompu, ou pointe vers un chemin de disque qui a changé, souvent après une mise à jour du noyau qui n'a pas régénéré correctement la configuration, ou après une modification manuelle de partitionnement qui a décalé les identifiants de disque utilisés par GRUB. Un firmware UEFI mal configuré, qui charge une entrée de démarrage obsolète, peut produire un symptôme similaire.

## Diagnostic

```bash
ls (hd0,gpt1)/
cat /boot/grub/grub.cfg | head -30
efibootmgr -v
grub-probe --target=fs_uuid /
```

Depuis l'invite `grub>`, la commande `ls` permet de lister le contenu des partitions détectées et de vérifier qu'un noyau et un initramfs sont bien présents à l'emplacement attendu. Cela distingue un problème de configuration GRUB (fichiers présents mais non référencés) d'un problème plus grave de partition manquante ou illisible.

## Résolution

1. Depuis l'invite `grub>`, charger manuellement le noyau pour confirmer que le système démarre bien une fois la bonne partition identifiée : `set root=(hd0,gpt1)`, `linux /vmlinuz root=/dev/sda1`, `initrd /initrd.img`, puis `boot`.
2. Une fois le système démarré, régénérer la configuration GRUB avec `grub-mkconfig -o /boot/grub/grub.cfg` (ou `update-grub` selon la distribution).
3. Réinstaller le chargeur d'amorçage sur le disque concerné avec `grub-install /dev/sda` si la configuration régénérée ne suffit pas.
4. Sur un système UEFI, vérifier avec `efibootmgr` que l'entrée de démarrage pointe vers le bon chemin, et la recréer si nécessaire.
