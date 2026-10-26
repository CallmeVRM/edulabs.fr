---
title: "introduction"
weight: 1
---

## 1. Un peu d'histoire

Il n'y a pas si longtemps que ça, la gestion des processus sous Linux était binaire : soit un processus avait des privilèges (`UID 0`, avec un accès total aux appels noyau, ce qu'on appel vulgairement root), soit il n'avait presque rien, et ne pouvait faire grand chose. De ce fait dès qu'un utilisateur standard devait lancer un programme nécessitant un accès spécifique, on utilisait le compte root pour maquer le binaire avec le `bit SUID`, ainsi ce dernier s'exécute avec les droits de son propiétaire (`root`). Bonjour la sécu !

Résultat : pour un simple utilitaire comme ping, qui a seulement besoin d'ouvrir une socket ICMP brute, le programme tournait avec les pleins pouvoirs de root. Il héritait donc, par extension, du droit de lire n'importe quel fichier sensible du système.

Vous l'avez compris, le problème du `SUID`, c'est la granulartié, un binaire marqué par un `SUID` de root, se voit attribué les pleins pouvoir sur la machine.

Pour y remédier, on a découper les appels privilégiés en catégories fonctionnelles (une partie pour les mounts, le réseau, etc.), et ne donner au processus que celles dont il a besoin (principe du moindre privilège), ping ne reçoit plus que CAP_NET_RAW. **Aujourd'hui on passe par une autre mécanique pour ping, mais pour garder le raisonnement j'ai continuer avec l'exemple de** `ping`**.**

## 2. Les idées a retenir :

- **Un processus normal a zéro capability,** mais il accède bien à la mémoire et au disque, surprenant n'est ce pas ? Eh bien cet accès est garanti parcequ'il passe par les permissions de fichiers classiques. Les capabilities ne servent généralement qu'aux tâches dites "Système" (mounts, réseau, etc.)
- **Elles s'attachent aux threads, pas aux utilisateurs.** Un thread est créé par `clone`, un nouveau programme est chargé par `execve`. Les capabilities intervienent aux deux étapes, et toute la compléxité vient de la deuxième : Que devient mon jeu de capabilities quand je lance un autre programme ?
- **Les fichiers peuvent aussi en porter.** A condition que le sytème de fichier support le `xattr`, via l'attribue etendu `security.capability`
- **Elles s'ajoutent aux permission classiques, sans les remplacer.** `CAP_NET_ADMIN` ne permet pas de lire `/etc/shadow`. Seules `CAP_DAC_OVERRIDE` et `CAP_DAC_READ_SEARCH` contournent les permissions de fichiers. Donc attention attention !

> [!IMPORTANT]
> Un `processus` est un simple contenant (thread group) qui peut regrouper plusieurs `threads`, lesquels partagent mémoire et ressources mais possèdent chacun leurs propres capabilities (c'est le `thread` appelant qui est vérifié par le noyau). Donc dans le contexte des **capabilities**, l'unité qui compte est le `thread`, et « processus » n'est qu'un raccourci valable techniquement que pour les programmes mono-thread. Je me permet parfois d'utiliser le terme de `processus`, par abus de langage pour les besoins pédagogiques : dans ce contexte, c'est bien le `thread` qui est visé.

## 3. Le découpage :

Le noyau expose des centaines d’*appels système* et d’*ioctl*. Parmi eux, ceux qui touchent à la sécurité de la machine (réseau, fichiers des autres, horloge, modules…) étaient autrefois réservés à `UID 0`. Les développeurs du noyau les ont regroupés par famille fonctionnelle, et chaque famille est devenue une capability.

Le principe est plutôt simple, quand un thread tente une opération, le noyau pose une question précise du type : « Ce thread a-t-il `CAP_NET_RAW` dans son ensemble **effective** ? » Si la réponse est oui, l'opération sera effectué, sinon le noyau bloque l'opération et renvoie `EPERM` - (EPERM = permission denied).

Avant d'enchainer, il faut qu'on clarifie les définitions de certains concepts :

> Il y a deux façons de penser un droit :
>
> - Un droit sur un objet précis : Bob peut lire le fichier `/etc/ssh/sshd_config`. Le droit est lié à un objet identifié (le fichier `sshd_config`)
>
> - Une dispense de règle : Bob est exemptée de la règle qui limite qui peut envoyer un signale réseau. Le droit n'est pas attaché à un objet, il supprimer le contrôle pour une certaine catégorie d'opérations. Une capabality Linux est toujours de ce type.

Sans `capability`, quand un `Processus/Thread` envoie un signal, le noyau applique une règle simple : `l’UID` réel ou effectif de l’expéditeur doit correspondre à `l’UID` réel ou sauvegardé de la cible. Autrement dit, vous ne pouvez tuer que vos propres processus. C’est pourquoi Bob ne peut pas tuer un processus de Alice, encore moins un processus de root.

`CAP_KILL` par exemple et par défaut, ne dit pas « tu peux tuer le processus X ». Elle dit : « ce test d’UID ne s’applique plus à toi ». Le noyau ne regarde plus à qui appartient la cible, ce qui vaut pour tous les processus visibles, ceux de root compris.

> Conséquence directe : on ne peut pas donner `CAP_KILL` pour un seul processus. Il n’existe pas de « `CAP_KILL` sur le PID 1234 ». C’est tout ou rien sur la catégorie « envoyer un signal à n’importe qui ».

Avec une nuance très importante : Cette capability ne vaut que pour les `Thread` des `Processus` visibles depuis PID namespace.  
Dans un conteneur `CAP_KILL` ne permet pas d'atteindre les processus de l'hôte, parce qu'ils n'y aparaissent pas, ou que d'autres mécanismes restent actifs, comme `SELinux`, `AppArmor`, qui peuvent encore bloquer le signal.

Le même raisonnement s’applique à toutes les autres capabilities :

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

Voici un tableau des capabilities Linux et leur numéro :

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
- Il est impossible de l’**étroiter** à un objet précis (ce processus, ce fichier, ce port). On contrôle son effet autrement : namespaces, LSM, utilisateur dédié, `sudo` avec commande figée.
- C’est pourquoi on choisit la capability la plus fine possible, et pourquoi certaines (`SYS_ADMIN`, `DAC_OVERRIDE`, `SETUID`) sont dangereuses : leur catégorie est trop vaste.
- Si une documentation vous demande l’une de ces capabilities, vous allez simplement donnez root mais sous un autre nom : `CAP_SYS_ADMIN` (fourre-tout), `CAP_SETUID`/`CAP_SETGID`, `CAP_DAC_OVERRIDE`, `CAP_SYS_MODULE`, `CAP_SYS_PTRACE`, `CAP_DAC_READ_SEARCH` (lecture de tout).
- Les capabilities n'est PAS une frontière d'isolation. Puisque les threads partagent l'espace d'adressage, un thread peu privilégié peut lire/écrire la mémoire d'un thread privilégié du même processus. Donc donner des capabilities différentes à deux threads n'isole rien : le thread non privilégié peut détourner le privilégié.  
Une vraie frontière de sécurité exige un processus séparé (voire un namespace). C'est la raison pour laquelle les capabilities sont décrites comme un mécanisme qui élargit les droits, pas comme un sandbox qui isole.

## 4. Les cinq ensemble d'un Thread :

Chaque thread possède cinq ensembles de capabilities (`CapEff`, `CapPrm`, `CapInh`, `CapBnd`, `CapAmb`), visibles dans `/proc/<PID>/status`. De leur côté, les fichiers exécutables sur le disque ne possèdent que deux ensembles (`Permitted` et `Inheritable`) accompagnés d'un simple bit booléen (`Effective`), configurables via `setcap`.

Pour comprendre l'interaction entre ces ensembles, il faut distinguer deux moments dans le cycle de vie d'un `thread` :

1. **Pendant que le programme s'exécute :** comment le noyau contrôle les privilèges à un instant $T$ et comment le processus gère ses propres droits en mémoire.
2. **Lorsqu'un programme en lance un autre (**`execve`**) :** comment les droits sont recalculés, transmis ou supprimés au chargement d'un nouveau binaire.

#### A. Pendant l'exécution du programme (`Effective` et `Permitted`)

- **Effective (**`CapEff`**) : Les droits actifs :** C'est le **seul** ensemble que le noyau vérifie lorsqu'un `Thread` effectue un appel système privilégié (ex: ouvrir un socket brut ou monter une partition). Si la capability requise n'est pas dans `CapEff` à cet instant précis, le noyau renvoie `EPERM` (l'équivalent d'un permission denied). ***Invariant fondamental :*** `CapEff` ***est toujours un sous-ensemble de*** `CapPrm`***.***
- **Permitted (**`CapPrm`**) : La réserve maximale du** `Thread` **:** Cet ensemble définit le plafond des capabilities que le `Thread` a le droit d'activer dans `CapEff`. Une fois lancé, un `Thread` (et même un administrateur à chaud) ne peut **jamais** ajouter de nouvelles capabilities dans `CapPrm` : ce capital est fixé au démarrage du `processus`. En revanche, le `Thread` peut utiliser l'appel système `capset()` pour basculer un droit de `CapPrm` vers `CapEff`, ou pour retirer **définitivement** un droit de `CapPrm`.
  - *Cas concret :* Un démon « capability-aware » (comme `tcpdump` ou `chronyd`) démarre avec une capability dans `CapPrm`, la retire de `CapEff` pendant qu'il traite des données non fiables, l'active dans `CapEff` uniquement le temps d'un appel système critique (ouverture de socket ou réglage d'horloge), puis vide définitivement `CapEff` et `CapPrm` dès qu'il n'en a plus besoin.

#### B. Lors du lancement d'un nouveau programme  (`Inheritable`, `Ambient` et `Bounding`)

Lorsqu'un `Thread` exécute un nouveau binaire, le noyau remet `CapPrm` et `CapEff` à zéro, puis recalcule les nouveaux droits à partir des trois ensembles suivants et des attributs du fichier cible :

- **Inheritable (**`CapInh`**) : La transmission par double consentement :** Cet ensemble indique quelles capabilities le `thread` parent accepte de transmettre à l'exécutable qu'il lance. Cependant, `CapInh` seul ne donne aucun privilège : le fichier binaire exécuté doit lui aussi posséder cette même capability dans son ensemble `Inheritable` sur le disque (`F(I)`). Le noyau effectue un `ET` logique (`CapInh & F(I)`) pour déterminer ce qui entrera dans le nouveau `CapPrm`.
  - *Cas concret :* Restreindre l'usage d'un binaire privilégié à certains utilisateurs (via `pam_cap.so`) en posant uniquement le flag `i` sur l'exécutable (`setcap cap_net_raw=ie binaire`), afin que seuls les utilisateurs ayant `CAP_NET_RAW` dans leur `CapInh` de session puissent l'exécuter avec ce privilège.
- **Ambient (**`CapAmb`**) : L'héritage automatique vers les binaires standards :** Introduit pour pallier la lourdeur d'`Inheritable`, `CapAmb` permet de transmettre des capabilities à des binaires standards (scripts Python, binaires Go/Node.js) qui n'ont aucun attribut `setcap` sur le disque. Toute capability présente dans `CapAmb` survit au `execve()` et est directement injectée dans le `CapPrm` et le `CapEff` du `Thread` enfant (sauf si le binaire cible est `setuid` ou possède déjà des capabilities sur le disque, auquel cas `CapAmb` est vidé par sécurité).
  - *Invariant strict :* Une capability ne peut être placée dans `CapAmb` que si le `thread` la possède déjà à la fois dans `CapPrm` **et** dans `CapInh`. La retirer de `CapPrm` ou de `CapInh` la supprime immédiatement de `CapAmb`.
  - *Cas concret :* Un service `systemd` configuré avec `User=www-data` et `AmbientCapabilities=CAP_NET_BIND_SERVICE` permet à un serveur web non privilégié d'écouter sur le port 443 sans modifier le fichier exécutable sur le disque.
- **Bounding (**`CapBnd`**) : Le masque de restriction irréversible :** Cet ensemble agit comme un filtre de sécurité (plafond absolu) pour le  et toute sa descendance. Il masque les capabilities qu'un fichier binaire sur le disque peut octroyer lors d'un `execve()` (`F(P) & CapBnd`). On ne peut que retirer des bits de `CapBnd`, jamais en rajouter, et cette limite s'impose **même à** `root` **(UID 0)** et aux binaires `setuid root`.
  - *Cas concret :* Les runtimes de conteneurs (`runc`, Docker, Kubernetes) ou `systemd` (`CapabilityBoundingSet=`) retirent des capabilities critiques comme `CAP_SYS_MODULE` ou `CAP_SYS_TIME` de `CapBnd` afin d'empêcher un processus — même exécuté en `root` dans le conteneur — de charger un module noyau ou de modifier l'horloge de l'hôte.

<table>
  <tr><th>Ensemble</th><th>Question à laquelle il répond</th><th>Évolution possible</th></tr>
  <tr><td>Effective</td><td>Qu’ai-je en main maintenant ?</td><td>activer/désactiver depuis P</td></tr>
  <tr><td>Permitted</td><td>Qu’ai-je le droit de prendre ?</td><td>retrait seulement</td></tr>
  <tr><td>Inheritable</td><td>Que suis-je d’accord pour transmettre ?</td><td>ajout limité à ce qui est dans P (ou B avec CAP_SETPCAP)</td></tr>
  <tr><td>Ambient</td><td>Qu’est-ce qui suit automatiquement à l’exec ?</td><td>ajout si dans P et I ; disparaît si l’un des deux le perd</td></tr>
  <tr><td>Bounding</td><td>Qu’est-ce qui peut encore exister dans mon arbre de processus ?</td><td>retrait seulement, irréversible</td></tr>
</table>

#### Lire un masque hexadécimal

`/proc/PID/status` affiche des masques de 64 bits : chaque chiffre hexa code 4 capabilities, et la capability n° *n* correspond au bit *n*. Quelques numéros utiles : `CAP_NET_BIND_SERVICE` = 10, `CAP_NET_ADMIN` = 12, `CAP_NET_RAW` = 13, `CAP_SYS_ADMIN` = 21.

Exemple : `0000000000000400`. Or 0x400 = 1 << 10, donc c’est **bit 10 =** `CAP_NET_BIND_SERVICE`, seule capability présente. On peut décoder avec `capsh --decode=0000000000000400`.

### 4 Bis. Un fichier peut porter des capabilities

Les `file capabilities` remplacent le bit `setuid` : on les stocke dans l’attribut étendu `security.capability` du fichier. Un fichier porte trois informations : **F(P)** (permitted), **F(I)** (inheritable) et **un seul bit effective** F(E).

Notation de `setcap` : `cap_net_raw=ep` signifie « permitted + effective ».

<table>
  <tr><th>Notation</th><th>Sens</th><th>Pour quel programme</th></tr>
  <tr><td>=ep</td><td>la clé est déjà en main au démarrage</td><td>programmes classiques (nginx, tcpdump…)</td></tr>
  <tr><td>=p</td><td>la clé est en poche, le programme doit la sortir</td><td>programmes conçus pour (libcap)</td></tr>
  <tr><td>=i</td><td>transmissible si l’appelant l’a aussi</td><td>rare</td></tr>
</table>

Point d’attention : F(P) reste **toujours filtré par B** (le Bounding du processus appelant).

### 5. Que devient un programme qui en lance un autre ? (règle d’`execve`)

Quand vous tapez `./monprogramme` dans un shell, il se passe ceci :

1. Votre shell crée un nouveau processus enfant (via `fork()`).
2. Ce processus enfant charge `./monprogramme` via`execve()`.

La question de cette section : **à cet instant précis, quelles capabilities le nouveau programme reçoit-il ?** Il y a trois acteurs :

<table>
  <tr><th>Acteur</th><th>Notation</th><th>Exemple</th></tr>
  <tr><td>Le processus avant l’exec (celui qui lance)</td><td>P</td><td>votre shell</td></tr>
  <tr><td>Le fichier qu’on lance</td><td>F</td><td>./monprogramme</td></tr>
  <tr><td>Le processus après l’exec (le programme qui tourne)</td><td>P’</td><td>monprogramme en cours d’exécution</td></tr>
</table>

Note : après `execve()`, le PID n'est pas perdu, c'est le même processus avec un nouveau code en mémoire

#### La règle de base : par défaut, rien ne passe

C’est le point le plus important, et celui qui surprend :

> **Un programme qui démarre n’emporte PAS les capabilities de son parent.**

Même si votre shell avait des capabilities, un programme ordinaire lancé depuis ce shell démarre avec zéro. Autrement, un simple `ls` lancé par un processus privilégié hériterait de tous ses pouvoirs, ce qui serait dangereux.

Pour qu’une capability arrive dans le nouveau programme, il faut qu’une des **trois portes** s’ouvre.

```plaintext
        Le parent lance un programme (execve)
                        │
   ┌────────────────────┼─────────────────────────┐
   │                    │                         │
 Porte 1             Porte 2                  Porte 3
 Le FICHIER          L'AMBIENT du parent      L'INHERITABLE du parent
 porte lui-même      (suit automatiquement)   ET celui du fichier
 des capabilities                             (double accord)
   │                    │                         │
 (filtré par           │                         │
  le Bounding)         │                         │
   └────────────────────┴─────────────────────────┘
                        │
              = ce que le nouveau programme
                a « en poche » (Permitted)
                        │
              Interrupteur : sont-elles aussi
              « en main » (Effective) ?

```

Voyons chaque porte, puis l’interrupteur.

#### 5.1 Porte 1 : le fichier porte des capabilities

Le fichier a était marqué avant le lancement avec `setcap :`

```plaintext
sudo setcap cap_net_raw=ep ./monprogramme
```

Le `setcap` est le remplaçant du vieux `SUID`, il permet d'accorder une capability précise au lieu de tout `root`.

N'importe quel utilisateur qui lance `./monprogramme` l'utilisera avec la capability `CAP_NET_RAW` sans être root.

Mais attention, le `Bounding` est là, et fait office de garde fou, si une capability à été retirée du `Bounding` du parent, elle ne passera pas, même si le fichier la porte en lui même. Le `Bounding` parent est le plafond absolu.

#### 5.2 Porte 2 : l'ambient du parent

Le parent a placé des capabilities dans son `CapAmb` (via `prctl(PR_CAP_AMBIENT_RAISE)`). Celles-ci suivent automatiquement le nouveau programme : elles sont injectées directement dans `CapPrm` et dans `CapEff`.

Avantage : le fichier n'a besoin d'aucun marquage `setcap`. C'est la porte idéale pour les binaires non préparés (scripts Python, binaires Go/Node.js).

Deux réserves de sécurité :

- `CapAmb` doit satisfaire l'invariant : chaque capability doit être à la fois dans `CapPrm` et `CapInh`.
- `CapAmb` est vidé si le binaire cible est `setuid`/`setgid` ou possède déjà des capabilities sur le disque.

#### 5.3 Porte 3 : le double accord Inheritable

Il faut ici deux consentements :

- le parent a la capability dans son `CapInh` (il accepte de la transmettre) ;
- le fichier l'a dans son Inheritable, noté `F(I)` (il accepte de la recevoir).

Si les deux sont d'accord, leur intersection entre dans le `CapPrm` du nouveau programme :

```plaintext
CapInh & F(I)
```

C'est la porte la plus contraignante, et la raison pour laquelle l'ambient (porte 2) a été inventée : elle évite d'avoir à marquer le fichier.

#### 5.4 L'interrupteur : « en poche » vs « en main »

Les trois portes remplissent le `CapPrm`, c'est-à-dire ce que le programme a en poche. Mais on l'a vu au tout début : le noyau ne vérifie que `CapEff`, ce que le programme a en main.

Deux façons d'avoir une capability aussi dans `CapEff` dès le démarrage :

- Le fichier a le bit Effectif (le `e` du `setcap`) : alors toutes les capabilities nouvellement Permitted deviennent également Effectives.
- La capability vient de l'ambient : elle est Effective automatiquement.

Sinon, le programme démarre avec la capability en poche mais pas en main, et devra l'activer lui-même via `capset()` le temps de l'usage.

#### 5.4 La formule (résumé)

Maintenant que les trois portes et l'interrupteur sont compris, la formule officielle n'est plus qu'une façon compacte de redire la même chose.

| Symbole | Signification                                                      |
| ------- | ------------------------------------------------------------------ |
| `P`     | les capabilities du parent **avant** l'`execve`                    |
| `F`     | les capabilities du **fichier** lancé (celles posées par `setcap`) |
| `P'`    | les capabilities du **nouveau programme** après l'`execve`         |
| `&`     | ET logique (intersection : il faut les deux)                       |
| `\|`    | OU logique (union : une seule porte suffit)                        |

#### La formule

```text
P'(permitted) =   ( P(inheritable) & F(inheritable) ) | ( F(permitted) & P(bounding) ) | P'(ambient)

P'(effective) = F(effective) ? P'(permitted) : P'(ambient)

```

#### Comment la lire

La première ligne dit simplement : **le nouveau Permitted est la réunion (**`|`**) de trois contributions**, une par porte.

| Terme                             | Porte | Traduction en français                                          |
| --------------------------------- | ----- | --------------------------------------------------------------- |
| `P(inheritable) & F(inheritable)` | 3     | le double accord : le parent **et** le fichier doivent accepter |
| `F(permitted) & P(bounding)`      | 1     | les caps du fichier, **filtrées** par le Bounding du parent     |
| `P'(ambient)`                     | 2     | l'ambient du parent, qui suit automatiquement                   |

La deuxième ligne est **l'interrupteur** :

- `F(effective) ?` → « si le fichier a le bit Effectif » (le `e` du `setcap`) :
  - **oui** → tout ce qui vient d'entrer dans le Permitted devient aussi **Effectif** ;
  - **non** → on ne met en Effectif que ce qui vient de l'**ambient** (qui est actif automatiquement).

Autrement dit : les portes remplissent la **poche** (`Permitted`), et l'interrupteur décide ce qui passe en **main** (`Effective`).

#### 5.5 Le cas particulier de root

Pour un processus d’UID 0 (root), le noyau applique un raccourci : il traite **tous les fichiers comme s’ils portaient toutes les capabilities, avec le bit effective à 1**. Résultat : un programme lancé par root reçoit **tout ce que son Bounding permet**, et directement en main.

C’est ce qui rend root « tout-puissant ». Mais cela veut dire aussi que **si on retire une capability du Bounding, root la perd** (le plafond s’applique toujours).

#### 5.6  Le piège du Bounding

Regardez la formule : le Bounding n'apparaît que sur une seule ligne, celle de la porte 1 (`F(permitted) & Bounding`). Il n'apparaît pas sur la porte 3 (`P(inheritable) & F(inheritable)`).

Je cite le man page sur Linux :

> « the bounding set masks the file permitted capabilities, but not the inheritable capabilities. If a thread maintains a  
> capability in its inheritable set that is not in its bounding set, then it can still gain that capability in its │ permitted set by executing a file that has the capability in its inheritable set. »

Traduction : retirer une capability du Bounding bloque bien les étiquettes de fichier (porte 1), mais pas un double accord Inheritable (porte 3) déjà en place. Une capability présente à la fois dans l'Inheritable du processus et dans celui du fichier arrivera donc « en poche » après l'`execve`, même si elle a été retirée du Bounding.

Nuance importante : depuis Linux 2.6.25, on ne peut pas ajouter une capability absente du Bounding dans son Inheritable via `capset()`. Le piège ne joue donc que pour une capability déjà présente dans l'Inheritable avant le retrait du Bounding (ou héritée telle quelle).

Le piège ne se produit donc que dans un cas précis :

> La capability a été rangée dans l'Inheritable avant le retrait du Bounding (ou elle a été héritée telle quelle d'un │ parent)

Conseil pratique : quand vous durcissez un service, retirez du Bounding ce qui n'est pas nécessaire, et vérifiez que les ensembles non bornés sont vides ou minimaux dans `/proc/PID/status` :

- `CapInh` (Inheritable) : le vecteur du piège ci-dessus ;
- `CapAmb` (Ambient) : lui non plus n'est pas filtré par le Bounding à l'`execve`.

#### 5.6 La remise à zéro : fichier « privilégié » = ambient vidé

Une dernière règle de sécurité. Quand le fichier lancé est **privilégié** (il a le bit setuid/setgid, ou des file capabilities), **l’ambient est remis à zéro** pour ce nouveau programme.

Pourquoi ? Pour éviter qu’un programme déjà privilégié ne soit détourné en lui injectant, via l’ambient, des pouvoirs en plus. Un programme à étiquette ne reçoit que ce que son étiquette dit.

#### 5.7 La logique en cinq questions :

Pour savoir ce qu’un programme obtiendra, posez ces questions dans l’ordre :

1. **Le processus qui lance est-il root (UID 0) ?** Si oui : il reçoit tout son `Bounding`, en main. Fin de la discussion.
2. **Le fichier porte-t-il des capabilities (F(P)) ?** Si oui, elles arrivent en poche, filtrées par le Bounding (porte 1).
3. **Le parent a-t-il de l’ambient, et le fichier est-il non privilégié ?** Si oui, l’ambient arrive en poche **et** en main (porte 2). Si le fichier est privilégié : l’ambient est perdu.
4. **Le parent et le fichier ont-ils tous deux la capability en Inheritable ?** Si oui, elle arrive en poche (porte 3).
5. **Le fichier a-t-il le bit effective ?** Si oui, tout ce qui est en poche passe en main. Sinon, seul ce qui vient de l’ambient est en main.

Ce qui reste inchangé : le **Inheritable** et le **Bounding** du nouveau programme sont ceux de son parent.

#### 5.8 Exemples concrets, un par un

Notation : `B` = Bounding. Dans tous les cas le parent est un utilisateur normal, sauf indication contraire.

<table>
  <tr><th>#</th><th>Situation</th><th>Résultat</th><th>Explication</th></tr>
  <tr><td>1</td><td>Fichier avec cap_net_raw=ep. Parent sans rien.</td><td>Poche = {net_raw} ; main = {net_raw}</td><td>Porte 1, bit E à 1 : active dès le départ</td></tr>
  <tr><td>2</td><td>Fichier avec cap_net_raw=p (sans E). Parent sans rien.</td><td>Poche = {net_raw} ; main = ∅</td><td>Porte 1, mais pas de bit E : le programme doit l’activer lui-même</td></tr>
  <tr><td>3</td><td>Fichier ordinaire. Parent avec net_bind_service en ambient.</td><td>Poche = {bind} ; main = {bind} ; ambient = {bind}</td><td>Porte 2 : l’ambient suit</td></tr>
  <tr><td>4</td><td>Fichier ordinaire. Parent avec net_raw en Inheritable seulement.</td><td>Poche = ∅ ; main = ∅</td><td>Le fichier n’a pas donné son accord (Inheritable côté fichier)</td></tr>
  <tr><td>5</td><td>Fichier avec net_raw=ei. Parent avec net_raw en Inheritable.</td><td>Poche = {net_raw} ; main = {net_raw}</td><td>Porte 3 : double accord ; bit E à 1 donc en main</td></tr>
  <tr><td>6</td><td>Fichier avec net_raw=p. Parent avec bind en ambient.</td><td>Poche = {net_raw} ; main = ∅ ; ambient = ∅</td><td>Fichier privilégié : l’ambient est perdu (§5.8)</td></tr>
  <tr><td>7</td><td>Fichier avec net_raw=ep. Parent dont le Bounding n’a pas net_raw.</td><td>Poche = ∅ ; main = ∅</td><td>Le plafond filtre la porte 1</td></tr>
  <tr><td>8</td><td>Parent root, fichier ordinaire, Bounding complet.</td><td>Poche = main = Bounding complet</td><td>Raccourci root</td></tr>
  <tr><td>9</td><td>Parent root, fichier ordinaire, Bounding sans net_raw.</td><td>Poche = main = tout sauf net_raw</td><td></td></tr>
</table>
