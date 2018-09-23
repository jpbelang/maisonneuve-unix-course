    
# Unix, la base

## jpbelang@gmail.com

---

# Qu'est-ce qu'un système d'exploitation ?

C'est du logiciel qui permet de gérer les ressources d'un ordinateur.
* Il contrôle les accès aux périphériques.
* Il assigne l'utilisation de la mémoire (mémoire virtuelle).
* Il distribue l'utilisation des processeurs (céduleur). 
* Il offre une façon de présenter des données (système de fichiers).
* Il offre aussi une manière de lancer des applications (processus).

---

# Unix, c'est vieux

La première version d'Unix a été développée en 1970, à AT&T.  Depuis ce temps, plusieurs groupes ont développé leur propre version d'Unix.

* L'Université de Berkeley a développé une version d'Unix (BSD Unix)
* Sun Microsystems (Solaris), IBM (AIX), DEC (OSF, Tru64).

Dans toutes ces versions d'Unix, l'interface s'est solidifiée.

---

# Linux 

Au travers de tous ces groupes, Linus Torvalds développe un noyau Unix gratuit. 
* Il est basé sur un autre noyau gratuit:  Minix.
* Il attire beaucoup de développeurs et le noyau grandit.
* Il existe présentement deux grandes familles Linux: les versions descendues par RedHat(RHEL, Fedora, CentOS), et les versions descendues de Debian (Ubuntu, Mint).
* Il existe d'autres versions Unix gratuites (OpenBSD, FreeBSD, NetBSD).

---

# Le noyau (kernel)

Dans le monde Unix, la base du système d'exploitation s'appelle le noyau.  C'est effectivement le niveau le plus bas du système d'exploitation.

* Linux, en fait, n'est qu'un kernel. On bâti des distributions autour du kernel (Red Hat, Ubuntu)
* 99% des développeurs ne développent pas vraiment dans le noyau:  ils utilisent les services du noyau.

---

# Les processus

Un processus est en fait une application qui exécute sous Linux.

`% sleep 10`

Cette commande lance la commande sleep (qui dormira 10 secondes et terminera).
On peut lancer les commandes en arrière plan.

`% sleep 10 &`

---
# Lister les processus

On peut lister les processus qui exécutent sur le système avec la commande ps:
```
  PID TTY          TIME CMD
25670 pts/0    00:00:00 bash
25693 pts/0    00:00:00 ps
```
---

# ps aux

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.2  0.3  45976  6300 ?        Ss   08:33   0:04 /usr/lib/systemd...
root         2  0.0  0.0      0     0 ?        S    08:33   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    08:33   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   08:33   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        S    08:33   0:01 [kworker/u2:0]
jpbelang 25611  0.0  0.1 115572  2312 tty2     Ss+  09:01   0:00 -bash
root     25665  0.0  0.3 158788  5692 ?        Ss   09:02   0:00 sshd: jpbelang [priv]
jpbelang 25669  0.0  0.1 158788  2604 ?        S    09:02   0:00 sshd: jpbelang@pts/0
jpbelang 25670  0.0  0.1 115436  2032 pts/0    Ss   09:02   0:00 -bash
```
---
# Parenthèse de sécurité

Le modèle de sécurité Unix, en général, est très simple: il y a l'usager root, et il y a le reste de la population.

* root peut tout faire, dans à peu près toutes les circonstances.
* Les autres ne peuvent, en général, qu'affecter leur vie personnelle (sauf dans le cas des permissions de fichier).

Dans la vraie vie, on n'opère que dans son compte personnel, et les opérations administratives sont gérées par la commande sudo.

---

# Status

D: bloqué en entrée-sortie.

R: essaie d'exécuter.
             
S: en attente d'un événement.
             
T: arrêté par un terminal.
             
t:  arrêté en mode debug.
             
Z: Zombie == application problématique.

---

# Kill - Arrêt des processus

La commande kill est une manière, en fait, d'envoyer un signal à un processus.  Accidentellement, la majorité des signaux tuent les processus.

`% kill 21345`

Tue le processus 21345 en envoyant le signal 15 (`TERM`).
La plupart des signaux peuvent être interceptés par l'application.  Seuls les signaux 9(`KILL`) et 19(`STOP`) ne peuvent pas être interceptés.

---
# Autres commandes de gestion de processus

`top`: Liste les processus en ordre de gourmandise.

`pgrep`: Liste les PIDs de certains processus par leur nom (généralement).

`pkill`:  Envoie un signal auxdits processus.

---

# nice

On peut être gentil:

```
jpbelang     25872  0.5  0.2 218512  4044 pts/0    S    09:53   0:00 nice -n 10 sleep 30
jpbelang     25880  0.0  0.0 107948   352 pts/0    SN   09:54   0:00 sleep 30
```
`root` peut être méchant
```
root     25917  1.0  0.2 218516  4016 pts/0    S    09:54   0:00 sudo nice -n -10 sleep 30
root     25918  0.1  0.0 107948   348 pts/0    S<   09:54   0:00 sleep 30
```

---
# nohup

Nohup permet à une commande d'exécuter sans avoir de terminal contrôlant.  
Il est important de lancer des tâches qui doivent exécuter longtemps avec la commande nohup, 
sinon, elles pourraient terminer de manière inattendue.
