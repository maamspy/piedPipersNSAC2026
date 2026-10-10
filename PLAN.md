# VIGIL Hackathon Plan

NASA Space Apps 2026, Team Pied Pipers, Cumilla, Bangladesh

---

## The Challenge

Build health monitoring software that gathers data, evaluates it, and helps astronauts act.

The gap exists because astronauts are poor sensors of their own bodies. They under-report symptoms. No lab in flight. No blood draws on Mars. Conditions develop slowly and affect multiple systems at once.

VIGIL's idea is to learn each astronaut's personal baseline in 14 days. Then detect drift they don't report. Use real NASA omics data, radiation physics, and wearable sensors.

---

### The Scientific Claim

Two independent NASA datasets identify the same gene (CD36) as the top spaceflight marker.

- OSD-53 shows 6 astronauts, fold-change equals +1.64, p-adj equals 0.022. Live API verified on 2026-10-10.
- OSD-530 shows 48 in-flight samples and also names CD36 as "hallmark of space environment." Live.

This is traceable, citable, and real. AI dashboards cannot claim this.

### Data Sources

| Source              | Status  | Use                                    |
| ------------------- | ------- | -------------------------------------- |
| OSDR Biological API | ✅ Live | OSD-53, OSD-530, omics data            |
| SWPC Radiation API  | ✅ Live | Kp index, proton flux, real-time dose  |
| ISS Location API    | ✅ Live | Crew position for rigidity calculation |
| NTRS Research       | ✅ Live | SANS papers and NASA documents         |

### Why We're Different

| What            | Us                                                          | Others                                        |
| --------------- | ----------------------------------------------------------- | --------------------------------------------- |
| Data            | Real NASA omics (130 in-flight samples, OSD-53 and OSD-530) | Mock data or generic vitals                   |
| Baseline        | Personal adaptive (14-day on-orbit per astronaut)           | Earth normal ranges (invalid in microgravity) |
| Countermeasures | Cited from NASA-STD-3001, HRP, NTRS papers                  | Generic advice                                |
| Hardware        | Wristband and cabin sensor (2 ESP32s, real sensors)         | Software only                                 |
| Validity Score  | Real physics, real in-flight data, zero mock data           | 1-2 out of 5 (judges see through fake data)   |

---

## System Design (4 Layers)

### Layer 1: Personal Baselines

Earth normal ranges do not work in microgravity. Use the crew member's own Day 1-14 data as the reference. Detect percentage change from that baseline. Heart rate rises 15-20 percent, plasma volume falls 10-15 percent, and so forth.

### Layer 2: Omics-Guided Monitoring

NASA OSDR shows which genes and systems change in orbit

- Inflammation (cytokines, immune markers)
- Circadian (clock genes, melatonin)
- Blood-forming (erythropoiesis, iron metabolism)
- Mitochondrial (energy metabolism)

Use those to select which sensors to read.

### Layer 3: Wearable Surrogates

Cannot run a sequencer in flight. Continuous sensors measure

- Heart rate variability (HRV) becomes immune and stress proxy
- Skin temperature becomes circadian phase marker
- Sleep staging becomes circadian quality indicator
- Respiratory rate and sweat conductance become autonomic function proxies

These form a non-invasive molecular proxy.

### Layer 4: Cite Everything

Do not report a risk score (0.72). Report cause and actions.

> Your HRV is 18 percent below baseline. Sleep efficiency is 11 points below baseline. Cabin CO2 equals 4,100 ppm. This matches NASA circadian and hypercapnia pathway. Countermeasures are move ARED 2 hours before sleep, increase air exchange to 6 ACH, take 2000 IU D3, limit screen to 50 cd per m². (NASA-STD-3001 V2-6004, OCHMO-TB-041)

---

## Hardware (Separates You From Software-Only Teams)

### VIGIL Band (wrist, ESP32)

- MAX30102 PPG does HR, HRV, SpO2 (reflection photoplethysmography)
- ADS1115 with ECG pads does R-peak detection, HRV quality, respiratory rate (breath-induced impedance change)
- DS18B20 does skin and core temperature (via armpit thermocouple)
- MPU6050 IMU does accelerometer for activity level and sleep staging proxy

### VIGIL Node (cabin, ESP32)

- SCD41 or MH-Z19 measures CO2 (ppm), which drives P2 and P3 models
- BME280 measures temperature, humidity, barometric pressure
- INA219 measures power consumption (proxy for ECLSS LiOH canister consumption)

### Full Stack

```
VIGIL Band (ESP32)      VIGIL Node (ESP32)
    down arrow MQTT          down arrow MQTT
    back arrow up
             down arrow
      Edge Gateway (Raspberry Pi) offline-first caching
             down arrow
      NASA Data plus Physics Engine
      P1 Radiation dose
      P2 CO2 dose-response
      P3 CO2 to ICP to SANS
      P4 Cardiovascular
      P5 Circadian debt
      P6 Bone loading
                 down arrow
      Adaptive Baseline plus Anomaly
      14-day learning window
      IsolationForest plus rules
                 down arrow
        In-Flight UI     Ground Dashboard
        EIMO offline     trends, audit log
        Countermeasures  Multi-crew view
```

### Risk Mitigation

1. Parts
   - Local Dhaka supplier or LCSC ePacket
   - Budget is approximately 100 dollars per device (band and node)

2. Write simulator mode before hackathon.
   - Generate physiologically plausible sensor streams
   - If hardware fails, demo switches to simulator (no blank screens)

3. Bring 2 complete ESP32 sets.
   - Prototype on breadboard first
   - Solder final circuit last
   - Bring spares of every sensor

---

## Tech Stack

| Layer     | Choice                               | Why                                  | Fallback                   |
| --------- | ------------------------------------ | ------------------------------------ | -------------------------- |
| Firmware  | ESP32 and Arduino or MicroPython     | Fast, 8 dollars each, reliable       | Simulated data from Python |
| Messaging | MQTT and Mosquitto                   | Standard IoT, low setup              | Simulate MQTT publishes    |
| DB        | TimescaleDB (or SQLite if tight)     | Range queries on time-series         | In-memory dict if DB fails |
| Backend   | FastAPI                              | Fast to write, auto-docs, async      | Flask if stuck             |
| Frontend  | React, Vite, Recharts                | Dashboard and many charts            | Plain HTML and Chart.js    |
| Viz       | Three.js Sankey                      | P3 causal chain (CO2 to ICP to SANS) | SVG static diagram         |
| ML        | scikit-learn IsolationForest         | Explainable, no GPU, 1-liner         | Simple threshold rules     |
| Deploy    | Vercel (UI) and Railway or Fly (API) | Free tier for hackathon              | Localhost demo if needed   |

### Sponsor Credits (Claim by Nov 14)

- GoDaddy gives free `.space` domain (gives you credible URL for judging)
- Google provides cloud credits (use for API calls)
- Microsoft provides Azure credits (use for compute if needed)
- Meteomatics provides weather API (optional, evaluate it for context)

---

## Physics Models to Implement

| Model | Input                                                | Output                                                 | Demo                                | Priority  |
| ----- | ---------------------------------------------------- | ------------------------------------------------------ | ----------------------------------- | --------- |
| P1    | GOES protons and solar wind Bz and GCR               | Radiation dose (mGy) and percent of 600 mSv career PEL | Show live GOES flux and Kp and dose | MUST HAVE |
| P2    | Cabin CO2 (ppm), lookup table (NASA-STD)             | Headache and cognitive risk                            | Slider from 400 to 5000 ppm         | MUST HAVE |
| P3    | CO2 hypercapnia to vasodilation to ICP rise          | SANS risk proxy                                        | 3D Sankey: CO2 to ICP to SANS       | MUST HAVE |
| P4    | Cephalad fluid shift (1-2 litres)                    | BP and HR orthostatic ramp                             | Optional (nice to have)             | Low       |
| P5    | 16 sunrises per day and melanopic lux and sleep debt | Phase shift and 18 minutes per day accumulation        | Show debt accumulation              | Medium    |
| P6    | Exercise load (ARED)                                 | Bone BMD trajectory                                    | Optional (nice to have)             | Low       |

Demo focus is on P2 and P3. Add one slider for CO2, drag from 4,100 ppm to 2,200 ppm. Watch SANS risk curve fall. This single interaction wins more points than 5 static dashboards.

---

## What Wins (And Why Others Lose)

| Competitor             | Problem                                 | Your Edge                               |
| ---------------------- | --------------------------------------- | --------------------------------------- |
| AI health dashboard    | No real data. Judges see through it.    | OSD-53 and OSD-530 are traceable.       |
| Generic vitals tracker | Fitbit in a suit. No specificity.       | Personal baseline and omics-guided.     |
| LLM chatbot            | Black box. No measurement.              | Every claim cites peer review and NASA. |
| Mock sensors           | Judges inspect data sources. Game over. | Real SWPC live data on stage.           |

---

## Cut List (If Falling Behind)

Do these in order. Save P1, P2, P3 and demo always.

1. Remove login, accounts, profiles
2. Remove crew admin panel and mission control CRUD screens
3. Remove native mobile app (responsive web equals enough)
4. Remove ground dashboard (keep in-flight only)
5. Remove offline mode
6. Remove ML beyond IsolationForest (black-box equals validity penalty)
7. Remove complex 3D scenes (one causal-chain Sankey wins over habitat model)
8. Implement only P1, P2, P3 (skip P4, P5, P6 if tight)

Golden rule is do well equals better score than do many badly. A working P3 slider beats 10 broken models.

---

## Resources

- [NASA Challenge](https://www.spaceappschallenge.org/2026/challenges/create-health-monitoring-software-for-astronauts-on-space-missions/)
- [NASA HRP Risk Portfolio](https://www.nasa.gov/hrp/risks/)
- [NASA-STD-3001 Vol. 1 PDF](https://www.nasa.gov/wp-content/uploads/2020/10/2022-01-05_nasa-std-3001_vol.1_rev._b_final_draft_with_signature_010522.pdf)
- [OCHMO Technical Briefs](https://www.nasa.gov/ochmo/hsa-standards/ochmo-technical-briefs/)
- [OSDR API (live)](https://visualization.osdr.nasa.gov/biodata/api/)
- [NOAA SWPC (live)](https://services.swpc.noaa.gov/json/)
- [NTRS (600k plus papers, live)](https://ntrs.nasa.gov/)
