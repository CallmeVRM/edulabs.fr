---
title: "Alpine Linux : légèreté et musl"
weight: 1
---

## Présentation

Alpine Linux s'est fait connaître comme l'image de base d'une bonne partie des conteneurs Docker, mais c'est aussi une distribution complète et utilisable directement sur une machine. Elle remplace glibc par musl libc et utilise BusyBox pour la plupart des utilitaires système, ce qui donne une empreinte disque minuscule.

## Ce qui est intéressant

- Une image de base pesant seulement quelques mégaoctets, très appréciée pour les conteneurs
- Le gestionnaire de paquets apk, rapide et simple, avec un dépôt bien fourni malgré la taille du système
- Un modèle de sécurité orienté durcissement par défaut, avec PIE et SSP activés sur l'ensemble des paquets
- OpenRC comme système d'init, plus lisible que systemd pour qui veut comprendre chaque étape du démarrage

## Cas d'usage

Parfait pour des conteneurs légers ou des serveurs à ressources limitées. Installer un paquet se résume à :

```
apk add nom-du-paquet
```

Attention cependant : certains logiciels compilés en supposant glibc peuvent nécessiter des ajustements avec musl.

## Avis en bref

Alpine tient une promesse rare : rester minuscule sans devenir pénible à utiliser. Un excellent choix par défaut pour les conteneurs, à condition de vérifier la compatibilité musl des logiciels les plus exotiques.
