---
title: "Le cloud nous a-t-il fait oublier l'administration système ?"
date: 2026-07-12
tags:
  - cloud
  - administration
  - infrastructure
---

Je forme régulièrement des jeunes ingénieurs qui savent écrire un manifeste Kubernetes impeccable mais qui n'ont jamais eu à diagnostiquer une charge CPU anormale avec `top` ou à comprendre pourquoi un service ne redémarre pas après un crash. Ce n'est pas un jugement, c'est un constat qui m'inquiète un peu.

<!--more-->

Le cloud a rendu énormément de choses plus simples, et c'est tout à son mérite : provisionner une machine en quelques secondes, faire du scaling automatique, déléguer la gestion du stockage et du réseau à un fournisseur qui a des équipes dédiées à ça. Mais cette simplicité a un coût caché : quand l'abstraction fuit — et elle fuit toujours, un jour ou l'autre — il faut redescendre au niveau du système, et c'est précisément la compétence qui s'érode le plus vite chez les nouvelles générations d'administrateurs.

J'ai vu des incidents de production traités à coup de redémarrage de pod en boucle, alors que le vrai problème était une fuite mémoire noyau parfaitement diagnosticable avec `dmesg` et un peu de patience. Le réflexe cloud-natif, quand il n'est pas accompagné de bases système solides, devient un pansement qui masque le symptôme sans jamais toucher à la cause.

Mon avis : le cloud n'est pas le problème, notre façon de former les gens qui l'utilisent l'est. Comprendre ce qui se passe sous l'abstraction — le noyau, les processus, le réseau — reste, à mes yeux, un prérequis et non une option, même pour quelqu'un qui ne touchera jamais un serveur physique de sa carrière.
