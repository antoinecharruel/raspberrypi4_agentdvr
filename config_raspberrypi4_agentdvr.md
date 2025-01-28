# Guide : Configuration d'un raspberry Pi 4 en serveur de vidéosurveillance NVR (Network Video Recorder) avec AgentDVR et mise en place d'un Bot Telegram pour recevoir les alertes.

<div align="center">
  <img src="data/raspberry-pi-logo.png" alt="raspberry-pi" width="16%" height="auto" />
  <img src="data/agent-dvr-logo.png" alt="agent-dvr" width="16%" height="auto" />
  <img src="data/telegram-logo.png" alt="telegram" width="16%" height="auto" />
</div>

## Sommaire.

- [Guide : Configuration d'un raspberry Pi 4 en serveur de vidéosurveillance NVR (Network Video Recorder) avec AgentDVR et mise en place d'un Bot Telegram pour recevoir les alertes.](#guide--configuration-dun-raspberry-pi-4-en-serveur-de-vidéosurveillance-nvr-network-video-recorder-avec-agentdvr-et-mise-en-place-dun-bot-telegram-pour-recevoir-les-alertes)
  - [Sommaire.](#sommaire)
  - [0 - Pré-requis.](#0---pré-requis)
  - [1 - Configuration initial du système.](#1---configuration-initial-du-système)
    - [1.1 - Identifiant par defaut.](#11---identifiant-par-defaut)
    - [1.2 - Initialiser root.](#12---initialiser-root)
    - [1.3 - Configurer le clavier (AZERTY).](#13---configurer-le-clavier-azerty)
    - [1.4 - Configurer la timezone (Europe/Paris).](#14---configurer-la-timezone-europeparis)
    - [1.5 - Ajouter et supprimer un utilisateur.](#15---ajouter-et-supprimer-un-utilisateur)
      - [1.5.1 - Ajouter l'utilisateur user.](#151---ajouter-lutilisateur-user)
      - [1.5.2 - Supprimer l'utilisateur ubuntu.](#152---supprimer-lutilisateur-ubuntu)
    - [1.6 - Monter un stockage externe.](#16---monter-un-stockage-externe)
  - [2 - Configurations du réseau.](#2---configurations-du-réseau)
    - [2.1 - Mettre une IP static.](#21---mettre-une-ip-static)
    - [2.2 - Configuration du serveur SSH.](#22---configuration-du-serveur-ssh)
      - [2.2.1 - Éditer le fichier de configuration de SSH.](#221---éditer-le-fichier-de-configuration-de-ssh)
      - [2.2.2 - Créer une clé SSH (Depuis Windows).](#222---créer-une-clé-ssh-depuis-windows)
      - [2.2.3 - Changer le port du serveur SSH.](#223---changer-le-port-du-serveur-ssh)
    - [2.3 - Installer SAMBA pour le partage de fichiers.](#23---installer-samba-pour-le-partage-de-fichiers)
    - [2.4 - Mise en place du serveur FTP.](#24---mise-en-place-du-serveur-ftp)
  - [3 - Installer AgentDVR.](#3---installer-agentdvr)
    - [3.1 Installer Docker.](#31-installer-docker)
    - [3.2 - Quelques commandes utiles.](#32---quelques-commandes-utiles)
  - [4 - Programmation \& automatisation.](#4---programmation--automatisation)
    - [4.1 - Programmer le reboot automatique de la machine.](#41---programmer-le-reboot-automatique-de-la-machine)
    - [4.2 - Supprimer les fichiers de plus de 7 jours automatiquement.](#42---supprimer-les-fichiers-de-plus-de-7-jours-automatiquement)
    - [4.3 - Programmation en python d'un bot Telegram.](#43---programmation-en-python-dun-bot-telegram)
      - [4.3.1 - Créer une interface utilisateur afin d'intéragir avec le bot Telegram.](#431---créer-une-interface-utilisateur-afin-dintéragir-avec-le-bot-telegram)
      - [4.3.2 - Envoyer sur le canal Telegram les alertes générées par la camara et reçues via le serveur ftp.](#432---envoyer-sur-le-canal-telegram-les-alertes-générées-par-la-camara-et-reçues-via-le-serveur-ftp)
  - [5 - Créer une image de la carte SD.](#5---créer-une-image-de-la-carte-sd)
    - [5.1 - Créer une image depuis windows.](#51---créer-une-image-depuis-windows)
    - [5.2 - Créer une image depuis le serveur Ubuntu.](#52---créer-une-image-depuis-le-serveur-ubuntu)

## 0 - Pré-requis.

Installer Ubuntu Server 20.04.5 LTS 64-bit avec le logiciel raspberry.imager.

> https://downloads.raspberrypi.org/imager/

---

## 1 - Configuration initial du système.

### 1.1 - Identifiant par defaut.

```bash
login : ubuntu
password : ubuntu
```

### 1.2 - Initialiser root.

```bash
sudo passwd root
password <Le mot de passe>
```

### 1.3 - Configurer le clavier (AZERTY).

> https://wiki.debian.org/fr/Keyboard

```bash
su root
```

```bash
dpkg-reconfigure keyboard-configuration
```

```bash
service keyboard-setup restart
```

### 1.4 - Configurer la timezone (Europe/Paris).

```bash
timedatectl
```

```bash
sudo timedatectl set-timezone Europe/Paris
```

### 1.5 - Ajouter et supprimer un utilisateur.

#### 1.5.1 - Ajouter l'utilisateur user.

```bash
sudo adduser user
```

#### 1.5.2 - Supprimer l'utilisateur ubuntu.

```bash
sudo userdel -r ubuntu
```

### 1.6 - Monter un stockage externe.

Afficher une liste des périphériques de stockage (disques, partitions).

```bash
lsblk -f
```

Formater la partition avec le système de fichiers ext4.

```bash
sudo mkfs.ext4 /dev/sda1
```

Monter le ssd sur le répertoire (point de montage) /home/user/ssd.

```bash
sudo mount -t ext4 /dev/sda1 /home/user/ssd
```

Afficher les informations d'identification de l'utilisateur.
```bash
id user
```

Modifier les permissions et le propriétaire du répertoire /home/user/ssd pour user et son groupe. 

```bash
sudo chown 1001:1001 /home/user/ssd
```

- Le propriétaire a le droit de lire, écrire et exécuter `(7)`.
- Le groupe propriétaire a la permission de lecture et d'exécution `(5)`.
- Les autres utilisateurs ont la permission de lecture et d'exécution `(5)`.

```bash
sudo chmod 755 /home/user/ssd
```
Monter les partitions automatiquement lors du démarrage du système.

```bash
sudo blkid
```

Ouvrir le fichier `/etc/fstab` avec un éditeur de texte comme nano :

```bash
sudo nano /etc/fstab
```

Repérer la ligne correspondant à la partition à monter. 
L'`UUID` sera affiché après `UUID=`.

```bash
UUID=9c3a89a3-83f1-4a4d-aa64-c273cf56b643  /home/user/ssd  auto  defaults,nofail  0  1
```

- `auto` : Ce champ spécifie le type de système de fichiers de la partition. 
Le paramètre `auto` indique que le système de fichiers doit être détecté automatiquement.

- `defaults`, `nofail` : Ces options définissent les paramètres de montage. `defaults` utilise les options de montage par défaut pour le système de fichiers spécifié, et `nofail` indique au système de continuer le démarrage même si le montage de cette partition échoue.

- `0` : C'est un indicateur pour le système de fichiers sur s'il faut sauvegarder cette partition lors de la vérification des systèmes de fichiers.

- `1` : C'est un indicateur pour le système de fichiers sur l'ordre dans lequel monter les partitions. Si plusieurs partitions ont la même valeur de priorité, elles sont montées en parallèle.

## 2 - Configurations du réseau.

### 2.1 - Mettre une IP static.

Afin de pouvoir utiliser la commande `ifconfig`, il faut installer `net-tools` :

```bash
sudo apt-get install net-tools
```

Afficher l'adresse ip du raspberry :

```bash
ifconfig -a
```

Créer un fichier de configuration autre que `50-cloud-init.yaml` dans le répertoire `/etc/netplan`.

```bash
cd /etc/netplan
```

```bash
cp 50-cloud-init.yaml 50-cloud-init.yaml.old # Créer une copie.
rm -rf 50-cloud-init.yaml # Supprime.
nano 01-network-manager.yaml
```

```bash
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.1.160/24]
      routes:
        - to: default
          via: 192.168.1.254
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
```

```bash
sudo netplan try
```

```bash
ifconfig -a
```

---

### 2.2 - Configuration du serveur SSH.

```bash
sudo apt install openssh-server # Installer openssh-server.
```

Voir l'état de openssh-server.
```bash
sudo systemctl status ssh
```

#### 2.2.1 - Éditer le fichier de configuration de SSH.

Créer une sauvegarde du fichier de configuration par default.
```bash
cp -r sshd_config.d sshd_config.d.old # Créer une copie.
``` 

Ouvrir et éditer le fichier de configuration.
```bash
sudo nano /etc/ssh/sshd_config
```

Pour autoriser la connection SSH en root, décommenter et remplacer "prohibit-password" par "yes" :
(remettre "no" une fois la configuration terminé).

```bash
#PermitRootLogin prohibit-password
PermitRootLogin yes
```

Redémarrer le service SSH.
```bash
sudo systemctl restart ssh
```

#### 2.2.2 - Créer une clé SSH (Depuis Windows).

L'emplacement où sera stocké la clé :
```bash
C:\Users\%USERNAME%\.ssh
```

Sur windows dans Powershell :
```bash
ssh-keygen -t ecdsa -b 521
```

Transférer la clé publique sur le serveur distant :
```bash
scp C:\Users\%USERNAME%\.ssh\id_ecdsa.pub user@192.168.1.160:~/
```

Se connecter avec root.
```bash
ssh root@192.168.1.160
```

Créer un répertoire .ssh pour user et ajouter les permissions.
```bash
mkdir .ssh # Créer nouveau répertoire .ssh.
chown user .ssh # Change le propriétaire du répertoire .ssh pour l'utilisateur user.
chmod 0700 .ssh # Seul le propriétaire de ce répertoire peut y accéder en lecture, écriture et exécution.
```
Ajouter la clé dans la liste des clés autorisées.
```bash
cat id_ecdsa.pub >> .ssh/authorized_keys # Ajoute la clé publique dans le fichier authorized_keys.
```

```bash
chown user .ssh/authorized_keys
chmod 0700 .ssh/authorized_keys # Seul le propriétaire de ce répertoire peut y accéder en lecture, écriture et exécution.
```

```bash
sudo systemctl restart ssh # Redémmarer le service SSH.
``` 

#### 2.2.3 - Changer le port du serveur SSH.

```bash
sudo nano /etc/ssh/sshd_config
```

```bash
Port 76
```

```bash
sudo systemctl restart ssh # Redémmarer le service SSH.
``` 

### 2.3 - Installer SAMBA pour le partage de fichiers.

> [Install and Configure Samba](https://ubuntu.com/tutorials/install-and-configure-samba#1-overview)

```bash
sudo apt update # Mettre à jour la liste des paquets.
```

```bash
sudo apt install samba # Installer Samba.
```

```bash
whereis samba # Affiche les emplacements des fichiers liés à Samba sur votre système.
```

Créer une copie du fichier de configuration par defaut de SAMBA :
```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.old # Créer une copie.
```

Modifier le fichier de configuration SAMBA :
```bash
sudo nano /etc/samba/smb.conf
```

A la fin du fichier on ajoute les paramètres pour l'utilisateur `user`:
```bash
[user]
    comment = Default Samba folder # Commentaire décrivant le partage Samba.
    path = /home/user # Chemin du dossier partagé.
    guest ok = no # Spécifie si les utilisateurs invités sont autorisés à accéder au partage sans authentification.
    valid users = @user # Spécifie les utilisateurs autorisés à accéder au partage.
    read only = no # Spécifie si le partage est en lecture seule ou en lecture/écriture.
    browsable = yes # Spécifie si le partage est visible aux clients Samba lors de la navigation dans le réseau.
```

> [smb.conf Documentation](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)

Redémarrer le service Samba :
```bash
sudo service smbd restart
```

Mettre à jour les règles du pare-feu pour autoriser le trafic Samba.
```bash
sudo ufw allow samba
```

Mettre en place un mot de passe Samba pour user :
```bash
sudo smbpasswd -a user
```

Se connecter depuis windows dans la barre d'adresse du file explorer :
```bash
\\192.168.1.160\user
```

ou sur d'autre appareils (comme depuis linux) :
```bash
smb://ip-adress/user
smb://192.168.1.160/user
```

Utiliser la commande `netstat` pour afficher les ports TCP en écoute :

[Samba Domain Member Port Usage](https://wiki.samba.org/index.php/Samba_Domain_Member_Port_Usage)

```bash
netstat -tulpn | egrep "smbd|nmbd|winbind"
```

Voir les autorision d'user sur le fichier `sambashare` :
```bash
su - user -c 'ls -ld /home/user/<sambashare>'
```

Changer le propriétaire du dossier en user :
```bash
chown user:user /home/user/<sambashare>
```

Donner les droits d'écriture au propriétaire :
```bash
chmod 775 /home/user/<sambashare>
```

### 2.4 - Mise en place du serveur FTP.

> [ubuntu.com : set-up-an-ftp-server](https://ubuntu.com/server/docs/set-up-an-ftp-server)

```bash
sudo apt install vsftpd
```

Créer un copie du fichier de configuration par default.
```bash
cp /etc/vsftpd.conf /etc/vsftpd.conf.old # Créer une copie.
```

Les commandes :
```bash
sudo systemctl start vsftpd.service # Démarre le service vsftpd.
sudo systemctl restart vsftpd.service # Redémarre le service vsftpd.
sudo systemctl stop vsftpd.service # Arrête le service vsftpd.
sudo systemctl status vsftpd.service # Affiche le statut actuel du service vsftpd.
sudo systemctl enable vsftpd.service # Active le service vsftpd pour qu'il démarre automatiquement au démarrage du système.
```

Configurer le pare-feu Uncomplicated Firewall (UFW) afin de permettre le trafic sur les ports 20 et 21.
```bash
sudo ufw allow 20 && sudo ufw allow 21
```

Ajouter user à la liste des utilisateurs vsftpd.
```bash
echo "user" | sudo tee -a /etc/vsftpd.userlist
```

```bash
nano /etc/vsftpd.conf
```

```bash
sudo systemctl restart vsftpd.service
```

## 3 - Installer AgentDVR.

> [ispyconnect download page](https://www.ispyconnect.com/download)

> [Page Docker mekayelanik/ispyagentdvr](https://hub.docker.com/r/mekayelanik/ispyagentdvr)

> [Docker documentation](https://docs.docker.com/config/containers/start-containers-automatically/)

### 3.1 Installer Docker.

```bash
sudo apt install docker.io
```

> [Github Docker mekayelanik/ispyagentdvr](https://github.com/MekayelAnik/ispyagentdvr-docker)

Reprendre l'exemple mis à disposition sur le github officiel d'AgentDVR et l'adapter.
```bash
docker run -d \
  --name=ispyagentdvr \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Asia/Dhaka \
  -p 8090:8090 \
  -p 3478:3478/udp \
  -p 50000-50010:50000-50010/udp \
  -v /path/to/config:/AgentDVR/Media/XML \
  -v /path/to/recordings:/AgentDVR/Media/WebServerRoot/Media \
  -v /path/to/commands:/AgentDVR/Commands \
  --restart unless-stopped \
  mekayelanik/ispyagentdvr:latest
```

Créer le conteneur :
- Modifier la TimeZone ("TZ=Europe/Paris").
- Créer un volume sur le ssd pour Media et Commands ("/home/user/ssd/AgentDVR/").
- Redemarre automatiquement ("--restart always").
- Changer "latest" par la version 5.2.8.0.

```bash
docker run -d \
  --name=ispyagentdvr \
  -e PUID=1001 \
  -e PGID=1001 \
  -e TZ=Europe/Paris \
  -p 8090:8090 \
  -p 3478:3478/udp \
  -p 50000-50010:50000-50010/udp \
  -v /home/user/ssd/AgentDVR/Media:/AgentDVR/Media/ \
  -v /home/user/ssd/AgentDVR/Commands:/AgentDVR/Commands \
  --restart always  \
  mekayelanik/ispyagentdvr:5.2.8.0
```

### 3.2 - Quelques commandes utiles.

Pour afficher les containeurs.
```bash
docker ps -a
```

Pour aller à l'interieur du conteneur.
```bash
docker exec -it ispyagentdvr /bin/bash
```

Pour sortir du conteneur.
```bash
exit
```

Pour démarrer ispyagentdvr manuellement.
```bash
docker start ispyagentdvr
```

Pour redémarrer ispyagentdvr manuellement.
```bash
docker restart agentdvr
```

pour supprimer AgentDVR.

```bash
docker ps -a # Affiche tous les conteneurs.

docker stop ispyagentdvr # Arrête le conteneur.

docker rm ispyagentdvr # Supprime le conteneur. Le conteneur doit être arrêté pour pouvoir être supprimé.

docker volume ls # Liste tous les volumes Docker existants, qu'ils soient associés à des conteneurs ou non.

docker volume rm <volume_id> # Supprime le volume spécifié par "volume_id". Verifier que le volume n'est pas utilisé par un conteneur.
```

Commande qui supprime tous les volumes Docker qui ne sont actuellement associés à aucun conteneur.

```bash
docker volume prune
```
Configure le pare-feu UFW (Uncomplicated Firewall) pour autoriser le trafic sur certains ports et protocoles.

```bash
sudo ufw start

sudo ufw status

sudo ufw enable

sudo ufw allow 8090/tcp

sudo ufw allow 8090/udp

sudo ufw allow 3478/tcp

sudo ufw allow 3478/udp

sudo ufw allow 5353/tcp

sudo ufw allow 5353/udp

sudo ufw allow 50000:50010/tcp

sudo ufw allow 50000:50010/udp

sudo ufw status

sudo ufw reload
```

> pour ce connecter au service AgentDVR http://192.168.1.160:8090 or http://nvr:8090

---

## 4 - Programmation & automatisation.

### 4.1 - Programmer le reboot automatique de la machine.

Reboot tous les 4 heures.
```bash
sudo crontab -e
# Automatically reboot the system at the specified times.
0 0,4,8,12,16,20 * * * sudo reboot
```

Reboot à 6h00.
```bash
sudo crontab -e
0 6 * * * sudo reboot
```

```txt
Minute Heure * * * Commande. 
```

### 4.2 - Supprimer les fichiers de plus de 7 jours automatiquement.

- 180 = 6 mois.
- 180 = 3 mois.

```bash
sudo crontab -e
```

La commande sera executé tout les matin à 6h.

- Supprime tous les fichiers dans les répertoires indiqués.

```bash
5 0 * * * sudo find /home/user/ssd/IPCAM-Alertes/* -maxdepth 0 -type d -ctime +7 -exec rm -r {} \;
5 0 * * * sudo find /home/user/ssd/Hikvision-ftp/* -maxdepth 0 -type d -ctime +7 -exec rm -r {} \;
```

### 4.3 - Programmation en python d'un bot Telegram.

#### 4.3.1 - Créer une interface utilisateur afin d'intéragir avec le bot Telegram.

```bash
pip install python-telegram-bot
```

```python
import telebot
import os
import asyncio
import subprocess

# Token du bot Telegram
TOKEN = 'API_KEY'

# Récuperrer les personne autorise
authorized_users = []

authorized_users_file = open("authorized_users", "r")

for line in authorized_users_file:
    # Remplacer les séparateurs par un caractère unique (par exemple, '\t')
    for separators in [' ', '#']:
        line = line.split(separators)[0]
    line = line.strip()  # Supprime les espaces autour de la ligne
    if line:  # Ajoute seulement si la ligne n'est pas vide
        authorized_users.append(line)  # Ajoute l'utilisateur dans la liste

authorized_users_file.close()

# Initialisation du bot
bot = telebot.TeleBot(TOKEN)


async def check_ping(ip_address, camera_name, chat_id):
    # Utilisation de la commande ping pour vérifier la disponibilité de l'adresse IP
    result = subprocess.run(['ping', '-c', '1', ip_address], stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    if result.returncode == 0:
        # Si le ping réussit, envoyer un message indiquant que l'adresse IP est accessible
        status_animation = "/home/user/.script/bot_telegram/notifications/signal_ok.gif"
        current_status = "est connecté"
        caption_text = f"La caméra {camera_name} ({ip_address}) {current_status}"
        # Envoie du GIF avec la légende spécifiée
        await bot.send_animation(chat_id, animation=open(status_animation, 'rb'), caption=caption_text)
    else:
        # Si le ping échoue, envoyer un message indiquant que l'adresse IP n'est pas accessible
        status_animation = "/home/user/.script/bot_telegram/notifications/no_signal.gif"
        current_status = "perdu"
        caption_text = f"Le signal de la caméra {camera_name} ({ip_address}) {current_status}"
        # Envoie du GIF avec la légende spécifiée
        await bot.send_animation(chat_id, animation=open(status_animation, 'rb'), caption=caption_text)


def createHandler(commands, aKeyboard, keyboard_header):
    @bot.message_handler(commands)
    def cmd(message):
        if str(message.from_user.id) in authorized_users: # Vérifier si l'utilisateur est autorisé
            bot.send_message(message.chat.id, keyboard_header, reply_markup=aKeyboard)
        else:
            bot.send_message(message.chat.id, f'Vous n\'êtes pas autorisé à exécuter cette commande. Votre user id est {message.from_user.id}')


# Clavier de démarage.
def keyboard_start():
    keyboard = telebot.types.InlineKeyboardMarkup()
    keyboard.row(
        telebot.types.InlineKeyboardButton('Start/Stop notifications des cameras', callback_data='display_cmd_notifications')
    )
    keyboard.row(
        telebot.types.InlineKeyboardButton('Ping', callback_data='display_cmd_ping')
    )
    keyboard.row(
        telebot.types.InlineKeyboardButton('Commandes système', callback_data='display_cmd_system')
    )
    return keyboard

createHandler(commands=['start', 'cmd', 'help'], aKeyboard=keyboard_start(), keyboard_header="Liste des commandes disponibles (cliquer pour afficher les commandes) :")


# Clavier de gestion des notifications des caméras.
def keyboard_notifications():
    keyboard = telebot.types.InlineKeyboardMarkup()
    keyboard.row(
    telebot.types.InlineKeyboardButton('\U0001F514 Start notifications Ctronics01', callback_data='start_notifications_Ctronics01')
    )
    keyboard.row(
        telebot.types.InlineKeyboardButton('\U0001F515 Stop notifications Ctronics01', callback_data='stop_notifications_Ctronics01')
    )
    keyboard.row(
        telebot.types.InlineKeyboardButton('\U0001F514 Start notifications Hikvision', callback_data='start_notifications_Hikvision')
    )
    keyboard.row(
        telebot.types.InlineKeyboardButton('\U0001F515 Stop notifications Hikvision', callback_data='stop_notifications_Hikvision')
    )
    return keyboard

createHandler(commands=['notifications'], aKeyboard=keyboard_notifications(), keyboard_header="Liste des commandes disponibles (cliquer pour afficher les commandes) :")


# Clavier pour ping.
def keyboard_ping():
    keyboard = telebot.types.InlineKeyboardMarkup()

    keyboard.add(
        telebot.types.InlineKeyboardButton('Ping caméra Ctronics01', callback_data='ping_Ctronics01'),
        telebot.types.InlineKeyboardButton('Ping caméra Hikvision', callback_data='ping_Hikvision')
    )

    return keyboard

createHandler(commands=['ping'], aKeyboard=keyboard_ping(), keyboard_header="Choissiser un appareil à ping :")


# Clavier ssh.
def keyboard_ssh():
    keyboard = telebot.types.InlineKeyboardMarkup()

    # Ajouter des boutons en colonnes
    keyboard.add(
        telebot.types.InlineKeyboardButton('Start SSH', callback_data='ssh_start'),
        telebot.types.InlineKeyboardButton('Stop SSH', callback_data='ssh_stop')
    )
    keyboard.add(
        telebot.types.InlineKeyboardButton('Enable SSH', callback_data='ssh_enable'),
        telebot.types.InlineKeyboardButton('Disable SSH', callback_data='ssh_disable')
    )

    return keyboard

createHandler(commands=['ssh'], aKeyboard=keyboard_ssh(), keyboard_header="Liste des commandes pour interagir avec le service SSH du serveur:")

# Commande pour autorisé un utilisateur à utiliser le bot.
@bot.message_handler(commands=['adduser'])
def adduser_command(message):
    # Vérifier si l'utilisateur est autorisé
    if str(message.from_user.id) in authorized_users:
        
        id_from_text = message.text

        if len(id_from_text) == 8:
            bot.send_message(message.chat.id, "Argument de la commande /adduser vide")
        else:
            separators = ' '
            id_from_text = id_from_text.split(separators)[1]

        if id_from_text in authorized_users:
            bot.send_message(message.chat.id, f'Utilisateur {id_from_text} déja autorisé')
        elif message.text[9] == '@' or message.text[9] == ' ':
            bot.send_message(message.chat.id, "Argument de la commande /adduser invalide")
        else:
            with open('authorized_users', 'a') as authorized_users_file:
                authorized_users_file.write(f'\n{message.text[9:]}')
            authorized_users_file.close()
            bot.send_message(message.chat.id, f'Utilisateur : {id_from_text} autorisé')
            subprocess.run(["sudo", "systemctl", "restart", "bot_telegram.service"])
    else:
        bot.send_message(message.chat.id, f'Vous n\'êtes pas autorisé à exécuter cette commande. Votre user id est {message.from_user.id}')


# Gestion des requêtes.
@bot.callback_query_handler(func=lambda call: True)
def callback_query(call):
    
    # Requêtes du keyboard start.
    if call.data == 'display_cmd_notifications':
       bot.send_message(call.message.chat.id, text=r'/notifications')
    elif call.data == 'display_cmd_ping':
       bot.send_message(call.message.chat.id, text=r'/ping - Selectionner pour afficher la liste des appareils à ping.')
    elif call.data == 'display_cmd_system':
        bot.send_message(call.message.chat.id, text=r'/ssh - Selectionner pour afficher les commandes ssh. ')
        bot.send_message(call.message.chat.id, text=r'/restart  - Selectionner pour redémarrer le serveur.')
        bot.send_message(call.message.chat.id, text=r'/adduser <id> - Autoriser un utilisateur à utliser le bot.')

    # Requêtes du keyboard notifications.
    if call.data == 'start_notifications_Ctronics01':
        subprocess.run(["sudo", "systemctl", "start", "camera-alert-notification_Ctronics01.service"])
        bot.send_message(call.message.chat.id, 'Les notifications pour Ctronics01 ont été démarrées.')
    elif call.data == 'stop_notifications_Ctronics01':
        subprocess.run(["sudo", "systemctl", "stop", "camera-alert-notification_Ctronics01.service"])
        bot.send_message(call.message.chat.id, 'Les notifications pour Ctronics01 ont été arrêtées.')
    elif call.data == 'start_notifications_Hikvision':
        subprocess.run(["sudo", "systemctl", "start", "camera-alert-notification_Hikvision.service"])
        bot.send_message(call.message.chat.id, 'Les notifications pour Hikvision ont été démarrées.')
    elif call.data == 'stop_notifications_Hikvision':
        subprocess.run(["sudo", "systemctl", "stop", "camera-alert-notification_Hikvision.service"])
        bot.send_message(call.message.chat.id, 'Les notifications pour Hikvision ont été arrêtées.')

    # Requêtes du clavier ping.
    if call.data == 'ping_Ctronics01':
        asyncio.run(check_ping('192.168.1.199', 'Ctronics01', call.message.chat.id))
    elif call.data == 'ping_Hikvision':
        asyncio.run(check_ping('192.168.1.102', 'Hikvision', call.message.chat.id))

    # Requêtes du clavier ssh
    if call.data == 'ssh_start':
        subprocess.run(["sudo", "systemctl", "start", "ssh.service"])
        bot.send_message(call.message.chat.id, 'Le service SSH a démarré.')
    elif call.data == 'ssh_stop':
        subprocess.run(["sudo", "systemctl", "stop", "ssh.service"])
        subprocess.run(["python3", "/home/user/.script/bot_telegram/kill_ssh.py"])
        bot.send_message(call.message.chat.id, 'Le service SSH est stoppé.')
    elif call.data == 'ssh_enable':
        subprocess.run(["sudo", "systemctl", "enable", "ssh.service"])
        bot.send_message(call.message.chat.id, 'Le service SSH est activé.')
    elif call.data == 'ssh_disable':
        subprocess.run(["sudo", "systemctl", "disable", "ssh.service"])
        bot.send_message(call.message.chat.id, 'Le service SSH est désactivé.')

# Gestion des erreurs
@bot.message_handler(func=lambda message: True)
def echo_all(message):
    bot.reply_to(message, 'Désolé, je ne comprends pas cette commande.')

# Lancement du bot
bot.polling(none_stop=True)
```
Définir un service pour exécuter un bot Telegram.
```bash
nano /etc/systemd/system/bot_telegram.service
```

```bash
[Unit]
Description=Commandes du bot télégram.
After=network.target

[Service]
User=root
WorkingDirectory=/home/user/.script/bot_telegram/

ExecStart=/usr/bin/python3 /home/user/.script/bot_telegram/bot_telegram.py
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl start bot_telegram.service
sudo systemctl stop bot_telegram.service
sudo systemctl enable bot_telegram.service
sudo systemctl status bot_telegram.service
```

#### 4.3.2 - Envoyer sur le canal Telegram les alertes générées par la camara et reçues via le serveur ftp.

Une fois le bot invité dans le canal télégram, pour connaitre le CHAT_ID :
https://api.telegram.org/<bot(BOTToken)>/getUpdates

```python
import os
import asyncio
import telegram
import time

# Token du bot Telegram
TOKEN = 'API_KEY'
# ID du chat Telegram
CHAT_ID = 'CHAT_ID'
# Chemin du répertoire à surveiller
DIR = '/home/user/ssd/IPCAM-Alertes'
# Extension à surveiller pour les photos
PHOTO_EXT = '.jpg'
# Extension à surveiller pour les vidéos
VIDEO_EXT = '.265'

# Crée une instance de bot Telegram
bot = telegram.Bot(token=TOKEN)

# Fonction pour envoyer un message sur Telegram
async def send_message(message):
    await bot.send_message(chat_id=CHAT_ID, text=message)

# Fonction pour surveiller le répertoire
async def monitor_directory():
    while True:
        for root, dirs, files in os.walk(DIR):
            for file in files:
                file_path = os.path.join(root, file)
                # Vérifie si le fichier a été créé il +10 et -20 secondes
                if 10 <= time.time() - os.path.getctime(file_path) < 20:
                    if file.endswith(PHOTO_EXT):
                        with open(file_path, 'rb') as photo_file:
                            await bot.send_photo(chat_id=CHAT_ID, photo=photo_file)
                    elif file.endswith(VIDEO_EXT):
                        await send_message(f"\U0001F6A8 \U0001F3A5 Nouvelle vidéo créée : {file_path}")
        await asyncio.sleep(9)  # Vérifie toutes les 9 secondes

if __name__ == "__main__":
    asyncio.run(monitor_directory())
```

Définir un service pour exécuter les alertes sur le canal Telegram.
```bash
nano /etc/systemd/system/camera-alert-notification_Ctronics01.service
```

```bash
[Unit]
Description=Envois une notification sur télégram à chaque alerte de la caméra.
After=network.target

[Service]
User=root
WorkingDirectory=/home/user/.script/bot_telegram/notifications

ExecStart=/usr/bin/python3 /home/user/.script/bot_telegram/notifications/camera-alert-notification_Ctronics01.py
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl start camera-alert-notification_Ctronics01.service
sudo systemctl stop camera-alert-notification_Ctronics01.service
sudo systemctl enable camera-alert-notification_Ctronics01.service
sudo systemctl status camera-alert-notification_Ctronics01.service
```

```bash
sudo journalctl -u camera-alert-notification_Ctronics01.service
sudo journalctl -u camera-alert-notification_Ctronics01.service -n 10
```

## 5 - Créer une image de la carte SD.

### 5.1 - Créer une image depuis windows.

- Avec DiskGenius (logiciel en version gratuite) supprimer la partition vide.

- Faire une image de la carte SD avec Win32DiskImager en cochant "Read Only Allocated Partitions".

- Après avoir envoyé l'image sur une nouvelle carte SD, ne pas oublier de re-créer la partition vide avec DiskGenius.

### 5.2 - Créer une image depuis le serveur Ubuntu.

Afficher les informations des partitions :

```bash
lsblk -f
```

```bash
fdisk -l
```

autres commandes :
```bash
df -h /dev/mmcblk0p2

df -B1 /dev/mmcblk0p2

df -hB1M /dev/mmcblk0p2

df -hB1G /dev/mmcblk0p2
```

```bash
root@ubuntu:~# fdisk -l

Disk /dev/mmcblk0: 238.3 GiB, 255869321216 bytes, 499744768 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xfb276288

Device         Boot    Start       End   Sectors   Size Id Type
/dev/mmcblk0p1 *        2048    526335    524288   256M  c W95 FAT32 (LBA)
/dev/mmcblk0p2        526336  30320639  29794304  14.2G 83 Linux
/dev/mmcblk0p3      30320640 499744767 469424128 223.9G 83 Linux

```

- Ajouter un au nombre final de secteurs, car le décompte des secteurs commence à zéro : `499744767 + 1` = `499744768` => le nombre total de secteurs à copier.

Multiplier par le nombre d'octets dans un secteur (512) : `499744768 x 512` = `255869321216` => le nombre total d'octets à copier.

Dans la commande `dd`, remplacer le nombre d'octets calculé dans l'instruction `count=`.

Ne pas oublier d'inclure `iflag=count_bytes` comme indiqué dans la commande ci-dessous.

Commande pour créer l'image :

```bash
dd if=/dev/mmcblk0 of="<file and path you are copying to>" count=<size> status=progress iflag=count_bytes

dd if=/dev/mmcblk0 of="/home/user/ssd/backup/raspi4-AgentDVR_backup_16GO.img" count=255869321216 status=progress iflag=count_bytes
```

Pour compresser l'image :

```bash
gzip "raspi4-AgentDVR_backup.img" status=progress
```