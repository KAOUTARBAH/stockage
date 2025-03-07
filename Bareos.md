# Bareos - Gestion centralisée des sauvegardes

## Installation, Configuration et Test de Bareos

Maintenant à toi de jouer.

Reprends la documentation de Bareos depuis le début et parcours-la attentivement.

En la suivant, procède pas à pas à :

- **L'installation d'un serveur Bareos**  
  Pour cette mise en pratique, utilise un unique serveur GNU/Linux (qui peut évidemment être une VM) sur lequel tu dois installer **PostgreSQL** (pour le catalogue) ainsi que l'ensemble des composants Bareos qui s'installent via le méta-paquet `bareos`.

- **L'installation sur le même serveur de Bareos WebUI**

Ensuite, réalise toute la partie tutorielle qui t'amène à :

- Lancer et découvrir la console.
- Lancer un premier job et faire ta première sauvegarde.
- Faire une restauration des données sauvegardées.
- Ajouter un client. Ce client est une deuxième machine (ou VM) sur laquelle tu installes uniquement `bareos-fd`.
- Configurer le serveur et le client pour que la communication soit possible.
- Lancer une sauvegarde sur le client.



## Réponse:
### Installation, Configuration et Test de Bareos

#### Prérequis

- Avant de commencer, assurez-vous d'avoir :

- Un serveur GNU/Linux (physique ou VM)

- Une deuxième machine (ou VM) pour le client

- Un accès root ou sudo sur les machines

- Une connexion Internet active

### 1. Installation du Serveur Bareos

#### Mise à jour du système

```bash
sudo apt update && sudo apt upgrade -y   # Pour Debian/Ubuntu
```

#### Installation de PostgreSQL

Bareos utilise PostgreSQL comme catalogue de sauvegarde.
```bash
sudo apt install -y postgresql            # Debian/Ubuntu
```

#### Démarrer et initialiser PostgreSQL
```bash
sudo systemctl enable postgresql --now
```
Création de l'utilisateur et de la base de données Bareos
```bash
sudo -u postgres psql -c "CREATE USER bareos WITH PASSWORD 'bareos';"
sudo -u postgres psql -c "CREATE DATABASE bareos WITH OWNER bareos;"
```

![PostgreSQL](https://github.com/KAOUTARBAH/Stockage/blob/main/ImgBareos/PostgreSQL.png)
#### Installation de Bareos

Ajoutez le dépôt officiel de Bareos :
```bash
sudo apt install -y bareos
```
Activez et démarrez les services :
```bash
sudo systemctl enable --now bareos-dir
sudo systemctl enable --now bareos-sd
sudo systemctl enable --now bareos-fd
```

### 2. Installation de Bareos WebUI
```bash
sudo apt install -y bareos-webui
```
Configurez Apache :
```bash
sudo systemctl enable --now apache2  # Debian/Ubuntu
sudo systemctl enable --now httpd    # CentOS/RHEL
```

### 3. Tester Bareos

#### Lancer la Console Bareos
```bash
bconsole
```

####Vérifier l'état des services
Dans bconsole, tapez :
```bash
status
```

####Lancer un premier job de sauvegarde
```bash
run job=BackupClient1
```

#### Restaurer une sauvegarde
```bash
restore all
```
Suivez les instructions pour choisir les fichiers à restaurer.

### 4. Ajouter un Client

Sur la deuxième machine, installez uniquement l'agent Bareos :
```bash
sudo apt install -y bareos-fd
```

#### Configurer la communication serveur-client

Sur le serveur, ajoutez le client dans /etc/bareos/bareos-dir.conf :
```bash
Client {
  Name = client1-fd
  Address = IP_DU_CLIENT
  Password = "mot_de_passe_client"
}
```
Redémarrez Bareos :
```bash
sudo systemctl restart bareos-dir
```
Sur le client, éditez /etc/bareos/bareos-fd.conf pour permettre la connexion.

### Lancer une sauvegarde sur le client
```bash
bconsole
run job=BackupClient1
```

### Conclusion

Vous avez maintenant un serveur Bareos fonctionnel avec un client distant. Vous pouvez programmer des sauvegardes automatiques et tester des restaurations régulièrement.


