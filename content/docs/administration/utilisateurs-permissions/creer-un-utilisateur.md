---
title: "Créer et configurer un nouvel utilisateur"
weight: 1
---

## Objectif

Cette procédure décrit la création d'un compte utilisateur local, l'attribution d'un mot de passe et son ajout à un ou plusieurs groupes existants, afin de lui donner un accès contrôlé au système.

## Prérequis

- Accès root ou droits `sudo` sur la machine
- Le paquet `shadow-utils` (ou équivalent) installé, présent par défaut sur la quasi-totalité des distributions
- Connaître le groupe secondaire souhaité (par exemple `sudo`, `wheel` ou `docker`)

## Étapes

1. Créer le compte avec un répertoire personnel et un shell par défaut.
2. Définir un mot de passe initial que l'utilisateur devra changer à la première connexion.
3. Ajouter le compte à un groupe secondaire si nécessaire.
4. Forcer l'expiration du mot de passe pour imposer un changement.

```bash
sudo useradd -m -s /bin/bash amartin
sudo passwd amartin
sudo usermod -aG sudo amartin
sudo chage -d 0 amartin
```

Le drapeau `-m` garantit la création du répertoire `/home/amartin`, et `-s /bin/bash` fixe le shell de connexion. L'ajout au groupe `sudo` (ou `wheel` sur les distributions basées sur RHEL) permet d'exécuter des commandes privilégiées sans passer par le compte root directement.

## Vérification

Vérifiez que le compte existe, que son répertoire personnel a bien été créé et qu'il appartient au bon groupe :

```bash
id amartin
getent passwd amartin
```

La commande `id` doit lister le groupe secondaire attendu dans la sortie, et `getent passwd` doit confirmer le chemin du répertoire personnel et le shell configuré.
