---
title: "s6 et s6-rc : la suite logique"
weight: 2
---

## Présentation

s6 pousse l'idée de runit encore plus loin, avec une attention presque obsessionnelle portée à la robustesse et à la correction des cas limites : gestion propre des signaux, absence de conditions de concurrence dans le démarrage et l'arrêt des services, et une séparation nette entre la supervision de processus (s6) et la gestion des dépendances entre services (s6-rc).

## Ce qui est intéressant

- Une gestion explicite des dépendances entre services via s6-rc, avec un ordre de démarrage et d'arrêt calculé plutôt que déduit implicitement
- Des primitives de bas niveau (s6-svscan, s6-supervise) pensées pour être composées plutôt qu'utilisées telles quelles
- Une attention documentée aux conditions de concurrence lors des transitions d'état, un point souvent négligé ailleurs
- Utilisé comme brique de supervision dans des distributions comme Obarun ou dans des images de conteneurs minimalistes

## Cas d'usage

Pertinent pour qui construit un système ou une image de conteneur où la fiabilité du démarrage compte plus que la simplicité de configuration. La compilation d'une base de services se fait par exemple avec :

```
s6-rc-compile compiled/ source/
```

## Avis en bref

s6 demande un investissement d'apprentissage plus important que runit, mais pour qui construit une infrastructure où chaque détail du cycle de vie des processus compte, c'est un des projets les plus rigoureux du domaine.
