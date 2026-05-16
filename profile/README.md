<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=timeGradient&height=200&section=header&text=PulseCare&fontSize=70&fontColor=ffffff&animation=fadeIn" width="100%" />

  <h1>PulseCare System Overview</h1>
  <p>
    <b>PulseCare is an IoT + AI healthcare platform focused on continuous heart-rate monitoring and intelligent health support.</b><br>
    <i>This repository contains the backend service that powers device ingestion, data storage, realtime streaming, and medical chatbot responses.</i>
  </p>
</div>

<br/>

## 🎯 What PulseCare Solves

<p>
Cardiovascular issues often require early detection and long-term trend monitoring. PulseCare is designed to:
</p>

<ul>
  <li><b>Collect</b> physiological signals from wearable hardware.</li>
  <li><b>Process and store</b> measurements in near real time.</li>
  <li><b>Provide</b> data access for web/mobile applications.</li>
  <li><b>Assist</b> users with AI-driven medical Q&A using retrieval-augmented generation (RAG).</li>
</ul>

## 🔄 End-to-End System Introduction

<div align="center">
  <table>
    <tr>
      <td align="center">
        <b>⌚ Wearable Device</b><br/>
        <code>ESP32 + MAX30102</code>
      </td>
    </tr>
    <tr>
      <td align="center">
        ⬇️<br/>
        <i>📡 sends FFT/PPG data via HTTP</i>
      </td>
    </tr>
    <tr>
      <td align="center">
        <b>⚙️ PulseCare Backend</b><br/>
        <code>FastAPI</code>
      </td>
    </tr>
    <tr>
      <td align="center">
        ⬇️<br/>
        <i>🔒 validates device and user tokens</i><br/>
        <i>🧮 computes and stores measurements in <b>MongoDB</b></i><br/>
        <i>⚡ streams live updates through WebSocket</i><br/>
        <i>🤖 serves AI chat using <b>MongoDB Vector Search + Groq</b></i>
      </td>
    </tr>
    <tr>
      <td align="center">
        <b>📱 Client Apps</b><br/>
        <code>Web/Mobile</code>
      </td>
    </tr>
    <tr>
      <td align="center">
        ⬇️<br/>
        <i>🌐 consume REST APIs and WebSocket streams</i>
      </td>
    </tr>
  </table>
</div>

## 📂 Repository Context

<p>The full PulseCare ecosystem in the monorepo includes:</p>

<table align="center" width="100%">
  <tr>
    <td width="20%" align="center"><kbd>pulsecare-be</kbd></td>
    <td>Backend API and AI services (this service).</td>
  </tr>
  <tr>
    <td width="20%" align="center"><kbd>pulsecare-fe</kbd></td>
    <td>Frontend client application.</td>
  </tr>
  <tr>
    <td width="20%" align="center"><kbd>pulsecare-iot</kbd></td>
    <td>IoT firmware and device-side logic.</td>
  </tr>
  <tr>
    <td width="20%" align="center"><kbd>pulsecare-chat-bot</kbd></td>
    <td>Additional chatbot-related assets/modules.</td>
  </tr>
  <tr>
    <td width="20%" align="center"><kbd>.github</kbd></td>
    <td>CI/CD and repository automation.</td>
  </tr>
</table>

## 🛠️ What This Backend Provides

<ul>
  <li>Authentication and authorization endpoints.</li>
  <li>User and device management.</li>
  <li>Measurement ingestion pipeline for FFT-based heart data.</li>
  <li>Historical measurement queries for analytics and dashboards.</li>
  <li>Realtime measurement streaming through WebSocket.</li>
  <li>Medical chat endpoints powered by RAG.</li>
</ul>

## 🏗️ Backend Project Structure

```text
📦 pulsecare-be
 ┣ 📂 app
 ┃ ┣ 📂 api         # API routers and endpoints (v1)
 ┃ ┣ 📂 core        # configuration and settings
 ┃ ┣ 📂 database    # MongoDB client, indexes, counters
 ┃ ┣ 📂 models      # Pydantic request/response schemas
 ┃ ┣ 📂 services    # security, RAG, business logic
 ┃ ┗ 📜 main.py     # FastAPI application entrypoint
 ┣ 📜 requirements.txt
 ┣ 📜 Procfile
 ┗ 📜 runtime.txt
```

## 🌊 Main Data Flow

<ol>
  <li>Devices send FFT/PPG payloads to backend ingestion endpoints.</li>
  <li>Backend validates device token, computes heart-rate metadata, and writes to MongoDB.</li>
  <li>Frontend/mobile clients query history and subscribe to realtime streams.</li>
  <li>For chat requests, backend retrieves medical context from vector search and generates responses with an LLM.</li>
</ol>

## 💻 Technology Stack

<p align="center">
  <img src="https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi" />
  <img src="https://img.shields.io/badge/MongoDB-4EA94B?style=for-the-badge&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white" />
</p>

<ul>
  <li><b>Backend</b>: FastAPI, Uvicorn, Pydantic</li>
  <li><b>Database</b>: MongoDB (Motor async client + PyMongo sync client for RAG)</li>
  <li><b>AI/RAG</b>: LangChain MongoDB, Sentence Transformers, Groq API</li>
  <li><b>IoT</b>: ESP32 + MAX30102 sensor over HTTP</li>
  <li><b>Realtime</b>: WebSocket streaming for live data delivery</li>
</ul>

<br/>

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

</details>

## 🔑 Key API Areas

<table align="center" width="100%">
  <tr><td width="30%"><b>Auth</b></td><td><code>/api/v1/auth/*</code></td></tr>
  <tr><td width="30%"><b>Users</b></td><td><code>/api/v1/users/*</code></td></tr>
  <tr><td width="30%"><b>Devices</b></td><td><code>/api/v1/devices/*</code></td></tr>
  <tr><td width="30%"><b>Health measurements</b></td><td><code>/api/v1/health-measurements/*</code></td></tr>
  <tr><td width="30%"><b>AI chat</b></td><td><code>/api/v1/chat/*</code></td></tr>
  <tr><td width="30%"><b>FFT WebSocket stream</b></td><td><code>/api/v1/health-measurements/ws/fourier</code></td></tr>
</table>

## ☁️ Deployment Notes

<ul>
  <li>Production process is defined in <code>Procfile</code>.</li>
  <li>Use <code>GET /healthz</code> for frequent lightweight health checks.</li>
  <li>Use <code>GET /readyz</code> for deeper readiness validation.</li>
</ul>

---

<div align="center">
  <h3>⚠️ Important Disclaimer</h3>
  <p>
    PulseCare supports personal health monitoring and educational guidance.<br>
    <i>It is not a replacement for professional medical diagnosis or treatment.</i>
  </p>
</div>
