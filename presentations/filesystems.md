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
# Logging

Le logging sous Linux est en fait un peu disparate.

---
# Logging (kernel)

Les messages de boot sont accédés par la commande `dmesg`:

```
[jpbelang@localhost mnt]$ dmesg
[    0.000000] Initializing cgroup subsys cpuset
[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Initializing cgroup subsys cpuacct
[    0.000000] Linux version 3.10.0-862.11.6.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) ) #1 SMP Tue Aug 14 21:49:04 UTC 2018
[    0.000000] Command line: BOOT_IMAGE=/vmlinuz-3.10.0-862.11.6.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8
[    0.000000] e820: BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
[    0.000000] BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x000000007ffeffff] usable
[    0.000000] BIOS-e820: [mem 0x000000007fff0000-0x000000007fffffff] ACPI data
[    7.899586] ata3: SATA link up 3.0 Gbps (SStatus 123 SControl 300)
[    7.899730] ata3.00: ATA-6: VBOX HARDDISK, 1.0, max UDMA/133
[    7.899733] ata3.00: 41943040 sectors, multi 128: LBA48 NCQ (depth 31/32)
[    7.899955] ata3.00: configured for UDMA/133
[    7.900081] scsi 2:0:0:0: Direct-Access     ATA      VBOX HARDDISK    1.0  PQ: 0 ANSI: 5
[    8.208084] ata4: SATA link up 3.0 Gbps (SStatus 123 SControl 300)
[    8.208404] ata4.00: ATA-6: VBOX HARDDISK, 1.0, max UDMA/133
[    8.208407] ata4.00: 113584 sectors, multi 128: LBA48 NCQ (depth 31/32)
[    8.208759] ata4.00: configured for UDMA/133
[    8.208903] scsi 3:0:0:0: Direct-Access     ATA      VBOX HARDDISK    1.0  PQ: 0 ANSI: 5
[    8.388891] sr 1:0:0:0: [sr0] scsi3-mmc drive: 32x/32x xa/form2 tray
[    8.388921] cdrom: Uniform CD-ROM driver Revision: 3.20
[    8.389330] sr 1:0:0:0: Attached scsi CD-ROM sr0
[    8.396350] sd 2:0:0:0: [sda] 41943040 512-byte logical blocks: (21.4 GB/20.0 GiB)
[    8.396401] sd 2:0:0:0: [sda] Write Protect is off
[    8.396404] sd 2:0:0:0: [sda] Mode Sense: 00 3a 00 00
[    8.396464] sd 2:0:0:0: [sda] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
[    8.398398]  sda: sda1 sda2
[    8.398693] sd 2:0:0:0: [sda] Attached SCSI disk
[    8.400905] sd 3:0:0:0: [sdb] 113584 512-byte logical blocks: (58.1 MB/55.4 MiB)
[    8.400955] sd 3:0:0:0: [sdb] Write Protect is off
[    8.400958] sd 3:0:0:0: [sdb] Mode Sense: 00 3a 00 00
[    8.400980] sd 3:0:0:0: [sdb] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
[    8.402186] sd 3:0:0:0: [sdb] Attached SCSI disk
00:00:00.012924 main     Package type: LINUX_64BITS_GENERIC
[   47.346650] 00:00:00.021673 main     5.2.18 r124319 started. Verbose level = 0
[  130.985596] e1000: enp0s3 NIC Link is Down
[  132.986639] e1000: enp0s3 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
[  281.265049] e1000: enp0s3 NIC Link is Down
[  285.272735] e1000: enp0s3 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
[  331.352821] e1000: enp0s3 NIC Link is Down
[  333.358971] e1000: enp0s3 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
[  974.676607] e1000: enp0s3 NIC Link is Down
[  976.679478] e1000: enp0s3 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
```

On y voit les disques, interfaces réseau et autres periphériques apparaitre et se configurer.

---
# Logging

Le logging est géré apr un système nomme `rsyslogd`, une implémentation de syslog pour CentOS.  Il supporte toutes 
sortes de logging, mais spécifiquement, en fichier et réseau.

* Les fichiers de configuration sont dans `/etc/rsyslog.conf` et sous `/etc/rsislog.d`.

* rsyslog permet de classer les messages de logging des applications par `facility` et `priority`.  On peut utiliser 
ces champs pour router les messages dans les bons fichiers.

```
#### RULES ####

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure
# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog
# Log cron stuff
cron.*                                                  /var/log/cron
# Everybody gets emergency messages, via email
*.emerg                                                 :omusrmsg:*
# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler
# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
``` 
---
# Logging

Il y a quelques raisons pour lesquelles on ne parlera pas trop de ces choses:

* Les grandes applications (java, Apache, ...) ont leur propre système de logging.

* Il existe de bien meilleurs systèemes d'aggégation de logs dans des environments virtualisés.

* Différentes versions de syslog opèrent de manières différentes.  Toutes supportent des concepts similaires, mais toutes
le font de façon assez différentes.  Read ze docs.
