---
title: "Linux Capabilities - Introduction"
weight: 1
---

## 1. Un peu d'histoire
Il n'y a pas si longtemps, la gestion des processus sous Linux était binaire : soit un processus était privilégié (UID 0, avec un accès total aux appels noyau), soit il ne l'était pas. À première vu on se dirait qu'à l'époque, la sécurité n'était clairement pas la priorité. 

Dès qu'un utilisateur standard devait lancer un programme nécessitant un accès spécifique, on attribuait simplement le bit setuid au binaire.

Résultat : pour un simple utilitaire comme ping, qui a seulement besoin d'ouvrir une socket ICMP brute, le programme tournait avec les pleins pouvoirs de root. Il héritait donc, par extension, du droit de lire n'importe quel fichier sensible du système. Ce n'est plus le cas pour ping aujourd'hui je vous rassure, 

Pour y remédier, on a découper les appels privilégiés en catégories fonctionnelles (une partie pour les mounts, le réseau, etc.), et ne donner au processus que celles dont il a besoin (principe du moindre privilège), ping ne reçoit plus que CAP_NET_RAW. **Aujourd'hui on passe par une autre mécanique pour ping, mais pour garder le raisonnement j'ai continuer avec l'exemple.**

## 2. Les idées a retenir :

- **Un processus normal a zéro capability.** mais il accède bien à la mémoire et au disque, surprenant n'est ce pas ? Eh bien cet accès est garanti parcequ'il passe par les permissions de fichiers classiques. Les capabilities ne servent généralement qu'aux tâches dites "Système" (mounts, réseau, etc.)
- **Elles s'attachent aux threads, pas aux utilisateurs.** Un thread est créé par `clone`, un nouveau programme est chargé par `execve`. Les capabilities intervienent aux deux étapes, et c'est toute la compléxité vient de la deuxième : Que devient mon jeu de capabilities quand je lance un autre programme ?
- **Les fichiers peuvent aussi en porter.** A condition que le sytème de fichier support le `xattr`, via l'attribue etendu `security.capability`
- **Elles s'ajoutent aux permission classiques, sans les remplacer.** `CAP_NET_ADMIN` ne permet pas de lire `/etc/shadow`. Seules `CAP_DAC_OVERRIDE` et `CAP_DAC_READ_SEARCH` contournent les permissions de fichiers. Donc attention attention !

## 3. Le découpage :

Le noyau expose des centaines d’appels système et d’ioctl. Parmi eux, ceux qui touchent à la sécurité de la machine (réseau, fichiers des autres, horloge, modules…) étaient autrefois réservés à `UID 0`. Les développeurs du noyau les ont regroupés par famille fonctionnelle, et chaque famille est devenue une capability.

Le principe est plutôt simple, quand un thread tente une opération, le noyau pose une question précise du type : << Ce thread a-t-il `CAP_NET_RAW` dans son ensemble **effective** ?>> Si la réponse est oui, l'opération sera effectué, sinon le noyau bloque l'opération et renvoie `EPERM`

Avant d'enchainer, il faut qu'on clarifie les définitions de certains concepts :

Il y a deux façons de penser un droit :

- Un droit sur un objet précis : Bob peut lire le fichier `/etc/ssh/sshd_config`. Le droit est lié à un objet identifié (le fichier `sshd_config`)

- Une dispense de règle : Bob est exemptée de la règle qui limite qui peut envoyer un signale réseau. Le droit n'est pas attaché à un objet, il supprimer le contrôle pour une certaine catégorie d'opérations. Une capabality Linux est toujours de ce type.

> Sans capability, quand un processus envoie un signal, le noyau applique une règle simple : l’UID réel ou effectif de l’expéditeur doit correspondre à l’UID réel ou sauvegardé de la cible. Autrement dit, vous ne pouvez tuer que vos propres processus. C’est pourquoi alice ne peut pas tuer un processus de bob ni un processus de root.
>
> `CAP_KILL` ne dit pas « tu peux tuer le processus 1234 ». Elle dit : « ce test d’UID ne s’applique plus à toi ». Le noyau ne regarde plus à qui appartient la cible, ce qui vaut pour tous les processus visibles, ceux de root compris.

Conséquence directe : on ne peut pas donner `CAP_KILL` pour un seul processus. Il n’existe pas de « `CAP_KILL` sur le PID 1234 ». C’est tout ou rien sur la catégorie « envoyer un signal à n’importe qui ».

Avec une nuance, cette capability ne vaut que pour les processus visibles depuis PID namespace. Dans un conteneur CAP_KILL ne permet pas d'atteindre les processus de l'hôte, parce qu'ils n'y aparaissent pas ! Ou que d'autres mécanismes restent actifs, comme SELinux, AppArmor, qui peuvent encore bloquer le signal.

Le même raisonnement s’applique à toutes les capabilities :

- `CAP_NET_BIND_SERVICE` : non pas « écouter sur le port 80 », mais « ignorer la limite des ports inférieurs à 1024 », donc **n’importe quel** port bas.
- `CAP_CHOWN` : non pas « changer le propriétaire de ce fichier », mais « changer le propriétaire de **n’importe quel** fichier ».
- `CAP_DAC_READ_SEARCH` : « lire tout », pas « lire `/var/backups` ».

> #### **Une capability n'est pas un droit sur un objet *précis*.** 
>
> Avec `CAP_KILL` par exemple, elle ne dit pas tu peux tuer le processus 1234, mais donne le droit de tuer n'iumporte quel processus, même si l'UID ne correspond pas !
>
> #### **Une capability ne donne que ce que son nom promet (théoriquement).** 
>
> `CAP_NET_ADMIN` configure le réseau, elle ne permet pas de lire `/etc/shadow`. Il faut pour cela une capability de la famille << fichier >>.

### Hands-on Lab :

Un petit Lab pour renforcer la théorie :

```bash
# 1. Un processus appartenant à root
sudo systemd-run --unit=victim sleep 300
PID=$(systemctl show -p MainPID --value victim)
ps -o user,pid,cmd -p "$PID"

# 2. Sans capability : refusé (l'UID ne correspond pas)
kill "$PID"                       # Operation not permitted

# 3. Avec CAP_KILL uniquement (ambient, voir Lab 4)
sudo setpriv --reuid="$(id -u)" --regid="$(id -g)" --init-groups \
  --inh-caps=+kill --ambient-caps=+kill \
  /bin/kill "$PID"
systemctl is-active victim        # inactive ou failed : le processus root a bien été tué

sudo systemctl reset-failed victim 2>/dev/null
```

#### **Exercice 1.**

Décodez à la main, puis vérifiez avec `capsh --decode` :
	a) `0000000000000400`
	b) `0000000000003000`
	c) `0000000000000024`
	d) `0000000000200000`
	e) `0000000000002400`

#### **Exercice 2.**

Encodez : `CAP_CHOWN` + `CAP_SETUID` + `CAP_SETGID`. Donnez le masque, puis vérifiez avec `capsh --decode`.

#### **Exercice 3.**

Sur votre machine :

a) quel est le masque `CapBnd` de votre shell ? Combien de capabilities contient-il ?
	b) décodez le `CapEff` du PID 1, puis d’un service durci (`systemd-resolved`).

Voici un tableau des capability et leur numéro

```bash
Bit	Capability
0	CAP_CHOWN	
1	CAP_DAC_OVERRIDE	
2	CAP_DAC_READ_SEARCH	
3	CAP_FOWNER	
4	CAP_FSETID	
5	CAP_KILL	
6	CAP_SETGID	
7	CAP_SETUID	
8	CAP_SETPCAP	
9	CAP_LINUX_IMMUTABLE	
10	CAP_NET_BIND_SERVICE	
11	CAP_NET_BROADCAST	
12	CAP_NET_ADMIN	
13	CAP_NET_RAW	
14	CAP_IPC_LOCK	
15	CAP_IPC_OWNER	
16	CAP_SYS_MODULE	
17	CAP_SYS_RAWIO	
18	CAP_SYS_CHROOT	
19	CAP_SYS_PTRACE	
20	CAP_SYS_PACCT	
21	CAP_SYS_ADMIN
22	CAP_SYS_BOOT
23	CAP_SYS_NICE
24	CAP_SYS_RESOURCE
25	CAP_SYS_TIME
26	CAP_SYS_TTY_CONFIG
27	CAP_MKNOD
28	CAP_LEASE
29	CAP_AUDIT_WRITE
30	CAP_AUDIT_CONTROL
31	CAP_SETFCAP
32	CAP_MAC_OVERRIDE
33	CAP_MAC_ADMIN
34	CAP_SYSLOG
35	CAP_WAKE_ALARM
36	CAP_BLOCK_SUSPEND
37	CAP_AUDIT_READ
38	CAP_PERFMON (noyau ≥ 5.8)
39	CAP_BPF (noyau ≥ 5.8)
40	CAP_CHECKPOINT_RESTORE (noyau ≥ 5.9)
```

### Ce qu’il faut retenir

- Une capability est une **dispense de contrôle pour toute une catégorie d’opérations**, pas une clé pour un objet.
- Il est impossible de l’**étroiter** à un objet précis (ce processus, ce fichier, ce port). On borne son effet autrement : namespaces, LSM, utilisateur dédié, `sudo` avec commande figée.
- C’est pourquoi on choisit la capability la plus fine possible, et pourquoi certaines (`SYS_ADMIN`, `DAC_OVERRIDE`, `SETUID`) sont dangereuses : leur catégorie est trop vaste.
- Si une documentation vous demande l’une de ces capabilities, vous allez simplement donnez root mais sous un autre nom : `CAP_SYS_ADMIN` (fourre-tout), `CAP_SETUID`/`CAP_SETGID`, `CAP_DAC_OVERRIDE`, `CAP_SYS_MODULE`, `CAP_SYS_PTRACE`, `CAP_DAC_READ_SEARCH` (lecture de tout).

## 4. Les cinq ensemble d'un Thread :

Chaque thread possède cinq ensembles de capabilities (`CapEff`, `CapPrm`, `CapInh`, `CapBnd`, `CapAmb`), visibles dans `/proc/<PID>/status`. De leur côté, les fichiers exécutables sur le disque ne possèdent que deux ensembles (`Permitted` et `Inheritable`) accompagnés d'un simple bit booléen (`Effective`), configurables via `setcap`.

Pour comprendre l'interaction entre ces ensembles, il faut distinguer deux moments dans le cycle de vie d'un processus :

1. **Pendant que le programme s'exécute :** comment le noyau contrôle les privilèges à un instant $T$ et comment le processus gère ses propres droits en mémoire.
2. **Lorsqu'un programme en lance un autre (**`execve`**) :** comment les droits sont recalculés, transmis ou supprimés au chargement d'un nouveau binaire.

#### A. Pendant l'exécution du programme (`Effective` et `Permitted`)

- **Effective (**`CapEff`**) — Les droits actifs :** C'est le **seul** ensemble que le noyau vérifie lorsqu'un processus effectue un appel système privilégié (ex: ouvrir un socket brut ou monter une partition). Si la capability requise n'est pas dans `CapEff` à cet instant précis, le noyau renvoie `EPERM` équivalent du permission denied.
- **Permitted (**`CapPrm`**) — La réserve maximale du processus :** Cet ensemble définit le plafond des capabilities que le processus a le droit d'activer dans `CapEff`. Une fois lancé, un processus (et même un administrateur à chaud) ne peut **jamais** ajouter de nouvelles capabilities dans `CapPrm` : ce capital est fixé au démarrage lors du `execve()`. En revanche, le processus peut utiliser l'appel système `capset()` pour basculer un droit de `CapPrm` vers `CapEff`, ou pour retirer définitivement un droit de `CapPrm`.
  - *Cas concret :* Un démon « capability-aware » (comme `tcpdump` ou `chronyd`) démarre avec une capability dans `CapPrm`, la retire de `CapEff` pendant qu'il traite des données non fiables, l'active dans `CapEff` uniquement le temps d'un appel système critique (ouverture de socket ou réglage d'horloge), puis vide définitivement `CapEff` et `CapPrm` dès qu'il n'en a plus besoin.

#### B. Lors du lancement d'un nouveau programme via `execve()` (`Inheritable`, `Ambient` et `Bounding`)

Lorsqu'un processus exécute un nouveau binaire avec `execve()`, le noyau remet `CapPrm` et `CapEff` à zéro, puis recalcule les nouveaux droits à partir des trois ensembles suivants et des attributs du fichier cible :

- **Inheritable (**`CapInh`**) — La transmission par double consentement :** Cet ensemble indique quelles capabilities le processus parent accepte de transmettre à l'exécutable qu'il lance. Cependant, `CapInh` seul ne donne aucun privilège : le fichier binaire exécuté doit **lui aussi** posséder cette même capability dans son ensemble `Inheritable` sur le disque (`F(I)`). Le noyau effectue un `ET` logique (`CapInh & F(I)`) pour déterminer ce qui entrera dans le nouveau `CapPrm`.
  - *Cas concret :* Restreindre l'usage d'un binaire privilégié à certains utilisateurs (via `pam_cap.so`) en posant uniquement le flag `i` sur l'exécutable (`setcap cap_net_raw=ie binaire`), afin que seuls les utilisateurs ayant `CAP_NET_RAW` dans leur `CapInh` de session puissent l'exécuter avec ce privilège.
- **Ambient (**`CapAmb`**) — L'héritage automatique vers les binaires standards :** Introduit pour pallier la lourdeur d'`Inheritable`, `CapAmb` permet de transmettre des capabilities à des binaires standards (scripts Python, binaires Go/Node.js) qui n'ont aucun attribut `setcap` sur le disque. Toute capability présente dans `CapAmb` survit au `execve()` et est directement injectée dans le `CapPrm` et le `CapEff` du processus enfant (sauf si le binaire cible est `setuid` ou possède déjà des capabilities sur le disque, auquel cas `CapAmb` est vidé par sécurité).
  - *Invariant strict :* Une capability ne peut être placée dans `CapAmb` que si le processus la possède déjà à la fois dans `CapPrm` **et** dans `CapInh`. La retirer de `CapPrm` ou de `CapInh` la supprime immédiatement de `CapAmb`.
  - *Cas concret :* Un service `systemd` configuré avec `User=www-data` et `AmbientCapabilities=CAP_NET_BIND_SERVICE` permet à un serveur web non privilégié d'écouter sur le port 443 sans modifier le fichier exécutable sur le disque.
- **Bounding (**`CapBnd`**) — Le masque de restriction irréversible :** Cet ensemble agit comme un filtre de sécurité (plafond absolu) pour le processus et toute sa descendance. Il masque les capabilities qu'un fichier binaire sur le disque peut octroyer lors d'un `execve()` (`F(P) & CapBnd`). On ne peut que retirer des bits de `CapBnd`, jamais en rajouter, et cette limite s'impose **même à** `root` **(UID 0)** et aux binaires `setuid root`.
  - *Cas concret :* Les runtimes de conteneurs (`runc`, Docker, Kubernetes) ou `systemd` (`CapabilityBoundingSet=`) retirent des capabilities critiques comme `CAP_SYS_MODULE` ou `CAP_SYS_TIME` de `CapBnd` afin d'empêcher un processus — même exécuté en `root` dans le conteneur — de charger un module noyau ou de modifier l'horloge de l'hôte.
