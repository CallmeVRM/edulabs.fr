---
title: "Une unité systemd expire (timeout) au démarrage"
weight: 2
---

## Symptômes

Le démarrage du système prend nettement plus de temps que d'habitude, et `systemctl status mon-service` finit par afficher un état `failed` accompagné du message « start operation timed out ». Le service semble pourtant fonctionner correctement une fois lancé manuellement en dehors de systemd, mais sans jamais réussir son démarrage automatique dans le délai imparti.

## Cause probable

Le service met plus de temps à s'initialiser que la valeur `TimeoutStartSec` définie (90 secondes par défaut dans la plupart des cas), souvent parce qu'il attend une ressource externe lente à répondre (base de données distante, montage réseau, ou disque lent), ou parce qu'un service de type `notify` n'envoie jamais le signal `READY=1` attendu par systemd, même s'il fonctionne par ailleurs normalement.

## Diagnostic

```bash
systemd-analyze blame
journalctl -u mon-service -n 80 --no-pager
systemctl show mon-service -p TimeoutStartUSec -p Type
systemd-analyze critical-chain mon-service
```

`systemd-analyze blame` classe les unités par temps de démarrage et révèle rapidement si l'unité en cause dépasse largement les autres. `critical-chain` montre la chaîne de dépendances et le temps consommé à chaque étape, ce qui aide à localiser une attente anormale sur une ressource précise.

## Résolution

1. Si le service a réellement besoin de plus de temps pour s'initialiser (cas légitime, par exemple une base de données volumineuse à charger), augmenter `TimeoutStartSec` dans un fichier de surcharge (`systemctl edit mon-service`) plutôt que dans le fichier d'unité fourni par le paquet.
2. Si le service est de type `notify` mais n'envoie jamais le signal de disponibilité, vérifier que le binaire supporte réellement l'intégration `sd_notify`, ou passer temporairement le type à `simple` en attendant une correction amont.
3. Traiter la cause de la lenteur elle-même si elle est identifiée (connexion réseau lente, disque à latence anormale) plutôt que de se contenter d'augmenter le délai.
4. Relancer le service après ajustement et confirmer avec `systemd-analyze blame` que son temps de démarrage reste désormais dans une plage normale.
