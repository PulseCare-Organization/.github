<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=timeGradient&height=200&section=header&text=PulseCare&fontSize=70&fontColor=ffffff&animation=fadeIn" width="100%" />

  <h1>PulseCare System Overview</h1>
  <p>
    <b>PulseCare is an IoT + Edge AI + Cloud healthcare platform for continuous heart-rate monitoring and intelligent wellness support.</b><br>
    It spans two wearable hardware tracks that share one MAX30102 PPG sensor, one FastAPI backend, a Flutter client, and a RAG medical assistant.
  </p>
</div>

<br/>

## 🎯 What PulseCare Solves

Cardiovascular issues need early detection and long-term trend monitoring. Commercial wearables often show a BPM even when the finger contact is poor. PulseCare is designed to:

<ul>
  <li><b>Collect</b> photoplethysmogram (PPG) signals from finger-worn MAX30102 hardware.</li>
  <li><b>Gate trust on device</b> so noisy or no-finger windows are not treated as real heartbeats.</li>
  <li><b>Process and store</b> trusted 512-sample windows in near real time.</li>
  <li><b>Provide</b> live FFT/BPM access for web and mobile applications.</li>
  <li><b>Assist</b> users with AI-driven educational Q&amp;A using retrieval-augmented generation (RAG).</li>
</ul>

PulseCare is a wellness safety assistant. It does not diagnose arrhythmia and is not a certified medical device.

## 🔄 Dual Hardware Tracks, One Cloud

Both tracks read the same MAX30102 optical PPG sensor and sync trusted windows to the same FastAPI backend. They differ in where signal-quality decisions run.

<div align="center">
  <table>
    <tr>
      <td align="center" width="50%">
        <b>⌚ Track A — ESP32 wearable</b><br/>
        <code>pulsecare-iot</code><br/><br/>
        ESP32-WROOM-32 + MAX30102 + ST7789 TFT<br/>
        FreeRTOS sensing, display, and Wi-Fi<br/>
        On-device template matching + 512-point FFT
      </td>
      <td align="center" width="50%">
        <b>🧠 Track B — Arduino Uno Q Edge AI</b><br/>
        <code>pulsecare-arduino</code><br/><br/>
        Arduino Uno Q (STM32 MCU + Qualcomm Linux) + MAX30102<br/>
        MCU samples PPG; Linux runs quality MLP / wave recovery<br/>
        Only good 512-sample windows are POSTed
      </td>
    </tr>
  </table>
</div>

### Track A — ESP32 wearable (`pulsecare-iot`)

The original closed-loop prototype: a self-contained wearable that filters, transforms, displays, and uploads on one MCU.

| Layer | What it does |
| --- | --- |
| Hardware | ESP32-WROOM-32, MAX30102 (I2C PPG), ST7789 240×240 SPI TFT, LiPo power |
| Firmware | PlatformIO / C++ / Arduino framework, FreeRTOS tasks `SensorTask`, `NetworkTask`, `SerialTask` |
| Quality gate | Normalized cross-correlation template matching (`QUALITY_CAPTURE_GOOD` / `BAD`) to drop motion artifacts |
| Analytics | On-device 512-point FFT (`ArduinoFFT`) at ~50 Hz, ~10.24 s windows |
| UI | On-wrist screens: put finger, loading, AC waveform, BPM |
| Sync | HTTP POST of FFT JSON when quality is `GOOD` |

### Track B — Arduino Uno Q Edge AI (`pulsecare-arduino`)

The Hack Challenge / Edge-Cloud path: split MCU sampling and Linux inference on Arduino Uno Q (QRB2210 Debian side + MCU), packaged for Arduino App Lab.

| Layer | What it does |
| --- | --- |
| Hardware | Arduino Uno Q + MAX30102 (RED channel, I2C). Debug UI is a Linux WebUI, not a wrist TFT |
| MCU sketch | ~50 Hz sampling, DC/AC filter, finger detect, beat-to-beat BPM, quality 0–100, `Arduino_RouterBridge` |
| Linux gateway | 512-sample windows, measured `fs`, quality MLP (numpy), optional Tiny 1D U-Net + SPEAR wave recovery, FFT |
| Trust rule | POST only when the window is contiguous, finger is present, and Edge AI class is `good_quality` |
| Packaging | Arduino App Lab app (`app.yaml`, `python/main.py`, `sketch/sketch.ino`) |

```text
MAX30102
  -> Uno Q MCU: PPG, AC, BPM, quality
  -> Arduino_RouterBridge
  -> Uno Q Linux: 512-sample window, Edge AI, optional recon, FFT
  -> FastAPI: store trusted window + Edge AI metadata
  -> Flutter / WebUI: live BPM, FFT, history
```

Edge AI answers `good_quality` or `poor_quality`. It is not a disease classifier. If the `.npz` model is missing, the device reports `edge_ai_status = missing_model` and does not fake AI.

## 🌐 Shared Cloud And Clients

<div align="center">
  <table>
    <tr>
      <td align="center">
        <b>⌚ Wearable Device</b><br/>
        <code>ESP32 + MAX30102</code> or <code>Arduino Uno Q + MAX30102</code>
      </td>
    </tr>
    <tr>
      <td align="center">
        ⬇️<br/>
        <i>📡 HTTP POST of trusted FFT / PPG windows (<code>X-Device-Token</code>)</i>
      </td>
    </tr>
    <tr>
      <td align="center">
        <b>⚙️ PulseCare Backend</b><br/>
        <code>FastAPI + MongoDB Atlas</code>
      </td>
    </tr>
    <tr>
      <td align="center">
        ⬇️<br/>
        <i>🔒 validates device and user tokens</i><br/>
        <i>🧮 stores measurements; optional NeuroKit2 check on PPG windows</i><br/>
        <i>⚡ streams live updates through WebSocket</i><br/>
        <i>🤖 serves AI chat using <b>MongoDB Vector Search + Groq</b></i>
      </td>
    </tr>
    <tr>
      <td align="center">
        <b>📱 Client Apps</b><br/>
        <code>Flutter mobile</code> · <code>Uno Q WebUI</code> · <code>RAG chatbot</code>
      </td>
    </tr>
  </table>
</div>

## 📂 Organization Repositories

<table align="center" width="100%">
  <tr>
    <td width="24%" align="center"><kbd>pulsecare-iot</kbd></td>
    <td>ESP32 wearable firmware: FreeRTOS, MAX30102, ST7789, template matching, on-device FFT, HTTP sync.</td>
  </tr>
  <tr>
    <td width="24%" align="center"><kbd>pulsecare-arduino</kbd></td>
    <td>Arduino Uno Q App Lab app: MCU PPG pipeline, Linux Edge AI quality MLP, wave recovery, FFT gateway.</td>
  </tr>
  <tr>
    <td width="24%" align="center"><kbd>pulsecare-be</kbd></td>
    <td>FastAPI backend: auth, devices, measurement ingestion, WebSocket FFT stream, RAG chat.</td>
  </tr>
  <tr>
    <td width="24%" align="center"><kbd>pulsecare-fe</kbd></td>
    <td>Flutter client: live FFT/BPM charts, history, device pairing, chat.</td>
  </tr>
  <tr>
    <td width="24%" align="center"><kbd>pulsecare-chat-bot</kbd></td>
    <td>Standalone RAG medical chatbot assets (LangChain, embeddings, Streamlit).</td>
  </tr>
  <tr>
    <td width="24%" align="center"><kbd>pulsecare-docs</kbd></td>
    <td>Interactive HTML technical documentation.</td>
  </tr>
  <tr>
    <td width="24%" align="center"><kbd>.github</kbd></td>
    <td>Organization profile and shared GitHub automation.</td>
  </tr>
</table>

## 🌊 Main Data Flow

<ol>
  <li>MAX30102 produces finger PPG. ESP32 or Uno Q MCU filters DC/AC and estimates BPM / quality.</li>
  <li>A quality gate runs on device:
    <ul>
      <li><b>ESP32:</b> normalized cross-correlation template matching, then on-MCU FFT.</li>
      <li><b>Uno Q:</b> Linux quality MLP (and optional recon) on a 512-sample window, then FFT.</li>
    </ul>
  </li>
  <li>Only trusted windows are POSTed to <code>/api/v1/health-measurements</code> with a device token.</li>
  <li>Backend stores FFT, optional PPG traces, heart-rate metadata, and Uno Q Edge AI fields.</li>
  <li>Flutter / WebUI query history and subscribe to <code>/api/v1/health-measurements/ws/fourier</code>.</li>
  <li>Chat requests retrieve medical context from vector search and generate educational (not diagnostic) answers.</li>
</ol>

## 💻 Technology Stack

<p align="center">
  <img src="https://img.shields.io/badge/ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white" />
  <img src="https://img.shields.io/badge/Arduino_Uno_Q-008184?style=for-the-badge&logo=arduino&logoColor=white" />
  <img src="https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi" />
  <img src="https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white" />
</p>

<ul>
  <li><b>ESP32 IoT</b>: PlatformIO, FreeRTOS, ArduinoFFT, ST7789, MAX30102, HTTP client</li>
  <li><b>Arduino Uno Q</b>: MCU sketch + Python App Lab gateway, numpy quality MLP, Tiny 1D U-Net recon, FFT</li>
  <li><b>Sensor</b>: MAX30102 optical PPG (I2C)</li>
  <li><b>Backend</b>: Python 3.11, FastAPI, Uvicorn, Pydantic, JWT</li>
  <li><b>Database</b>: MongoDB Atlas (Motor async + PyMongo for RAG / Vector Search)</li>
  <li><b>AI / RAG</b>: LangChain, Sentence Transformers, Cross-Encoder rerank, Groq LLM</li>
  <li><b>Clients</b>: Flutter (FL Chart, WebSocket), Uno Q Linux WebUI</li>
  <li><b>Realtime</b>: WebSocket streaming of live Fourier / measurement updates</li>
</ul>

## 🔑 Key Backend API Areas

<table align="center" width="100%">
  <tr><td width="30%"><b>Auth</b></td><td><code>/api/v1/auth/*</code></td></tr>
  <tr><td width="30%"><b>Users</b></td><td><code>/api/v1/users/*</code></td></tr>
  <tr><td width="30%"><b>Devices</b></td><td><code>/api/v1/devices/*</code></td></tr>
  <tr><td width="30%"><b>Health measurements</b></td><td><code>/api/v1/health-measurements/*</code></td></tr>
  <tr><td width="30%"><b>AI chat</b></td><td><code>/api/v1/chat/*</code></td></tr>
  <tr><td width="30%"><b>FFT WebSocket stream</b></td><td><code>/api/v1/health-measurements/ws/fourier</code></td></tr>
</table>

Both hardware tracks authenticate with a device token and post compatible measurement payloads. Uno Q windows may include extra Edge AI fields (`edge_quality_class`, `edge_model`, recon RMSE, original vs recovered AC).

<details>
<summary><b>🚀 Quick Start (Local Backend)</b></summary>
<br/>

### 1) Requirements

- Python 3.9+
- MongoDB local instance or MongoDB Atlas

### 2) Install dependencies

```bash
cd pulsecare-be
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

### 3) Configure environment variables

Create a `.env` file in `pulsecare-be`:

```env
MONGODB_URL=mongodb://localhost:27017
DATABASE_NAME=pulsecare
JWT_SECRET=change-me
GROQ_API_KEY=
```

Leave `GROQ_API_KEY` empty if AI chat is not enabled yet.

### 4) Run the server

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

<ul>
  <li>Swagger UI: <a href="http://localhost:8000/docs">http://localhost:8000/docs</a></li>
  <li>Liveness endpoint: <code>GET /healthz</code></li>
  <li>Readiness endpoint (with DB check): <code>GET /readyz</code></li>
</ul>

Pair an ESP32 firmware build from <code>pulsecare-iot</code>, or import the Uno Q App Lab package from <code>pulsecare-arduino</code>, then register a device token before posting measurements.

</details>

## ☁️ Deployment Notes

<ul>
  <li>Backend production process is defined in <code>pulsecare-be/Procfile</code>.</li>
  <li>Use <code>GET /healthz</code> for frequent lightweight health checks.</li>
  <li>Use <code>GET /readyz</code> for deeper readiness validation.</li>
  <li>ESP32 devices need Wi-Fi credentials and the production ingest URL.</li>
  <li>Uno Q App Lab apps need a valid zip layout (<code>app.yaml</code>, <code>python/main.py</code>, <code>sketch/sketch.ino</code>) and the Edge AI <code>.npz</code> model on the Linux side.</li>
</ul>

---

<div align="center">
  <h3>⚠️ Important Disclaimer</h3>
  <p>
    PulseCare supports personal health monitoring and educational guidance.<br>
    <i>It is not a replacement for professional medical diagnosis or treatment.</i>
  </p>
</div>
