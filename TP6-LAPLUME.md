`Sorenza LAPLUME`

# TP 6 - Gestion des disques /Tâches d’administration (1)

Dans ce sixième TP, nous allons manipuler divers outils de gestion des disques et partitions, ainsi que le partitionnement
LVM. Nous verrons comment créer des points de montage et monter des partitions.

## Exercice 1. Disques et partitions


### 1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM

### 2. Vérifiez que ce nouveau disque dur est bien détecté par le système

Ce nouveau disque dur est bien détecté par le système nous pouvons le voir avec la commande **lsblk**

<pre>
laplume@serveur:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0   89M  1 loop /snap/core/7713
loop1    7:1    0 89,1M  1 loop /snap/core/7917
loop2    7:2    0 54,6M  1 loop /snap/lxd/11985
loop3    7:3    0 54,7M  1 loop /snap/lxd/12181
sda      8:0    0   10G  0 disk
├─sda1   8:1    0    1M  0 part
└─sda2   8:2    0   10G  0 part /
sdb      8:16   0    5G  0 disk ==> nouveau disque créer
sr0     11:0    1 1024M  0 rom
</pre>

### 3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)

<pre>
laplume@serveur:~$ sudo fdisk /dev/sdb
Command (m for help): p
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3ea4d04e

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (4196352-10485759, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-10485759, default 10485759): +3G
Value out of range.
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-10485759, default 10485759):

Created a new partition 2 of type 'Linux' and of size 3 GiB.
</pre>

Ici je dois changer Linux en NTFS pour cela j'utilise les commande suivante:
<pre>
Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 7
</pre>

je vérifie avec la commande lsblk:
<pre>
laplume@serveur:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0   89M  1 loop /snap/core/7713
loop1    7:1    0 54,7M  1 loop /snap/lxd/12181
loop2    7:2    0 54,6M  1 loop /snap/lxd/11985
loop3    7:3    0 89,1M  1 loop /snap/core/7917
sda      8:0    0   10G  0 disk
├─sda1   8:1    0    1M  0 part
└─sda2   8:2    0   10G  0 part /
sdb      8:16   0    5G  0 disk
├─sdb1   8:17   0    2G  0 part
└─sdb2   8:18   0    3G  0 part
sr0     11:0    1 1024M  0 rom
</pre>

L'on constate bien la création des 2 partition dans sdb.

### 4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions (pensez à consulter le manuel !)

<pre>
laplume@serveur:~$ sudo mkfs.ext3 -b 4096 /dev/sdb1
mke2fs 1.44.6 (5-Mar-2019)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: c2473ea6-99ab-46e3-908f-669513c93a24
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

laplume@serveur:~$ sudo mkfs.ntfs /dev/sdb2
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
</pre>

### 5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?

La commande df -T ne fonctionne pas sur notre disque car l'on à pas encore monter le système de fichier. 

### 6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)

L'on commence par créer 2 points de montage à la racine : /
**sudo mkdir /data**
**sudo mkdir /win**

Puis l'on procède au montage avec les commandes :

<pre>
laplume@serveur:~$ sudo mount /dev/sdb1 /data
laplume@serveur:~$ sudo mount /dev/sdb2 /win
</pre>


### 7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration

<pre>
/dev/sdb1 on /data type ext3 (rw,relatime)
/dev/sdb2 on /win type fuseblk (rw,relatime,user_id=0,group_id=0,allow_other,blksize=4096)
</pre>


### 8. Montez votre clé USB dans la VM



### 9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox

## Exercice 2. Partitionnement LVM

Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler
les disques et les partitions.

### 1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab

<pre>
laplume@serveur:~$ sudo umount /dev/sdb2 /win
umount: /dev/sdb2: not mounted.
umount: /win: not mounted.
laplume@serveur:~$ sudo umount /dev/sdb1 /data
umount: /data: not mounted.
</pre>

le fichier /etc/fstab contient maintenant :
<pre>
UUID=bfc9fd35-cfdd-494d-8d8e-e208696b50d4 / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
</pre>

### 2. Supprimez les deux partitions du disque, et créez une partition unique de type LVM La création d’une partition LVM n’est pas indispensable, mais vivement recommandée quand on utilise LVM sur un disque entier. En effet, elle permet d’indiquer à d’autres OS ou logiciels de gestion de disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données sur ce disque. Attention à ne pas supprimer la partition système !

Pour supprimer les deux partitions du disque j'utilise :
<pre>
sudo fdisk /dev/sdb
Command (m for help): d
Partition number (1,2, default 2): 1

Partition 1 has been deleted.

Command (m for help): d
Selected partition 2
Partition 2 has been deleted.

Command (m for help):n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.
Partition #1 contains a ext3 signature.

Do you want to remove the signature? [Y]es/[N]o: yes

The signature will be removed by a write command.

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e

</pre>



### 3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay. Toutes les commandes concernant les volumes physiques commencent par pv. Celles concernant les groupes de volumes commencent par vg, et celles concernant les volumes logiques par lv.

<pre>
root@serveur:~# pvcreate sdb1 /dev/sdb1
  Device sdb1 not found.
  Physical volume "/dev/sdb1" successfully created.

  root@serveur:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               VolumeGroupTest
  PV Size               2,00 GiB / not usable 4,00 MiB
  Allocatable           yes
  PE Size               4,00 MiB
  Total PE              511
  Free PE               511
  Allocated PE          0
  PV UUID               Uo45Vr-QZQi-prWr-1cU7-V8cl-roY9-gRxCVx

</pre>

### 4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay. Par convention, on nomme généralement les groupes de volumes vgxx (où xx représente l’indice du groupe de volume, en commençant par 00, puis 01...)

<pre>
root@serveur:~# vgcreate VolumeGroupTest /dev/sdb1
Volume group "VolumeGroupTest" successfully created

root@serveur:~# vgdisplay
  --- Volume group ---
  VG Name               VolumeGroupTest
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <2,00 GiB
  PE Size               4,00 MiB
  Total PE              511
  Alloc PE / Size       0 / 0
  Free  PE / Size       511 / <2,00 GiB
  VG UUID               OKvgg0-wzpI-mJuv-omSt-qicS-i9u9-FNo4Oy
</pre>

### 5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible. On peut renseigner la taille d’un volume logique soit de manière absolue avec l’option -L (par exemple -L 10G pour créer un volume de 10 Gio), soit de manière relative avec l’option -l : -l 60%VG pour utiliser 60% de l’espace total du groupe de volumes, ou encore -l 100%FREE pour utiliser la totalité de l’espace libre.

<pre>
root@serveur:~# lvcreate -n lvdata -l  100%FREE VolumeGroupTest
  Logical volume "lvdata" created.

root@serveur:~# lvdisplay
  --- Logical volume ---
  LV Path                /dev/VolumeGroupTest/lvdata
  LV Name                lvdata
  VG Name                VolumeGroupTest
  LV UUID                pdrcVB-8Y7z-NElN-1llu-Ewuu-xZ3o-jvXTAb
  LV Write Access        read/write
  LV Creation host, time serveur, 2019-10-14 19:27:01 +0000
  LV Status              available
  # open                 0
  LV Size                <2,00 GiB
  Current LE             511
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
</pre>

### 6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data. A ce stade, l’utilité de LVM peut paraître limitée. Il trouve tout son intérêt quand on veut par exemple agrandir une partition à l’aide d’un nouveau disque.

<pre>
root@serveur:~# mkfs.ext4 /dev/VolumeGroupTest/lvdata
mke2fs 1.44.6 (5-Mar-2019)
Found a dos partition table in /dev/VolumeGroupTest/lvdata
Proceed anyway? (y,N) y
Creating filesystem with 523264 4k blocks and 130816 inodes
Filesystem UUID: 7e728432-36a5-4429-a090-73fc0686608c
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
</pre>

modification du ficher /etc/fstab :
<pre>
Ajout de la ligne UUID=7e728432-36a5-4429-a090-73fc0686608c /ext4 defaults 0 0
</pre>

monter du volume :
<pre>
mount /dev/VolumeGroupTest/lvdata /data
mount /dev/VolumeGroupTest/lvdata /data
</pre>

### 7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.

<pre>
laplume@serveur:~$ lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                        7:0    0 54,7M  1 loop /snap/lxd/12181
loop1                        7:1    0 89,1M  1 loop /snap/core/7917
loop2                        7:2    0 54,6M  1 loop /snap/lxd/11985
loop3                        7:3    0   89M  1 loop /snap/core/7713
sda                          8:0    0   10G  0 disk
├─sda1                       8:1    0    1M  0 part
└─sda2                       8:2    0   10G  0 part /
sdb                          8:16   0    5G  0 disk
└─sdb1                       8:17   0    2G  0 part
  └─VolumeGroupTest-lvdata 253:0    0    2G  0 lvm
sdc                          8:32   0   10G  0 disk
sr0                         11:0    1 1024M  0 rom
</pre>
Le nouveau disque porte le nom **sdc**.

### 8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes

<pre>
root@serveur:~# vgextend VolumeGroupTest /dev/sdc
  Physical volume "/dev/sdc" successfully created.
  Volume group "VolumeGroupTest" successfully extended
</pre>

### 9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs. Il est possible d’aller beaucoup plus loin avec LVM, par exemple en créant des volumes par bandes (l’équivalent du RAID 0) ou du mirroring (RAID 1). Le but de cet exercice n’était que de présenter les fonctionnalités de base.

<pre>
root@serveur:~# lvresize -L +3G /dev/VolumeGroupTest/lvdata
  Size of logical volume VolumeGroupTest/lvdata changed from <2,00 GiB (511 extents) to <5,00 GiB (1279 extents).
  Logical volume VolumeGroupTest/lvdata successfully resized.
</pre>

<pre>
root@serveur:~# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                        7:0    0   89M  1 loop /snap/core/7713
loop1                        7:1    0 54,6M  1 loop /snap/lxd/11985
loop2                        7:2    0 89,1M  1 loop /snap/core/7917
loop3                        7:3    0 54,7M  1 loop /snap/lxd/12181
sda                          8:0    0   10G  0 disk
├─sda1                       8:1    0    1M  0 part
└─sda2                       8:2    0   10G  0 part /
sdb                          8:16   0    5G  0 disk
└─sdb1                       8:17   0    2G  0 part
  └─VolumeGroupTest-lvdata 253:0    0    5G  0 lvm
sdc                          8:32   0   10G  0 disk
└─VolumeGroupTest-lvdata   253:0    0    5G  0 lvm
sr0                         11:0    1 1024M  0 rom
</pre>

## Exercice 3. Exécution de commandes en différé : at et cron

### 1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez entre temps que la tâche est bien programmée.

<pre>
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*/3 3   * * *  echo "La réunion aura lieu dans 3 minutes." >>~/readme
</pre>

J'ai ajouté la ligne ***"*/3 3   * * *  echo "La réunion aura lieu dans 3 minutes." >> ~/readme"**

### 2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par exemple en vous aidant des logs, du manuel...)

La réponse ne s'est pas affiché.

### 3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple, l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.

*/3 * * * * echo "Il faut réviser pour l'examen !"

### 4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure

Tous les jours:
> 0 0 * * * echo "Il faut réviser pour l'examen !" 
Toute l'année:
> 0 0 1 1 * echo "Il faut réviser l'examen !"
Tous les quarts d'heure:
> */15 * * * * echo "Il faut réviser l'examen !"

### 5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18 heures les 1er et 15 du mois :

### 6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures

0 17 * * 1-5 echo "Il faut réviser l'examen !"

### 7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un fichier de log situé dans votre dossier personnel

### 8. Videz votre crontab
