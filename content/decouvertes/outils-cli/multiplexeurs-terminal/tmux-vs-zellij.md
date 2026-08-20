---
title: "tmux contre Zellij : deux approches du multiplexage"
weight: 1
---

## Présentation

tmux est depuis longtemps la référence pour multiplexer un terminal : plusieurs fenêtres, plusieurs panneaux, des sessions qui survivent à la déconnexion. Zellij est un projet plus récent, écrit en Rust, qui reprend la même idée mais avec une interface plus découvrable, des indications de raccourcis affichées à l'écran et une configuration en KDL plutôt qu'un langage de commandes maison.

## Ce qui est intéressant

- tmux bénéficie d'un écosystème immense de plugins et de scripts accumulés depuis des années
- Zellij affiche par défaut une barre de raccourcis contextuelle, ce qui réduit fortement la courbe d'apprentissage
- Zellij propose des "layouts" déclaratifs pour définir l'agencement d'un projet et le retrouver identique à chaque lancement
- Les deux permettent de détacher une session et de la retrouver plus tard, y compris après une déconnexion SSH

## Cas d'usage

tmux reste pertinent partout où sa présence est déjà acquise (serveurs distants, scripts existants). Une commande tmux typique :

```
tmux new -s travail
```

Zellij convainc davantage sur un poste personnel où l'on peut choisir son outil librement.

## Avis en bref

tmux garde l'avantage de l'omniprésence et de la maturité, mais Zellij donne un aperçu convaincant de ce à quoi un multiplexeur de terminal pourrait ressembler s'il était conçu aujourd'hui.
