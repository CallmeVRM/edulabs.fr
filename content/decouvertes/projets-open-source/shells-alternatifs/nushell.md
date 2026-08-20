---
title: "Nushell : quand le shell devient structuré"
weight: 2
---

## Présentation

Nushell repense plus radicalement encore la notion de shell : au lieu de faire circuler du texte brut entre les commandes comme le font bash ou fish, il fait circuler des données structurées (tables, enregistrements, listes typées). Chaque commande reçoit et produit ces structures, ce qui permet de filtrer, trier ou transformer des données sans jongler avec `awk`, `sed` ou `cut`.

## Ce qui est intéressant

- La sortie de commandes comme le listing de répertoire est une véritable table manipulable, triable et filtrable
- Les fichiers CSV, JSON ou TOML sont chargés directement comme des données structurées, sans parsing manuel
- Un système de types qui permet d'attraper certaines erreurs de script avant même l'exécution
- Multiplateforme par conception, avec un comportement identique sur Linux, macOS et Windows

## Cas d'usage

Particulièrement à l'aise pour des tâches d'exploration ou de transformation de données directement en ligne de commande :

```
ls | where size > 10mb | sort-by modified
```

## Avis en bref

Nushell demande de désapprendre certains réflexes du texte brut façon Unix, mais pour qui manipule régulièrement des données structurées, le gain d'expressivité est immédiatement palpable.
