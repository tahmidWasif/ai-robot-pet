# CompanionAI (AI Robot Pet)

CompanionAI is a voice-controlled robot-pet prototype built by a three-person team for HackUTA 7. A user records a command or conversational prompt in the web application, Gemini interprets the audio, Supabase shares the resulting action with a Raspberry Pi, and the Pi represents robot actions through GPIO-connected LEDs while optionally speaking responses with ElevenLabs.

> This repository is an educational hackathon prototype. It demonstrates the complete interaction pipeline, but it still requires security and hardware hardening before production use.

## Features

- Supabase email authentication with session restoration and protected application routes
- Device-connection workflow that gates access to microphone controls
- In-browser audio capture with `MediaRecorder`, echo cancellation, and noise suppression
- Gemini 2.5 Flash transcription and intent classification for commands and conversation
- Structured action output for forward, backward, left, and right commands
- Supabase-backed state exchange between the web application and Raspberry Pi
- Raspberry Pi polling client with GPIO LED output for visualizing actions
- Optional ElevenLabs text-to-speech playback for conversational responses
- Flask endpoints for audio processing, service health, status, history, pending commands, and image uploads

## How It Works

```text
Browser microphone
       |
       v
React + Vite web application
       |
       | POST /api/upload-audio
       v
Flask API
       |
       v
Gemini 2.5 Flash
(transcription + intent classification)
       |
       v
Supabase user_profiles
       |
       | Raspberry Pi polls for updates
       v
GPIO action LEDs + optional ElevenLabs speech
```

The AI processor returns an action code and a response. Command-like prompts activate the corresponding GPIO output; conversational prompts use action `0` and can be spoken by the Raspberry Pi.

| Action | Meaning | Raspberry Pi BCM pin |
| --- | --- | --- |
| `0` | No movement / conversation | None |
| `1` | Move forward | 17 |
| `2` | Move backward | 18 |
| `3` | Turn left | 23 |
| `4` | Turn right | 27 |

## Tech Stack

- **Frontend:** React 18, Vite, React Router, Tailwind CSS, Lucide React
- **Backend:** Python, Flask, Flask-CORS
- **AI:** Google Gemini 2.5 Flash
- **Authentication and data:** Supabase Auth, Postgres, and Storage
- **Hardware client:** Raspberry Pi, Python, `RPi.GPIO`
- **Speech:** ElevenLabs Text-to-Speech (optional)

## Project Structure

```text
.
├── my-app/
│   ├── src/
│   │   ├── backend/          # Flask API, AI processing, and Python dependencies
│   │   │   └── pi_client.py  # Raspberry Pi polling, GPIO, and speech client
│   │   ├── components/       # React interface components
│   │   ├── config/           # Supabase client configuration
│   │   └── context/          # Authentication state
│   └── package.json
├── test_backend.py           # Live Flask endpoint checks
├── test_supabase_connection.py
├── SETUP_INSTRUCTIONS.md
└── TESTING_GUIDE.md
```

## Getting Started

### Prerequisites

- Node.js 20 or later
- Python 3.10 or later
- A Supabase project
- A Google Gemini API key
- Optional: Raspberry Pi with GPIO-connected LEDs and an ElevenLabs API key

### 1. Clone the repository

```bash
git clone https://github.com/tahmidWasif/ai-robot-pet.git
cd ai-robot-pet
```

### 2. Configure the frontend

Create `my-app/.env`:

```env
VITE_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
VITE_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```

Install the frontend dependencies:

```bash
cd my-app
npm install
```

### 3. Configure the backend

Create `my-app/src/backend/.env`:

```env
GEMINI_API_KEY=YOUR_GEMINI_API_KEY
SUPABASE_URL=https://YOUR_PROJECT.supabase.co
SUPABASE_SERVICE_ROLE_KEY=YOUR_SUPABASE_SERVICE_ROLE_KEY
SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY

# Optional
ALLOWED_ORIGINS=http://localhost:5173
BUCKET_NAME=images
SAVE_PHOTO_METADATA=false
PI_SERVER_URL=http://YOUR_PI_ADDRESS
```

This repository does not currently include a complete database migration. Provision the required Supabase resources in your own project before running the full pipeline. At minimum, the current application expects a `user_profiles` record that can store `user_id`, `action`, `response`, and `is_command`.

Install and start the backend:

```bash
cd my-app/src/backend
python -m venv .venv
```

Activate the virtual environment:

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# macOS or Linux
source .venv/bin/activate
```

Then install the packages and run Flask:

```bash
pip install -r requirements.txt
python app.py
```

The API runs at `http://localhost:5000`.

### 4. Start the frontend

In a separate terminal:

```bash
cd my-app
npm run dev
```

Open `http://localhost:5173`, create an account or sign in, connect the displayed device, and open the microphone page.

### 5. Run the Raspberry Pi client (optional)

Before starting the Pi client:

1. Open `my-app/src/backend/pi_client.py` and replace the hard-coded `target_user_id` with the Supabase user ID you want the Pi to follow.
2. Set the required environment variables:

```env
SUPABASE_URL=https://YOUR_PROJECT.supabase.co
SUPABASE_SERVICE_ROLE_KEY=YOUR_SUPABASE_SERVICE_ROLE_KEY

# Optional for spoken responses
ELEVENLABS_API_KEY=YOUR_ELEVENLABS_API_KEY
```

3. Connect LEDs to BCM pins `17`, `18`, `23`, and `27` with appropriate resistors.
4. Install the Pi-side Python packages and run:

```bash
python my-app/src/backend/pi_client.py
```

The client polls Supabase every two seconds for the selected user's latest action.

## Testing

With the Flask server running, execute the live endpoint checks from the repository root:

```bash
python test_backend.py
```

To verify access to your configured Supabase project and storage:

```bash
python test_supabase_connection.py
```

These are integration and diagnostic scripts that depend on running services and valid environment configuration; they are not an isolated unit-test suite.

## Security and Current Limitations

- Never commit Supabase service-role, Gemini, or ElevenLabs secrets. The service-role key belongs only on trusted backend or Raspberry Pi systems.
- Use your own Supabase project and credentials; do not rely on project-specific values found in older setup notes.
- The current prototype trusts a user identifier supplied by the client and does not enforce production-grade API authorization.
- The Raspberry Pi client currently uses a hard-coded target user and polling rather than a device-registration or realtime-subscription flow.
- GPIO LEDs represent movement commands in the current version; integration with motor drivers and physical robot movement is future work.
- The repository contains prototype artifacts and requires cleanup, database migrations, stronger validation, and automated tests before deployment.

## Contributors

CompanionAI was developed collaboratively for HackUTA 7 by:

- [Adib Bin Kadir](https://github.com/AdibBinKadir)
- [Saad Khairullah](https://github.com/saadkhairullah)
- [Md Tahmid Wasif](https://github.com/tahmidWasif)

## License

No license has been added yet. Unless a license is added, the repository remains under standard copyright.
