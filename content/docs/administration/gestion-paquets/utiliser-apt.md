---
title: "Utiliser APT au quotidien"
weight: 1
---

## Objectif

Ce guide couvre les opérations les plus fréquentes avec APT, le gestionnaire de paquets des distributions basées sur Debian (Debian, Ubuntu, Mint...) : recherche, installation, mise à jour et nettoyage des paquets.

## Prérequis

- Distribution basée sur Debian ou Ubuntu
- Accès `sudo` pour les opérations d'installation et de suppression
- Une connexion réseau vers les dépôts configurés dans `/etc/apt/sources.list`

## Étapes

1. Rafraîchir la liste des paquets disponibles depuis les dépôts configurés.
2. Rechercher un paquet par mot-clé.
3. Installer un paquet et ses dépendances.
4. Mettre à jour l'ensemble des paquets installés.
5. Supprimer les paquets qui ne sont plus nécessaires (dépendances orphelines).

```bash
sudo apt update
apt search nginx
sudo apt install nginx
sudo apt upgrade
sudo apt autoremove
```

`apt update` ne fait que synchroniser les index locaux avec les dépôts distants, il n'installe rien. `apt upgrade` applique ensuite les mises à jour disponibles pour les paquets déjà installés, sans en supprimer. Pensez à distinguer `apt upgrade` de `apt full-upgrade`, qui autorise la suppression de paquets si cela est nécessaire pour résoudre les dépendances.

## Vérification

Vérifiez qu'un paquet est bien installé et consultez sa version :

```bash
dpkg -l | grep nginx
apt show nginx
```

`dpkg -l` doit afficher `ii` en début de ligne, ce qui signifie que le paquet est correctement installé et configuré. La commande `apt show` donne des détails complémentaires comme la version installée et la taille du paquet.
