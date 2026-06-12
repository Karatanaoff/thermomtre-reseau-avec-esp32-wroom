readme_v3_content = """# 🌡️ Guide Ultime : Thermomètre Connecté DIY (ESP32 WROOM & DS18B20)

Ce guide complet regroupe toutes les étapes, les prérequis, les configurations de sécurité et les solutions aux pièges matériels/logiciels pour créer un thermomètre connecté en Wi-Fi. Ce projet intègre un **ESP32 WROOM 32**, une sonde étanche **Dallas DS18B20**, le tout piloté par **ESPHome** et centralisé sur **Home Assistant**.

---

## 📋 1. Prérequis Obligatoires

### 🖥️ Logiciels
* **Un serveur Home Assistant** fonctionnel (installé sur Proxmox, Raspberry Pi, etc.).
* **L'add-on ESPHome** installé et configuré dans Home Assistant.
* Un navigateur web moderne basé sur Chromium (Google Chrome, Microsoft Edge) pour le flashage initial par câble.

### 🎛️ Matériels
* **Microcontrôleur :** ESP32 WROOM 32 (avec son câble data micro-USB).
* **Capteur :** Sonde de température étanche Dallas DS18B20 (3 fils : Rouge, Noir, Jaune).
* **Résistance :** 4.7kΩ (indispensable pour le protocole 1-Wire).

---

## 🪛 2. Schéma de Câblage Physique

Le capteur DS18B20 utilise le protocole 1-Wire et nécessite impérativement une résistance de "pull-up" pour que l'ESP32 puisse lire les données :

1.  **Fil Rouge (VCC) :** Connecté sur la broche **3V3** de l'ESP32.
2.  **Fil Noir (GND) :** Connecté sur une broche **GND** de l'ESP32.
3.  **Fil Jaune (Data) :** Connecté sur la broche **GPIO5** (notée **D5** ou **5** sur la carte).
4.  **La Résistance (4.7kΩ) :** Doit être insérée **entre le fil Rouge (3V3) et le fil Jaune (GPIO5)**.

*⚠️ Attention : Si le capteur devient instantanément brûlant au toucher, débranchez tout immédiatement, cela signifie que le VCC et le GND ont été inversés.*

---

## 🔐 3. Configuration des Identifiants Wi-Fi (Secrets HA)

Avant d'écrire le code de la carte, il faut enregistrer vos accès Wi-Fi de manière sécurisée dans Home Assistant pour pouvoir utiliser les variables `!secret`.

1.  Dans Home Assistant, allez dans l'onglet **ESPHome Builder**.
2.  En haut à droite de l'écran, cliquez sur le bouton **SECRETS**.
3.  Ajoutez ou complétez le fichier avec vos identifiants (respectez bien la casse) :
    ```
```text?code_stdout&code_event_index=2
README-v3.md généré avec succès.

```yaml
    wifi_ssid: "LE_NOM_DE_VOTRE_BOX"
    wifi_password: "VOTRE_MOT_DE_PASSE_WIFI"
    ```
4.  Cliquez sur **Save** pour enregistrer.

---

## 🚀 4. Étape 1 : Le Flash Initial Express (Méthode Web)

*Pourquoi cette méthode ?* Lors du premier flashage, compiler l'intégralité du système d'exploitation de l'ESP32 sur une machine virtuelle (comme Proxmox) un peu limitée peut prendre plus de 20 minutes ou planter. Utiliser l'outil web officiel permet d'installer un micrologiciel de base en 1 minute chrono sans solliciter votre serveur.

1.  Branchez l'ESP32 en USB sur votre ordinateur.
2.  Ouvrez le site officiel : **[web.esphome.io](https://web.esphome.io/)**
3.  Cliquez sur **CONNECT** et sélectionnez le port USB de votre carte.
4.  Cliquez sur **PREPARE FOR FIRST USE** et laissez la jauge aller jusqu'à 100 %.
    * *💡 Astuce de dépannage ("Failed to initialize") : Si le site affiche une erreur au démarrage, maintenez le bouton physique **BOOT** (ou IO0) enfoncé sur l'ESP32, cliquez sur Connexion/Retry sur l'écran, puis relâchez le bouton dès que l'installation commence.*
5.  À la fin du flashage, une fenêtre vous demande vos identifiants Wi-Fi : renseignez votre réseau **2.4 GHz** (l'ESP32 ne gère pas le 5 GHz). 
6.  Une fois connecté, la carte obtiendra une adresse IP et apparaîtra en vert avec la mention **DISCOVERED** dans l'interface ESPHome de Home Assistant.

---

## 📝 5. Étape 2 : Le Code YAML Complet et Final

Dans votre interface **ESPHome** sur Home Assistant, cliquez sur **TAKE CONTROL / ADOPTER** sur l'appareil détecté, donnez-lui un nom (ex: `thermo-final`), puis ouvrez l'éditeur (**EDIT**). 

Remplacez ou complétez le contenu pour obtenir ce code exact (qui active les logs, configure le capteur sur la bonne broche et accélère le rafraîchissement) :

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

# Active le journal de bord à l'écran (Indispensable pour le débug technique)
logger:

# Autorise la communication avec Home Assistant
api:

# Permet les futures mises à jour sans fil à travers les airs
ota:
  - platform: esphome

# Connexion Wi-Fi via le fichier Secrets configuré à l'étape 3
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

# Configuration du bus de communication Dallas sur le GPIO5
one_wire:
  - platform: gpio
    pin: 5

# Déclaration et personnalisation de la sonde de température
sensor:
  - platform: dallas_temp
    name: "Temperature"
    update_interval: 5s  # Actualisation rapide toutes les 5 secondes
