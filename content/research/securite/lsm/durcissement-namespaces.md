---
title: "Isolation par namespaces : jusqu'où aller ?"
weight: 2
---

## Contexte

Les namespaces Linux forment le socle de l'isolation des conteneurs, mais le degré d'isolation réellement obtenu varie beaucoup selon la combinaison utilisée. Cette note fait le point sur les namespaces disponibles et sur les limites concrètes de cette isolation, en particulier vis-à-vis du noyau partagé.

## Analyse technique

Linux propose actuellement huit types de namespaces : PID, réseau, montage, UTS, IPC, utilisateur, cgroup et, plus récemment, temps (`time namespace`). Chacun isole une facette différente : le namespace PID donne à un processus l'illusion d'être le PID 1 d'un système isolé, le namespace réseau lui donne sa propre pile réseau complète, le namespace utilisateur permet de mapper un UID root dans le conteneur vers un UID non privilégié côté hôte.

La limite structurelle est que tous les conteneurs partagent le même noyau. Un namespace ne cloisonne pas les appels système eux-mêmes : une vulnérabilité dans le traitement noyau d'un appel système reste exploitable pour sortir du confinement, quel que soit le nombre de namespaces empilés. C'est pour cette raison que le filtrage `seccomp` et les LSM viennent compléter les namespaces plutôt que s'y substituer.

```bash
# lister les namespaces d'un processus
lsns -p 1234

# lancer un processus avec un namespace utilisateur mappé
unshare --map-root-user --user --pid --fork bash
```

## Observations

En testant une combinaison complète de namespaces (PID, réseau, montage, utilisateur) sur un processus non privilégié :

- l'isolation vis-à-vis du système de fichiers hôte et de la table de processus s'est révélée efficace pour empêcher toute observation directe depuis le conteneur ;
- le mappage UID via le namespace utilisateur a bien empêché une élévation de privilège via un binaire setuid présent dans l'image, celui-ci restant borné à l'UID mappé côté hôte ;
- aucune combinaison de namespaces n'a permis, dans ce test, de compenser l'absence d'un profil `seccomp` restreignant les appels système disponibles, confirmant que les deux mécanismes sont complémentaires et non substituables.

## Pour aller plus loin

Il resterait à évaluer l'apport des machines virtuelles légères (Kata Containers, gVisor) qui ajoutent une couche d'isolation matérielle ou d'émulation d'appels système, pour les charges où le partage de noyau est jugé insuffisant.
