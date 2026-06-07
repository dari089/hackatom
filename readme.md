# Siren Direction Detection — PSoC Edge E84 AI Kit

**Infineon EESTech Challenge Hackathon 2026**

| | |
|---|---|
| **Team** | [Name 1], [Name 2], [Name 3] |
| **ATomic** |
| **Board** | PSoC Edge E84 AI Evaluation Kit (`KIT_PSE84_AI`) |
| **Live-Demo (UI)** | https://wuhuisland.eu/[pfad-zur-index.html] |
| **Repository** | https://github.com/[user]/[repo] |

---

## 1. Projektübersicht

Ziel: Mit zwei Mikrofonen auf dem PSoC Edge AI Kit erkennen, **aus welcher Richtung** ein Sirenen- / Einsatzfahrzeug-Sound kommt.

- **8 Richtungen:** North, Northeast, East, Southeast, South, Southwest, West, Northwest  
- **Zusätzliche Klassen:** `noise`, `unlabeled`  
- **Inferenz:** On-device auf **Arm Cortex-M55** (Edge AI, kein Cloud-Inference zur Laufzeit)  
- **Visualisierung:** Web-Radar (`index.html`) auf [wuhuisland.eu](https://wuhuisland.eu)

---

## 2. Architektur
