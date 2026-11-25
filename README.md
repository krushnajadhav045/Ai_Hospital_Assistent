# Sarah — AI Hospital Voice Receptionist

A FastAPI-based multilingual voice AI receptionist web application for booking hospital appointments. It pairs a browser voice interface (`frontend.html`) with a FastAPI backend (`app.py`), Google Gemini (`gemini-2.5-flash` via `google-genai`) with function calling, SQLite (`appointments_poc.db`) for doctor and appointment scheduling, and `gTTS` for spoken responses.

---

## Features

- **Voice & Chat Call Interface:** Browser UI (`frontend.html`) with patient name/email intake, language selection, call timer, animated call states (`idle`, `listening`, `thinking`, `speaking`), mute toggle, and live transcript display.
- **Browser Speech Recognition & Audio Playback:** Uses the Web Speech API (`SpeechRecognition` / `webkitSpeechRecognition`) for voice input in Chrome/Edge and plays base64-encoded MP3 audio returned by the backend.
- **Multilingual Support:** Supports `en-IN` (English), `hi-IN` (Hindi), and `te-IN` (Telugu), with text sanitization (`clean_text_for_audio`) to strip markdown and parenthetical text before generating speech with `gTTS`.
- **Appointment Scheduling Tools (Gemini Function Calling):** Exposes backend tools to Gemini so the assistant can:
  - List available doctors and specialties (`list_doctors`)
  - Check slot availability for a given doctor and timestamp (`check_slot`)
  - Book confirmed appointments in SQLite while preventing double-bookings (`book_appointment`)
- **Session Management:** Maintains per-session Gemini chat instances in memory keyed by `session_id` (`/agent/new_session` and `/agent/message`).

---

## Project Structure

```text
.
├── app.py                # FastAPI server, SQLite setup, Gemini tools, gTTS audio & session logic
├── frontend.html         # Tailwind CSS + vanilla JS voice call UI and Web Speech client
├── requirements.txt      # Python dependencies
├── .env                  # Local environment variables (create manually; keep out of Git)
└── appointments_poc.db   # SQLite database (auto-created on startup with sample doctors)
