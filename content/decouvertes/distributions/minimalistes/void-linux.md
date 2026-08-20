---
title: "Void Linux et son init runit"
weight: 2
---

## Présentation

Void Linux est une distribution indépendante, construite à partir de zéro plutôt que dérivée d'une autre. Elle utilise son propre gestionnaire de paquets XBPS et remplace systemd par runit, un système d'init volontairement minimal. C'est aussi l'une des rares distributions grand public à proposer musl comme option en plus de glibc.

## Ce qui est intéressant

- XBPS, un gestionnaire de paquets binaire rapide, avec des paquets sources faciles à modifier via xbps-src
- runit comme init : quelques centaines de lignes de code, des services démarrés par de simples scripts shell dans des répertoires dédiés
- Un modèle "rolling release" sans version figée, avec des mises à jour continues plutôt que des sauts de version majeurs
- Une communauté qui documente activement les choix techniques et les alternatives à systemd

## Cas d'usage

Convient à qui veut comprendre précisément ce qui démarre sur sa machine et pourquoi. Un service runit s'active en général ainsi :

```
ln -s /etc/sv/nom-du-service /var/service/
```

et il tourne alors sous supervision continue.

## Avis en bref

Void demande de mettre un peu les mains dans le cambouis, mais c'est justement ce qui en fait une excellente distribution pédagogique pour qui veut sortir de l'écosystème systemd sans sacrifier la stabilité.
