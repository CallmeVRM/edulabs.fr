---
title: "Une interface réseau reste à l'état DOWN"
weight: 1
---

## Symptômes

Après un redémarrage ou le branchement d'un câble, l'interface `enp3s0` (ou équivalent) reste affichée en état `DOWN` dans la sortie de `ip link`, aucune adresse IP n'est attribuée et aucun trafic ne circule. Le voyant du port sur le commutateur reste éteint alors que le câble est visiblement branché aux deux extrémités.

## Cause probable

Les causes les plus fréquentes sont un pilote de carte réseau qui n'a pas été chargé correctement après une mise à jour du noyau, un câble ou un port physiquement défectueux, ou une politique de gestion réseau (NetworkManager, systemd-networkd) qui ne prend pas en charge l'interface parce qu'aucun profil de connexion ne lui est associé. Un problème d'alimentation sur une carte PCIe mal fixée peut aussi expliquer une absence totale de négociation de lien.

## Diagnostic

```bash
ip link show enp3s0
ethtool enp3s0
dmesg | grep -i enp3s0
nmcli device status
```

`ethtool` indique si un lien physique est détecté (`Link detected: yes/no`) indépendamment de la configuration logicielle. Si le lien physique n'est pas détecté, le problème est matériel ; s'il l'est mais que l'interface reste DOWN, la cause est plus probablement logicielle ou liée à la gestion des connexions.

## Résolution

1. Si `ethtool` ne détecte aucun lien, tester un autre câble puis un autre port sur le commutateur avant toute intervention logicielle.
2. Forcer l'activation de l'interface avec `ip link set enp3s0 up` et observer si elle passe en `UP` puis retombe.
3. Vérifier les messages du noyau autour du chargement du pilote (`dmesg | grep -i eth` ou le nom du pilote concerné) et recharger le module si nécessaire avec `modprobe -r <module> && modprobe <module>`.
4. Si NetworkManager gère l'interface, créer ou réactiver un profil de connexion avec `nmcli device connect enp3s0`.
