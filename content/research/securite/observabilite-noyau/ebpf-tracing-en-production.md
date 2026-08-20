---
title: "Tracer un système en production avec eBPF"
weight: 1
---

## Contexte

eBPF permet d'attacher du code vérifié directement à des points d'accroche du noyau (appels système, fonctions internes, événements réseau) sans module noyau custom ni redémarrage. L'intérêt en production est évident, mais la promesse d'un traçage "sans risque ni surcoût" mérite d'être vérifiée avant de généraliser son usage sur des systèmes critiques.

## Analyse technique

Un programme eBPF est écrit en C restreint, compilé en bytecode, puis vérifié par le vérificateur du noyau avant chargement : celui-ci s'assure que le programme termine (pas de boucle non bornée), qu'il n'accède pas à de la mémoire hors limites, et qu'il respecte les types de données attendus aux points d'accroche utilisés. Une fois validé, le bytecode est en général compilé en JIT vers du code machine natif, ce qui limite fortement le surcoût par rapport à une solution de traçage interprétée.

Les points d'accroche courants incluent les kprobes (fonctions internes du noyau), les tracepoints (points d'instrumentation stables définis par le noyau lui-même) et les uprobes (fonctions d'un binaire utilisateur). Les tracepoints sont généralement préférés en production car leur interface reste stable entre versions de noyau, contrairement aux kprobes qui dépendent de noms de fonctions internes pouvant changer sans préavis.

```bash
# tracer les ouvertures de fichiers avec bpftrace
bpftrace -e 'tracepoint:syscalls:sys_enter_openat 
  { printf("%s %s\n", comm, str(args->filename)); }'
```

## Observations

En instrumentant un serveur applicatif en production avec un script `bpftrace` ciblant les appels `openat` et `connect` :

- le surcoût CPU mesuré sur la durée de la session de traçage est resté sous 2 %, en cohérence avec la réputation de légèreté d'eBPF par rapport à `strace` ;
- l'usage de tracepoints plutôt que de kprobes a permis de réutiliser le même script sans modification après une montée de version du noyau ;
- la principale difficulté rencontrée a été la volumétrie des événements sur un système très actif, nécessitant un filtrage par PID ou par commande dès l'écriture du script pour rester exploitable.

## Pour aller plus loin

Il serait pertinent d'évaluer des outils construits au-dessus d'eBPF comme Cilium Tetragon ou Falco, qui packagent des règles de détection prêtes à l'emploi plutôt que d'écrire des scripts `bpftrace` ad hoc à chaque investigation.
