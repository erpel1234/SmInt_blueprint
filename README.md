# 🧪 SmInt - Smart Interfaces

[![Home Assistant](https://img.shields.io/badge/home%20assistant-%2341BDF5.svg?style=for-the-badge&logo=home-assistant&logoColor=white)](https://www.home-assistant.io/)
[![MQTT](https://img.shields.io/badge/MQTT-660066?style=for-the-badge&logo=mqtt&logoColor=white)](https://mqtt.org/)
[![ESPHome](https://img.shields.io/badge/ESPHome-black?style=for-the-badge&logo=esphome&logoColor=white)](https://esphome.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

**SmInt** (Smart Interfaces) ist ein modulares System zur Erfassung und Zuordnung von Ressourcenverbräuchen in Laborumgebungen.

Es verbindet RFID-Tagreader und Smart Plugs über MQTT mit Home Assistant, um Gerätefreigaben zu steuern und den Energieverbrauch (kWh) exakt einem Experiment, einer Projektgruppe oder einem Nutzer zuzuordnen.

---

## 🎯 Kernfunktionen

* **🔒 Zugangskontrolle:** Geräte laufen nur, wenn eine autorisierte Karte aufliegt.
* **⚡ Live-Tracking:** Automatische Erstellung von MQTT-Geräten in Home Assistant bei Experiment-Start.
* **📊 Exakte Zuordnung:** Verbrauchswerte werden spezifischen Experiment-IDs zugeschrieben.
* **🧩 Modular:** Basiert auf wiederverwendbaren Blueprints – einfach skalierbar auf viele Laborgeräte.

---

## 🏗 Systemarchitektur

Das System besteht aus drei logischen Schichten:

1.  **Hardware Layer (ESP):** Liest die UID der Karte und meldet sie an Home Assistant.
2.  **Logic Layer (Blueprints):**
    * *Mapper:* Erkennt die UID und schaltet den virtuellen Schalter.
    * *Name Generator:* Erzeugt eine eindeutige Experiment-ID.
    * *MQTT Generator:* Erstellt das Gerät dynamisch und übermittelt Daten.
3.  **Data Layer (Helpers):** Speichert Status, Namen und Messwerte temporär.

---

## 📦 Installation & Einrichtung

### Schritt 1: Blueprints importieren

Klicke auf die Buttons, um die Logik-Bausteine direkt in dein Home Assistant zu importieren.

| Modul | Beschreibung | Import |
| :--- | :--- | :--- |
| **Card Mapper** | Verbindet eine physische UID mit einem Schalter. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/DEIN_GITHUB_USER/DEIN_REPO/blob/main/blueprints/automation/smint_card_mapper.yaml) |
| **MQTT Device** | Erzeugt das Home Assistant Gerät & Logik. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/DEIN_GITHUB_USER/DEIN_REPO/blob/main/blueprints/automation/smint_mqtt_device.yaml) |
| **Name Gen** | Skript zur Erzeugung von Experiment-IDs. | [![Import](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/DEIN_GITHUB_USER/DEIN_REPO/blob/main/blueprints/script/smint_name_generator.yaml) |

*(Falls die Buttons nicht gehen: Kopiere die URL der YAML-Dateien aus dem Ordner `/blueprints` und nutze den manuellen Import in HA)*

### Schritt 2: Helfer anlegen (Voraussetzung)

Für **jeden** Tag-Slot (Nutzungskanal) müssen unter `Einstellungen -> Geräte & Dienste -> Helfer` drei Entitäten angelegt werden.

*Beispiel für Tag 001:*
1.  **Input Boolean (Schalter):** `input_boolean.tr_k_01_switch_tag_001`
2.  **Input Text (Experiment Name):** `input_text.experiment_tag001`
3.  **Input Number (Messwert):** `input_number.tr_k_01_kwh_um_added_tag001`

### Schritt 3: Automatisierungen erstellen

Erstelle nun mithilfe der Blueprints die Logik.

#### A. Karten zuweisen (Card Mapper)
Erstelle für jede gültige Karte eine Automation.
* **Reader Sensor:** Der Sensor vom ESP (z.B. `sensor.tr_k_01_uid`)
* **Karten UID:** Die ID auf der Karte (z.B. `90-9D-25-20`)
* **Ziel Schalter:** Der oben erstellte Boolean.

#### B. Das MQTT Gerät (Hauptlogik)
Erstelle eine Automation pro Tag-Slot.

| Feld | Erklärung | Beispiel |
| :--- | :--- | :--- |
| **Reader ID** | **Wichtig:** Kennung für das MQTT Topic. Muss zur Hardware passen. | `tr_k_01_kwh` oder `tr_k_02_kwh` |
| **Auslöser** | Der Boolean Helfer. | `input_boolean.tr_k_01...` |
| **Messwert** | Der Number Helfer. | `input_number.tr_k_01...` |
| **Name** | Der Text Helfer. | `input_text.experiment...` |

---

## ⚡ Funktionsablauf

1.  **Start:** Nutzer legt Karte auf **TR-K-01**.
2.  **Auth:** Card Mapper erkennt UID → Schaltet `input_boolean` auf **ON**.
3.  **Init:** MQTT Generator reagiert auf ON:
    * Sendet MQTT Discovery Config.
    * Ein **neues Gerät** erscheint in Home Assistant (z.B. "03_Mikrowelle").
4.  **Run:** Experiment läuft, Sensoren in `configuration.yaml` berechnen den Verbrauch.
5.  **Stop:** Nutzer entfernt Karte → Boolean geht auf **OFF**.
6.  **Log:** MQTT Generator wartet kurz und sendet den finalen **kWh-Wert** an das System.

---

## 🛠 Troubleshooting

* **Gerät taucht nicht auf:** Prüfe in der Automation, ob die `Reader ID` (z.B. `tr_k_01_kwh`) exakt stimmt und keine Leerzeichen hat.
* **Import Button Fehler:** Stelle sicher, dass dieses GitHub-Repository auf **Public** gestellt ist oder nutze den manuellen Import.
* **Keine Verbrauchswerte:** Stelle sicher, dass die Template-Sensoren in deiner `configuration.yaml` definiert sind und Werte liefern.

---

*SmInt Project - Open Source Lab Automation*
