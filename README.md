# 🏠 Home Assistant + VMC S&P Domeo EVO 225 FL (Modbus TCP/RTU)

Intégration complète de la **VMC double flux S&P Domeo EVO 225 FL** dans **Home Assistant** à l’aide du **convertisseur RS485 ↔ Ethernet Waveshare RS485 TO POE ETH (B)**.  
Cette configuration permet la lecture des données de température, humidité, débit et état du filtre, ainsi que l’envoi de commandes (bypass, mode vacances, etc.).

---

## ⚙️ Matériel utilisé

| Équipement | Référence | Description |
|-------------|------------|--------------|
| 💨 VMC double flux | S&P Domeo EVO 225 FL | Version RD, sortie Modbus |
| 🔌 Convertisseur RS485 | Waveshare RS485 TO POE ETH (B) | Convertisseur RS485 ↔ Ethernet, alimentation PoE |
| 🧠 Serveur domotique | Raspberry Pi 5 | Sous Home Assistant OS |
| 🌐 Réseau local | Freebox Ultra | IP fixe et routage local |

---

## 🌐 Configuration réseau

| Périphérique | IP fixe | Port | Protocole |
|---------------|---------|------|------------|
| Waveshare | `192.168.1.200` | `502` | TCP Server |
| Home Assistant | `192.168.1.142` | — | Client Modbus |
| Passerelle | `192.168.1.254` | — | Freebox Ultra |

---

## 🧩 Paramètres Waveshare

- **Work Mode** : `TCP Server`
- **Device Port** : `502`
- **Baudrate** : `9600`
- **Databits** : `8`
- **Stopbits** : `2`
- **Parity** : `Even`
- **Protocol** : `Modbus TCP to RTU`
- **Enable Multi-host** : `Yes`
- **IP mode** : `Static`

🟢 *Les jumpers JP4 et JP1 de la VMC doivent être fermés pour activer le Modbus.*

---

## 📄 Table Modbus officielle

La table complète des registres est fournie dans le document :

📘 **[Télécharger la table Modbus officielle (PDF)](./TABLE%20MODBUS%20DOMEO%20EVO_fr.pdf)** 
*(S&P France – Version officielle)*

---

## 🧠 Configuration Home Assistant

Fichier : `configuration.yaml`

```yaml
# ────────────────────────────────────────────────
#  VMC S&P Domeo EVO 225 FL - Modbus TCP/RTU
# ────────────────────────────────────────────────
modbus:
  - name: domeo225
    type: tcp
    host: 192.168.1.200
    port: 502
    timeout: 10
    delay: 3
    retries: 5
    message_wait_milliseconds: 300

    sensors:
      - name: "VMC Débit actuel"
        slave: 1
        input_type: input
        address: 3
        unit_of_measurement: "m³/h"
        state_class: measurement

      - name: "VMC HR"
        slave: 1
        input_type: input
        address: 4
        unit_of_measurement: "%"
        device_class: humidity
        state_class: measurement

      - name: "VMC T air extrait"
        slave: 1
        input_type: input
        address: 10
        scale: 0.1
        precision: 1
        unit_of_measurement: "°C"
        device_class: temperature

      - name: "VMC T air rejeté"
        slave: 1
        input_type: input
        address: 11
        scale: 0.1
        precision: 1
        unit_of_measurement: "°C"
        device_class: temperature

      - name: "VMC T air extérieur"
        slave: 1
        input_type: input
        address: 12
        scale: 0.1
        precision: 1
        unit_of_measurement: "°C"
        device_class: temperature

      - name: "VMC T air soufflé"
        slave: 1
        input_type: input
        address: 13
        scale: 0.1
        precision: 1
        unit_of_measurement: "°C"
        device_class: temperature

      - name: "VMC T avant échangeur"
        slave: 1
        input_type: input
        address: 14
        scale: 0.1
        precision: 1
        unit_of_measurement: "°C"
        device_class: temperature

      - name: "VMC Mode courant (code)"
        slave: 1
        input_type: input
        address: 2
        data_type: uint16

    binary_sensors:
      - name: "VMC Filtre sale"
        slave: 1
        input_type: discrete_input
        address: 10
        device_class: problem

      - name: "VMC Moteur 1 erreur"
        slave: 1
        input_type: discrete_input
        address: 1
        device_class: problem

      - name: "VMC Moteur 2 erreur"
        slave: 1
        input_type: discrete_input
        address: 2
        device_class: problem

    switches:
      - name: "VMC By-pass manuel"
        slave: 1
        address: 5
        write_type: coil
        verify:
          input_type: coil
          address: 5

      - name: "VMC Mode vacances"
        slave: 1
        address: 3
        write_type: coil
        verify:
          input_type: coil
          address: 3
```

---

## 🖥️ Interface Home Assistant

Une fois la configuration rechargée, les entités suivantes apparaissent automatiquement dans **Home Assistant** :  

- 🌡️ `sensor.vmc_t_air_extrait` — température air extrait  
- 🌡️ `sensor.vmc_t_air_rejete` — température air rejeté  
- 🌡️ `sensor.vmc_t_air_exterieur` — température air extérieur  
- 🌡️ `sensor.vmc_t_air_souffle` — température air soufflé  
- 🌡️ `sensor.vmc_t_avant_echangeur` — température avant échangeur  
- 💧 `sensor.vmc_hr` — humidité relative  
- 💨 `sensor.vmc_debit_actuel` — débit instantané  
- ⚙️ `sensor.vmc_mode_courant_code` — mode de fonctionnement  
- ⚠️ `binary_sensor.vmc_filtre_sale` — filtre à remplacer  
- ⚠️ `binary_sensor.vmc_moteur_1_erreur` — erreur moteur extraction  
- ⚠️ `binary_sensor.vmc_moteur_2_erreur` — erreur moteur insufflation  
- 🔘 `switch.vmc_by_pass_manuel` — commande du by-pass  
- 🌴 `switch.vmc_mode_vacances` — mode absence

---

## 🧩 Exemple de carte Lovelace

```yaml
type: vertical-stack
cards:
  - type: markdown
    content: |
      ## 🌬️ VMC Domeo 225 FL
      Suivi : débit, humidité, températures, état du filtre & commandes.
  - type: entities
    title: Ventilation & Hygrométrie
    entities:
      - entity: sensor.vmc_debit_actuel
        name: Débit actuel (m³/h)
      - entity: sensor.vmc_hr
        name: Humidité (%)
      - entity: sensor.vmc_mode_courant_code
        name: Mode courant (code brut)
      - entity: binary_sensor.vmc_filtre_sale
        name: Filtre sale
  - type: entities
    title: Températures
    entities:
      - entity: sensor.vmc_t_air_extrait
        name: Air extrait (°C)
      - entity: sensor.vmc_t_air_rejete
        name: Air rejeté (°C)
      - entity: sensor.vmc_t_air_exterieur
        name: Air extérieur (°C)
      - entity: sensor.vmc_t_air_souffle
        name: Air soufflé (°C)
      - entity: sensor.vmc_t_avant_echangeur
        name: Avant échangeur (°C)
  - type: horizontal-stack
    cards:
      - type: gauge
        name: Débit
        entity: sensor.vmc_debit_actuel
        min: 0
        max: 500
      - type: gauge
        name: Humidité
        entity: sensor.vmc_hr
        min: 0
        max: 100
  - type: horizontal-stack
    cards:
      - type: button
        name: 🚀 Boost ON
        icon: mdi:fan-chevron-up
        tap_action:
          action: call-service
          service: script.vmc_boost_on
      - type: button
        name: 🛑 Boost OFF
        icon: mdi:fan-off
        tap_action:
          action: call-service
          service: script.vmc_boost_off
  - type: entities
    title: Modes & by-pass
    entities:
      - entity: switch.vmc_mode_vacances
        name: Mode vacances
      - entity: switch.vmc_by_pass_manuel
        name: By-pass manuel
grid_options:
  columns: 48
  rows: auto
```

## ⚡ Exemple d’automatisation

Exemple : activer le mode boost de la VMC si l’humidité dépasse 70 % dans la salle de bain.
Pour ma part j'ai des capteurs de températures et humidité zigbee



```yaml
# ═════════════════════════════════════════════════════════════════════════════
# VMC - Gestion automatique humidité SDB & SDE
# ═════════════════════════════════════════════════════════════════════════════

description: Active le boost VMC si humidité > 70 %, désactive si < 60 %
trigger:
  - platform: numeric_state
    entity_id: sensor.capteur_h_et_t_sde_master_humidite
    above: 70
    id: boost_rdc
  - platform: numeric_state
    entity_id: sensor.capteur_h_et_t_sde_master_humidite
    below: 60
    id: stop_rdc
  - platform: numeric_state
    entity_id: sensor.capteur_h_et_t_sde_rdj_humidite
    above: 70
    id: boost_rdj
  - platform: numeric_state
    entity_id: sensor.capteur_h_et_t_sde_rdj_humidite
    below: 60
    id: stop_rdj

condition: []

action:
  - choose:
      # --- Salle de bain RDC (Master) ---
      - conditions:
          - condition: trigger
            id: boost_rdc
        sequence:
          - service: modbus.write_register
            data:
              hub: domeo225
              address: 2
              slave: 1
              value: 3   # Boost
      - conditions:
          - condition: trigger
            id: stop_rdc
        sequence:
          - service: modbus.write_register
            data:
              hub: domeo225
              address: 2
              slave: 1
              value: 1   # Normal

      # --- Salle d’eau RDJ ---
      - conditions:
          - condition: trigger
            id: boost_rdj
        sequence:
          - service: modbus.write_register
            data:
              hub: domeo225
              address: 2
              slave: 1
              value: 3
      - conditions:
          - condition: trigger
            id: stop_rdj
        sequence:
          - service: modbus.write_register
            data:
              hub: domeo225
              address: 2
              slave: 1
              value: 1

mode: single
```

## 🧰 Dépannage

Symptôme	Cause possible	Solution
❌ Not connected [AsyncModbusTcpClient]	Mauvais port ou IP	Vérifier IP statique et port 502 
⚠️ No response after 3 retries	Câblage RS485 inversé	Inverser A/B et vérifier les jumpers JP1/JP4 ou Ou la configuration du wareshare notament Baud rate, parity et Stopbits
💾 Data incohérente	Parité incorrecte	Utiliser Even et 2 stop bits
⏳ Pas de lecture	Adresse Modbus incorrecte	Vérifier la table officielle fournie
🔌 Alim instable	Mauvais PoE ou câble RJ45	Vérifier la tension du port PoE
🚫 Interface injoignable (192.168.1.200)	Conflit d’IP ou reset non appliqué	Redémarrer le Waveshare et la Freebox
🔄 Connexion aléatoire	Timeout trop court	Passer timeout: 10 et delay: 3 dans configuration.yaml
🧱 Blocage total	Modbus figé après coupure	Redémarrer la VMC + Waveshare (PoE OFF/ON 10s)

 ## Schéma de câblage

VMC (RS485)             WAVESHARE RS485 TO POE ETH (B)
------------------------------------------------------
A -------------------  485A
 
B -------------------  485B 

GND ----------------  GND 

Alim PoE -----------  RJ45 (PoE IN) 

💡 Si tu ajoutes plusieurs appareils RS485, termine la ligne avec une résistance de 120 Ω. Jumpers JP4 ouvert

## 🧑‍💻 Auteur

@xtozez

📅 Dernière mise à jour : Novembre 2025

## 📜 Licence

Ce projet est distribué sous la licence MIT.
Vous êtes libre de l’utiliser, le modifier et le redistribuer à condition de conserver la mention d’auteur.
⭐ Contribuer
Les contributions sont les bienvenues !
Si vous améliorez la configuration (autres modèles Domeo, débits adaptatifs, etc.),
n’hésitez pas à proposer une pull request ou à ouvrir une issue.

## Remerciements

S&P France pour la documentation Modbus. très réactifs par mail et tel.
Waveshare pour son convertisseur RS485 ↔ Ethernet.
Communauté Home Assistant France pour le partage de configurations.
