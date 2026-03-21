# MuseStream

AI music generation and streaming server. Full reference: **SKILL.md**

## Check if running
```bash
curl -s http://localhost:5000/library | python3 -m json.tool | head -5
```

## Start server
```bash
source .env && ./restart_musestream.sh
```

## Required env vars (copy .env.example → .env and fill in)
- `SONAUTO_API_KEY` — get at https://sonauto.ai
- `MINIMAX_API_KEY` — optional, for context prompts
- `ANTHROPIC_API_KEY` — optional, fallback for context prompts

## Quick test
```bash
curl "http://localhost:5000/start?prompt=upbeat+indie+rock+morning+energy"
```
