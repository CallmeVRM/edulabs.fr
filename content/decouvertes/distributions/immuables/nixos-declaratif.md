---
title: "NixOS et la configuration déclarative"
weight: 2
---

## Présentation

NixOS pousse l'idée d'immuabilité un peu plus loin que ses cousines : tout le système, des paquets installés aux services activés en passant par la configuration réseau, est décrit dans un ou plusieurs fichiers `.nix`. Le gestionnaire de paquets Nix construit alors un nouvel environnement système à partir de cette description, sans jamais écraser l'ancien.

## Ce qui est intéressant

- Une configuration complète tient dans un fichier texte versionnable avec git, reproductible sur une autre machine
- Chaque génération du système reste disponible dans le menu de démarrage, avec retour en arrière garanti
- Les dépendances de chaque paquet sont isolées dans le Nix store, ce qui élimine une grande partie des conflits de versions
- Les environnements de développement éphémères avec `nix-shell` évitent de polluer le système pour un simple test

## Cas d'usage

Particulièrement adapté aux équipes qui veulent des environnements identiques entre plusieurs machines ou entre le poste de dev et le serveur. Un service se déclare simplement dans `configuration.nix` :

```nix
services.nginx.enable = true;
```

puis `nixos-rebuild switch` applique le changement.

## Avis en bref

La courbe d'apprentissage du langage Nix est réelle et parfois rugueuse, mais la garantie de reproductibilité qu'on obtient en échange n'a pas vraiment d'équivalent ailleurs.
