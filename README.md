# 🌡️ Thermomètre Connecté DIY - ESP32 WROOM & DS18B20

Ce guide complet retrace toutes les étapes, les prérequis indispensables, les pièges évités et la configuration finale pour créer un thermomètre connecté en Wi-Fi à l'aide d'un **ESP32 WROOM 32**, d'une sonde de température **Dallas DS18B20**, et du système **ESPHome / Home Assistant**.

---

## 📋 Prérequis Matériels & Logiciels
* **Un serveur Home Assistant** fonctionnel (installé sur Proxmox, Raspberry Pi ou autre) avec l'add-on **ESPHome** installé.
* **Microcontrôleur :** ESP32 WROOM 32.
* **Capteur :** Sonde étanche Dallas DS18B20.
* **Résistance :** 4.7kΩ (indispensable pour le pull-up du signal).

### 🪛 Branchement physique :
* **Rouge (VCC) :** Connecté sur la broche **3V3** de l'ESP32.
* **Noir (GND) :** Connecté sur une broche **GND** de l'ESP32.
* **Jaune (Data) :** Connecté sur la broche **GPIO5** (D5) de l'ESP32.
* *Note : La résistance de 4.7kΩ doit impérativement être placée entre le fil Rouge (3V3) et le fil Jaune (GPIO5).*

---

## 🔐 Étape Importante : Les Identifiants Wi-Fi (Secrets)
Pour que le code YAML ci-dessous fonctionne sans erreur, vous devez enregistrer vos identifiants Wi-Fi dans le fichier de sécurité de Home Assistant.

1. Dans Home Assistant, allez dans l'onglet **ESPHome**.
2. En haut à droite, cliquez sur le bouton **SECRETS** (ou modifiez directement le fichier `secrets.yaml`).
3. Ajoutez-y vos identifiants sous cette forme exacte :
