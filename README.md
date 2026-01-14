# SmInt_blueprint
SmInt (Smart Interfaces): Ein System zur personalisierten Ressourcen- und Verbrauchserfassung in Laboren. Ermöglicht die sichere Gerätefreigabe und Experiment-Zuordnung mittels Tag-Readern und ID-Karten über MQTT und Home Assistant.

# 🧩 SmInt - Universal MQTT Device Generator (Blueprint)

Dieser Blueprint verbindet die **SmInt Tag-Reader Hardware** mit Home Assistant. Er sorgt dafür, dass dynamisch MQTT-Sensoren erstellt werden, sobald ein Experiment gestartet wird, und übermittelt den Verbrauchswert, wenn das Experiment endet.

**Funktion:**
* **Start (Switch ON):** Sendet eine MQTT Auto-Discovery Konfiguration. Ein neues Gerät erscheint in Home Assistant.
* **Stopp (Switch OFF):** Sendet den finalen Verbrauchswert (kWh) an das zuvor erstellte Gerät.

---

## 📋 Voraussetzungen

Bevor du diesen Blueprint nutzt, müssen für jeden Tag-Reader folgende **Helfer (Helpers)** in Home Assistant existieren:

1.  **Input Boolean** (Der Schalter/Trigger)
    * *Beispiel:* `input_boolean.tr_k_01_switch_tag_001`
2.  **Input Text** (Name des Experiments/Users)
    * *Beispiel:* `input_text.experiment_tag001`
3.  **Input Number** (Der gemessene kWh Wert)
    * *Beispiel:* `input_number.tr_k_01_kwh_um_added_tag001`

---

## ⚙️ Konfiguration

Erstelle eine neue Automatisierung basierend auf diesem Blueprint und fülle die Felder wie folgt aus:

| Feld | Beschreibung | Beispiel |
| :--- | :--- | :--- |
| **Auslöser-Schalter** | Der Boolean, der signalisiert, ob eine Karte aufliegt. | `input_boolean.tr_k_01_switch...` |
| **Experiment Name** | Der Text-Helfer, der die ID des aktuellen Experiments enthält. | `input_text.experiment_tag001` |
| **Messwert** | Der Nummer-Helfer mit dem aktuellen Verbrauch. | `input_number.tr_k_01_kwh...` |
| **Reader ID für MQTT** | **Wichtig:** Die eindeutige Kennung des Readers im MQTT Topic. Muss exakt zur Hardware passen. | `tr_k_01_kwh` (für Reader 1)<br>`tr_k_02_kwh` (für Reader 2) |
| **Anzeigename** | Wie soll das Gerät in Home Assistant heißen? | `Mikrowelle` oder `Absaugung` |
| **Hersteller** | (Optional) Name für die Geräte-Info. | `Karte 001` |

---

## 🛠 Funktionsweise im Detail

Der Blueprint nutzt `mqtt.publish` mit dynamischen Templates.

1.  **Topic Struktur:**
    `homeassistant/sensor/[EXPERIMENT_ID]/[READER_ID]/config`
    `homeassistant/sensor/[EXPERIMENT_ID]/[READER_ID]/state`

2.  **Ablauf:**
    * Wenn der **Schalter auf ON** geht, wird ein JSON-Payload an das `config`-Topic gesendet. Home Assistant erkennt dies als **MQTT Discovery** und erstellt (oder aktualisiert) das Sensor-Gerät sofort.
    * Wenn der **Schalter auf OFF** geht, wartet die Automation **1 Sekunde** (um sicherzustellen, dass der ESP den letzten Wert gesendet hat) und pusht dann den Wert aus dem Input Number Helfer an das `state`-Topic.

---

## 📥 Installation

Klicke auf den Button unten, um diesen Blueprint direkt in deine Home Assistant Instanz zu importieren:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=(https://github.com/erpel1234/SmInt_blueprint/edit/main/blueprints/automation))
