---
title: "Gérer les permissions avancées avec les ACL"
weight: 2
---

## Objectif

Le modèle de permissions Unix classique (propriétaire/groupe/autres) devient rapidement limitant lorsque plusieurs utilisateurs ou groupes doivent accéder différemment à un même fichier. Les listes de contrôle d'accès (ACL) permettent d'attribuer des droits fins à des utilisateurs ou groupes spécifiques, en plus du modèle standard.

## Prérequis

- Paquet `acl` installé (`apt install acl` ou `dnf install acl`)
- Système de fichiers monté avec l'option `acl` (activée par défaut sous ext4 et XFS récents)
- Droits suffisants sur le fichier ou répertoire cible

## Étapes

1. Vérifier les ACL déjà présentes sur un répertoire.
2. Accorder un droit de lecture/écriture à un utilisateur précis, sans modifier le propriétaire.
3. Définir une ACL par défaut pour que les nouveaux fichiers créés dans le répertoire héritent des mêmes droits.
4. Retirer une entrée ACL si elle n'est plus nécessaire.

```bash
getfacl /srv/projets/partage
setfacl -m u:amartin:rwx /srv/projets/partage
setfacl -d -m u:amartin:rwx /srv/projets/partage
setfacl -x u:amartin /srv/projets/partage
```

L'option `-m` modifie une entrée existante ou en ajoute une nouvelle, tandis que `-d` définit une ACL par défaut appliquée aux futurs fichiers et sous-répertoires. Notez que l'ajout d'une ACL fait apparaître un `+` à la fin de la ligne de permissions dans la sortie de `ls -l`.

## Vérification

Confirmez que l'entrée ACL est bien appliquée et visible dans les permissions :

```bash
ls -ld /srv/projets/partage
getfacl /srv/projets/partage
```

La présence du signe `+` après les droits classiques (`drwxrwxr-x+`) indique qu'une ACL est active, et `getfacl` doit lister l'entrée `user:amartin:rwx` ajoutée précédemment.
