# MuseStream Skill

AI music generation and streaming. Give the user a shareable player URL — music generates and plays continuously in their browser. All songs are saved to a local library.

**Provider-agnostic** — Sonauto is the default. Adding new music generation APIs requires only a config entry.

---

## Features

- **Continuous streaming player** — Agent sends user a URL; browser streams AI-generated music song after song with no interruption
- **Auto-queue** — Automatically requests the next song after 120 seconds of playback while the browser window stays active; click "Stop Stream" or close the window to stop queuing and save your Sonauto credits
- **Shareable links** — Player URLs can be shared externally; expose the server via a reverse proxy (e.g., Nginx, Caddy) or a tunnel (e.g., ngrok, Cloudflare Tunnel) with HTTPS and authentication to keep your stream secure
- **Context-aware prompts** — Converts real-world context (weather, mood, activity, traffic) into music generation prompts via MiniMax M2.7 or Claude Haiku, with a rule-based fallback
- **Persistent library** — All generated songs are saved locally with metadata (title, tags, lyrics) and browsable via a built-in player
- **Mobile context UI** — Form at `/context-ui` for sharing context from any device
- **Background save on stop** — Clicking Stop finishes saving the current song before ending
- **Messenger bot integration** — Connect MuseStream to your messaging bots (Telegram, Discord, Slack, etc.) so you can request AI music streams directly from a chat message and receive a playable link in reply

> [!CAUTION]
> Remember to click **Stop Stream** or close the browser window when you're done listening. The auto-queue will keep requesting new songs every 120 seconds, which consumes your Sonauto credits.

---

## Quick start for agent

### 1. Check if server is running
```bash
curl -s http://localhost:5000/library | python3 -m json.tool | head -5
```
If it returns JSON → server is up. If connection refused → start it:
```bash
source .env && ./restart_musestream.sh
```

### 2. Quick test
```bash
curl "http://localhost:5000/start?prompt=upbeat+indie+rock+morning+energy"
```

### 3. Generate music from a prompt
```
GET http://localhost:5000/start?prompt=<url-encoded description>
```
Returns `{ "url": "http://localhost:5000/player?s=<key>", "key": "...", "prompt": "..." }`

Send the `url` to the user. They open it in a browser — music streams automatically.

**Prompt tips:** describe genre, mood, energy, texture. No real artist or song names.
- `"upbeat indie rock with jangly guitars, morning energy"`
- `"dark ambient electronic, late night focus, minimal percussion"`
- `"smooth jazz piano trio, warm and intimate, chill evening"`

### 4. Generate from user context
```
POST http://localhost:5000/api/context
Content-Type: application/json

{
  "time": "evening",
  "weather": "rainy",
  "mood": "relaxed",
  "activity": "working from home",
  "driving": "",
  "traffic": "",
  "destination": ""
}
```
Returns `{ "url": "...", "prompt": "<LLM-generated music prompt>", "key": "..." }`

Mobile-friendly form: `http://localhost:5000/context-ui`

### 5. Browse the library
```
GET http://localhost:5000/library      # JSON list of all saved songs
GET http://localhost:5000/             # Browser library player
```

### 6. Stop streaming
```
POST http://localhost:5000/stop
{"task_ids": ["<task_id>"]}
```
Current song finishes saving before stopping.

---

## All endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/start?prompt=...` | GET | Create session → player URL |
| `/player?s=<key>` | GET | Streaming player page |
| `/generate?prompt=...&session=...` | GET | Start generation job |
| `/status/<task_id>` | GET | Check generation status |
| `/stream/<task_id>` | GET | Audio stream |
| `/metadata/<task_id>` | GET | Title, tags, lyrics |
| `/stop` | POST | Stop tasks `{"task_ids": [...]}` |
| `/api/context` | POST/GET | Context → prompt → player URL |
| `/context-ui` | GET | Mobile context form |
| `/library` | GET | JSON list of saved songs |
| `/files/<filename>` | GET | Serve saved audio (range-capable) |
| `/` | GET | Library player UI |

---

## Setup (first time)

### 1. Get API keys

Ask the user which LLM they use for context prompts (MiniMax, Claude, etc.), then guide them to get the required keys:

- **Sonauto** (required) — sign up at **https://sonauto.ai** → copy API key
- **MiniMax** (optional, preferred for context prompts) — get key from MiniMax
- **Anthropic** (optional, fallback for context prompts) — get key from Anthropic

### 2. Configure environment

Once the user provides their keys:
```bash
cp .env.example .env
```
Set the keys in `.env`:
```
SONAUTO_API_KEY=<user's key>
MINIMAX_API_KEY=<user's key, if provided>
ANTHROPIC_API_KEY=<user's key, if provided>
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Start the server
```bash
source .env && ./restart_musestream.sh
# Server at http://localhost:5000
```

The port is configurable via `MUSESTREAM_PORT`. The agent should pick an available port to avoid conflicts.

### Environment variables

| Variable | Default | Description |
|---|---|---|
| `MUSIC_PROVIDER` | `sonauto` | Active provider |
| `SONAUTO_API_KEY` | — | **Required** — get at https://sonauto.ai |
| `MUSESTREAM_OUTPUT_DIR` | `~/Music/MuseStream` | Where songs are saved |
| `MUSESTREAM_PORT` | `5000` | Server port |
| `MINIMAX_API_KEY` | — | MiniMax for context prompts (preferred) |
| `ANTHROPIC_API_KEY` | — | Claude Haiku fallback for context prompts |

---

## Adding a new provider

Add an entry to `PROVIDERS` in `musestream_server.py`:

```python
"myprovider": {
    "name":         "MyProvider",
    "register_url": "https://myprovider.com",
    "key_env":      "MYPROVIDER_API_KEY",
    "generate_url": "https://api.myprovider.com/v1/generate",
    "stream_base":  "https://api.myprovider.com/v1/stream",
    "status_url":   "https://api.myprovider.com/v1/status",
    "meta_url":     "https://api.myprovider.com/v1/songs",
    "audio_fmt":    "mp3",
    "mime":         "audio/mpeg",
},
```
Add a branch in `start_generation()` for the provider's payload format.
Set `MUSIC_PROVIDER=myprovider` and restart.
