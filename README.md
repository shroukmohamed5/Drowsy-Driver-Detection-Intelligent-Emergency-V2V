<div align="center">

<img width="2080" height="572" alt="banner" src="https://github.com/user-attachments/assets/2f75e8ce-334a-4840-8746-c63e9fa0b5a3" />


<br/>

![Python](https://img.shields.io/badge/Python-3.10-3776AB?style=for-the-badge&logo=python&logoColor=white)
![MediaPipe](https://img.shields.io/badge/MediaPipe-Tasks_API-0F9D58?style=for-the-badge)
![CARLA](https://img.shields.io/badge/CARLA-0.9.15-F59E0B?style=for-the-badge)
![SUMO](https://img.shields.io/badge/SUMO-1.27.1-2563EB?style=for-the-badge)
![Streamlit](https://img.shields.io/badge/Streamlit-Dashboard-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![Tests](https://img.shields.io/badge/automated_tests-220%2B-22C55E?style=for-the-badge)
![Source](https://img.shields.io/badge/source_code-included-8B5CF6?style=for-the-badge)

<img width="1244" height="887" alt="image" src="https://github.com/user-attachments/assets/7f6fb384-47d2-41b4-bc44-6b62a241af6e" />





### A complete, working research platform that turns a **webcam** into a **drowsiness-aware V2V safety system** — from face landmarks all the way to a car pulling over in a 3D simulator.

**[▶ Watch the demo](https://youtu.be/AtQNb1WovZo)** &nbsp;·&nbsp; **[💳 Commercial license available ](#-pricing--licenses)** &nbsp;·&nbsp; **[❓ FAQ](#-faq)**

</div>

---

## ▶ See it in action

[![Watch the live demo on YouTube](https://img.youtube.com/vi/AtQNb1WovZo/maxresdefault.jpg)](https://youtu.be/AtQNb1WovZo)
<img width="1920" height="1022" alt="image" src="https://github.com/user-attachments/assets/0281ff2c-db97-44c5-b7b5-270a4f174993" />


> Real webcam on the left, CARLA 3D world on the right. Close your eyes for ~3 seconds and watch the whole chain fire: **alarm → emergency → V2V broadcast → surrounding cars react → the vehicle brakes, changes lane and pulls over onto the shoulder.** Zero collisions, no teleporting — real vehicle control.

<!--
SCREENSHOTS (optional, recommended): add 3-4 frames from the video to assets/screenshots/ and uncomment.

| Webcam HUD | CARLA pull-over | Streamlit dashboard |
|:--:|:--:|:--:|
| <img src="assets/screenshots/hud.png"/> | <img src="assets/screenshots/carla_pullover.png"/> | <img src="assets/screenshots/dashboard.png"/> |
-->

---

## 📌 Table of contents

- [Why this project](#-why-this-project) · [Who it is for](#-who-it-is-for) · [Key features](#-key-features)
- [How it works](#-how-it-works) · [Architecture](#-architecture) · [Driver state machine](#-driver-state-machine) · [V2V protocol](#-v2v-protocol) · [CARLA scenario](#-the-carla-highway-scenario)
- [Dashboard & logging](#-dashboard--logging) · [Evaluation](#-transparent-evaluation) · [Engineering quality](#-engineering-quality)
- [What you get](#-what-you-get) · [Pricing & licenses](#-pricing--licenses) · [How to buy](#-how-to-buy) · [Requirements](#-requirements) · [FAQ](#-faq) · [Safety & scope](#-safety--scope) · [Cite](#-citing-this-work) · [Contact](#-contact)

---

## 💡 Why this project

Drowsy driving is a leading cause of serious road accidents, and most "driver monitoring" demos stop at *"beep when eyes close."* This project goes the whole distance and asks the harder question:

> **If a vehicle detects that its driver is falling asleep, can it tell the cars around it — and does that actually change what they do?**

You get a complete, runnable answer: **perception → explainable decision-making → V2V messaging → coordinated response → safe pull-over**, wired together, tested, logged, and measurable. It is built as a Master's thesis project, so every component is configurable, every result is reproducible, and every limitation is written down.

## 🎯 Who it is for

| You are… | What this gives you |
|---|---|
| **A student** (BSc / MSc / capstone) | A finished, defensible, well-documented project with a full thesis-style write-up, experiments and a live demo to learn from or build upon. |
| **A researcher / lab** | A modular testbed: swap detectors, risk weights, V2V policies or simulators, then measure with the built-in ablation and V2V experiment harness. |
| **An automotive / ADAS / IoT startup** | A fast prototype of a driver-monitoring → V2X pipeline, with a clean simulator abstraction you can extend. |
| **An educator** | A teaching-ready example of CV, state machines, messaging protocols and simulation, with Russian/Arabic quick-start guides. |

## ✨ Key features

<table>
<tr>
<td width="50%" valign="top">

### 👁️ Real-time driver monitoring
- Webcam → **MediaPipe Face Landmarker** (Tasks API, actively maintained)
- **EAR** (eye closure), **MAR** (yawning), **3-axis head pose** (nodding, drooping)
- **Temporal aggregation** — a single bad frame can *never* trigger an alert
- Handles lost tracking safely (`MONITORING_UNCERTAIN` never implies "drowsy")

### 🧠 Explainable decision engine
- Weighted multi-signal **risk fusion** → LOW / MEDIUM / HIGH / CRITICAL
- **8-state driver state machine** with hysteresis both ways and a mandatory RECOVERY path
- Optional **microsleep rule** for live demos (eyes closed ≥ N s → emergency)
- Every threshold lives in one file: `app/config.py`

</td>
<td width="50%" valign="top">

### 📡 Realistic V2V messaging
- Typed JSON **EMERGENCY / CANCEL** messages with UUIDs
- Validation: **range, TTL, per-receiver duplicate prevention, malformed-message rejection**
- Receiver-side **relevance filtering** (behind / ahead / oncoming)
- Collision-risk monitor (TTC + gap) escalates message priority

### 🚗 Coordinated vehicle response
- **Role-aware** reactions: following, adjacent, path-blocking vehicles
- Staged behaviour: warn → reduce speed → larger headway
- **Emergency pull-over**: decelerate → find a safe gap → lane change → shoulder → stop
- Real steering / throttle / brake — **no teleporting**

</td>
</tr>
<tr>
<td colspan="2" valign="top">

### 🧪 Built to be measured, not just shown
Ablation study harness · 8-condition V2V experiment suite · structured CSV/JSONL event logs · Streamlit live dashboard · **three interchangeable simulators** (CARLA, SUMO, Mock) behind one interface · **220+ automated tests**.

</td>
</tr>
</table>

---

## 🔬 How it works

One closed-eyes event, end to end:

<img width="1562" height="443" alt="image" src="https://github.com/user-attachments/assets/eee4a7a2-0d94-4457-a43b-3e24957cc526" />


1. **Perceive** — landmarks → EAR / MAR / head pose, per frame.
2. **Aggregate** — durations, persistence and nodding patterns over time (the *only* place allowed to say "sustained").
3. **Assess** — fuse the signals into a risk score and level.
4. **Decide** — the state machine escalates with hysteresis; EMERGENCY fires **once per episode**.
5. **Alert** — on-screen HUD + audible alarm (slow beeps → fast beeps → siren).
6. **Broadcast** — one validated V2V message to every vehicle in range.
7. **Respond** — each receiver decides if it is relevant and reacts according to its role.
8. **Pull over** — the ego vehicle decelerates, changes lane only when it is safe, and stops on the shoulder.

---

## 🏗 Architecture

<img width="1690" height="1199" alt="image" src="https://github.com/user-attachments/assets/374469a0-8bae-4fc8-8cef-a26fe4fdc570" />


**The simulator boundary is real, not just a diagram.** Everything above the `SimulationBackend` interface — all detection, risk, decision and V2V logic — never imports `traci` or `carla`. This is enforced mechanically by tests, and the *same* behavioural test-suite runs against both the Mock and the real SUMO backend. Plugging in a different simulator means writing one adapter file.

| Backend | Purpose |
|---|---|
| 🟣 **CARLA 0.9.15** | Photorealistic 3D live demo with active vehicle control |
| 🔵 **SUMO 1.27.1 + TraCI** | Repeatable traffic-flow batch experiments |
| ⚪ **Mock** | Instant, dependency-free backend for fast tests and CI |

---

## 🚦 Driver state machine

<img width="3119" height="732" alt="image" src="https://github.com/user-attachments/assets/dccd3640-ca51-41e4-9aa5-477936c20130" />


- **Escalation is earned, not instant** — each rung needs the higher risk to persist.
- **Recovery is earned too** — CRITICAL/EMERGENCY can never jump straight back to AWAKE; they pass through `RECOVERY`.
- **False-alarm resistant** — brief blips, normal blinks and a head tilt with open eyes do not escalate (all covered by tests).

---

## 📡 V2V protocol

<img width="3810" height="722" alt="image" src="https://github.com/user-attachments/assets/18edca66-92ee-43a6-93a2-5cbc8e08c7ad" />


```json
{
  "message_id": "8f2e…-uuid",
  "vehicle_id": 12,
  "message_type": "EMERGENCY",
  "reason": "DRIVER_DROWSINESS",
  "priority": "HIGH",
  "timestamp": 1234567.5,
  "position": { "x": 500.0, "y": 0.0, "z": 0.0 },
  "speed": 68.0,
  "heading": 0.0
}
```

A `CANCEL` message (same shape) is sent when the emergency clears, so receivers can reset cleanly.

---

## 🛣 The CARLA highway scenario

<img width="1537" height="586" alt="image" src="https://github.com/user-attachments/assets/ef9648eb-e17d-4d16-a0e5-f623ada9d811" />


A reproducible 3-lane convoy with 9 NPC vehicles (fixed 20 Hz synchronous mode, seeded Traffic Manager). Each receiver picks its response from its **role** relative to the drowsy vehicle:

| Role | Who | Response |
|---|---|---|
| **Following** | Behind, same lane | Reduce speed, increase time headway |
| **Adjacent** | Alongside / behind in another lane | Same staged response, keeps clear |
| **Path-blocking** | Ahead within 90 m, in ego lane or to its right | Ease to 80 % speed; move left **only if** the feasibility check passes, otherwise just slow down |
| **Unaffected** | Far ahead / oncoming | Informational only |

**Emergency pull-over state machine:**
`NORMAL_DRIVING → EMERGENCY → DECELERATING → SEARCHING_SAFE_PULL_OVER → CHANGING_LANE → PULLING_OVER → STOPPED`

It aborts unsafe lane changes (and logs why), falls back to stopping in-lane with hazards if no gap appears, and never gets cancelled mid-manoeuvre by a driver who suddenly wakes up.

---

## 📊 Dashboard & logging

- **Streamlit dashboard** with live *Driver*, *Vehicle*, *Collision*, *V2V* and *Events* panels; switch between SUMO / Mock / CARLA and tune the V2V range from the sidebar.
- **Webcam HUD** — driver state, risk, eye/yawn/head-pose readouts, tracking confidence, vehicle speed/lane, V2V counters and a full-width EMERGENCY banner; composed side by side with the CARLA view.
- **Structured event logs** (`events.csv` + `events.jsonl`) for every run: state changes, alarms, V2V sent/received, vehicle responses, lane changes, pull-over stages, collisions and more — ready for analysis or your thesis tables.

---

## 🧪 Transparent evaluation

Most projects show you only the flattering numbers. This one ships the **experiment harness and the honest results**, so you can reproduce them or re-run them on your own data.

### Detection ablation (30 synthetic sessions · 83,206 frames · 159 drowsiness episodes)

<img width="1936" height="622" alt="image" src="https://github.com/user-attachments/assets/6563adec-3c3e-4b07-b24b-5f05551abf3f" />


| Configuration | Accuracy | F1 | False-positive rate | Mean latency |
|---|:--:|:--:|:--:|:--:|
| Eye-only baseline | 91.8 % | 0.699 | 1.5 % | 2.96 s |
| **Full multi-signal system** | 87.9 % | 0.612 | 6.4 % | 3.16 s |
| No head pose | 86.9 % | 0.582 | 7.0 % | 2.96 s |
| No head nodding | 85.6 % | 0.572 | 9.0 % | 3.16 s |
| No temporal smoothing | 87.2 % | 0.697 | 13.3 % | **0.88 s** |

**What this shows**
- ⏱ **Temporal smoothing is a real trade-off knob**: removing it makes detection ~3.6× faster (3.16 s → 0.88 s) but roughly doubles the false-positive rate (6.4 % → 13.3 %).
- 🧩 Head-pose and nodding evidence measurably reduce false positives *within the full system*.
- 🔍 On this synthetic benchmark the simple eye-only baseline is hard to beat — the ablation harness is exactly the tool you need to find out whether fusion pays off on **your** data.

> **Important:** all detection numbers come from synthetic, generator-labelled sessions. They are **not** real-world accuracy claims. See [FAQ](#-faq).

### V2V + response experiments (real SUMO · 8 conditions)

<img width="1937" height="649" alt="image" src="https://github.com/user-attachments/assets/fffb13a9-5da4-4917-b3a4-6d24f9edc356" />


- ✅ **0 collisions in all 8 conditions** (baseline, short/long V2V range, close/far following, faster/slower follower, dense traffic).
- 📡 Delivery is correctly gated by **communication range and receiver geometry**, not blindly broadcast.
- 🚗 The following vehicle's speed measurably dropped **0.8 s after message delivery** in the conditions where it stayed in range.

Raw data: `data/results/*.csv|json` · Full methodology and discussion: `docs/thesis.md`.

---

## 🛡 Engineering quality

| | |
|---|---|
| **~9,000 lines** of typed, documented Python | Modular packages: `app/`, `simulation/`, `dashboard/`, `experiments/`, `tests/` |
| **220+ automated tests** | Unit, scenario, V2V, relevance, emergency, abstraction and full-session tests — including **real-SUMO integration tests** and a faithful fake-CARLA adapter test (live-CARLA tests run automatically when a server is present) |
| **One-file configuration** | Every threshold, weight and timing in `app/config.py` — no magic numbers hidden in logic |
| **Reproducible** | Seeded scenarios, deterministic synchronous simulation, logged runs |
| **Graceful fallbacks** | No SUMO? Mock backend. No CARLA? Labelled kinematic preview. No camera? Scripted demo scenarios |
| **Environment check script** | `python scripts/check_environment.py` diagnoses your setup in seconds |

### Demo modes

```bash
python live_demo.py                      # real webcam + CARLA 3D, full emergency chain
python live_demo.py --controlled-demo    # no webcam: scripted, clearly labelled signals
python live_demo.py --backend mock       # no CARLA: labelled kinematic preview
python main.py --demo                    # CLI end-to-end demo (SUMO / Mock)
streamlit run dashboard/app.py           # live dashboard
pytest -q                                # full test suite
```

---

## 📦 What you get

```
project/
├── app/            Detection · risk · state machine · alarm · V2V · vehicle response · pull-over · logging
├── simulation/     CARLA / SUMO / Mock adapters · road frame · lane logic · vehicle control · scenarios
├── dashboard/      Streamlit live dashboard
├── experiments/    Ablation + V2V experiment harness
├── tests/          220+ automated tests
├── docs/           Thesis write-up · CARLA setup & integration · run-from-scratch (AR) · demo guide (RU/AR)
├── data/results/   Experiment results (CSV + JSON)
├── scripts/        Environment check · landmark check · CARLA map probe
├── main.py         End-to-end CLI demo
├── live_demo.py    One-command CARLA + webcam demonstration
└── requirements*.txt
```

- ✅ Complete Python source code
- ✅ Full thesis-style research write-up (`docs/thesis.md`): problem, research questions, methodology, baselines, ablation, results, limitations, future work
- ✅ Step-by-step setup guides (English · Arabic · Russian)
- ✅ Experiment data and the scripts that produced it
- ✅ Test suite and environment checker

---

## 💳 Pricing & licenses

<!-- EDIT: fill in your prices and the checkout link before publishing -->

| | 🎓 **Student** | 🔬 **Researcher / Academic Lab** | 🏢 **Professional / Commercial** |
|---|:--:|:--:|:--:|
| **Price** | **$79** | **$299** | **From $1,500** |
| Full source code | ✅ | ✅ | ✅ |
| Docs, thesis write-up & setup guides | ✅ | ✅ | ✅ |
| Experiment data & scripts | ✅ | ✅ | ✅ |
| Use for coursework, theses & publications | ✅ | ✅ | ✅ |
| Use within a research group | — | ✅ | ✅ |
| Commercial / internal product prototyping | — | — | ✅ |
| Setup support | Email | Email | Priority email |
| **Eligibility** | Valid student ID or university email | University / institute email or ORCID | Anyone |
> Prices are indicative starting prices. Commercial licensing terms depend on the intended use, scope, and support requirements. Exclusive intellectual property transfer is negotiated separately.

> 🎓 **Student & researcher pricing:** send a student ID, a university email address, or your ORCID profile when you order and the discounted price is applied.
>
> 📄 Source is licensed to the purchaser (and, for lab / commercial tiers, the stated team). Redistribution, resale or publishing the source code is not permitted. If you publish research that uses it, please [cite it](#-citing-this-work).

## 🛒 How to buy

1. **Choose your tier** and order via **[Contact me on Telegram: @shro5k]**, or email **[srukmohamed@gmail.com]** with the subject `Drowsy Driver V2V — <your tier>`.
2. **Student / researcher?** Include your verification (student ID, university email or ORCID).
3. **Pay** — you receive the source code package (ZIP) and a license note.
4. **Run** — follow `docs/run_from_scratch.md`; `python scripts/check_environment.py` tells you if anything is missing.

Not sure it fits your use case? **[Watch the demo](https://youtu.be/AtQNb1WovZo)** first, or message me with your requirements.

---

## ⚙ Requirements

| | Minimum | Recommended |
|---|---|---|
| **OS** | Windows 10 / 11 (primary, tested) | Windows 10 / 11 |
| **Python** | 3.10 (matches the CARLA 0.9.15 Windows wheel) | 3.10 |
| **Webcam** | Any USB / built-in camera | Front-facing, even lighting |
| **GPU** | Not needed for detection, SUMO or Mock | NVIDIA 4060, ≥ 6 GB VRAM for the CARLA 3D demo (low-quality mode works on laptops) |
| **Disk** | ~2 GB (without CARLA) | ~20 GB (with CARLA server) |

Core stack: OpenCV · MediaPipe · NumPy · Streamlit · pandas · pytest · SUMO + TraCI + sumolib (installed via pip) · CARLA (optional, for the 3D demo).

> The audible alarm uses the Windows standard library (`winsound`); on other platforms the alarm is logged but silent. The detection, V2V and simulation layers are platform-independent Python.

---

## ❓ FAQ

<details>
<summary><b>Does it work with a normal laptop webcam?</b></summary>

Yes. Detection runs on a standard webcam with MediaPipe — no special hardware, no GPU required for the monitoring pipeline itself. Only the optional CARLA 3D view benefits from a dedicated GPU.
</details>

<details>
<summary><b>How accurate is the drowsiness detection?</b></summary>

The quantitative results in this README come from <b>synthetic, labelled sessions</b>, because collecting and labelling real drivers' footage was out of scope for the thesis. They demonstrate the pipeline's behaviour and trade-offs (e.g. the effect of temporal smoothing), <b>not</b> real-world accuracy. Detection quality on real faces also depends on lighting, camera position and the individual user, and thresholds should be calibrated for your setup. The experiment harness is included precisely so you can validate on your own data.
</details>

<details>
<summary><b>Can I use this in a real car?</b></summary>

No. This is a research and education prototype. It controls nothing real — all "vehicle control" happens inside a simulator — and it is not a certified automotive safety system. See <a href="#-safety--scope">Safety & scope</a>.
</details>

<details>
<summary><b>Do I need CARLA?</b></summary>

No. CARLA is for the visual 3D demo. SUMO (installed automatically via pip) powers the batch experiments, and the Mock backend lets everything run with zero simulator installed.
</details>

<details>
<summary><b>Can I swap in another simulator or my own detector?</b></summary>

Yes — that is the point of the architecture. Simulators plug in through one adapter implementing the `SimulationBackend` interface. The detector, risk weights and thresholds are all configurable and covered by tests.
</details>

<details>
<summary><b>Can I use it for my thesis or paper?</b></summary>

Yes, with any tier. Please <a href="#-citing-this-work">cite the project</a> in published work. The included `docs/thesis.md` shows one way to structure the research write-up and experiments.
</details>

<details>
<summary><b>What happens after I pay?</b></summary>

You receive the full source package and license note, plus the setup guides. If something does not run on your machine, email me with the output of <code>python scripts/check_environment.py</code> and I will help you get unblocked.
</details>

---

## 🛑 Safety & scope

> **This is a software simulation / research prototype only.** It does not control any real vehicle, steering, brakes, throttle, CAN bus or automotive hardware. The nearby-vehicle response behaviour is a research prototype, not certified automotive safety behaviour, and is not intended for deployment. V2V is an application-level simulation (no network delay / loss model yet). Detection accuracy has not been validated on real driver footage, and no real-world accuracy claim is made.

## 🗺 Roadmap

- Validate and recalibrate thresholds on real, labelled driver datasets
- V2V network delay / packet-loss model
- Multi-seed timing experiments and confidence intervals
- Curved-road support in the ego controller
- Per-user calibration of eye / mouth thresholds

## 📚 Citing this work

```bibtex
@mastersthesis{drowsy_v2v_2026,
  title  = {Intelligent Risk Assessment and V2V Communication for Emergency Response to Driver Drowsiness},
  author = {Gebriel Shrouk Mohamed},
  school = {Belgorod State Technological University named after V.G. Shukhov},
  year   = {2026},
  note   = {Software prototype, CARLA / SUMO simulation}
}
```

## 📬 Contact

**Gebriel Shrouk Mohamed** · GitHub: [@shroukmohamed5](https://github.com/shroukmohamed5) · Email: **[srukmohamed@gmail.com]** · **[Contact me on Telegram: @shro5k]** · Demo: [youtu.be/AtQNb1WovZo](https://youtu.be/AtQNb1WovZo)

<div align="center">

<sub>Built with OpenCV, MediaPipe, SUMO, CARLA and Streamlit. All trademarks belong to their respective owners.</sub>

</div>
