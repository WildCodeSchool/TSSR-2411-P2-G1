# Guide de paramétrages des machines :

Pour ce projet, nous devons configurer les adresses IP suivantes sur les machines : 

| **Machines**        | **Adresses IP** |
|---------------------|-----------------|
| Win Server 2022     | 172.16.10.5     |
| Debian 12           | 172.16.10.10    |
| Win 10              | 172.16.10.20    |
| Ubuntu 22.04/24.04  | 172.16.10.30    |


## Étapes d'installation et de configuration

Nous utiliserons ici des VM sur Proxmox.

### Configuration des VM

### Mise en reseau 

les cartes réseau pour le serveur Debian et la machine Ubuntu ont été configurées en bridge sur le pont vmbrg1.

les cartes réseau pour le serveur et la machine Windows ont été configurées en bridge sur le pont vmbr4.

Pour pouvoir communiquer entre elles, nos machines doivent avoir des adresses IP faisant partie du même réseau 172.16.10.XX/24.

Ensuite, il faut configurer les adresses IP fixes manuellement : 

**Pour les machines Windows :**

Clique droit sur l'icone **réseau**en bas à droite de l'écran puis**Ouvrir les paramètres réseaux et internet**.    
On sélectionne le menu Ethernet à gauche et on clique sur **Modifier les options d'adaptateur**  

Double-cliquer sur la connexion ethernet / propriétés / Protocole Internet version 4 (TCP/IP)

Cocher "utiliser l'adresse ip suivante" et renseigner les adresses IP correspondantes aux 2 machines 

| **Machines**        | **Adresses IP** |
|---------------------|-----------------|
| Windows Server 2022 | 172.16.10.5     |
| Windows 10          | 172.16.10.20    |

Le masque de sous réseau a renseigné est : **255.255.255.0** car les adresses IP sont en /24.

Cliquer sur OK pour quitter. Les nouvelles adresses ont bien été modifiées

On peut vérifier ces adresses IP avec la commande suivante dans Powershell :

```PowerShell
ipconfig
```

  
**Pour la machine Ubuntu :**  

En interface graphique :

Cliquer sur l'icone réseau en haut à droite de l'écran et on sélectionne wired (filaire en français) / wired settings (paramètres filaires)

Cliquer sur le petite engrenage du pavé "wired" puis IPv4 / manual 

sur la ligne adresses, renseigner l'adresse IP 172.16.10.30 et le masque de sous-réseau 255.255.255.0 puis valider en cliquant sur apply. 
 
En ouvrant un terminal, on peut vérifier que l'adresse IP a été modifiée avec la commande suivante :

```Bash
ip a
```


  
**Pour la machine Debian :**  

En interface graphique :

Cliquer sur l'icone réseau en haut à droite de l'écran et on sélectionne wired (filaire en français) / wired settings (paramètres filaires)

Cliquer sur le petite engrenage du pavé "wired" puis IPv4 / manual 

sur la ligne adresses, renseigner l'adresse IP 172.16.10.10 et le masque de sous-réseau 255.255.255.0 puis valider en cliquant sur apply. 
 
En ouvrant un terminal, on peut vérifier que l'adresse IP a été modifiée avec la commande suivante :

```Bash
ip a
```


### Renommer les machines : 

Les machines doivent avoir les noms suivants :

- Debian : SRVLIN01  
- Ubuntu : CLILIN01
- Windows Server : SRVWIN01 
- Windows 10: CLIWIN01

**Pour les machines Windows :**

Tapez *nom* dans le menu démarrer / afficher le nom de votre PC / Renommer ce PC :

Saisir le nom de la machine et redémarrez pour vérifier que le changement est effectif, on peut voir ci-dessous que le nom a été mis à jour :

![renommer PC](https://github.com/user-attachments/assets/e714e605-cccc-43cb-8359-7713819c3d05)

 

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
sudo apt install openssh-serveur
```

Une fois l'installation achevée, il est recommander de regarder si le service est actif

``` bash
sudo systemctl status ssh
```
cela nous montre que si le service ssh est actif ou si l'on doit le rendre actif.

``` bash
sudo systemctl start service ssh
ou sudo systemctl enable ssh
```


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


Le message d'erreur n'est pas a prendre en compte, le plus important et que vous voyez le message :
"Le service WinRM a été démarré"

WinRM est donc configurer et pret à l'emploi

## Divers 
Pour les besoins de notre script, quelques étapes rapides sont a effectuer en amont :
1. Création d'un dossier export
   Pour avoir l'export des informations nous devons crée un dossier export sur le bureau de notre Serveur Windows et dans l'utilisateur courant du serveur Debian (/home/<NomDelUtilisateur/)
2. Activation de l'utilisateur Root sur votre Ubuntu
   Pour les actions nécessitant les droits sudo (comme par exemple ajout d'un utilisateur) nous devons nous connecter sur l'utilisateur Root
