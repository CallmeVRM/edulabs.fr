---
title: "CFS contre EEVDF : deux philosophies d'ordonnancement"
weight: 1
---

## Contexte

Depuis la version 6.6, le noyau Linux remplace progressivement le Completely Fair Scheduler (CFS) par EEVDF (Earliest Eligible Virtual Deadline First) comme ordonnanceur par défaut pour les tâches normales. Cette bascule mérite qu'on s'y attarde : CFS a structuré vingt ans de comportements d'ordonnancement, et un changement de cette ampleur affecte potentiellement toute charge sensible à la latence.

## Analyse technique

CFS repose sur un temps virtuel (`vruntime`) maintenu dans un arbre rouge-noir : la tâche avec le plus petit `vruntime` est toujours élue. Le mécanisme est simple, mais il traite mal les tâches qui alternent rapidement entre calcul et attente (les charges dites "latency-sensitive"), car il ne raisonne qu'en termes de partage équitable du temps CPU, pas d'échéance.

EEVDF introduit une notion de deadline virtuelle par tâche, calculée à partir d'un poids et d'une durée de tranche demandée. Une tâche devient "éligible" quand son `vruntime` dépasse la moyenne pondérée du groupe d'exécution, et parmi les tâches éligibles, celle dont la deadline est la plus proche est choisie. Cela permet de mieux honorer les tâches qui demandent explicitement de petites tranches (via `sched_setattr` et le champ `sched_runtime`), sans sacrifier l'équité globale.

```c
/* extrait conceptuel : condition d'éligibilité EEVDF */
if (se->vruntime >= cfs_rq->avg_vruntime)
        entity_is_eligible = true;
```

## Observations

Sur une charge mixte (serveur web + tâches batch de compression) testée sur un noyau 6.8 :

- la latence de réveil (`wakeup latency`) des requêtes web diminue d'environ 15 à 20 % par rapport à un noyau 6.5 sous CFS, dans des conditions de charge comparables ;
- le débit global des tâches batch reste stable, sans régression notable ;
- le comportement devient plus prévisible sous forte contention, car la notion de deadline limite les cas de starvation observés occasionnellement avec CFS sur des groupes `cgroup` très déséquilibrés.

## Pour aller plus loin

Il serait intéressant de comparer EEVDF à `sched_ext` (BPF schedulers) sur les mêmes charges, pour voir si un ordonnanceur programmable spécialisé fait mieux qu'un ordonnanceur générique optimisé.
