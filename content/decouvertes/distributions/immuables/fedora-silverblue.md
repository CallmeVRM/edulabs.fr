---
title: "Fedora Silverblue : une base immuable au quotidien"
weight: 1
---

## Présentation

Fedora Silverblue reprend la base de Fedora Workstation mais remplace le gestionnaire de paquets classique par un système d'images gérées avec rpm-ostree. Le système racine est monté en lecture seule, et chaque mise à jour crée un nouveau déploiement complet plutôt que de modifier les fichiers en place. On bascule d'un déploiement à l'autre au redémarrage, ce qui rend les mises à jour ratées beaucoup moins angoissantes.

## Ce qui est intéressant

- Rollback quasi instantané en cas de mise à jour problématique, simplement en choisissant l'ancien déploiement au démarrage
- Les applications graphiques passent par Flatpak, ce qui les isole proprement du système de base
- Pour le développement, Toolbox (des conteneurs podman légers) permet d'avoir un environnement mutable sans toucher à l'hôte
- Deux machines avec le même déploiement ostree sont, par construction, strictement identiques

## Cas d'usage

Idéal pour qui veut un poste de travail stable et reproductible sans sacrifier les nouveautés de Fedora. Installer un paquet système supplémentaire se fait avec :

```
rpm-ostree install nom-du-paquet
```

suivi d'un redémarrage pour activer le nouveau déploiement.

## Avis en bref

Le changement d'habitudes est réel au début, notamment pour les paquets système, mais une fois le réflexe Flatpak/Toolbox acquis, la tranquillité d'esprit apportée par l'immuabilité se ressent vite au quotidien.
