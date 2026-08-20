---
title: "ripgrep et fd : la recherche réinventée"
weight: 1
---

## Présentation

ripgrep (`rg`) et fd sont deux outils écrits en Rust qui remplacent respectivement grep et find. Ils reprennent l'esprit des commandes historiques mais avec des valeurs par défaut pensées pour l'usage courant : ignorer automatiquement les fichiers listés dans `.gitignore`, sauter les fichiers binaires, et paralléliser la recherche sur plusieurs cœurs.

## Ce qui est intéressant

- ripgrep ignore par défaut les répertoires `.git`, `node_modules` et tout ce qui est listé dans `.gitignore`, ce qui évite de fouiller dans du bruit
- Une vitesse de recherche nettement supérieure à grep sur de gros arbres de fichiers, grâce à un moteur d'expressions régulières optimisé
- fd propose une syntaxe bien plus courte que find, avec une recherche récursive et une coloration des résultats par défaut
- Les deux outils gèrent nativement l'Unicode et l'UTF-8 sans configuration supplémentaire

## Cas d'usage

Utile dès qu'on cherche du texte ou des fichiers dans un projet de taille respectable. Par exemple, chercher toutes les occurrences d'une fonction en ignorant le dossier de build :

```
rg "ma_fonction" --type rust
```

## Avis en bref

Une fois qu'on a goûté à la rapidité et aux valeurs par défaut sensées de ripgrep et fd, revenir à grep et find nu paraît presque punitif. Deux outils qui méritent largement leur place dans les alias du quotidien.
