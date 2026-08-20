---
title: "systemd, le débat qui ne meurt jamais"
date: 2026-02-10
tags:
  - systemd
  - init
---

Il y a des sujets qu'on ne peut plus aborder sereinement sur un forum Linux. systemd en fait partie depuis plus de dix ans, et je commence à comprendre pourquoi le débat ne s'éteint pas : les deux camps ont raison, chacun sur un point différent.

<!--more-->

D'un côté, les pragmatiques : systemd démarre plus vite, gère proprement les dépendances entre services, journalise tout au même endroit et propose des primitives (sockets, timers, cgroups) qu'on aurait dû avoir depuis longtemps. Sur un serveur qu'on administre à dix, ce confort n'est pas négociable.

De l'autre, les puristes UNIX : un PID 1 qui absorbe la gestion des logs, du réseau, du temps et des points de montage, ça viole un principe qu'on a mis quarante ans à faire comprendre à tout le monde — un outil, une responsabilité. Et honnêtement, `journalctl` reste plus pénible à parcourir qu'un bon vieux fichier texte avec `grep`.

Ce que je retiens après des années à administrer les deux mondes : le vrai problème n'est pas systemd lui-même, c'est l'absence d'alternative crédible qui aurait pu tirer la gestion des services vers le haut sans absorber le reste. runit et s6 prouvent qu'on peut faire un init minimaliste et fiable, mais ils n'ont jamais eu la surface d'adoption nécessaire pour peser dans la discussion.

Je continuerai à utiliser systemd sur mes machines de tous les jours parce que l'écosystème s'est construit autour, mais je regarde toujours avec un peu de nostalgie les distributions qui ont fait un autre choix.
