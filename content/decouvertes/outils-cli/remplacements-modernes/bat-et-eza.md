---
title: "bat et eza : cat et ls modernisés"
weight: 2
---

## Présentation

bat et eza s'attaquent à deux commandes que tout le monde tape des dizaines de fois par jour : cat et ls. bat affiche le contenu d'un fichier avec coloration syntaxique et numéros de ligne, tandis qu'eza (une continuation communautaire d'exa) enrichit le listing de répertoires avec des icônes, des couleurs par type de fichier et une intégration git.

## Ce qui est intéressant

- bat détecte automatiquement le langage d'un fichier et applique la coloration syntaxique correspondante, comme le ferait un éditeur
- La pagination intégrée de bat s'active automatiquement pour les fichiers longs, sans avoir à penser à `less`
- eza affiche l'état git (modifié, ajouté, ignoré) directement dans le listing d'un dépôt
- eza propose une vue arborescente intégrée qui remplace avantageusement une invocation séparée de `tree`

## Cas d'usage

Pratique pour quiconque passe sa journée à lire du code ou à naviguer dans des répertoires de projet. Un alias classique :

```
alias cat=bat
alias ls="eza --icons --git"
```

## Avis en bref

Ce sont de petits outils, mais l'accumulation de détails soignés (couleurs, icônes, intégration git) change réellement le confort d'utilisation du terminal au quotidien.
