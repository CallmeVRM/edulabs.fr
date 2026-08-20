---
title: "Détection de secteurs défectueux via SMART"
weight: 2
---

## Symptômes

Le système ralentit ponctuellement lors d'accès disque, avec parfois de courtes pauses de plusieurs secondes sur des opérations de lecture ou d'écriture pourtant simples. Le journal noyau contient des messages d'erreurs I/O (`I/O error`, `ata error`) sur le périphérique concerné, et certaines applications signalent des erreurs de lecture de fichiers qui fonctionnaient normalement la veille.

## Cause probable

Le disque physique présente des secteurs défectueux détectés par sa surveillance interne SMART. Ces secteurs, une fois identifiés par le firmware du disque, sont normalement réalloués vers une zone de réserve, mais un nombre croissant de réallocations est un signal fort de dégradation mécanique progressive, en particulier sur un disque mécanique (HDD) approchant ou dépassant sa durée de vie théorique.

## Diagnostic

```bash
smartctl -a /dev/sda
smartctl -H /dev/sda
smartctl -l selftest /dev/sda
journalctl -k | grep -i "ata1\|I/O error"
```

Les attributs `Reallocated_Sector_Ct`, `Current_Pending_Sector` et `Offline_Uncorrectable` dans la sortie de `smartctl -a` sont les indicateurs clés : une valeur non nulle et croissante dans le temps confirme une dégradation active plutôt qu'un incident isolé.

## Résolution

1. Lancer un autotest complet du disque avec `smartctl -t long /dev/sda` puis consulter le résultat après quelques heures avec `smartctl -l selftest /dev/sda`.
2. Sauvegarder immédiatement les données critiques présentes sur ce disque, indépendamment du résultat du test, dès que des secteurs réalloués ou en attente sont détectés.
3. Si le nombre de secteurs défectueux continue de croître entre deux relevés, planifier le remplacement physique du disque plutôt qu'une simple surveillance continue.
4. En attendant le remplacement, envisager de migrer les données vers un autre support via `dd` ou un outil de clonage tolérant aux erreurs (`ddrescue`) pour limiter les pertes en cas de défaillance totale.
