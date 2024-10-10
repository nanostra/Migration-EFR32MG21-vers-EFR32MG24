# Migration-EFR32MG21-vers-EFR32MG24
Process de migration d'un dongle zigbee EFR32MG21 vers EFR32MG24 sans ré-appairage, **par clonage**
Ce process peut également être utilisé pour réaliser un clone sans migration d'un dongle Zigbee EFR32MG21 ou EFR32MG24 

Merci à **@Nerivec** pour son assistance et son travail sur ember-zli
Merci à **@darkxst** pour la mise à disposition des firmware spécifiques à l'EFR32MG24

**Remarques - Recommandations**

- Ce guide ne concerne que la migration d’une clé à base de EFR32MG21 vers EFR32MG24.
- mais il peut également être utilisé pour réaliser un clone sans migration d'un dongle Zigbee EFR32MG21 ou EFR32MG24 
- Il se base sur l’utilisation de WSL (Windows subsystem for linux) et Ubuntu sous Windows 11.
- Ce process permet de conserver la totalité du réseau sans avoir à réappairer les équipements.
- Prendre toutes les précautions nécessaires, faire un backup de home assistant ou snapshot de VM par exemple.
- Il est important que les deux dongles aient la même version de firmware zigbee ember, dans mon cas 7.4.4
- Je ne suis pas l'auteur du package ember-zli, en conséquence merci de consulter la documentation pour tout complément d'informations.

## A propos du firmware EFR32MG24

- Depuis **09/10/2024**, **darkxst** a mis à disposition une branche pour le SLZB-07MG24 à cette adresse :
[https://github.com/darkxst/silabs-firmware-builder/tree/SLZB-07Mg24](https://github.com/darkxst/silabs-firmware-builder/tree/SLZB-07Mg24)

- Pour flasher cela se passe à cette adresse en choisissant **custom firmware** comme option :
[https://darkxst.github.io/silabs-firmware-builder/](https://darkxst.github.io/silabs-firmware-builder/)

- ⚠ Pour éviter les erreurs de manipulation, connecter **1 clé à la fois**.

## Installer WSL – Windows subsystem for linux

- La virtualisation au niveau BIOS doit être activée. 
- Aller dans le Windows Store et installer **Ubuntu**
  
## Transfert de port USB vers WSL :
[https://learn.microsoft.com/en-us/windows/wsl/connect-usb](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)

- Sur windows, installer **usbipd-win** et pour permettre le partage de port avec WSL (VM ubuntu)
[https://github.com/dorssel/usbipd-win/releases](https://github.com/dorssel/usbipd-win/releases)

- ⚠ **Lancer WSL command prompt** (chercher WSL et lancer), obligatoire pour laisser la VM active

- **Lancer PowerShell (admin)**, connecter la clé sur un port USB et lancer la commande **usbipd list** pour identifier le port du dongle source.
```
$ usbipd list
Connected:
BUSID  VID:PID    DEVICE                                    STATE
1-8    10c4:ea60  Silicon Labs CP210x USB to UART Bridge (COM11)  Not shared
```

- Toujours sous Powershell **Partager et rattacher le port USB du dongle à WSL**, avec les commandes **usbpid bind --busid <port_identifier_précédemment>**, puis **usbipd attach --wsl --<port_identifier_précédemment>**

```  
$ usbipd bind --busid 1-8

$ usbipd attach --wsl --busid 1-8
```

Il est possible de refaire une commande **usbipd list** pour vérifier que le port est bien attaché.

## Installations des packages nécessaires sur Ubuntu

- **Rechercher et lancer Ubuntu sur windows**
- Installation des packages **usbutils, nodejs, et npm**
- la commande **lsusb** permet de lister les ports usb liés
  
```
$ sudo apt install usbutils

$ sudo apt-get install nodejs

$ sudo apt install npm

```

```
$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
```

**Installation ember-zli**
[https://github.com/Nerivec/ember-zli](https://github.com/Nerivec/ember-zli)

- Informations utiles :
  * https://github.com/Nerivec/ember-zli
  * https://github.com/Nerivec/ember-zli/issues/5
  * https://github.com/Nerivec/ember-zli/wiki/Stack#backup-nvm3-tokens

```
$ sudo npm install -g ember-zli
```

## Backup de la clé d’origine ZBDONGLE-E EFR32MG21 dans mon cas

- Bien entendu, il faut que la clé soit montée sur le WSL comme indiqué dans les étapes précédentes.
- **Sous Ubuntu**, lancer la commande **ember-zli stack**
- Dans les choix, l'on choisira :
  * connection type serial,
  * firmware baudrate 115200,
  * le port de notre clé logiquement facilement identifiable,
  * **flow control software**,
  * et finalement **Backup NVM3 tokens** ce qui produira un fichier tokens_backup.nvm3 que nous allons ensuite copier sur la nouvelle clé.

```
$ ember-zli stack

[2024-10-10 08:36:50.585] info: cli: Data folder: /home/fred/ember-zli.
✔ Adapter connection type Serial
✔ Adapter firmware baudrate 115200
✔ Serial port Itead usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2 (/dev/ttyUSB1)
✔ Flow control Software Flow Control (rtscts=false)
[2024-10-10 08:58:53.532] info: zh:ember:ezsp: ======== EZSP starting ========
[2024-10-10 08:58:53.664] info: ember: NCP EZSP protocol version (13) lower than Host. Switched.
....
....
✔ Menu Backup NVM3 tokens
✔ Tokens backup save file Use default (/home/fred/ember-zli/tokens_backup.nvm3)
[2024-10-10 09:00:32.155] info:         zh:ember:tokens: [TOKENS] Saving tokens...
[2024-10-10 09:00:32.980] info:         cli: Tokens backup written to '/home/fred/ember-zli/tokens_backup.nvm3'.
✔ Menu Exit
✔ Restart? (If no, exit) no
```

## Restore sur la nouvelle clé EFR32MG24, dans mon cas une SLZB-07MG24
- ⚠ Je recommande ici de déconnecter la clé source pour éviter un écrasement accidentel.

Lancer à nouveau la commande **ember-zli stack** après vous être assuré que la clé est bien montée et partagée comme indiqué dans les étapes précédentes.
- Dans les choix, l'on choisira :
  * connection type serial,
  * firmware baudrate 115200,
  * le port de notre clé logiquement facilement identifiable,
  * **flow control hardware**,
  * et finalement **Restore NVM3 tokens** qui prendra par défaut le fichier tokens_backup.nvm3 produit précédemment.
- Après ces étape, vous avez maintenant un clone de votre clé

```
ember-zli stack
✔ Adapter connection type Serial
✔ Adapter firmware baudrate 115200
✔ Serial port SMLIGHT usb-SMLIGHT_SLZB-07Mg24 (/dev/ttyUSB0)
✔ Flow control Hardware Flow Control (rtscts=true)
[2024-10-10 09:04:41.374] info: cli: Restored tokens.
```

**Remarque importante**
L’ancien dongle n’est plus utilisable sur le réseau ou à proximité car les deux dongles ont maintenant le même EUI64.
Si l’on souhaite recycler la clé, il faut lui donner un nouvel EUI64. Il faut alors faire un **Reset NVM3 Tokens** puis un **Write EUI64 NVM3 token**. Changer les 2 ou 3 derniers caractères en s’assurant qu’ils ne sont pas pris par un périphérique existant. Je n'ai pas tester ce point souhaitant garder mon ancienne clé en backup.


