---
title: "screen est-il encore utile en 2026 ?"
weight: 2
---

## Présentation

GNU Screen est l'ancêtre des multiplexeurs de terminal modernes, présent par défaut sur une quantité de systèmes depuis des décennies. Face à tmux et Zellij, on pourrait le croire dépassé, mais il reste étonnamment pertinent dans certains contextes précis, notamment grâce à sa disponibilité quasi universelle.

## Ce qui est intéressant

- Présent par défaut sur énormément de distributions et d'images de conteneurs, sans installation supplémentaire
- Une syntaxe de commandes minimaliste, largement suffisante pour l'usage le plus courant : lancer une tâche longue et s'en détacher
- Une empreinte mémoire très faible, appréciable sur du matériel ancien ou des environnements très contraints
- Une stabilité éprouvée par des années d'usage en production sur des serveurs critiques

## Cas d'usage

Toujours pertinent pour lancer rapidement une tâche longue sur un serveur distant sans se soucier de l'installation d'un outil tiers :

```
screen -S ma-tache
```

puis `Ctrl-a d` pour se détacher et `screen -r ma-tache` pour reprendre.

## Avis en bref

screen a clairement perdu la bataille des fonctionnalités face à tmux, mais sa disponibilité par défaut presque partout en fait encore un filet de sécurité précieux quand on ne peut rien installer.
