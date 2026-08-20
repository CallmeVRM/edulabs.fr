---
title: "Wayland est-il vraiment prêt pour tout le monde ?"
date: 2026-04-30
tags:
  - wayland
  - x11
  - bureau
---

Après avoir basculé mon poste principal sous Wayland il y a quelques mois, je peux enfin répondre sans faire de politique : oui, presque. Et ce « presque » cache encore de vraies aspérités.

<!--more-->

Ce qui fonctionne bien, et même très bien : le rendu est plus fluide, la gestion du multi-écran avec des résolutions mixtes n'a plus rien à voir avec les bricolages d'antan sous X11, et la sécurité par défaut (chaque application ne peut plus espionner les autres à volonté) est un vrai progrès qu'on sous-estime trop souvent dans les comparatifs.

Ce qui coince encore : le partage d'écran dans certaines applications de visioconférence reste capricieux selon le compositeur utilisé, quelques outils historiques de capture ou de gestion des fenêtres n'ont jamais été portés, et les pilotes propriétaires NVIDIA, même après les progrès récents, restent le point de friction numéro un cité par tous ceux qui hésitent encore à migrer.

Mon avis, pour ce qu'il vaut : sur du matériel récent avec des pilotes libres (Intel, AMD), la bascule est aujourd'hui indolore et je ne reviendrais pas en arrière. Sur une configuration NVIDIA un peu datée ou avec des besoins de capture d'écran spécifiques, X11 garde encore, pour l'instant, un vrai rôle à jouer.
