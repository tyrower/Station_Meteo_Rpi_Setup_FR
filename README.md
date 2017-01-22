# Station_Meteo_Rpi_Setup_FR
Tutoriel Pas-à-Pas pour installation manuelle de la Station Météo Oracle/Raspberry Pi de la RPF

* Télécharger et installer la dernière version de Raspbian à partir de https://www.raspberrypi.org/downloads/raspbian/
* Activer SSH par défaut en ajoutant un fichier "à vide" nommé "ssh" dans la partition "boot"
* Démarrer et reperer adresse IP sur le réseau (ou avec un écran hostname -I)
* Modifier le mot de passe par défaut (raspberry)
* Mettre à jour le système sudo apt-get update && sudo apt-get upgrade
* Mettre à jour le firmware sudo rpi-update
* Redemarrer

Installation Station Météo RPi/Oracle propre
* Localisation pour mettre système en Français et clavier AZERTY
CLI > sudo raspi-config > Internationalisation Options
Change locale : fr_FR.UTF8 UTF-8 et mettre "par défaut"
Change timezone : Paris
Change keyboard layout : French

Ou via mode Graphique : 
Menu Preferences > Raspberry Pi Configuration > Onglet "Localisation"

* Activer I2C interface : 
CLI > sudo raspi-config > Interfacing options 

Ou via mode Graphique : 
Preferences > Raspberry Pi Configuration > Onglet 4 "Localisation"

* Télécharger les fichiers nécessaires à l'installation de la station météo : 
cd ~ && git clone https://github.com/raspberrypi/weather-station

Nous avons quelques modifications à faire au démarrage du système donc on va modifier la configuration dans le fichier "config.txt"
* sudo nano /boot/config.txt
Ajouter les lignes suivantes en bas de fichier : 
* dtoverlay=w1-gpio
* dtoverlay=pcf8523-rtc
CTRL + O, Entrer, CTRL + X pour enregistrer et sortir.
Puis 
* sudo nano /etc/modules
Ajouter en bas de fichier : 
* i2c-dev
* w1-therm
CRTL + O, Entrer, CTRL + X pour enregistrer et sortir.

* sudo halt
Brancher le Weather Station HAT

Partie Horloge Temps Réel ("RTC")
Redémarrer puis tester que l'horloge embarqué "RTC" est bien active avec la commande :
* ls /dev/rtc* 
Ca devrait donner "/dev/rtc0"

Après reboot, vérifier l'heure et date du système avec : 
* date
Si tout est correct, valider les informations avec 
* sudo hwclock -w

Mettre à jour le fichier :
* sudo nano /lib/udev/hwclock-set
avec les détails suivants : 
if [ yes = "$BADYEAR" ] ; then
    /sbin/hwclock --rtc=$dev --hctosys --badyear
else
    /sbin/hwclock --rtc=$dev --hctosys
fi

Enlever le paquet d'horloge factice :
* sudo update-rc.d fake-hwclock remove
* sudo apt-get remove fake-hwclock -y

Vérifier les capteurs 
Commencer par installer les paquets dépendents : 
* sudo apt-get install i2c-tools python-smbus telnet -y
Retrouver les modules I2C avec la commande : 
* sudo i2cdetect -y 1
qui devrait donner ceci (ou similaire) : 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: 40 -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- UU 69 6a -- -- -- -- -- 
70: -- -- -- -- -- -- -- 77        

Clé : 
40 = HTU21D, la sonde d'humidié et temperature.
77 = BMP180, capteur de pression atmosphérique
68 = PCF8523, l'horloge "temps réel" (RTC). Ça devrait s'afficher comme "UU" car il est déjà réservé par le pilote. 
69 = MCP3427, le convertisseur Anaogue-Numérique sur la carte principale.
6a = MCP3427, le convertisseur Anaogue-Numérique sur la carte secondaire de qualité d'air.

Création et configuration de la base de données : 
Installer les paquets requis : 
* sudo apt-get update
* sudo sudo apt-get install apt-transport-https
* sudo apt-get install apache2 mysql-server python-mysqldb php5 libapache2-mod-php5 php5-mysql -y

Créer un mot de passe "root" pour votre accès aux bases de données.

Créer une nouvelle bdd : 
* mysql -u root -p
* CREATE DATABASE weather;
* USE weather;

Créer une table pour stocker les données : 
CREATE TABLE WEATHER_MEASUREMENT(
    ID BIGINT NOT NULL AUTO_INCREMENT,
    REMOTE_ID BIGINT,
    AMBIENT_TEMPERATURE DECIMAL(6,2) NOT NULL,
    GROUND_TEMPERATURE DECIMAL(6,2) NOT NULL,
    AIR_QUALITY DECIMAL(6,2) NOT NULL,
    AIR_PRESSURE DECIMAL(6,2) NOT NULL,
    HUMIDITY DECIMAL(6,2) NOT NULL,
    WIND_DIRECTION DECIMAL(6,2) NULL,
    WIND_SPEED DECIMAL(6,2) NOT NULL,
    WIND_GUST_SPEED DECIMAL(6,2) NOT NULL,
    RAINFALL DECIMAL (6,2) NOT NULL,
    CREATED TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY ( ID )
  );
  
Sortir de MYSQL avec la commande "exit" ou CTRL + D
  
Installer l'application pour les capteurs : 
* cd ~
* git clone https://github.com/raspberrypi/weather-station.git
Il est fort probable que ce repertoire existe déjà. 

Démarrer et tester le serveur (daemon) "Weather Station"
* sudo ~/weather-station/interrupt_daemon.py start

Se connecter pour tester les données : 
* telnet localhost 49501

Envoyer des commandes :
Les commandes de texte peuvent être envoyées : 
RAIN: affichage de la quantité de pluie en ml
WIND: affichage de la vitesse du vent en kph
GUST: affichage de la vitesse maximale de rafale de vent en kph
RESET: Remise à zéro des valeurs de pleuviométrie et de l'anénomètre 
BYE: quitter

Lancer le daemon/service Station Météo au Démarrage du Pi
* sudo nano /etc/rc.local

Ajouter les lignes : 
* echo "Starting Weather Station daemon..."
* /home/pi/weather-station/interrupt_daemon.py start
avant "exit 0" 
Enregistrer et quitter CTRL + O, Entrer et CTRL +X.

Paramétrer les accès BDD dans MYSQL
* cd ~/weather-station
* nano credentials.mysql
Ajouter le mot de passe configuré à la création de la base de données. 

Automatiser la mise à jour de la base de données : 
Pour démarrer : 
* crontab < crontab.save
Pour arreter : 
* crontab -r
Réactiver :
* crontab < ~/weather-station/crontab.save

Lancement manuel du script : 
* sudo ~/weather-station/log_all_sensors.py

Consulter les données : 
* mysql -u root -p
* USE weather;
* SELECT * FROM WEATHER_MEASUREMENT;

Paramétrer Identifiant/MdP pour Oracle DB
* cd ~/weather-station
* nano credentials.oracle.template
* mv credentials.oracle.template credentials.oracle

Vérifier connexion en téléversement : 
* sudo ~/weather-station/upload_to_oracle.py













Note : Installation et paramétrage basé sur https://www.raspberrypi.org/learning/weather-station-guide/manual-setup.md disponible sous Créative Commons également via : https://github.com/raspberrypilearning/weather-station-guide
