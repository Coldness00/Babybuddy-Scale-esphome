# 🍼 Smart Baby Bottle Scale  
### Measure every feeding automatically — ESP32 + ESPHome + Home Assistant + BabyBuddy  
<div align="center">
  
**Fully 3D‑printed • Open‑source • Wifi‑connected • Accurate to the gram**

</div>

---

## 🎯 Overview

This project is a **smart baby bottle scale** built with:

- an **ESP32‑S3**,  
- an **HX711 load cell amplifier**,  
- a **load cell**,  
- an **OLED display**,  
- RGB LEDs,  
- and **ESPHome**.

It automatically measures how much your baby drinks and sends the result directly to **Home Assistant**, which then creates a feeding entry in **BabyBuddy**.

Everything is fully autonomous, compact, and 3D‑printed.

---

## 🧩 Features

- **Accurate milk consumption tracking** (using calibrated load cell + HX711)
- **Automatic BabyBuddy feeding logs** through Home Assistant
- 3D‑printed enclosure (Bambu Studio `.3mf` included)
- **OLED screen** showing:
  - Bottle weight
  - Consumption
  - Last feeding time
  - Time since last feeding  
- **RGB LED feedback**
  - White fade in → bottle placed  
  - Yellow fade out → feeding started  
  - Green fade out → feeding finished  
  - Red pulse → error or invalid state  
- **Two physical buttons**
  - FULL → marks the bottle before feeding  (when pressed a circle is drawn on the top right corner of the screen)
  - EMPTY → marks the bottle after feeding  
- **Static IP + fallback Wi‑Fi AP**
- **State restored after reboot** (timestamps, full weight, etc.)

---

## 🛒 Bill of Materials (BOM)

### Electronics

| Item | Description | Link |
|------|-------------|------|
| Load Cell + HX711 Kit | 50kg scale sensor with amplifier | https://fr.aliexpress.com/item/1005006165499175.html |
| ESP32‑S3 DevKitC‑1 | Main controller running ESPHome | - |
| SSD1306 OLED (128×64) | I²C display | https://fr.aliexpress.com/item/1005006262908701.html |
| WS2812 RGB LED strip | 3 LEDs | — |
| 2× Momentary push buttons | FULL + EMPTY | — |
| 8× M3 screws | — |

### 3D‑Printed Parts
- Complete `.3mf` Bambu Studio project included  
- Designed for snap‑fit assembly  
- Supports:
  - Load cell mount  
  - Front panel for OLED + buttons  
  - Compartment for ESP32 + HX711  

<img width="639" height="340" alt="image" src="https://github.com/user-attachments/assets/24b4defe-aa6a-4d42-8ad7-b2d8dc9e890e" />

---

## 📸 Photos  

<img width="471" height="630" alt="image" src="https://github.com/user-attachments/assets/8f585d04-5fa4-4840-8bbd-40016426f995" />
<img width="540" height="619" alt="image" src="https://github.com/user-attachments/assets/3c43166e-ca98-4610-9e9d-ce2b23d729dc" />
<img width="470" height="450" alt="image" src="https://github.com/user-attachments/assets/48e586a4-5adc-463d-955f-6cf984e1a416" />


---

## 📡 How It Works

### 1. Press **FULL** before feeding  
- Weight is stored as “full bottle”  
- Timestamp saved  
- Yellow LED effect  
- Display shows full bottle icon  

### 2. Press **EMPTY** after feeding  
- New weight measured  
- Consumption is calculated  
- Data is sent to Home Assistant  
- Green LED effect  

### 3. Home Assistant Automation  
Home Assistant receives the values and automatically calls the **BabyBuddy API** to create a feeding entry (Formula / Bottle).

---

## 📦 ESPHome YAML Configuration

Full ESPHome config used in this project is available here:

`/esphome/biberon.yaml`

It includes:
- I²C OLED setup  
- HX711 calibration table  
- RGB LED effects  
- State persistence  
- Two GPIO buttons (full/empty)  
- Consumption calculation logic  
- Timestamp handling via Home Assistant time source  

---

## 🏠 Home Assistant Automation

Place this YAML inside your HA automations and edit accordingly the device_id:

```yaml
alias: Baby feeding bottle
description: ""
triggers:
  - trigger: state
    entity_id: sensor.biberon_bottle_consumption

conditions:
  - condition: template
    value_template: >
      {{ states('sensor.biberon_bottle_full_button_last_press') not in
         ['unknown','unavailable',''] and
         states('sensor.biberon_bottle_empty_button_last_press') not in
         ['unknown','unavailable',''] }}

  - condition: template
    value_template: >
      {{ states('sensor.biberon_bottle_consumption') | int(0) > 0 }}

actions:
  - delay: "00:00:02"
  - action: babybuddy.add_feeding
    target:
      device_id: 51762e63db1082c325ccfa7d02a4f4d6
    data:
      type: Formula
      method: Bottle
      start: "{{ states('sensor.biberon_bottle_full_button_last_press') }}"
      end: "{{ states('sensor.biberon_bottle_empty_button_last_press') }}"
      amount: "{{ states('sensor.biberon_bottle_consumption') | int(0) }}"

mode: queued
```

---

## 🧪 Calibration

Load cell calibration is performed using ESPHome’s `calibrate_linear` filter.  
You can recalibrate using known weights, then update the YAML accordingly.

---

## 📁 Project Structure

```
enclosure.3mf
biberon.yaml
README.md
LICENSE
```

---

## ❤️ Why I built this

Feeding logs are incredibly useful for newborns, but typing them manually is tedious.  
This project removes all friction by logging feedings **automatically** and accurately.

---


## 🙌 Contributions Welcome

Feel free to open issues or PRs if you'd like to improve the design, YAML, or integration.

---
