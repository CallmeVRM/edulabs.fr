---
title: "Installer Docker sur une distribution Linux"
weight: 1
---

## Objectif

Installer le moteur Docker (Docker Engine) à partir du dépôt officiel plutôt que depuis les paquets génériques de la distribution, afin de disposer d'une version récente et des composants Compose associés.

## Prérequis

- Distribution basée sur Debian/Ubuntu (adaptable à Fedora/CentOS via leurs dépôts respectifs)
- Droits `sudo`
- Accès réseau sortant vers `download.docker.com`

## Étapes

1. Installer les dépendances nécessaires à l'ajout d'un dépôt via HTTPS.
2. Ajouter la clé GPG et le dépôt officiel Docker.
3. Installer le moteur Docker ainsi que le plugin Compose.
4. Ajouter son utilisateur au groupe `docker` pour éviter de préfixer chaque commande avec `sudo`.

```bash
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER
```

Après l'ajout au groupe `docker`, une déconnexion/reconnexion (ou `newgrp docker`) est nécessaire pour que le changement de groupe prenne effet dans la session courante.

## Vérification

Vérifiez que le service tourne et qu'un conteneur de test peut être lancé sans `sudo` :

```bash
sudo systemctl status docker
docker run --rm hello-world
```

Le service doit apparaître comme `active (running)`, et la commande `docker run` doit afficher le message de bienvenue confirmant que le client a pu contacter le démon et récupérer une image.
