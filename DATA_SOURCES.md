# NASA Data Sources for VIGIL

---

## 1. NASA Astronaut Biology Data (OSDR)

Six months research with real astronauts, 650+ studies, 4,357 human spaceflight samples

### Quick Start

```bash
# Human spaceflight omics
curl "https://visualization.osdr.nasa.gov/biodata/api/v2/query/metadata/?study.characteristics.organism=homo%20sapiens&study.factor%20value.spaceflight"

# Download OSD-53
curl -sLO "https://osdr.nasa.gov/geode-py/ws/studies/OSD-53/download?source=datamanager&file=GLDS-53_microarray_E-GEOD-47126.additional.zip"
```

### 4 Key Studies (All Live)

| Accession           | Data                               | Why Use                                                                          | Samples         | Status  |
| ------------------- | ---------------------------------- | -------------------------------------------------------------------------------- | --------------- | ------- |
| **OSD-53**          | Blood, microarray, 6 astronauts    | Pre and post-flight with p-values. CD36 gene shows +1.64 log2 ratio (benchmark). | 12              | ✅ LIVE |
| **OSD-530**         | 48 in-flight samples, 6 astronauts | Also identifies CD36 as top spaceflight marker. Links two independent datasets.  | 48              | ✅ LIVE |
| **OSD-174**         | Hair follicles (ISS crew)          | Non-invasive tissue. Supports wearable surrogate concept.                        | 20              | ✅ LIVE |
| **OSD-569/570/571** | Inspiration4, 4 civilians          | Most data types, blood RNA, proteins, metabolites, microbiome.                   | 4 crew pre-post | ✅ LIVE |

### OSD-53 Details (CD36 Discovery Dataset)

Confirmed via API call on 2026-10-10.

- 6 astronauts (A-F), 12 total samples
- Each astronaut has 1 Baseline Control (Launch -10d) and 1 Space Flight (Landing +0d)
- Sample type is Whole Blood, microarray
- Organism is Homo sapiens

Download fold-change table

```bash
curl -sLO "https://osdr.nasa.gov/geode-py/ws/studies/OSD-53/download?source=datamanager&file=GLDS-53_microarray_E-GEOD-47126.additional.zip"
unzip -o GLDS-53_microarray_E-GEOD-47126.additional.zip
# Look for GSE47126_Fold_change_data.txt
# Find row for K00650 (CD36), log2_ratio = +1.64, p-adj = 0.022
```

Sample breakdown (4,357 human spaceflight samples)

- Pre-flight 331, In-flight 130, Post-flight 430, Ground control 210

Download key files before Nov 14 to reduce network risk. OSD-53 ZIP is 43 KB.

---

## 2. Real-Time Radiation and Weather (NOAA SWPC)

Live data every 1 minute, used for dose calculations. All live verified 2026-10-10.

### Endpoints (Tested Working)

```bash
# Radiation (Solar Proton Events) returns multi-energy flux
https://services.swpc.noaa.gov/json/goes/primary/integral-protons-1-day.json    # ✅ Live, 236 KB
https://services.swpc.noaa.gov/json/goes/primary/xrays-1-day.json              # ✅ Live

# Geomagnetic storms (needed for dose triggers)
https://services.swpc.noaa.gov/json/rtsw_mag_1m.json                            # ⚠️ Sometimes 404, add retry
https://services.swpc.noaa.gov/json/planetary_k_index_1m.json                   # ✅ Live, returns Kp index

# Crew location (for dose calculations)
http://api.open-notify.org/iss-now.json          # ✅ Live ISS lat/lon (currently -5.41 degrees, 62.03 degrees)
https://api.open-notify.org/astros.json          # ✅ Live crew roster
```

### Live Data Example (2026-10-10 12:40 UTC)

```json
{
  "time_tag": "2026-10-10T12:40:00Z",
  "kp_index": 2,
  "iss_position": {
    "latitude": "-5.4131",
    "longitude": "62.0269"
  },
  "proton_flux_1mev": 3.87,
  "proton_flux_10mev": 0.51,
  "proton_flux_100mev": 0.25
}
```

### Radiation Dose Limits (hardcode in your code)

| Threshold    | Value              | Use                   |
| ------------ | ------------------ | --------------------- |
| ISS typical  | 0.5 mSv per day    | Baseline dose display |
| Career limit | 600 mSv (lifetime) | Main PEL reference    |
| Mars mission | 300-450 mGy        | Context               |
| Lens damage  | 4,000 mGy (career) | Eye health alert      |

P1 Model (Radiation Dose) converts GOES flux to absorbed dose (mGy), then to effective dose (mSv), then to percent PEL.

---

## 3. NASA Research Papers (600k plus public-domain documents)

Use these to cite your alerts. All live verified 2026-10-10.

```bash
# Search SANS literature
curl "https://ntrs.nasa.gov/api/citations/search?q=SANS&page.size=5"

# Get paper metadata
curl "https://ntrs.nasa.gov/api/citations/20180000936"

# Download PDF (no auth needed)
curl "https://ntrs.nasa.gov/api/citations/20180000936/downloads/20180000936.pdf"

# Get grep-able full text
curl "https://ntrs.nasa.gov/api/citations/20180000936/downloads/20180000936.pdf.txt"
```

### Key Papers (Verified Live)

| Risk        | Paper ID    | Title                                                                    | Status  |
| ----------- | ----------- | ------------------------------------------------------------------------ | ------- |
| SANS (eye)  | 20180000936 | Evidence Report Risk of SANS                                             | ✅ LIVE |
| SANS Update | 20220014606 | Spaceflight Associated Neuro-ocular Syndrome (SANS) 2023 Clinical Update | ✅ LIVE |
| Muscle      | 20220019338 | Evidence Report Reduced Muscle Mass, Strength and Endurance              | ✅ LIVE |
| Bone        | 20240005190 | Risk of Bone Fracture from Spaceflight-induced Bone Changes              | ✅ LIVE |
| Immune      | 20150008177 | Risk of Crew Adverse Health Event Due to Altered Immune Response         | ✅ LIVE |

Example citation for countermeasure card

> Limit screen luminance to 50 cd per m² before sleep (20:00-06:00 local). Source NASA HRP Risk of SANS (NTRS 20180000936), OCHMO-TB-041 Sleep Technical Brief

---

## 4. NASA Safety and Physiology Limits

### CO₂ Thresholds (hardcode this table)

Source is NASA-STD-3001 Vol. 2 Rev. F (approved 2026-07-14) and OCHMO-TB-004 Rev B.

| Level                          | Effect                        | Severity | Use in Alert |
| ------------------------------ | ----------------------------- | -------- | ------------ |
| Less than 2.5 mmHg (7-day avg) | Headache risk less than 1%    | Safe     | Green        |
| 2.8-4.5 mmHg                   | Headache onset                | Yellow   | Caution      |
| Greater than 4.5 mmHg          | Fatigue, nausea, poor sleep   | Red      | Alert        |
| Greater than 5.0 mmHg          | 5th cranial nerve dysesthesia | Critical | Alarm        |

Conversion formula is mmHg equals ppm times 0.00076, so 2,600 ppm is approximately 2 mmHg.

ISS design is 267 Pa, which is approximately 2 mmHg or approximately 2,600 ppm.

Real ISS headache incidence comes from Experiment 2-31 (N=49). Result is 19 out of 49 equals 38.7% had headaches.

### Cabin Requirements

- O₂ is 145-155 mmHg
- Temp is 20-25 degrees C, 30-60 percent RH (performance zone)
- Water is minimum 2.5 litres per crew per day
- Food is average 3,035 kilocalories per crew per day

### Physiology on ISS (Model Inputs)

- Fitness loss is 1 percent per week. You need at least 80 percent of pre-flight VO₂ max.
- Bone loss is 1-1.5 percent per month. Hip and spine must stay at least 95 percent, femoral neck at least 90 percent.
- Sleep is only 6-6.5 hours (versus 7-9 on Earth), which equals 18 minutes sleep debt per day.
- Immune response shows EBV at 96 percent, VZV at 65 percent, CMV greater than 50 percent reactivation on long missions.
- SAS affects up to 73 percent of crew in days 2-3.

P2 Model (CO₂) looks up mmHg in table and returns headache and cognitive risk.

P5 Model (Sleep) starts with zero debt on Day 1 and accumulates plus 18 minutes per day until countermeasure is applied.

---

## 5. Complete Python Ingest Template

```python
import httpx, time, json

OSDR = "https://visualization.osdr.nasa.gov/biodata/api/v2/query/metadata/"
SWPC = "https://services.swpc.noaa.gov/json/"
OPEN_NOTIFY = "http://api.open-notify.org/"

def resilient_fetch(url, retries=3, timeout=20):
    """Retry with exponential backoff. SWPC files rotate sometimes."""
    for attempt in range(retries):
        try:
            r = httpx.get(url, timeout=timeout)
            if r.status_code == 200:
                return r
        except Exception as e:
            print(f"Attempt {attempt+1} failed {e}")
        if attempt < retries - 1:
            time.sleep(1.5 * (attempt + 1))
    raise RuntimeError(f"Failed after {retries} retries")

# Example: Fetch live radiation and crew location
def get_radiation_context():
    """Get current space weather for dose calculation"""
    protons = resilient_fetch(SWPC + "goes/primary/integral-protons-1-day.json").json()
    kp = resilient_fetch(SWPC + "planetary_k_index_1m.json").json()[-1]
    iss = resilient_fetch(OPEN_NOTIFY + "iss-now.json").json()

    return {
        "proton_flux_1mev": protons[-1]["flux"],
        "kp_index": kp["kp_index"],
        "iss_latitude": float(iss["iss_position"]["latitude"]),
        "iss_longitude": float(iss["iss_position"]["longitude"]),
        "timestamp": iss["timestamp"]
    }

# Example: Fetch OSD-53 metadata
def get_osd53_metadata():
    """Get 6 astronauts from CD36 discovery dataset"""
    url = OSDR + "?id.accession=OSD-53&study.factor%20value&study.characteristics"
    r = resilient_fetch(url)
    lines = r.text.strip().split('\n')
    return lines
```

Packages to use are httpx for HTTP client with JSON parsing, and fastapi to serve endpoints to React frontend.

Key conversions

- CO₂ formula is mmHg equals ppm times 0.00076
- Dose formula is absorbed_dose_mgy equals proton_flux times exposure_time times shielding_factor
- Career percent formula is (effective_dose_mSv divided by 600.0) times 100

---

## 6. What's NOT Available (Don't build on these)

- ECLSS telemetry (CO₂, O₂, humidity streams) does not exist publicly
- Per-astronaut medical records are restricted by design
- DONKI and DOdoset hosts have DNS failures or have moved
- Flight crew physiological ZIP files return 404
- ISS cabin pressure and ECLSS data have no public API

Why this gap exists is privacy and operational security. Only OSDR omics and published research are available.

Why VIGIL fills it is by bridging wearable sensor signatures to molecular baselines from real astronauts.

---

## Key Documents to Download (Before Nov 14)

- [NASA-STD-3001 Vol. 1 (877 KB)](https://www.nasa.gov/wp-content/uploads/2020/10/2022-01-05_nasa-std-3001_vol.1_rev._b_final_draft_with_signature_010522.pdf) - All crew health thresholds
- [OCHMO Technical Briefs](https://www.nasa.gov/ochmo/hsa-standards/ochmo-technical-briefs/) - Especially TB-004 (CO₂ and headache risk) and TB-041 (sleep debt accumulation)
- [OSD-53 fold-change data](https://osdr.nasa.gov/geode-py/ws/studies/OSD-53/download?source=datamanager&file=GLDS-53_microarray_E-GEOD-47126.additional.zip) (43 KB, no network risk)
