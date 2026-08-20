---
title: "Un service systemd redémarre en boucle"
weight: 1
---

## Symptômes

`systemctl status mon-service` affiche un état qui alterne rapidement entre `activating` et `failed`, avec un compteur de redémarrages qui augmente continuellement. Les journaux montrent le même message d'erreur qui se répète toutes les quelques secondes, et le service n'atteint jamais un état `active (running)` stable.

## Cause probable

Le processus lancé par le service échoue systématiquement dès son démarrage, le plus souvent à cause d'un fichier de configuration invalide, d'une dépendance non disponible (base de données, socket, ou autre service qui n'est pas encore prêt), ou d'un port déjà occupé par un autre processus. La politique de redémarrage automatique définie dans l'unité (`Restart=always`) masque alors l'échec en relançant indéfiniment le processus sans jamais le stabiliser.

## Diagnostic

```bash
systemctl status mon-service
journalctl -u mon-service -n 50 --no-pager
systemctl show mon-service -p Restart -p RestartUSec
ss -tlnp | grep <port_attendu>
```

Le journal détaillé du service donne presque toujours l'erreur exacte à l'origine de l'échec (fichier introuvable, port déjà utilisé, erreur de syntaxe de configuration). Vérifier avec `ss` si un autre processus occupe déjà le port attendu permet d'écarter rapidement cette cause fréquente.

## Résolution

1. Isoler l'erreur précise dans les journaux et la corriger à la source (chemin de configuration, permissions de fichier, dépendance manquante).
2. Si la cause est une dépendance non prête au démarrage, ajouter une directive `After=` et `Requires=` appropriée dans le fichier d'unité pour ordonner correctement le démarrage.
3. Réinitialiser le compteur d'échecs et relancer le service proprement : `systemctl reset-failed mon-service && systemctl start mon-service`.
4. Une fois la cause corrigée, laisser tourner le service quelques minutes en surveillant `systemctl status` pour confirmer qu'il atteint un état stable sans redémarrage supplémentaire.
