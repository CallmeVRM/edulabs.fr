---
title: "Flatpak, Snap, AppImage : le chaos des formats universels"
date: 2026-06-05
tags:
  - flatpak
  - snap
  - appimage
  - paquets
---

On nous promettait, avec les formats de paquets universels, la fin du morcellement entre distributions. Une application, un fichier, n'importe quelle distribution. Sur le papier, magnifique. Dans les faits, on a surtout ajouté une couche de complexité au-dessus de l'ancienne.

<!--more-->

Flatpak s'est imposé sur le bureau grâce à Flathub et à un bon compromis entre isolation et intégration, mais chaque application embarque ses propres runtimes, ce qui gonfle l'espace disque utilisé et complique le débogage quand quelque chose ne fonctionne pas comme prévu. Snap fait un pari similaire côté Canonical, avec en prime une intégration parfois trop appuyée dans Ubuntu, ce qui a fini par agacer une partie de la communauté. AppImage, de son côté, séduit par sa simplicité — un fichier exécutable, aucune installation — mais laisse à l'utilisateur la charge des mises à jour et de l'intégration au bureau, ce qui n'est pas rien pour un public non technique.

Résultat : au lieu d'un standard unique, on a maintenant trois écosystèmes qui coexistent, chacun avec ses forces, et un utilisateur qui doit encore choisir. La fragmentation qu'on voulait éliminer entre distributions s'est simplement déplacée d'un cran plus haut.

Je ne crois pas qu'un des trois formats « gagnera » à court terme. Je pense plutôt qu'on va continuer à vivre avec cette coexistence, et que le vrai progrès viendra plutôt d'une meilleure interopérabilité entre eux que d'une hypothétique unification.
