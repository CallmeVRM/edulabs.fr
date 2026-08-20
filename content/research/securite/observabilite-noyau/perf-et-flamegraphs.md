---
title: "Générer des flamegraphs avec perf"
weight: 2
---

## Contexte

Face à un processus qui consomme plus de CPU qu'attendu sans cause évidente, l'échantillonnage de piles d'appel via `perf` reste l'un des outils les plus directs pour localiser le problème. La difficulté n'est pas tant de collecter les données que de les rendre lisibles, ce qui a motivé l'usage systématique des flamegraphs plutôt que la sortie brute de `perf report`.

## Analyse technique

`perf record` échantillonne périodiquement (par défaut autour de 1000 Hz configurable) la pile d'appel de tous les threads d'un processus ou du système entier, en s'appuyant sur un compteur matériel ou temporel. Chaque échantillon capture la chaîne complète des fonctions actives à cet instant. Un flamegraph agrège ensuite tous ces échantillons en empilant les fonctions communes depuis la racine, la largeur de chaque barre représentant la proportion du temps total CPU passé dans cette fonction et ses descendants.

La lecture est intuitive une fois l'habitude prise : les fonctions larges en haut de la pile sont les points chauds à investiguer en priorité, indépendamment de la profondeur de la pile d'appel qui, elle, n'a pas de signification en termes de coût.

```bash
# capturer 30 secondes d'échantillons sur tout le système
perf record -F 999 -a -g -- sleep 30

# générer les données pliées puis le flamegraph SVG
perf script | stackcollapse-perf.pl > out.folded
flamegraph.pl out.folded > out.svg
```

## Observations

Sur un service applicatif consommant environ 40 % de CPU de plus que sur un déploiement précédent :

- le flamegraph a permis d'identifier en quelques minutes une fonction de sérialisation JSON représentant plus de 25 % du temps CPU total, invisible dans les métriques applicatives classiques ;
- la comparaison visuelle entre deux flamegraphs (avant/après un correctif) s'est révélée beaucoup plus rapide à interpréter qu'une comparaison de tableaux de `perf report` ;
- la résolution des symboles a nécessité de compiler le binaire avec les informations de debug (`-g`) et sans certaines optimisations d'inlining trop agressives, sans quoi plusieurs fonctions apparaissaient fusionnées de façon trompeuse.

## Pour aller plus loin

Une piste complémentaire serait de générer des flamegraphs différentiels directement, qui colorent les écarts entre deux captures plutôt que de les comparer côte à côte visuellement.
