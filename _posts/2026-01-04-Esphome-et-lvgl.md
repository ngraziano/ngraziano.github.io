---
layout: post
title: "ESPHome et LVGL : prototypage rapide avec la plateforme host et SDL"
date: 2026-01-04 10:50:00 +0100
---

## Pourquoi

J'ai acheté un affichage ePaper : reTerminal E1002. J'en suis content, l'affichage est très lisible, le seul défaut est peut le temps de mise à jour vu qu'il est en couleur.
Cependant j'utilise ESPHome et les graphiques LVGL pour générer l'affichage, le temps de développement s'en ressent car il faut : modifier le code, compiler, charger le code sur l'écran et donc l'avoir à porté de main, vérifier et recommencer. Cela peut prendre beaucoup de temps pour chaque itération.

## Solution

ESPhome possède une plateforme pour résoudre ce problème : host qui s'exécute sur le PC hôte et SDL qui simule un affichage. De plus avec la notion de "package" et de "include" on peut garder la configuration source tel quel et avoir une version qui s'exécute sur le PC.
J'ai utilisé cela sous Windows 10 grâce à WSL.

### Préparation

Installation des dépendance :
sous un WSL Debian :

```bash
# Installer les dépendance de build
sudo apt install libsdl2-dev libsodium-dev build-essential git
# Installer python pour esphome
sudo apt install python3 python3-venv

mkdir epapersimul
cd epapersimul
# Création d'un environement pour installer esphome
python3 -m venv venv
source venv/bin/activate
pip install esphome
```

### Création d'un template pour le simulateur

TODO

### Configuration du transfert de port et ajout à home assistant

TODO

## Conclusion

TODO