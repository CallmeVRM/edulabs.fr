---
title: "fish : un shell pensé pour l'utilisateur"
weight: 1
---

## Présentation

fish (friendly interactive shell) part d'un postulat différent de bash ou zsh : plutôt que de viser une compatibilité stricte avec POSIX, il privilégie l'expérience interactive dès l'installation, sans configuration préalable. Autocomplétion intelligente, coloration syntaxique en temps réel et suggestions basées sur l'historique sont actives par défaut.

## Ce qui est intéressant

- Coloration syntaxique en direct pendant la frappe, qui signale par exemple une commande inconnue avant même de valider
- Suggestions automatiques tirées de l'historique, affichées en grisé et validables avec une simple flèche droite
- Une syntaxe de script volontairement simplifiée par rapport à bash, notamment pour les conditions et les boucles
- Une autocomplétion des options de commande générée automatiquement à partir des pages de manuel

## Cas d'usage

Très adapté à un usage interactif quotidien, moins pensé pour l'écriture de scripts destinés à être portables. Un exemple de condition en fish :

```fish
if test -f fichier.txt
    echo "présent"
end
```

## Avis en bref

fish assume de ne pas être POSIX et cela se voit dans la qualité de l'expérience interactive, nettement plus agréable dès la première utilisation que bash sans configuration additionnelle.
