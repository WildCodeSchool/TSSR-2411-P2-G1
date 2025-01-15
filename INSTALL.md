# Guide d'installation :

## Besoins initiaux : besoins du projet

Pour ce projet, vous aurez besoin de quatre machines mises en réseaux. Pour l'exemple, nous prendrons les adresses IP suivantes : 

| **Machines**   | **Adresses IP** |
|----------------|-----------------|
| Win Sever 2022 | 192.168.1.10    |
| Debian 12      | 192.168.1.20    |
| Win 10         | 192.168.1.100   |
| Ubuntu 22.04   | 192.168.1.200   |

Nous allons ici détailler les différentes étapes d'installation pour pouvoir utiliser deux scripts (un Bash et un Powershell) d'un ordinateur serveur vers un ordinateur client.
Nous allons utiliser une serveur Windows server 2022 à destination d'un client Windows 10. De même nous utiliserons un serveur Debian à destination d'un client Ubuntu.


## Étapes d'installation et de configuration : instruction étape par étape

Nous utiliserons ici des VM sur VirtualBox.

### Prérequis VM

Toutes nos machines ont été configurées avec 2 processeurs et 4 gigas de RAM mais il est possible de diminuer notamment pour Debian à un processeur et 2 gigas de RAM.  

### Mise en reseau 

Pour la mise en réseau, il faut tout d'abord aller dans l'onglet réseau de virtualbox sur la machine et sélectionner **Réseau Interne**.

![image1](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/blob/Thomas/Ressources/1_r%C3%A9seau_interne.JPG?raw=true)  

Pour pouvoir communiquer entre eux, nos machines doivent avoir des adresses IP faisant partie du même réseau. Nous utilisons ici les adresses 192.168.1.X/24.
Nous allons ensuite mettre en place des adresses IP fixes manuellement grâces aux étapes ci-dessous : 

**Pour les ordinateurs Windows :**

On clique droit sur l'onglet **réseau**en bas à droite de l'écran et on va dans **Ouvrir les paramètres réseaux et internet**.    
On sélectionne ensuite l'onglet Ethernet et on clique sur **Modifier les options d'adaptateur**  

![image2](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/blob/Thomas/Ressources/2_ethernet.JPG?raw=true)  

On rejoint ensuite les propriétés de l'Ethernet comme le montre l'image ci-dessous : 

![image3](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/blob/Thomas/Ressources/3_propri%C3%A9t%C3%A9s_ethernet.JPG?raw=true)  

Dans ces propriété, un double clique sur les **propriétés IPv4** nous permet de passer l'adressage en manuel et de renseigner l'adresse IP et le masque de sous-réseau désirés.  

![image4](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/blob/Thomas/Ressources/4_ipv4.JPG?raw=true)  

Notre adresse IP est bien configurée après un redémarrage de la machine.
On peut vérifier l'adresse IP avec la commande suivante :
```PowerShell
ipconfig
```

  
**Pour l'ordinateur Ubuntu :**  

On clique sur l'onglet en haut à droite de l'écran et on sélectionne **Paramètres**.  
Dans l'onglet **Réseau**, on clique sur le petit engrenage à droite au niveau de **Ethetnet (enp0s3)**.    


![image5](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/assets/144700255/12ad7d26-c6ed-48e7-acf9-eccdd468dcfd)   


On va ensuite dans IPv4 et on renseigne l'adresse IP et le masque de sous-réseau voulus.   

![image6](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/blob/Thomas/Ressources/6_ipv4ubuntu.JPG?raw=true)  

Notre adresse IP est bien configurée après un redémarrage de la machine.  
On peut vérifier l'adresse IP avec la commande suivante :
```Bash
ip a
```


  
**Pour l'ordinateur Debian :**  


En **root**, nous allons éditer le ficher */etc/network/interfaces* :
```Bash
nano /etc/network/interfaces

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface - vbox
allow-hotplug enp0s3
iface enp0s3 inet static
        addresse 192.168.1.20
        netmask 255.255.255.0

# NAT interface si on a rajouté un second adapter
auto enp0s8
iface enp0s8 inet dhcp
```
Etant sur une VM notre interfaces s'appelle "**enp0s3**", penser à verifié le nom de votre interface réseau avec la commande "**ip a**"

Un fois édité comme ci-dessus, on peut redémarrer le système réseau et vérifier notre adresse IP.
```Bash
systemctl restart networking

ip a
```


Toutes nos machines sont maintenant en réseau, nous pouvons passer à la suite.


### Renommer les machines : 

Pour ce projet, nous allons donner les noms suivants aux machines : 
- Debian : srvlin  
- Ubuntu : clilin  
- Windows Server : srvwin  
- Windows 10: cliwin

**Pour les machines Windows :**

Nous allons chercher *nom* dans le menu démarrer.   
Nous cliquons sur **Afficher le nom de votre PC** et nous le modifions.   
Nous redémarrons la machine.   

Nous allons aussi modifier le fichier hosts qui est sous C:\Windows\System32\drivers\etc : 

- Pour le serveur, ajouter (avec le compte Administrator) :  
```
127.0.0.1 localhost  
192.168.1.20 srvlin 
192.168.1.100 cliwin
192.168.1.200 clilin
```
  
- Pour le client, ajouter (avec le compte Administrateur) :  
```
127.0.0.1 localhost  
192.168.1.20 srvlin  
192.168.1.10 srvwin  
192.168.1.200 clilin
```

**Pour les machines Linux :**

Nous allons éditer le ficher /etc/hostname et changer le nom du PC : 
```Bash
nano /etc/hostname
```

Nous allons aussi modifier le fichier /etc/hosts : 

- Pour le serveur, ajouter (en root) :
```
127.0.0.1 localhost
127.0.1.1 srvlin
192.168.1.10 srvwin
192.168.1.100 cliwin
192.168.1.200 clilin
```

  
- Pour le client, ajouter (en éditant avec un sudo) :
```
127.0.0.1 localhost
127.0.1.1 clilin
192.168.1.200 clilin
192.168.1.20 srvlin
192.168.1.10 srvwin
192.168.1.100 cliwin
```


L'ensemble de nos noms de machines est maintenant modifié.   

## Installation et configuration du ssh sur le client Ubuntu

Nous allons commencer par installer OpenSSH si vous ne l'avez pas fait avec la commande

``` bash
sudo apt update && sudo apt install
sudo apt install open-ssh-serveur
```

Une fois les paquets récupérer et installée nous allons modifié le fichier de configuration afin de pouvoir effectuer la connection en ssh  
Pour cela il suffit de vous rendre dans le fichier "**/etc/ssh/sshd_config**", décommenter la ligne "**Port 22**" et saisir les deux lignes suivante

``` Bash
AllowUsers <NomDelUtilisateur>
PermitRootLogin Yes
```
\<NomDelUtilisateur> : correspond a l'utilisateur sur lequel vous souhaitez vous connecter

Il vous suffit de redemarrer votre VM et le tour est jouer !

## Configuration de WinRM sur le client Windows 10

Pour commencer, on ouvre le menu démarrer, et on clique sur le petit rouage pour ouvrir les paramètres de votre machine  
On selectionne Système puis Bureau à distance sur le menu de gauche   
On active le bureau à distance  
![img](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/blob/Thomas/Ressources/Capture%20d'%C3%A9cran%202023-10-31%20103701.png?raw=true)

Ensuite ouvrez votre CLI "**invite de commandes**" en tant qu'administrateur     
Et entrez la ligne de commande suivante :  

``` Batch
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```
On redemarre notre VM

Enfin ouvrez votre CLI "**PowerShell**" en tant qu'administrateur

On va ajouter TOUTE les machines dans notre Trusted host avec la commande suivante :   
Set-Item WSMan:\localhost\Client\TrustedHosts -Value *

On va activer le service WinRm avec la commande suivante :

``` PowerShell
Enable-PSRemoting
```
![img](https://github.com/ThomasDominici/TSSR-Projet-Groupe_2-TheScriptingProject/blob/Thomas/Ressources/Capture%20d'%C3%A9cran%202023-10-31%20110453.png?raw=true)

Le message d'erreur n'est pas a prendre en compte, le plus important et que vous voyez le message :
"Le service WinRM a été démarré"

WinRM est donc configurer et pret à l'emploi

## Divers 
Pour les besoins de notre script, quelques étapes rapides sont a effectuer en amont :
1. Création d'un dossier export
   Pour avoir l'export des informations nous devons crée un dossier export sur le bureau de notre Serveur Windows et dans l'utilisateur courant du serveur Debian (/home/<NomDelUtilisateur/)
2. Activation de l'utilisateur Root sur votre Ubuntu
   Pour les actions nécessitant les droits sudo (comme par exemple ajout d'un utilisateur) nous devons nous connecter sur l'utilisateur Root
