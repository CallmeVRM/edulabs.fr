---
title: "Configurer le réseau d'un conteneur nspawn"
weight: 2
---

## Objectif

Par défaut, un conteneur systemd-nspawn partage l'espace réseau de l'hôte, ce qui n'est pas toujours souhaitable. Ce guide montre comment lui attribuer une interface réseau virtuelle dédiée, isolée de celle de l'hôte.

## Prérequis

- Conteneur déjà créé (voir la page « Créer un conteneur léger avec systemd-nspawn »)
- Droits root sur l'hôte
- Le module `bridge` du noyau disponible si vous souhaitez relier le conteneur à un pont existant

## Étapes

1. Démarrer le conteneur avec une interface réseau virtuelle privée (paire veth).
2. Configurer l'interface côté hôte pour lui donner accès au réseau.
3. Attribuer une adresse IP à l'interface côté conteneur.
4. Rendre la configuration persistante dans le fichier de service du conteneur.

```bash
sudo systemd-nspawn -D /var/lib/machines/monconteneur --network-veth
sudo ip addr add 10.10.10.1/24 dev ve-monconteneur
sudo ip link set ve-monconteneur up
```

Dans le conteneur, il faut ensuite configurer l'interface `host0` (nom donné automatiquement côté conteneur à l'autre extrémité de la paire veth) :

```bash
ip addr add 10.10.10.2/24 dev host0
ip link set host0 up
ip route add default via 10.10.10.1
```

Pour que cette configuration réseau soit appliquée automatiquement à chaque démarrage via `machinectl`, ajoutez `--network-veth` dans un fichier de configuration sous `/etc/systemd/nspawn/monconteneur.nspawn`, dans la section `[Network]`.

## Vérification

Depuis l'hôte, vérifiez que l'interface veth est active et que le conteneur répond aux requêtes ping :

```bash
ip addr show ve-monconteneur
ping -c 3 10.10.10.2
```

L'interface `ve-monconteneur` doit apparaître à l'état `UP` avec l'adresse configurée, et les requêtes ping vers l'adresse du conteneur doivent obtenir une réponse.
