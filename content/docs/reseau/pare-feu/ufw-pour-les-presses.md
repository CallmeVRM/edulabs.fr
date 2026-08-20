---
title: "UFW pour configurer un pare-feu rapidement"
weight: 2
---

## Objectif

UFW (Uncomplicated Firewall) simplifie la gestion du pare-feu en fournissant une syntaxe simple par-dessus nftables ou iptables. Ce guide couvre la mise en place d'une politique minimale : tout bloquer en entrée, autoriser explicitement quelques services.

## Prérequis

- Paquet `ufw` installé (présent par défaut sur Ubuntu desktop et serveur)
- Droits `sudo`
- Liste des ports à ouvrir (SSH, HTTP, HTTPS...)

## Étapes

1. Définir les politiques par défaut : refuser en entrée, autoriser en sortie.
2. Autoriser le SSH avant toute autre chose, pour ne pas perdre l'accès distant.
3. Autoriser les ports HTTP et HTTPS si un service web tourne sur la machine.
4. Activer le pare-feu.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

`ufw allow OpenSSH` s'appuie sur un profil d'application préenregistré, équivalent à `allow 22/tcp`, mais plus lisible. Il est possible de limiter une règle à une IP source précise avec `ufw allow from 192.168.1.0/24 to any port 22`, ce qui est utile pour restreindre l'accès SSH à un réseau d'administration.

## Vérification

Affichez l'état du pare-feu et la liste des règles numérotées :

```bash
sudo ufw status verbose
sudo ufw status numbered
```

La sortie doit indiquer `Status: active`, la politique par défaut `deny (incoming), allow (outgoing)`, ainsi que les règles ouvertes pour SSH, HTTP et HTTPS dans l'ordre où elles ont été ajoutées.
