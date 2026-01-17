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

Exemple de configuration qui importe le YAML du epaper et désactive les configurations spécifique à l'ESP 32 et active la configuration pour SDL sur l'hote.

```yaml
# Import real epaper display configuration
packages: 
  - !include ./epaper.yaml

# Remove ESP32 configuration 
# and create simulated value for exemple for the battery_voltage
esp32: !remove

wifi: !remove
    ssid: !remove
    password: !remove
    on_connect: !remove

ota: !remove

psram: !remove

deep_sleep: !remove

spi: !remove

i2c: !remove

logger:
  hardware_uart: !remove

sensor: 
  - id: !remove sht4x_sensor
  - id: !remove battery_voltage
  - id: battery_voltage
    platform: template
    lambda: |-
      return 2.0;

time:
  - id: !remove int_time
  - id: int_time
    platform: homeassistant
  - id: !extend ha_time
    on_time_sync: !remove

# Simulate Keyboard keys
binary_sensor:
  - id: !extend gkey
    pin: !remove
    platform: sdl
    key: SDLK_UP
  - id: !extend prevkey
    pin: !remove
    platform: sdl
    key: SDLK_LEFT
  - id: !extend nextkey
    pin: !remove
    platform: sdl
    key: SDLK_RIGHT
  - id: !remove prevent_deepsleep


script:
  - id: !remove readtime
  - id: readtime
    then:
      - logger.log: "Reading time from RTC"
  - id: !remove update_if_not_sleeping
  - id: update_if_not_sleeping
    then:
      - script.execute: update_forcast
  - id: !remove goto_sleep
  - id: goto_sleep
    then:
      - logger.log: "Going to sleep"

# Ass host part

esphome:
  name: epaper_simul

display:
  - id: !remove epaper_display
  - platform: sdl
    id: epaper_display
    dimensions:
      width: 800
      height: 480
    # auto_clear_enabled: true
    show_test_card: false
    update_interval: never

touchscreen:
  platform: sdl

host:
  mac_address: "99:35:69:ab:f6:79"

```

### Configuration du transfert de port et ajout à home assistant

Dans WSL récupérer l'adresse IP du système linux

```bash
ip addr
```

Sortie :

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:cd:f9:ad brd ff:ff:ff:ff:ff:ff
    inet 172.31.100.81/20 brd 172.31.111.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fecd:f9ad/64 scope link
       valid_lft forever preferred_lft forever
```

Pour l'adresse IP 172.31.100.81 la commande pour configurer le proxy de port :

```powershell
netsh interface portproxy add v4tov4 listenport=6053 listenaddress=0.0.0.0 connectport=6053 connectaddress=172.31.100.81
```

Ajouter à Home Assistant en utilisant l'adresse IP du PC sous windows.

## Conclusion

TODO