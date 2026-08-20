---
title: "Résolution DNS anormalement lente"
weight: 1
---

## Symptômes

Sur plusieurs postes du même réseau, l'ouverture d'un site web ou une simple commande `ping vers-un-nom-de-domaine` met entre 3 et 8 secondes avant d'obtenir une réponse, alors que le `ping` vers une adresse IP directe est instantané. Le phénomène est intermittent : certaines requêtes passent en moins de 100 ms, d'autres traînent systématiquement. Les utilisateurs remontent des pages qui « chargent longtemps au début puis vont vite ».

## Cause probable

Dans la grande majorité des cas, le résolveur configuré (souvent le premier serveur DNS de `/etc/resolv.conf`) ne répond pas ou répond avec une latence importante, et le système attend le délai de timeout complet avant de basculer sur le serveur suivant. On retrouve aussi ce symptôme quand le service local `systemd-resolved` ou un cache DNS applicatif tente d'abord une requête IPv6 (AAAA) qui échoue silencieusement avant de retomber sur IPv4 (A).

## Diagnostic

```bash
dig +stats exemple.fr
resolvectl status
time getent hosts exemple.fr
tcpdump -ni any port 53 -c 20
```

Le champ `Query time` renvoyé par `dig` donne la latence réelle côté serveur DNS interrogé. Si `resolvectl status` montre plusieurs serveurs configurés et que le premier de la liste ne répond jamais dans `tcpdump`, c'est probablement lui qui pénalise chaque requête avant le basculement.

## Résolution

1. Vérifier la joignabilité de chaque serveur DNS listé avec `dig @<ip_serveur> exemple.fr`.
2. Retirer ou reclasser en dernière position tout serveur qui ne répond pas ou répond très lentement.
3. Réduire le délai d'attente avec l'option `timeout:1` et `attempts:2` dans `/etc/resolv.conf` si la configuration n'est pas gérée automatiquement.
4. Envisager l'ajout d'un résolveur local en cache (`systemd-resolved`, `dnsmasq` ou `unbound`) pour absorber la latence des requêtes répétées.
