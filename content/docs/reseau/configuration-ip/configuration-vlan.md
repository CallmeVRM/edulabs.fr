---
title: "Configurer un VLAN sur une interface"
weight: 2
---

## Objectif

Segmenter le trafic réseau en créant une interface virtuelle taguée (VLAN 802.1Q) au-dessus d'une interface physique, afin d'isoler différents flux (production, administration, invités...) sur un même câblage.

## Prérequis

- Un commutateur configuré pour transporter le tag VLAN souhaité sur le port concerné (mode trunk)
- Module noyau `8021q` chargé
- Droits `sudo`

## Étapes

1. Charger le module VLAN si nécessaire.
2. Créer l'interface VLAN au-dessus de l'interface physique en précisant l'identifiant.
3. Attribuer une adresse IP à cette nouvelle interface.
4. Activer l'interface.

```bash
sudo modprobe 8021q
sudo ip link add link eth0 name eth0.20 type vlan id 20
sudo ip addr add 10.0.20.5/24 dev eth0.20
sudo ip link set eth0.20 up
```

Ici, `eth0.20` reçoit le trafic tagué avec l'identifiant VLAN 20 depuis l'interface physique `eth0`, qui doit rester elle-même active. Pour rendre la configuration persistante après un redémarrage, il est préférable de la déclarer dans Netplan ou dans le gestionnaire réseau de la distribution plutôt que de répéter ces commandes manuellement.

Exemple équivalent avec Netplan :

```yaml
network:
  version: 2
  vlans:
    eth0.20:
      id: 20
      link: eth0
      addresses: [10.0.20.5/24]
```

## Vérification

Vérifiez que l'interface VLAN est active avec la bonne adresse, et que le tag est correctement reconnu :

```bash
ip -d link show eth0.20
ping -c 3 10.0.20.1
```

La sortie de `ip -d link show` doit mentionner `vlan protocol 802.1Q id 20`, confirmant que l'interface transporte bien le tag attendu.
