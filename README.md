# Station_Meteo_Rpi_Setup_FR
Tutoriel Pas-à-Pas pour installation manuelle de la Station Météo Oracle/Raspberry Pi de la RPF

* Télécharger et installer la dernière version de Raspbian à partir de https://www.raspberrypi.org/downloads/raspbian/
* Activer SSH par défaut en ajoutant un fichier "à vide" nommé "ssh" dans la partition "boot"
* Démarrer et reperer adresse IP sur le réseau (ou avec un écran hostname -I)
* Modifier le mot de passe par défaut (raspberry)
* Mettre à jour le système sudo apt-get update && sudo apt-get upgrade
* Mettre à jour le firmware sudo rpi-update
