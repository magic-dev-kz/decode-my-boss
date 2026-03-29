# 🔍 Decode My Boss

**What they say vs what they mean.**

Paste a message from your boss — get the real meaning, red flags, toxicity score, and an actionable playbook. AI-powered corporate message decoder.

---

## Features

- **Decoded Translation** — Every phrase analyzed: what was said vs what was meant, with tone tags (passive-aggressive, power move, genuine, etc.)
- **Red Flags** — Detects patterns: passive aggression, gaslighting, blame shifting, weaponized positivity, and more
- **Toxicity Score (1-10)** — Gradient bar with 5-category breakdown
- **Playbook** — 2-3 specific, actionable steps for what to do next
- **Share Card** — PNG card with score + flags (original text NOT included — privacy first)
- **BYOK** (Bring Your Own Key) — works with OpenAI, Google Gemini, or Anthropic
- **Single HTML file** — no backend, no account, no tracking

## How It Works

1. Paste a message from your boss / HR / colleague
2. Select context (Corporate Email, Slack, Teams, HR Letter, Performance Review)
3. Click "Decode"
4. Get the translation, red flags, toxicity score, and playbook

## Privacy

- **Your messages never touch our servers** — text goes directly from your browser to the AI provider
- **API key stays in localStorage** — we never see it
- **Share card contains NO original text** — only score, flags, and verdict
- **No analytics, no cookies, no tracking**

## Supported AI Providers

| Provider | Model | Cost per decode | Notes |
|----------|-------|----------------|-------|
| OpenAI | gpt-4o-mini | ~$0.01-0.03 | Best quality |
| Google Gemini | 2.0 Flash | Free (free tier) | Great for testing |
| Anthropic | Claude Sonnet | ~$0.01-0.03 | CORS limitations in browser |

## Red Flag Types Detected

- Passive Aggression
- Gaslighting
- Blame Shifting
- Power Move
- Exclusion Signal
- False Urgency
- Weaponized Positivity
- Plausible Deniability
- Scope Creep Trap
- Credit Theft

## Running Locally

```bash
# Simple HTTP server (needed for PWA)
python3 -m http.server 8000
# Visit http://localhost:8000
```

## License

MIT — free for personal and commercial use.

---

Made by [OpenClaw](https://github.com/openclaw). We make $0 from this.
