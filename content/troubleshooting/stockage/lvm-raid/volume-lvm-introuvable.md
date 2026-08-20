---
title: "Un volume logique LVM n'est plus détecté au démarrage"
weight: 1
---

## Symptômes

Au démarrage, le système bascule en mode secours (emergency shell) car un point de montage attendu dans `/etc/fstab` n'est pas trouvé. La commande `lvs` ne renvoie aucun volume logique alors qu'il existait la veille, ou renvoie le volume avec un statut inactif.

## Cause probable

Le plus souvent, un disque physique appartenant au groupe de volumes n'a pas été détecté à temps par le noyau avant que `lvm2` ne tente d'activer les volumes (cas fréquent avec un disque USB ou un support lent), ou les métadonnées LVM ont été partiellement écrasées par une opération de partitionnement effectuée sur un autre disque du système. Un service `initramfs` mal reconstruit après une mise à jour peut aussi omettre les modules nécessaires à la détection du groupe de volumes.

## Diagnostic

```bash
pvs
vgs
lvs -a -o+devices
journalctl -b | grep -i lvm
```

Si `pvs` ne liste pas le volume physique attendu alors qu'il apparaît dans `lsblk`, le problème vient de la détection ou de l'activation LVM plutôt que d'une perte de disque. La colonne `devices` de `lvs -a` permet de vérifier sur quel support physique repose chaque extent logique.

## Résolution

1. Vérifier que le disque physique est bien reconnu par le noyau avec `lsblk` et `fdisk -l` avant toute action sur LVM.
2. Réactiver manuellement le groupe de volumes avec `vgchange -ay nom_du_vg` puis vérifier l'état avec `lvs`.
3. Si les métadonnées semblent corrompues, consulter les sauvegardes automatiques conservées par LVM dans `/etc/lvm/backup/` et `/etc/lvm/archive/` pour une restauration via `vgcfgrestore`.
4. Reconstruire l'initramfs (`update-initramfs -u` ou équivalent) si le module LVM ou le pilote du contrôleur disque manquait à l'activation, puis redémarrer pour confirmer que le montage se fait automatiquement.
