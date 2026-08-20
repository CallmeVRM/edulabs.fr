---
title: "Configurer une adresse IP statique"
weight: 1
---

## Objectif

Attribuer une adresse IP fixe à une interface réseau, plutôt que de dépendre d'un serveur DHCP, afin de garantir la stabilité de l'adresse d'une machine (serveur, passerelle, équipement de supervision...).

## Prérequis

- Droits root ou `sudo`
- Connaître le plan d'adressage du réseau local (adresse, masque, passerelle, DNS)
- Utilitaire `netplan` (Ubuntu récent) ou fichier `ifcfg` / `NetworkManager` selon la distribution

## Étapes

1. Identifier le nom de l'interface réseau à configurer.
2. Éditer le fichier de configuration Netplan correspondant.
3. Déclarer l'adresse statique, la passerelle et les serveurs DNS.
4. Appliquer la configuration.

```bash
ip link show
sudo nano /etc/netplan/01-netcfg.yaml
sudo netplan apply
ip addr show eth0
```

Le fichier YAML doit ressembler à ceci :

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.1.50/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

Sur les distributions n'utilisant pas Netplan, la même configuration peut être posée via `nmcli con mod` puis `nmcli con up`, ou directement dans un fichier `ifcfg-eth0` sous `/etc/sysconfig/network-scripts/`.

## Vérification

Confirmez que l'adresse est bien active et que la passerelle répond :

```bash
ip addr show eth0
ping -c 3 192.168.1.1
```

L'adresse configurée doit apparaître dans la sortie de `ip addr show`, et les paquets ping vers la passerelle doivent obtenir une réponse sans perte.
