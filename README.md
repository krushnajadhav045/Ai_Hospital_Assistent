<h1 align="center">— AI Hospital Voice Receptionist</h1>

<div align="center">

<img alt="Python" src="https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white" />
<img alt="FastAPI" src="https://img.shields.io/badge/FastAPI-Backend-009688?logo=fastapi&logoColor=white" />
<img alt="Gemini" src="https://img.shields.io/badge/LLM-gemini--2.5--flash-4285F4?logo=googlegemini&logoColor=white" />
<img alt="gTTS" src="https://img.shields.io/badge/TTS-gTTS-0F9D58?logo=google&logoColor=white" />
<img alt="SQLite" src="https://img.shields.io/badge/Database-SQLite-003B57?logo=sqlite&logoColor=white" />

</div>

<h3 align="center">Multilingual Voice AI Receptionist for Hospital Appointment Scheduling.</h3>

---

## Overview

**Sarah** is a voice-enabled AI hospital receptionist built with **FastAPI**, **Google Gemini (`gemini-2.5-flash`)**, **gTTS**, **SQLite**, and a browser voice interface (`frontend.html`). Patients can enter their details, select a language (`en-IN`, `hi-IN`, or `te-IN`), speak with Sarah using browser speech recognition, and book appointments backed by SQLite tool calls.

---

## Features

- **Voice & Chat Call Interface:** Browser UI (`frontend.html`) with patient name and email intake, language selection, live call timer, animated call states (`idle`, `listening`, `thinking`, `speaking`), mute toggle, and live transcript display.
- **Browser Speech Recognition & Audio Playback:** Uses the Web Speech API (`SpeechRecognition` / `webkitSpeechRecognition`) for voice input in Chrome/Edge and plays base64-encoded MP3 audio returned by the backend.
- **Multilingual Support:** Supports `en-IN` (English), `hi-IN` (Hindi), and `te-IN` (Telugu), with text sanitization (`clean_text_for_audio`) to strip markdown and parenthetical text before generating speech with `gTTS`.
- **Appointment Scheduling Tools (Gemini Function Calling):** Exposes backend tools to Gemini so the assistant can:
  - List available doctors and specialties (`list_doctors`)
  - Check slot availability for a given doctor and timestamp (`check_slot`)
  - Book confirmed appointments in SQLite while preventing double-bookings (`book_appointment`)
- **Session Management:** Maintains per-session Gemini chat instances in memory keyed by `session_id` (`/agent/new_session` and `/agent/message`).

---

## Architecture & Flow

```text
Browser (frontend.html)
   │
   ├── POST /agent/new_session  ──▶ Creates UUID chat session in memory
   │
   ├── Web Speech API (STT)     ──▶ Captures patient speech in selected language
   │
   ├── POST /agent/message      ──▶ Sends text + language instruction to Gemini
   │                                  ├── Tool Calls: list_doctors, check_slot, book_appointment
   │                                  ├── Reads/Writes: appointments_poc.db (SQLite)
   │                                  └── Sanitizes text & generates MP3 via gTTS (base64)
   │
   └── HTML5 Audio              ──▶ Plays spoken response & triggers next listen cycle
```

---

## Project Structure

```text
.
├── app.py                # FastAPI endpoints, SQLite init & tools, Gemini chat, gTTS audio
├── frontend.html         # Voice call UI, Web Speech API client, visual ring & audio player
├── requirements.txt      # Project Python dependencies
├── .env                  # Environment variables (GEMINI_API_KEY; keep out of Git)
└── appointments_poc.db   # SQLite database (auto-created with doctors & appointments tables)
```

---

## Installation

### 1. Create and activate a virtual environment

```bash
python -m venv .venv
# Linux / macOS:
source .venv/bin/activate
# Windows (PowerShell):
.venv\Scripts\Activate.ps1
# Windows (Command Prompt):
.venv\Scripts\activate.bat
```

### 2. Install dependencies

```bash
pip install --upgrade pip
pip install fastapi uvicorn python-dotenv gtts google-genai
```

### 3. Configure environment variables

Create a `.env` file in the project root (do not commit real API keys to Git):

```env
GEMINI_API_KEY=your_gemini_api_key_here
```

---

## Usage

Start the backend server:

```bash
python app.py
```

Or run with Uvicorn:

```bash
uvicorn app:app --host 127.0.0.1 --port 8000 --reload
```

Open `http://127.0.0.1:8000` in **Google Chrome** or **Microsoft Edge**, enter optional patient name/email, choose a language (`English (India)`, `Hindi`, or `Telugu`), and click **Start Voice Call**.

---

## API Endpoints

- `GET /` — Serves `frontend.html`.
- `POST /agent/new_session` — Initializes a new session and returns `{"session_id": "<uuid>"}`.
- `POST /agent/message` — Processes user text and `language_code`, runs Gemini tool calls against SQLite, synthesizes speech with `gTTS`, and returns `{"session_id": "...", "text": "...", "audio": "<base64-mp3>"}`.

---

## Database Schema (`appointments_poc.db`)

On startup, `init_db()` creates `appointments_poc.db` with two tables and seeds initial doctors:

- **`doctors`**: `id`, `name` (`UNIQUE`), `specialty`
  - Seeded records: `Dr. Meera Patel` (`Cardiology`) and `Dr. Arjun Rao` (`Neurology`)
- **`appointments`**: `id`, `patient_name`, `doctor_name`, `appointment_time`, `patient_email`, `status`, with a `UNIQUE(doctor_name, appointment_time)` constraint

---

## Dependencies

```text
fastapi
uvicorn
python-dotenv
google-generativeai
TTS
torch
soundfile
```

*(Note: `app.py` specifically imports `google-genai` and `gtts` for Gemini and text-to-speech synthesis.)*

---

## Notes & Limitations

- **Browser Speech API:** Voice input relies on `window.SpeechRecognition` / `window.webkitSpeechRecognition`, which is supported in Chromium-based browsers like Chrome and Edge and requires microphone permissions.
- **In-Memory Sessions:** `AGENT_SESSIONS` is stored in memory per process, so restarting the server clears active conversation sessions.

---

## License

Add a `LICENSE` file to specify your license terms for this project.
