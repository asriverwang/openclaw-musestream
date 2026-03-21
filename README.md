# MuseStream

AI music generation and streaming server. Describe a mood or context — get a continuous, browser-playable music stream with songs saved to a local library.

**Provider-agnostic** — Sonauto is the default. Adding new music generation APIs requires only a config entry.

---

## Features

- **Continuous streaming player** — Agent sends user a URL; browser streams AI-generated music song after song with no interruption
- **Context-aware prompts** — Converts real-world context (weather, mood, activity, traffic) into music generation prompts via MiniMax M2.7 or Claude Haiku, with a rule-based fallback
- **Persistent library** — All generated songs are saved locally with metadata (title, tags, lyrics) and browsable via a built-in player
- **Mobile context UI** — Form at `/context-ui` for sharing context from any device
- **Background save on stop** — Clicking Stop finishes saving the current song before ending

---

## Setup

### 1. Get a Sonauto API key
Register at **https://sonauto.ai** and copy your API key.

### 2. Configure
```bash
cp .env.example .env
# Set at minimum: SONAUTO_API_KEY=your_key
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Start the server
```bash
./restart_musestream.sh
# or: python3 musestream_server.py
```

Server runs on **http://localhost:5000** by default.

---

## Usage

### Generate music from a prompt
```bash
curl "http://localhost:5000/start?prompt=upbeat+indie+rock+morning+energy"
# → {"url": "http://localhost:5000/player?s=abc12345", ...}
```
Open the returned URL in a browser. Music starts streaming immediately.

### Generate from context (weather, mood, activity)
```bash
curl -X POST http://localhost:5000/api/context \
  -H "Content-Type: application/json" \
  -d '{"time": "evening", "weather": "rainy", "mood": "relaxed", "activity": "working"}'
# → {"url": "...", "prompt": "ambient lo-fi with soft rain texture, focused evening energy"}
```

### Browse the library
Open **http://localhost:5000/** in a browser — prev/next/shuffle player for all saved songs.

---

## API Reference

| Endpoint | Method | Description |
|---|---|---|
| `/start?prompt=...` | GET | Create session → player URL |
| `/player?s=<key>` | GET | Streaming player page |
| `/generate?prompt=...` | GET | Start a generation job |
| `/status/<task_id>` | GET | Check job status |
| `/stream/<task_id>` | GET | Audio stream (browser connects here) |
| `/metadata/<task_id>` | GET | Song title, tags, lyrics |
| `/stop` | POST | Stop tasks `{"task_ids": [...]}` |
| `/api/context` | POST/GET | Context JSON → player URL |
| `/context-ui` | GET | Mobile-friendly context form |
| `/library` | GET | JSON list of saved songs |
| `/files/<filename>` | GET | Serve saved audio (seekable) |
| `/` | GET | Library player UI |

---

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `MUSIC_PROVIDER` | `sonauto` | Active provider |
| `SONAUTO_API_KEY` | — | **Required** — get at https://sonauto.ai |
| `MUSESTREAM_OUTPUT_DIR` | `~/Music/MuseStream` | Where songs are saved |
| `MUSESTREAM_PORT` | `5000` | Server port |
| `MINIMAX_API_KEY` | — | MiniMax M2.7 for context prompts (preferred) |
| `ANTHROPIC_API_KEY` | — | Claude Haiku fallback for context prompts |

---

## Adding a New Provider

Add an entry to `PROVIDERS` in `musestream_server.py`:

```python
"udio": {
    "name":         "Udio",
    "register_url": "https://udio.com",
    "key_env":      "UDIO_API_KEY",
    "generate_url": "https://api.udio.com/v1/generate",
    "stream_base":  "https://api.udio.com/v1/stream",
    "status_url":   "https://api.udio.com/v1/status",
    "meta_url":     "https://api.udio.com/v1/songs",
    "audio_fmt":    "mp3",
    "mime":         "audio/mpeg",
},
```

Then add a branch in `start_generation()` for the provider's request format, set `MUSIC_PROVIDER=udio`, and restart.

---

## OpenClaw Integration

See **`SKILL.md`** for the agent reference — all endpoints, examples, and setup steps written for a fresh agent with no prior context.

---

## License

MIT
