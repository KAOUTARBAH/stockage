# stockage LVM
Logical Volume Manager (LVM) est un ensemble d'outil permettant une gestion avancée du stockage sous Linux

# Pré-requis

Avant de commencer, vous devez vous assurer que vous avez installé Ubuntu et configuré votre machine virtuelle dans VirtualBox. Il vous faut également un disque supplémentaire ou une partition à utiliser pour LVM.

## Vérifiez la configuration de la machine virtuelle

1. Assurez-vous que la machine virtuelle dispose de suffisamment d'espace disque. Vous pouvez ajouter un disque virtuel supplémentaire à la VM via les paramètres de VirtualBox.
2. Allez dans **Paramètres > Stockage > Ajouter un disque dur** et ajoutez un nouveau disque dur virtuel.
3. Démarrez Ubuntu dans la machine virtuelle VirtualBox et ouvrez un terminal pour commencer à travailler avec LVM.


Ouvre un terminal et tape la commande suivante pour voir la liste des partitions actuelles et des volumes LVM :

```bash
lsblk
```
Cela te donnera un aperçu de la structure de tes disques et partitions.

## 2. Créer un Volume Physique (PV)

Si vous avez ajouté un disque supplémentaire à votre machine virtuelle, vous pouvez le transformer en volume physique (PV) avec la commande suivante :

```bash
sudo pvcreate /dev/sdb  # Créez un PV sur le disque /dev/sdb
```
![PV](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgLVM/PV.png)

## 3. Créer un Groupe de Volumes (VG)

Une fois que vous avez un PV, vous pouvez créer un groupe de volumes (VG) pour regrouper plusieurs PV. Si vous avez plusieurs disques (par exemple /dev/sdb et /dev/sdc), vous pouvez les ajouter à un VG.

```bash
sudo vgcreate mon_vg /dev/sdb /dev/sdc  # Créez un VG 'mon_vg' avec /dev/sdb et /dev/sdc
```
![VG](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgLVM/VG.png)

## 4. Créer un Volume Logique (LV)

Une fois le VG créé, vous pouvez créer un LV. Par exemple, pour créer un LV de 10 Go :

```bash
sudo lvcreate -n mon_lv -L 10G mon_vg  # Crée un LV 'mon_lv' de 10 Go dans 'mon_vg'
```
![LV](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgLVM/LV.png)

## 5. Formater et monter le LV

Après avoir créé le LV, vous devez le formater avec un système de fichiers (par exemple ext4) et le monter.

```bash
# Formater le LV avec ext4
sudo mkfs.ext4 /dev/mon_vg/mon_lv

# Créer un point de montage
sudo mkdir /mnt/mon_lv

# Monter le LV
sudo mount /dev/mon_vg/mon_lv /mnt/mon_lv
```
![Formatage](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgLVM/Formatage.png)

![mont](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgLVM/mont.png)



# Ajouter ou convertir un LV en RAID 1

RAID 1 implique la duplication des données entre deux disques pour une redondance. Voici comment convertir un LV en RAID 1 :

```bash
# Créez un LV sur un VG existant
sudo lvcreate -n mon_lv -L 10G mon_vg

# Créer un miroir RAID 1 du LV (nécessite un autre PV, ici /dev/sdc)
sudo lvconvert --type raid1 /dev/mon_vg/mon_lv /dev/sdc
# Ajouter /devv/sdc au groupe de volumes mon_vg
sudo vgextend /mon_vg/mon_lv

# Vérifier l'état du RAID
sudo lvdisplay /dev/mon_vg/mon_lv
```

![toRaid](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgLVM/toRaid.png)

![raid1](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgLVM/raid1.png)

## 5. Ajouter ou convertir un LV en RAID 0

RAID 0 répartit les données entre plusieurs disques, offrant une meilleure performance mais sans redondance. Voici comment créer un RAID 0 avec un LV :

```bash
# Créer un LV sur un VG existant
sudo lvcreate -n mon_lv -L 10G mon_vg

# Convertir le LV en RAID 0 (nécessite un second disque /dev/sdc)
sudo lvconvert --type raid0 --size 10G --name mon_lv_raid /dev/mon_vg/mon_lv /dev/sdc

# Vérifier l'état du RAID
sudo lvdisplay /dev/mon_vg/mon_lv
```

## 6. Créer un snapshot d'un LV

Un snapshot permet de capturer l'état actuel d'un volume logique (LV) à un instant donné, ce qui est utile pour la sauvegarde ou la récupération des données.

```bash
# Créer un snapshot du LV existant 'mon_lv' dans le VG 'mon_vg'
sudo lvcreate -s -n mon_lv_snapshot -L 1G /dev/mon_vg/mon_lv

# Monter le snapshot pour y accéder
sudo mount /dev/mon_vg/mon_lv_snapshot /mnt/snapshot

# Vérifier que le snapshot est bien monté
df -h /mnt/snapshot
```
## 7. Détruire un LV inutile (comme un snapshot)

Si un LV (ou snapshot) n'est plus nécessaire, vous pouvez le supprimer de cette manière :

```bash
# Supprimer un snapshot du LV
sudo lvremove /dev/mon_vg/mon_lv_snapshot

# Supprimer un LV complet
sudo lvremove /dev/mon_vg/mon_lv
```