---
title: "Créer un conteneur léger avec systemd-nspawn"
weight: 1
---

## Objectif

systemd-nspawn permet de créer et démarrer un conteneur à partir d'une simple arborescence de fichiers, sans image intermédiaire ni démon supplémentaire, en s'appuyant uniquement sur les composants déjà fournis par systemd.

## Prérequis

- Paquet `systemd-container` installé
- Droits root
- Outil `debootstrap` (ou équivalent) pour préparer une arborescence minimale d'une distribution

## Étapes

1. Créer un répertoire qui accueillera l'arborescence racine du conteneur.
2. Installer un système minimal dans ce répertoire avec `debootstrap`.
3. Démarrer un shell interactif dans le conteneur pour vérifier son bon fonctionnement.
4. Enregistrer le conteneur comme une machine gérée par `machinectl` pour un démarrage persistant.

```bash
sudo mkdir -p /var/lib/machines/monconteneur
sudo debootstrap stable /var/lib/machines/monconteneur http://deb.debian.org/debian
sudo systemd-nspawn -D /var/lib/machines/monconteneur
sudo machinectl start monconteneur
```

Une fois dans le shell interactif ouvert par `systemd-nspawn -D`, vous êtes root dans le conteneur et pouvez y installer des paquets ou configurer des services comme sur une machine classique. La commande `machinectl start` démarre le conteneur en arrière-plan comme un service systemd, ce qui permet de le superviser avec les outils habituels (`systemctl`, `journalctl`).

## Vérification

Listez les machines actives et confirmez que le conteneur répond :

```bash
machinectl list
machinectl shell monconteneur
```

`machinectl list` doit afficher `monconteneur` avec l'état `running`, et `machinectl shell` doit ouvrir une session à l'intérieur du conteneur sans erreur.
