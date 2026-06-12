# 🌡️ Thermomètre Connecté DIY - ESP32 WROOM & DS18B20

Ce guide complet retrace toutes les étapes, les pièges évités et la configuration finale pour créer un thermomètre connecté en Wi-Fi à l'aide d'un **ESP32 WROOM 32**, d'une sonde de température **Dallas DS18B20**, et du système **ESPHome / Home Assistant**.

---

## 🛠️ Le Matériel
* **Microcontrôleur :** ESP32 WROOM 32.
* **Capteur :** Sonde étanche Dallas DS18B20.
* **Résistance :** 4.7kΩ (indispensable pour le pull-up du signal).
* **Branchement physique :**
    * **Rouge (VCC) :** Connecté sur la broche **3V3** de l'ESP32.
    * **Noir (GND) :** Connecté sur une broche **GND** de l'ESP32.
    * **Jaune (Data) :** Connecté sur la broche **GPIO5** (D5) de l'ESP32.
    * *Note : La résistance de 4.7kΩ doit être placée entre le fil Rouge (3V3) et le fil Jaune (GPIO5).*

---

## 🚀 Étape 1 : Le Flash Initial Express (Sans surcharger le serveur)
Lors de la première installation, la compilation complète du système par un serveur virtuel (ex: Proxmox) peut prendre plus de 20 minutes si la machine manque de puissance. Pour contourner ce problème, nous avons utilisé la méthode Web ultra-rapide :

1.  Brancher l'ESP32 en USB sur l'ordinateur.
2.  Aller sur le site officiel : [web.esphome.io](https://web.esphome.io/).
3.  Cliquer sur **Connect** et sélectionner le port USB correspondant.
4.  Cliquer sur **PREPARE FOR FIRST USE**.
    * *💡 Astuce de dépannage : Si la connexion échoue ("Failed to initialize"), maintenir le bouton **BOOT** physique de la carte enfoncé au moment de cliquer sur Connexion, puis le relâcher dès que le flashage démarre.*
5.  Une fois le flashage générique terminé (1 min), renseigner les identifiants Wi-Fi directement dans l'interface web pour connecter la carte au réseau local.

---

## 📝 Étape 2 : La Configuration YAML Finale (Le bon code)
Une fois la carte connectée au réseau, elle est automatiquement détectée par l'extension ESPHome de Home Assistant. 

Voici le code YAML complet et optimisé qui fonctionne. Il intègre le module `logger:` (pour voir ce qu'il se passe), la recherche du capteur sur le **GPIO5**, et une **actualisation fluide toutes les 5 secondes** :

```yaml
esphome:
  name: esphome-web-4d7564
  friendly_name: thermo-final
  min_version: 2026.4.0
  name_add_mac_suffix: false

esp32:
  variant: esp32
  framework:
    type: esp-idf

# Active le journal de bord à l'écran (indispensable pour le diagnostic)
logger:

# Autorise la communication avec Home Assistant
api:

# Permet les futures mises à jour sans fil à travers les airs
ota:
  - platform: esphome

# Connexion Wi-Fi (gérée de manière sécurisée par les secrets HA)
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

# Configuration du bus de communication Dallas sur la broche 5
one_wire:
  - platform: gpio
    pin: 5

# Déclaration et personnalisation du thermomètre
sensor:
  - platform: dallas_temp
    name: "Temperature"
    update_interval: 5s

