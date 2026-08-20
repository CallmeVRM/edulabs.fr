---
title: "Écrire des règles de base avec nftables"
weight: 1
---

## Objectif

nftables est le successeur d'iptables sur les noyaux Linux récents. Ce guide montre comment créer une table, une chaîne de filtrage et quelques règles simples pour autoriser le trafic essentiel tout en bloquant le reste par défaut.

## Prérequis

- Noyau Linux 3.13 ou supérieur avec le sous-système nftables activé
- Paquet `nftables` installé
- Droits root, car une règle mal écrite peut couper l'accès distant à la machine

## Étapes

1. Créer une table dédiée au filtrage IPv4/IPv6.
2. Ajouter une chaîne d'entrée avec une politique par défaut de rejet.
3. Autoriser le trafic déjà établi, le loopback et le SSH.
4. Charger la configuration.

```bash
sudo nft add table inet filtre
sudo nft add chain inet filtre entree { type filter hook input priority 0 \; policy drop \; }
sudo nft add rule inet filtre entree ct state established,related accept
sudo nft add rule inet filtre entree iif lo accept
sudo nft add rule inet filtre entree tcp dport 22 accept
```

Il est fortement recommandé de tester ces règles via une connexion console ou un accès physique avant de les appliquer sur une machine distante accessible uniquement par SSH, au cas où une erreur de syntaxe bloquerait le port 22. Pour rendre la configuration persistante, il faut l'écrire dans `/etc/nftables.conf` et activer le service `nftables` au démarrage.

## Vérification

Listez les règles actives pour confirmer qu'elles sont bien chargées dans l'ordre attendu :

```bash
sudo nft list ruleset
```

La sortie doit afficher la table `inet filtre`, la chaîne `entree` avec sa politique `drop`, suivie des règles d'acceptation pour les connexions établies, le loopback et le port 22.
