# Le système de fichiers

---
# Les périphériques

Les périphériques se trouvent dans le répertoire `/dev`.
```
[jpbelang@localhost dev]$ ls -la
total 0
crw-rw-rw-.  1 root    root      1,   3 Oct 18 13:20 null
brw-rw----.  1 root    disk      8,   0 Oct 18 13:20 sda
brw-rw----.  1 root    disk      8,   1 Oct 18 13:20 sda1
brw-rw----.  1 root    disk      8,   2 Oct 18 13:20 sda2
brw-rw----.  1 root    disk      8,  16 Oct 18 13:20 sdb
crw-rw-rw-.  1 root    tty       5,   0 Oct 18 13:20 tty
crw--w----.  1 root    tty       4,   0 Oct 18 13:20 tty0
crw--w----.  1 root    tty       4,   1 Oct 18 13:21 tty1
```

---
# Les périphériques

`sdaX`: ce sont les disques dure, en forme "raw".  Ils sont inutilisables sous cette forme, à part pour le processus de 
montage et démontage. Le `b` indique que c'est un périphérique qui fonctionne en mode bloc.

`ttyX`: ce sont les terminaux (la console, par exemple).  Le `c` indique que ce periphérique fonctionne en mode caractère
par caractère.

`null`: c'est un périphérique qui peut servir de poubelle our les sorties.  On utilise la redirection pour se débarasser de ces données:
`ls -l > /dev/null` fait disparaître les sorties.

---
# Les disques

Un disque, sous Linux est un périphérique avec un système de fichiers formatté de manière à organiser les données de façon efficace.
Il existe plusieurs types de système de fichiers dans CentOS.

* ext3, ext4:  ce sont des types souvent utilisé dans les distributions Linux. Ce sont en fait les systèmes de fichiers "natifs" de Linux.  
Ils sont en fait assez efficaces et supportent de très grands disques.

* xfs:  c'est un système de fichier originalement développé chez Silicon Graphics Incorporated (SGI).  En terme de caractéristiques, ses gains
sont plus apparents pour de très grands disques ( > 16 TB).

---
# Les disques

Comment trouve-t-on quels disques sont connectés à notre système ?

```
[jpbelang@localhost dev]$ sudo fdisk -l

Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00048637

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM

Disk /dev/sdb: 58 MB, 58155008 bytes, 113584 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

# Disques /dev/mapper omis....
```
---
# Les disques

Donc notre système a deux disques physiques:  `/dev/sda` et `/dev/sdb`.  Le premier est sous divisé en partitions (1 et 2).
On pourrait partitionner notre nouveau disque `/dev/sdb`, mais ça ne ferait que nous contraindre:  chaque partition est considérée
comme un disque séparé.  Pour les connecter, il nous faudrait utiliser LVM.

Notre disque n'est pas de système de fichiers installé.  La commande utilisée pour ajouter un système de fichiers 
est `mkfs` (MaKe FileSystem).

On l'utilise comme suit:
```
[jpbelang@localhost dev]$ sudo mkfs -t ext4 /dev/sdb
[sudo] password for jpbelang: 
mke2fs 1.42.9 (28-Dec-2013)
/dev/sdb is entire device, not just one partition!
Proceed anyway? (y,n) y
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
14224 inodes, 56792 blocks
2839 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33685504
7 block groups
8192 blocks per group, 8192 fragments per group
2032 inodes per group
Superblock backups stored on blocks: 
	8193, 24577, 40961

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
---
# Les disques

On peut aussi utiliser d'autres types de systàmes de fichier.
```
[jpbelang@localhost dev]$ sudo mkfs -t xfs /dev/sdb
meta-data=/dev/sdb               isize=512    agcount=2, agsize=7099 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=14198, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=855, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

Les sorties sont différentes dépendant du type de systeme de fichier.

```
[jpbelang@localhost dev]$ sudo yum install dosfstools
...pour installer les outils...

[jpbelang@localhost dev]$ sudo mkfs -t vfat -I -v /dev/sdb
mkfs.fat 3.0.20 (12 Jun 2013)
/dev/sdb has 255 heads and 63 sectors per track,
logical sector size is 512,
using 0xf8 media descriptor, with 113584 sectors;
filesystem has 2 16-bit FATs and 4 sectors per cluster.
FAT size is 112 sectors, and provides 28331 clusters.
There is 1 reserved sector.
Root directory contains 512 slots and uses 32 sectors.
Volume ID is b90e736f, no volume label.
```

Voila, un systeme de fichiers vfat installé....

---
# Les disques

Le périphérique a maintenant une structure qui permet de créer des fichiers et répertoires.  On doit maintenant monter le
disque.

```
[jpbelang@localhost mnt]$ df -k .
Filesystem              1K-blocks    Used Available Use% Mounted on
/dev/mapper/centos-root  17811456 1739896  16071560  10% /

... Le répertoire est dur le périphérique /dev/mapper/centos-root ...

[jpbelang@localhost /]$ sudo mount /dev/sdb /mnt/
jpbelang@localhost mnt]$ cd /mnt
[jpbelang@localhost mnt]$ df -k .
Filesystem     1K-blocks  Used Available Use% Mounted on
/dev/sdb           53372  2948     50424   6% /mnt

... Le répertoire est sur le périphérique /dev/sdb ...
```
---
# Les disques

Choses à réaliser:

* Le contenu du répertoire monté sera "caché" par le nouveau périphérique.  Aucune donné n'est perdue, mais elles sont 
entièrements  inaccessibles.

* Un disque peut être monté sur un autre disque a été monté, etc...  En fait, le noyau monte automatiquement `/` et 
tout le reste s'attache ensemble a partir de `/`.

* Om peut dé-monter un disque: `umount /mnt`, par exemple.  Personne ne doit utiliser le disque (soit en étant assis 
dans un des ses répertoires ou bien en tenant un de ses fichiers ouverts) 

---
# fstab

C'est en écrivant dans ce fichier que l'on s'assure que le montage des répertoires est persistent (survit au reboot)
```
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=5fce7a9b-bd48-41da-b4b7-f9519fd54ace /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/sdb  /mnt ext4 defaults 0 0
``` 
---


