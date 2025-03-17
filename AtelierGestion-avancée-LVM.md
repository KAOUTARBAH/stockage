# Atelier : Gestion avancée de LVM

## 1. Prérequis

Pour cet atelier, tu as besoin :

- Un hyperviseur comme **VirtualBox** pour pouvoir créer des VM  
- **1 VM avec Ubuntu 22.04** installé et mise à jour, avec en plus du disque système :
  - Un disque de **10 Go**
  - Un disque de **20 Go**
  - Un disque de **25 Go**

## Étape 2 - Initialisation et création des LV
### Utilisation de 2 disques de 10 et 20 Go.

- Identifier les 2 disques non-système avec fdisk -l, normalement /dev/sdb et /dev/sdc.
- Initialiser les disques pour l'utilisation de LVM avec la commande pvcreate sur chacun des 2 disques.
- Créer un groupe de volume vg_datas avec la commande vgcreate avec les 2 disques
- Vérifier avec vgdisplay que la création s'est bien passée (tu dois avoir un VG Size de la taille totale des 2 disques)
- Créer un volume logique lv_datas de 25 Go avec la commande lvcreate

### Pendant l'installation d'Ubuntu :  
Lorsque tu arrives à l'étape de partitionnement du disque dur, choisis l'option **"Manuel"**.  

Crée les partitions suivantes :  
- **Système** de **8 Go** pour `/`  
- **Swap**, d'une taille **supérieure ou égale** à la quantité de RAM disponible sur la VM  
- `/home` utilise **le reste de l'espace disque principal**  

### 2. Créer un Volume Physique (PV)

Si vous avez ajouté un disque supplémentaire à votre machine virtuelle, vous pouvez le transformer en volume physique (PV) avec la commande suivante :

```bash
sudo pvcreate /dev/sdb  # Créez un PV sur le disque /dev/
sdb
sudo pvcreate /dev/sdc
```

### 3. Créer un Groupe de Volumes (VG)

- Une fois que vous avez un PV, vous pouvez créer un groupe de volumes (VG) nommé **vg_datas** pour regrouper plusieurs PV. Si vous avez plusieurs disques (par exemple /dev/sdb et /dev/sdc), vous pouvez les ajouter à un VG.

```bash
sudo vgcreate vg_datas /dev/sdb /dev/sdc  # Créez un VG 'vg_datas' avec /dev/sdb et /dev/sdc
```

- Vérifier avec vgdisplay que la création s'est bien passée (tu dois avoir un VG Size de la taille totale des 2 disques)
```bash
sudo vgdisplay
```
### 4. Créer un Volume Logique (LV)

- Une fois le VG créé, vous pouvez créer un LV. Par exemple, pour créer un LV de 10 Go :

```bash
sudo lvcreate -L 25G -n lv_datas vg_datas
 # Crée un LV 'lv_datas' de 25 Go dans 'vg_datas'
```
- Vérifier avec lvdisplay que tout est bien crée (tu dois avoir un LV Size de la taille que tu as choisi).
```bash
sudo lvdisplay
```
- Essaye maintenant de créer un volume logique de 35Go ? observe le résultat.

- resultat : espace insuffusante .

## Étape 3 - Formatage et montage de FS
### 5. Formater et monter le LV

Après avoir créé le LV, vous devez le formater avec un système de fichiers (par exemple ext4) et le monter.

```bash
# Formater le LV avec ext4
sudo mkfs.ext4 /dev/vg_datas/lv_datas

# Créer un point de montage
sudo mkdir /mnt/datas


# Monter le LV
sudo mount /dev/vg_data/lv_data /mnt/datas
``` 
- Pour le montage automatique au démarrage, ajouter au fichier /etc/fstab la ligne /dev/vg_datas/lv_datas /mnt/datas ext4 defaults 0 2

```bash
/etc/fstab
/etc/fstab la ligne /dev/vg_datas/lv_datas /mnt/datas ext4 defaults 0 2
```
## Étape 4 - Étendre le LV

### 6.Étendre le LV
- Pour vérifier la place restante sur le VG, exécuter vgs
Pour etendre le LV avec la place restante :
```bash
sudo vgs
sudo lvextend -l +100%FREE /dev/vg_datas/lv_datas
```
![Extend](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/lvextend.png)

- Ensuite il faut étendre le FS :
```bash
resize2fs /dev/vg_datas/lv_datas
```
![resize2fs](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/resize2fs.png)

- Vérifier avec lvs que l'opération a réussi
```bash
lvs
```
![lvs](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/lvs.png)


## Étape 5 - Ajout d'un disque au PV existant
- Utilisation du disque de 25 Go.

- Initialiser le nouveau disque :
```bash
# On estime que le nouveau disque est /dev/sdd
pvcreate /dev/sdd

# Ajouter ce nouveau disque au PV existant :
vgextend vg_datas /dev/sdd

# Vérifier avec vgs que l'information de VFree correspond à la taille du disque ajouté
```
![pvExistant](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/pvExistant.png)


- Créer un nouveau LV lv_datas2 de 15 Go :
```bash
lvcreate  -L 15G -n lv_datas2 vg_datas
# Formater ce LV :
mkfs.ext4 /dev/vg_datas/lv_datas2
# Effectuer le montage :
mkdir /mnt/datas2
mount /dev/vg_datas/lv_datas2 /mnt/datas2
# Ajouter dans le fichier /etc/fstab la ligne /dev/vg_datas/lv_datas2 /mnt/datas2 ext4 defaults 0 2
/dev/vg_datas/lv_datas2 /mnt/datas2 ext4 defaults 0 2
```

![lvc-form](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/lvc-form.png)

![montage](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/montage.png)


## Étape 6 - Création d'un snapshot
- Avant de commencer : créé un fichier dans le dossier /mnt/datas2 :
```bash
# si ta le droit
echo "Contenu avant création du snapshot !" > /mnt/datas2/test_file.txt

# si tu n'a pas le droit
echo "Contenu avant création du snapshot !" | sudo tee /mnt/datas2/test_file.txt > /dev/null 
```
- Exécute maintenant les commandes suivantes pour faire un snapshot et monter ce snapshot dans /mnt/datas_snap :
```bash
lvcreate --size 5G --snapshot --name lv_datas_snap /dev/vg_datas/lv_datas2
mkdir /mnt/datas_snap
mount /dev/vg_datas/lv_datas_snap /mnt/datas_snap/
```
![snapshot](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/snapshot.png)

- Vérifie le contenu du snapshot :
```bash
ls /mnt/datas2
ls /mnt/datas_snap
```

![lsSnapshot](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/lsSnapshot.png)

- Si c'est bien un snapshot, le contenu de datas_snap est identique à celui de lv_datas2

- Créé un fichier dans le répertoire /mnt/datas2 :
```bash
echo "Après création du snapshot" | sudo tee /mnt/datas2/test_file1.txt > /dev/null 
```

- Vérifie si ce fichier apparaît également dans le snapshot ou s'il est bien maintenant indépendant du sa source.

- Pour supprimer le snapshot :
```bash
umount /mnt/datas_snap
lvremove /dev/vg_datas/lv_datas_snap
```

![supSnapshot](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgAtelierLVM/supSnapshot.png)


## Étape 7 - Redimensionnement d'un LV
Avant de redimensionner le système de fichiers, il est fortement recommandé d'en vérifier l'intégrité avec e2fsck.

Tout d'abord, il faut le démonter avec umount /mnt/datas
Pour le redimensionner à 15 Go :
lvresize -L 15G /dev/vg_datas/lv_datas
Un message d'alerte te prévient que tes données ainsi que ton FS peuvent être détruite !
C'est normal car tu vas réduire la taille.
LVM ne gère pas les données, donc même si le LV est vide, comme ici, tu as le message d'alerte.

Ne pas oublier de redimensionner le FS :
resize2fs /dev/vg_datas/lv_datas
Pourquoi cela ne marche pas ?
Comme indiqué à l'écran, essaye d'exécuter la commande e2fsck -f /dev/vg_datas/lv_datas.
Est-ce que cela marche ?

Tu as ce message, et les erreurs associées car la taille du FS dépasse actuellement la taille physique du LV.
Tu as ce dysfonctionnement car tu as réduit la taille du LV sans réduire d'abord la taille du FS ! Cette manipulation amène une corruption dans la structure du FS.

Pour pouvoir faire cela correctement, il faut tout d'abord revenir à la taille initiale du LV :
lvresize -L 29.99G /dev/vg_datas/lv_datas
Puis tu peux réparer le FS :
e2fsck -f /dev/vg_datas/lv_datas
Cela va te permettre de réduire le FS :
resize2fs /dev/vg_datas/lv_datas 15G
Uniquement là tu peux réduire la taille du LV :
lvresize -L 15G /dev/vg_datas/lv_datas
Tu peux vérifier que l'opération a réussi avec la commande :
e2fsck -f /dev/vg_datas/lv_datas