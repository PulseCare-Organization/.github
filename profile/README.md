# PulseCare System Overview

PulseCare is an IoT + AI healthcare platform focused on continuous heart-rate monitoring and intelligent health support.  
This repository contains the backend service that powers device ingestion, data storage, realtime streaming, and medical chatbot responses.

## What PulseCare Solves

Cardiovascular issues often require early detection and long-term trend monitoring. PulseCare is designed to:

- Collect physiological signals from wearable hardware.
- Process and store measurements in near real time.
- Provide data access for web/mobile applications.
- Assist users with AI-driven medical Q&A using retrieval-augmented generation (RAG).

## End-to-End System Introduction

```text
Wearable Device (ESP32 + MAX30102)
   -> sends FFT/PPG data via HTTP
PulseCare Backend (FastAPI)
   -> validates device and user tokens
   -> computes and stores measurements in MongoDB
   -> streams live updates through WebSocket
   -> serves AI chat using MongoDB Vector Search + Groq
Client Apps (Web/Mobile)
   -> consume REST APIs and WebSocket streams
```

## Repository Context

The full PulseCare ecosystem in the monorepo includes:

- `pulsecare-be`: Backend API and AI services (this service).
- `pulsecare-fe`: Frontend client application.
- `pulsecare-iot`: IoT firmware and device-side logic.
- `pulsecare-chat-bot`: Additional chatbot-related assets/modules.
- `.github`: CI/CD and repository automation.

## What This Backend Provides

- Authentication and authorization endpoints.
- User and device management.
- Measurement ingestion pipeline for FFT-based heart data.
- Historical measurement queries for analytics and dashboards.
- Realtime measurement streaming through WebSocket.
- Medical chat endpoints powered by RAG.

## Backend Project Structure

```text
pulsecare-be/
  app/
    api/         # API routers and endpoints (v1)
    core/        # configuration and settings
    database/    # MongoDB client, indexes, counters
    models/      # Pydantic request/response schemas
    services/    # security, RAG, business logic
    main.py      # FastAPI application entrypoint
  requirements.txt
  Procfile
  runtime.txt
```

## Main Data Flow

1. Devices send FFT/PPG payloads to backend ingestion endpoints.
2. Backend validates device token, computes heart-rate metadata, and writes to MongoDB.
3. Frontend/mobile clients query history and subscribe to realtime streams.
4. For chat requests, backend retrieves medical context from vector search and generates responses with an LLM.

## Technology Stack

- **Backend**: FastAPI, Uvicorn, Pydantic
- **Database**: MongoDB (Motor async client + PyMongo sync client for RAG)
- **AI/RAG**: LangChain MongoDB, Sentence Transformers, Groq API
- **IoT**: ESP32 + MAX30102 sensor over HTTP
- **Realtime**: WebSocket streaming for live data delivery

## Quick Start (Local Backend)

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

- Swagger UI: `http://localhost:8000/docs`
- Liveness endpoint: `GET /healthz`
- Readiness endpoint (with DB check): `GET /readyz`

## Key API Areas

- Auth: `/api/v1/auth/*`
- Users: `/api/v1/users/*`
- Devices: `/api/v1/devices/*`
- Health measurements: `/api/v1/health-measurements/*`
- AI chat: `/api/v1/chat/*`
- FFT WebSocket stream: `/api/v1/health-measurements/ws/fourier`

## Deployment Notes

- Production process is defined in `Procfile`.
- Use `GET /healthz` for frequent lightweight health checks.
- Use `GET /readyz` for deeper readiness validation.

## Important Disclaimer

PulseCare supports personal health monitoring and educational guidance.  
It is not a replacement for professional medical diagnosis or treatment.
