---
title: "Conflit d'adresse IP sur le réseau local"
weight: 2
---

## Symptômes

Un poste perd la connectivité réseau de façon intermittente, avec des messages système du type « duplicate address detected » ou des déconnexions brèves suivies d'une reprise automatique. D'autres machines du même sous-réseau signalent parfois un comportement erratique similaire au même moment, et un `arp -a` fait apparaître deux adresses MAC différentes associées à la même IP à quelques secondes d'intervalle.

## Cause probable

Deux équipements du réseau se voient attribuer la même adresse IP, généralement parce qu'une machine a été configurée avec une IP statique qui appartient en réalité à la plage gérée par le serveur DHCP, ou parce qu'un bail DHCP expiré a été réattribué à un nouvel appareil alors que l'ancien poste, resté endormi, continue de croire qu'il détient toujours cette adresse.

## Diagnostic

```bash
ip addr show
arping -D -I enp3s0 192.168.1.50
journalctl -k | grep -i "duplicate address"
arp -a | grep 192.168.1.50
```

`arping -D` interroge activement le réseau pour savoir si une adresse est déjà utilisée par une autre machine, indépendamment de la table ARP locale déjà en cache. Un résultat positif confirme le conflit et donne l'adresse MAC de l'équipement concurrent.

## Résolution

1. Identifier physiquement l'équipement portant l'adresse MAC concurrente (recherche dans l'inventaire réseau ou sur le commutateur via sa table MAC).
2. Retirer l'adresse IP statique fautive ou la faire sortir de la plage DHCP réservée, en la déplaçant si besoin vers une plage d'exclusion dédiée.
3. Relancer un cycle DHCP propre sur les postes concernés avec `dhclient -r enp3s0 && dhclient enp3s0`.
4. Sur le serveur DHCP, envisager une réservation d'adresse par adresse MAC pour les postes qui nécessitent une IP stable, afin d'éviter toute nouvelle collision.
