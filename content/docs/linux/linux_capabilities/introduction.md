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

- **Un processus normal a zéro capability.** mais il accède bien à la mémoire et au disque, surprenant n'est ce pas ? Eh bien cet accès est garanti parcequ'il passe par les permissions de fichiers classiques. Les capabilities ne servent qu'aux tâches dites "Système" (mounts, réseau, etc.)

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

Sans capability, quand un processus envoie un signal, le noyau applique une règle simple : l’UID réel ou effectif de l’expéditeur doit correspondre à l’UID réel ou sauvegardé de la cible. Autrement dit, vous ne pouvez tuer que vos propres processus. C’est pourquoi alice ne peut pas tuer un processus de bob ni un processus de root.

CAP_KILL ne dit pas « tu peux tuer le processus 1234 ». Elle dit : « ce test d’UID ne s’applique plus à toi ». Le noyau ne regarde plus à qui appartient la cible, ce qui vaut pour tous les processus visibles, ceux de root compris.




- **Une capability n'est pas un droit sur un objet *précis*.**

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

