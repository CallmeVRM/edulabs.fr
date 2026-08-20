---
title: "Cache DNS corrompu après une mise à jour"
weight: 2
---

## Symptômes

Juste après une mise à jour système incluant `systemd` ou le paquet `dnsmasq`, un serveur continue de résoudre un nom de domaine vers une ancienne adresse IP qui n'existe plus, alors que `dig` interrogé directement contre le DNS faisant autorité renvoie la bonne valeur. Le problème persiste même après redémarrage des applications concernées, et touche uniquement les résolutions passant par le cache local.

## Cause probable

Le service de cache DNS local (`systemd-resolved`, `nscd` ou `dnsmasq`) a conservé en mémoire une entrée avec un TTL anormalement long, ou n'a pas correctement invalidé son cache lors du redémarrage déclenché par la mise à jour du paquet. Il arrive aussi que la mise à jour ait modifié la configuration réseau sans relancer proprement le service de résolution, laissant tourner une instance avec un état obsolète.

## Diagnostic

```bash
resolvectl statistics
resolvectl query exemple.fr
systemctl status systemd-resolved
journalctl -u systemd-resolved --since "1 hour ago"
```

`resolvectl statistics` affiche le nombre d'entrées en cache et permet de confirmer qu'une entrée obsolète y est toujours présente. Le journal du service peut révéler des erreurs de rechargement de configuration survenues au moment de la mise à jour.

## Résolution

1. Vider le cache DNS local avec `resolvectl flush-caches` (ou `systemctl restart nscd` selon le service utilisé).
2. Si le problème réapparaît, redémarrer complètement le service de résolution : `systemctl restart systemd-resolved`.
3. Vérifier que le TTL renvoyé par le serveur faisant autorité est cohérent (`dig +noall +answer exemple.fr` côté serveur externe) et ajuster si un TTL trop long a été fixé par erreur côté zone.
4. Documenter l'incident : un redémarrage systématique du service de cache DNS après toute mise à jour touchant la pile réseau évite la récidive.
