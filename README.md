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

* 









Note : Installation et paramétrage basé sur https://www.raspberrypi.org/learning/weather-station-guide/manual-setup.md disponible sous Créative Commons également via : https://github.com/raspberrypilearning/weather-station-guide
