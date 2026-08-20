---
title: "Utiliser DNF et Pacman"
weight: 2
---

## Objectif

Au-delà de Debian et Ubuntu, deux autres gestionnaires de paquets sont très répandus : DNF sur les distributions basées sur Fedora/RHEL, et Pacman sur Arch Linux. Ce guide présente les commandes équivalentes pour installer, mettre à jour et supprimer des paquets avec chacun d'eux.

## Prérequis

- Une distribution basée sur Fedora/RHEL (pour DNF) ou Arch Linux/Manjaro (pour Pacman)
- Droits `sudo`
- Dépôts correctement configurés (`/etc/yum.repos.d/` pour DNF, `/etc/pacman.conf` pour Pacman)

## Étapes

Sur une distribution Fedora ou RHEL :

```bash
sudo dnf install httpd
sudo dnf update
sudo dnf remove httpd
sudo dnf autoremove
```

Sur Arch Linux, la logique est proche mais la syntaxe diffère :

```bash
sudo pacman -Sy httpd
sudo pacman -Syu
sudo pacman -R httpd
sudo pacman -Rns httpd
```

Avec Pacman, `-Syu` synchronise les dépôts et met à jour tout le système en une seule commande ; il est déconseillé de faire un simple `-Sy` suivi d'une installation sans mise à jour complète, car cela peut casser des dépendances (le fameux problème des « mises à jour partielles »). L'option `-Rns` supprime un paquet ainsi que ses dépendances désormais inutilisées et ses fichiers de configuration.

## Vérification

Pour DNF :

```bash
dnf list installed | grep httpd
```

Pour Pacman :

```bash
pacman -Qi httpd
```

Dans les deux cas, la commande doit renvoyer une ligne correspondant au paquet avec sa version installée ; une absence de résultat signifie que le paquet n'est pas (ou plus) présent sur le système.
