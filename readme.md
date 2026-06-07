
![logo](docs/images/atomic.png)

# Siren Direction Detection — PSoC Edge E84 AI Kit

**Infineon EESTech Challenge Hackathon 2026**

| | |
|---|---|
| **Team** | ATomic Trio |
| **Members** | Darius Lorenz, Fabian Lorenz, Laurence Sajuk |
| **Board** | PSoC Edge E84 AI Evaluation Kit (`KIT_PSE84_AI`) |
| **Live Demo (UI)** | <https://wuhuisland.eu> |
| **Repository** | <https://github.com/dari089/hackatom> |

---

## 1. Project Overview

**Objective:** Utilize the two on-board microphones of the PSoC Edge AI Kit to detect **the direction** of an approaching siren / emergency vehicle sound.

- **8 Directions:** North, Northeast, East, Southeast, South, Southwest, West, Northwest  
- **Additional Classes:** `noise`
- **Inference:** On-device on the **Arm Cortex-M55** (Edge AI, no cloud inference during runtime)  
- **Visualization:** Web-based Radar (`index.html`) hosted on [wuhuisland.eu](https://wuhuisland.eu)

---

## 2. Architecture

Siren → 2× Microphone (PSoC) → `audio.c` → ML Inference (CM55) → UART → Browser (`index.html` via Web Serial)

**Note regarding the website:** The visualization is hosted as a **single `index.html`** file on wuhuisland.eu. For the live demo, the board connects to the laptop via **USB**; the web page reads the incoming serial data using the **Web Serial API** (supported by Chrome/Edge).

---

## 3. Our Approach (Hackathon)

1. **Setup:** Installed ModusToolbox + DEEPCRAFT Studio; used the Infineon Deploy Audio example as our foundation.  
2. **Proof of Concept:** Verified the entire workflow (data collection → training → flashing) using a minimal 4-direction setup with a small dataset.  
3. **Final Model:** Expanded to 8 directions across 26 recording sessions (~10 seconds per recording, with 3 measurements per direction under the `data/` directory).
4. **Deployment:** Replaced `model.c` and `model.h` inside `proj_cm55/model/`; adapted `audio.c` for stereo processing.  
5. **UI:** Developed the radar visualization in parallel and published it on wuhuisland.eu.  

---

## Setup & Data Collection

The board was fixed centrally onto an 8-direction schema (N, NE, E, ...). Multiple ~10-second sessions of siren sounds were recorded for each direction using the DEEPCRAFT Live Data Collection tool.

![PSoC Edge E84 AI Kit — Setup](docs/images/boardsetup.png)

*Team ATomic — Measurement setup for direction detection (N / E / S / W marked)*

---

## 4. Model & Training (DEEPCRAFT Studio)

| Parameter | Value |
|-----------|------|
| Preprocessor | Contextual Window **1 s**, Shape `[16000, 2]` (Stereo) |
| Architecture | Conv1D **Small** |
| Quantization | Enabled (required for code generation) |
| Classes | 9 (8 directions + noise) |
| Export | `model.c`, `model.h` tailored for PSoC Edge |

![Deepcraft Training finished](docs/images/deepcraft.png)

**Base Reference Example (Infineon):** <https://github.com/Infineon/mtb-example-psoc-edge-ml-deepcraft-deploy-audio>

**Deployment Guide:** <https://developer.imagimob.com/deepcraft-studio/deployment/deploy-models-supported-boards/deploy-model-PSOC-6-PSOC-Edge>

---

## 5. Firmware Modifications

### Files in this Repository

| File | Description |
|-------|--------------|
| `firmware/audio.c` | Handles Stereo PDM (2 channels), separate ISR, mic gain, and UART transmission. |
| `firmware/model/model.c` | Generated ML model source code (DEEPCRAFT). |
| `firmware/model/model.h` | Model metadata including labels, buffer sizes, and detection thresholds. |

### Integration into the Infineon Example Project

1. **ModusToolbox:** Create or import the **PSOC Edge Machine Learning DEEPCRAFT Deploy Audio** project for the `KIT_PSE84_AI`.  
2. **Replace Files:**
   - Replace `proj_cm55/audio.c` with our `firmware/audio.c`
   - Replace `proj_cm55/model/model.c` and `model.h` with our newly generated model files.  
3. **Build:** Make sure to build the **entire application project** (not just the isolated `proj_cm55`).  
4. **Flash:** Program the application onto the target board.  

---

## 6. Web Visualization

| | |
|---|---|
| **File** | `web/index.html` |
| **Hosting** | <https://wuhuisland.eu/> |
| **Functionality** | 8-direction radar display, score indicators, animated pointer direction |
| **Data Source** | Web Serial (Board → Laptop → Browser connection) |

![Radar-UI — 8-Direction Display](docs/images/radar-ui.png)

### Running the Demo

1. Connect the flashed board to your laptop using a USB cable.  
2. Open **Google Chrome or Microsoft Edge** (required for Web Serial API support).  
3. Navigate to <https://wuhuisland.eu>.  
4. Click the **"Connect to PSoC"** button and select the active KitProg3 COM port.  
5. Play this [Siren Sound Sample](https://github.com/dari089/hackatom/blob/main/data/sirene/freesound_community-lafiresiren-55183.mp3) and monitor the live radar feedback.  

**Serial Configuration:** 115200 Baud, 8N1

---

## 7. Evaluation & Results

| Direction | Detection Performance (Subjective) |
|----------|------------------------|
| South | Good |
| East / West | Good (achieved after fixing inverted logic labels in the UI) |
| North | Frequently misclassified as South |
| Diagonals (NE, SE, SW, NW) | Partially accurate |

### Known Limitations

- **North vs. South Confusion:** Because the two microphones are aligned **horizontally**, distinguishing between front (North) and back (South) sound origins is physically restricted; this was further compounded by a limited training dataset.
- **Dataset Size constraints:** The model lacks optimal generalization accuracy due to small data sizes, leading to occasional value jumping between adjacent direction fields.
- **Web Serial Compatibility:** The browser-based interface runs most reliably on desktop environments utilizing Chromium-based browsers.

---

## 8. Key Takeaways & Lessons Learned

- Integrating the official Infineon documentation and deployment examples **early in the process** prevents significant configuration delays.  
- **DEEPCRAFT Studio:** Always verify the **Validation Set** manually rather than leaving it unallocated (0:00).  
- Adjusted the Preprocessor window to **1 s** instead of 2 s to mitigate on-board RAM overflow crashes.  
- Starting with a **Proof of Concept targeting fewer classes** before scaling up to all 8 directions is a much safer development route.  
- Mitigating the North classification error requires gathering a larger volume of spatially distinct training data.
- Encountered intermittent tool slow-downs and software bugs during the hackathon, which demanded extra troubleshooting time.

---

## 9. License & Acknowledgments

- This project builds upon the Infineon ModusToolbox example code bases (see corresponding file headers in `audio.c`).  
- The machine learning model was trained using **DEEPCRAFT Studio** (provided by Imagimob / Infineon).  
- Organized under the **Infineon EESTech Challenge**.
- While we utilized various AI tools (such as Gemini/Cursor) to assist with environment configuration, rapid debugging, and drafting documentation, all core engineering choices, target data collection, training runs, and hardware flashing tasks were executed purely by our team.

---

## Quick Links

- [Live Radar UI](https://wuhuisland.eu/)  
- [Our Repository (Team ATomic Trio)](https://github.com/dari089/hackatom)
- [Official Hackathon Repository (Infineon)](https://github.com/infineon/hackathon)  
- [Infineon Deploy Audio Example](
