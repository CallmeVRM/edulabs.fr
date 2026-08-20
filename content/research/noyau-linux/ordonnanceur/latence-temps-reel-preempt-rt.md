---
title: "Réduire la latence avec PREEMPT_RT"
weight: 2
---

## Contexte

Le patch PREEMPT_RT, fusionné dans le noyau mainline à partir de la version 6.12, transforme Linux en système capable de garanties temps réel dur. Cette note documente une première prise en main sur un banc de test, dans l'optique d'évaluer si l'infrastructure actuelle pourrait bénéficier d'un basculement vers ce mode pour des charges sensibles à la jitter.

## Analyse technique

PREEMPT_RT rend préemptibles la quasi-totalité des sections critiques du noyau, y compris une bonne partie de ce qui était protégé par des spinlocks classiques. Concrètement, les spinlocks deviennent des mutex "rt" qui peuvent céder le CPU, et les gestionnaires d'interruption sont convertis en threads noyau (`threaded irqs`) ordonnancés comme des tâches normales, avec une priorité configurable.

Le compromis est clair : on échange du débit contre de la prévisibilité. Chaque conversion de spinlock introduit un potentiel changement de contexte supplémentaire, ce qui augmente le coût moyen des opérations noyau tout en réduisant leur pire cas.

```bash
# vérifier si PREEMPT_RT est actif
cat /sys/kernel/realtime
# 1 si le noyau RT est chargé

# fixer la priorité d'un thread d'interruption
chrt -f -p 80 $(pgrep -f irq/44-eth0)
```

## Observations

Sur un banc équipé d'une carte réseau à faible latence et d'une charge de traitement de paquets en boucle serrée :

- la latence maximale observée (`cyclictest`) passe d'environ 800 microsecondes sous un noyau générique à moins de 40 microsecondes sous PREEMPT_RT ;
- le débit brut de traitement diminue d'environ 5 % en charge nominale ;
- la configuration des priorités d'IRQ threadées demande une vraie discipline : une priorité mal placée peut inverser les gains de latence sur d'autres flux.

## Pour aller plus loin

Une piste naturelle est de combiner PREEMPT_RT avec l'isolation de CPU (`isolcpus`, `nohz_full`) pour voir si les gains se cumulent ou si l'un des deux mécanismes domine largement l'autre.
