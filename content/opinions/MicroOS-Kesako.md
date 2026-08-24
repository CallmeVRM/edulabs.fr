---
title: "MicroOS ? En 2026 tu dois abosulement savoir ce que c'est"
date: 2026-08-24
tags:
  - os-immuable
  - atomic os
  - opensuse
  - microos
---

### Introduction :

Il existe un principe que presque tout le monde considère comme normal, au point que personne ne le remet en question : pourquoi acceptons-nous qu'un simple apt upgrade puisse, un jour, rendre notre système inutilisable ?

On a finit par considéré ça comme une fatalité. Au mieux, on backup avant de patcher, souvent, on laisse des environnements sans mise à jour par peur de les casser. openSuse ont parmis tant d'autres remis en question ce principe, avec microOS ils ont simplement refuser cette "fausse fatalité" !

Dans cette article j'aborde l'OS immuable sous le contexte de composants edge utilisant microOS.

### **Un problème tellement récurrent qu'il est devenu normal et accepté comme tel :**

Je mets à jour des composants en production sur des environnements critiques ; je n’ai clairement pas besoin de saut en parachute pour ressentir des sensations fortes.

Les distributions classiques partagent un même modèle de fonctionnement depuis des décenies, un état mutable, on installe des paquets, on les mets à jour, on les supprime, le tout directement sur le filesystème en production, et chaque opération modifie l'état actuel de la machine, instantanément, sans filet de sécurité.

Ce modèle bien que simple au début, a un coût invisible qui s'accumule avec le temps : le configuration drift. Après deux ans de patchs manuels, de paquets installés "pour dépanner un truc, un vendredi soir" plus aucune machine de votre parc ne ressemble vraiment à une autre et une mise à jour peut fonctionner sur 9 et foiré la sur la 10 ème. Le pire c'est le retour en arrière, impossible sur des environnements non virtualisé (équipements edge par ex) !

### **L'idée de MicroOS : rendre le système en lecture seule**

Pour mettre les points sur les i, openSuse n'ont pas inventé le système immuable bien évidemment, mais c'est européen donc mettons le en valeur.

MicroOS est basé sur Tumbleweed, c'est une distribution immuable, ce que veut dire que la racine / est monté en lecture seule pendant le fonctionnement normal du système. Vous ne pouvez tout simplement pas modifier le sytème en place, même par erreur, même en root...

Toute évolution du système passe par un mécanisme unique : transactional-update.

Voici ce qui se passe concrète quand vous demandez une mise à jour :

1. Le système Btrfs crée un nouveau snapshot à partir de l'état actuel.
2. Les changement (patchs, nouveaux paquets) sont appliqués dans ce snapshot, pas sur le système actif.
3. Au prochain redémarrage, GRUB bascule sur ce nouveau snapshot
4. Le système précédent reste intact, disponible, sélectionnable au boot.

Concrètement, cela veut dire qu'une mise à jour qui casse un driver réseau, un service critique ou une dépendance, ne se traduit jamais par un système en panne. Bam reboot, et retour vers le snapshot d'avant. Pas de procédure de disaster reconvery poussièreuse, mais une fonctionnalité de base qui permet de revenir en arrière simplement.

Tout cela biensur est basé sur le Btrfs, qui permet du faire copy-on-write (xfs le fait aussi), ça rend viable l'idée de garder plusieurs générations complète du système en parallèle sans exploser le stockage !

### **Bon, le prochaine point est chiant au quotidien, mais nécessaire !**

Sur une distribution classique vous faites apt install <paquet> et hop le paquet est installé et vous pouvez le configurer et l'utiliser quasi instantanément. Pas sur MicroOS !

Il propose deux pistes :

1. transactional-update pkg install <paquet> : ajoute le paquet dans un nouveau snapshot, mais ne sera effectif qu'après redémarrage. ça concerne tout ce qu'il doit vivre au niveau du système (drivers, outils bas niveau, dépendance kernel).
2. La conteneurisation : pour moi c'est podman pour tout le reste, podman est un choix dogmatique que je n'ai pas envie d'étaler içi (ce n'est pas le sujet hein !)

C'est un peu la philosophie de ce type d'OS, il y a le système et les applications, deux mondes différents, alors séparons les pour qu'ils vivent en paix !

### Rien n'est gratuit :

1. Chaque changement système nécessiste un reboot pour être appliqué (on peut chainer plusieurs installations avant un seul redémarrage avec --continue, mais le reboot reste incontournable si on veut utiliser ce qu'on a installer !
2. Le début à chaud est plus .... disons rigide : Réfléchir avant d'agir, je dois installer strace pour le debug, transctional-update pkg install strace + reboot ! Si tu veux ensuite installer tcpdump, pareil, install + reboot !
3. De la discipline pour le matériel bas niveau : Modules kernel tiers, Drivers hors arbre officiel, des outils spéficiques, doivent être intégrés proprement !
4. La gestion des fichiers config demande de comprendre ce qui survit aux transcations, /etc par exemple bénéficie d'un overlay, /var est persitant, le / est en lecture seule .

### Mais on gagne en production :

1. La cohérance de parc devient quasi automatique, on peut toujours bricoler, mais ça limite fortement ce phénomène.
2. La surface d'attaque diminue mécaniquement, un attaquant qui compromet un service applicatif ne peut pas persister en modifiant le sytème de fichiers racine, il est en lecture seule ! Ce n'est pas une garantie absolue, mais ça élimine une partie importante du risque.
3. Le rollback devient un réfglexe, pas une opération çà risque : Sur un système mutable, revenir en arrière après une mauvaise mise à jour implique soiuvent une restautration de sauvegarde complète, surtout pour des composants baremetal (donc non virtualisé), il faut potentiellement un déplacement phyusiuque et un certain  temps pour remettre la machine en marche. Revenir en arrière c'est un reboot sur MicroOS.

### Un changement de paradigme, pas juste une distribution de plus

MicroOS ne cherche pas à remplacer Debian ou Ubuntu sur toutes les tâches. C'est un outil pensé pour un usage précis : des machines dont le rôle est stable et bien défini (hôte de conteneurs, routeur, nœud d'infrastructure), où la fiabilité et la reproductibilité comptent plus que la flexibilité d'installation à la volée.

Le vrai apport, au fond, n'est pas technique, je dirais qu'il est culturel. MicroOS force à repenser la frontière entre "ce qui est système" et "ce qui est application", et à accepter que la stabilité d'une infrastructure ne vient pas de l'absence de changement, mais de la capacité à changer sans jamais pouvoir casser irrémédiablement ce qui fonctionnait la veille.

Dans un monde où on répète que l'infrastructure doit être "immutable" et "as code", MicroOS pousse cette idée jusqu'au système d'exploitation lui-même. Ce n'est pas un hasard si elle trouve un terrain naturel sur les infrastructures conteneurisées et les environnements Kubernetes — des mondes qui ont déjà fait le deuil de la machine qu'on bricole à la main.
