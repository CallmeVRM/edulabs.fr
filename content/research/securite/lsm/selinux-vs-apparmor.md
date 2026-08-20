---
title: "SELinux contre AppArmor : deux modèles de confinement"
weight: 1
---

## Contexte

Le choix entre SELinux et AppArmor dépend souvent davantage de la distribution utilisée (Red Hat pousse SELinux, Debian/Ubuntu privilégient AppArmor) que d'une décision technique réfléchie. Cette note revient sur les différences de modèle pour comprendre ce que chacun apporte réellement en matière de confinement.

## Analyse technique

SELinux applique un contrôle d'accès obligatoire basé sur des étiquettes (labels) attachées à chaque objet du système (fichiers, sockets, processus). Toute interaction est vérifiée contre une politique qui définit quels types de sujets peuvent agir sur quels types d'objets. C'est puissant et exhaustif, mais la politique de référence est complexe, et le mode "targeted" utilisé par défaut ne couvre qu'un sous-ensemble de services.

AppArmor, à l'inverse, raisonne par chemins de fichiers plutôt que par étiquettes abstraites : chaque profil décrit ce qu'un binaire donné a le droit de lire, écrire ou exécuter, en listant des chemins explicites. C'est plus lisible et plus rapide à écrire pour un profil ad hoc, mais moins robuste face à des attaques exploitant des liens symboliques ou des montages qui contournent le chemin attendu.

```bash
# SELinux : vérifier le contexte d'un fichier
ls -Z /var/www/html/index.html

# AppArmor : profil en mode "complain" pour observer sans bloquer
aa-complain /usr/sbin/nginx
```

## Observations

En confinant le même service nginx avec les deux approches, sur des distributions différentes :

- la politique SELinux ciblée a bloqué une tentative d'écriture hors des répertoires attendus sans configuration additionnelle, grâce au typage déjà présent dans la politique de référence ;
- le profil AppArmor équivalent a nécessité une phase d'apprentissage plus explicite (mode `complain` puis analyse des logs) pour couvrir tous les chemins légitimes utilisés par le service ;
- une fois les deux profils correctement calés, le niveau de confinement obtenu était comparable pour ce cas d'usage précis, la différence se jouant surtout sur la charge de configuration initiale.

## Pour aller plus loin

Un test avec un service plus dynamique, générant ses propres fichiers temporaires dans des chemins imprévisibles, permettrait de mieux départager les deux modèles sur leur résistance réelle au contournement.
