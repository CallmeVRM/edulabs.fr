---
title: "runit : simplicité et supervision de processus"
weight: 1
---

## Présentation

runit est un système d'init et de supervision de processus né du constat que les scripts d'init traditionnels (SysV) étaient devenus fragiles et difficiles à raisonner. Son fonctionnement repose sur une idée simple : chaque service est un petit script shell exécuté en avant-plan et surveillé en permanence par un processus superviseur qui le relance s'il s'arrête.

## Ce qui est intéressant

- Une base de code volontairement réduite, entièrement lisible en une soirée pour qui veut la comprendre
- Chaque service tourne sous surveillance continue, avec redémarrage automatique en cas d'arrêt inattendu
- Le démarrage du système se décompose en trois étapes claires (stage 1, 2, 3), sans dépendances implicites cachées
- Les journaux de service peuvent être gérés simplement en chaînant le processus à un journal dédié

## Cas d'usage

Adapté à qui veut un démarrage système entièrement transparent, ou à l'administration de services applicatifs sous supervision. Un service se déclare avec un simple script `run` :

```
#!/bin/sh
exec mon-programme --option
```

placé dans son propre répertoire de service.

## Avis en bref

runit ne cherche pas à rivaliser en fonctionnalités avec systemd, et c'est précisément sa force : on sait toujours ce qui se passe, sans avoir besoin de consulter une documentation épaisse pour comprendre le démarrage du système.
