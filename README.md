<div align="center">

![VoiceCortex banner](docs/assets/banner.svg)

# VoiceCortex

**Self-hosted, real-time phone AI assistant** — STT → LLM → TTS pipeline for Asterisk, FreeSWITCH, and SIP trunks.

[![License](https://img.shields.io/badge/license-MIT-%237c3aed?style=flat-square)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.11%2B-%233b82f6?style=flat-square&logo=python&logoColor=white)](https://www.python.org)
[![Release](https://img.shields.io/github/v/release/OneByJorah/VoiceCortex?style=flat-square&color=%2322c55e)](https://github.com/OneByJorah/VoiceCortex/releases)
[![Last Commit](https://img.shields.io/github/last-commit/OneByJorah/VoiceCortex?style=flat-square&color=%2306b6d4)](https://github.com/OneByJorah/VoiceCortex/commits)
[![Status](https://img.shields.io/badge/status-beta-%23f59e0b?style=flat-square)]()
[![Docker](https://img.shields.io/badge/docker-ready-%232496ED?style=flat-square&logo=docker&logoColor=white)](docker-compose.yml)

</div>

<p align="center">
  <img src="docs/screenshots/hero.png" alt="VoiceCortex Dashboard" width="90%">
</p>

## Features

- **Phone Integration** — Asterisk, FreeSWITCH, FreePBX/3CX via SIP trunking.
- **STT Pipeline** — Speech-to-text with faster-whisper and VAD filtering.
- **LLM Processing** — AI-powered conversation via Ollama, llama.cpp, Anthropic, or OpenAI.
- **TTS Output** — Natural speech synthesis with Piper TTS.
- **Real-Time Streaming** — Low-latency WebSocket transport with Voice Activity Detection.
- **Call Logging** — Record, transcribe, and archive all conversations.
- **Docker Ready** — Single `docker compose up` to deploy the full stack.
- **Fully Self-Hosted** — No external dependencies; run entirely on your own hardware.

## Architecture

```mermaid
flowchart LR
  Phone["📞 Phone Call"] --> PBX["FreeSWITCH / Asterisk / Twilio"]
  PBX --> VAD["VAD Voice Activity Detection"]
  VAD --> STT["STT faster-whisper"]
  STT --> LLM["LLM Ollama / llama.cpp / API"]
  LLM --> TTS["TTS Piper"]
  TTS --> Response["📞 Phone Response"]

  subgraph Legend [" "]
    dir((
    Audio stream
    Text stream
    ))
  end

  style STT fill:#3b82f6,color:#fff
  style LLM fill:#7c3aed,color:#fff
  style TTS fill:#06b6d4,color:#fff
```

### Pipeline Flow

| Layer | Technology | Function |
|-------|-----------|----------|
| **Transport** | WebSocket (mod_audio_fork) / HTTP | Streams PCM16 audio bidirectionally |
| **VAD** | silero-vad | Detects speech boundaries for natural turn-taking |
| **STT** | faster-whisper | Transcribes audio → text (configurable model size) |
| **LLM** | Ollama / llama.cpp / Anthropic / OpenAI | Generates conversational response |
| **TTS** | Piper | Synthesizes response → speech (resampled to codec rate) |

## Tech Stack

![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![faster-whisper](https://img.shields.io/badge/STT-faster--whisper-3B82F6?style=flat-square)
![Piper](https://img.shields.io/badge/TTS-Piper-06B6D4?style=flat-square)
![Ollama](https://img.shields.io/badge/LLM-Ollama-7C3AED?style=flat-square)
![llama.cpp](https://img.shields.io/badge/LLM-llama.cpp-7C3AED?style=flat-square)
![Anthropic](https://img.shields.io/badge/LLM-Anthropic-7C3AED?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)
![WebSocket](https://img.shields.io/badge/Transport-WebSocket-888888?style=flat-square)
![Twilio](https://img.shields.io/badge/Phone-Twilio-F22F46?style=flat-square)
![Asterisk](https://img.shields.io/badge/Phone-Asterisk-FF6600?style=flat-square)
![FreeSWITCH](https://img.shields.io/badge/Phone-FreeSWITCH-005C99?style=flat-square)

## Quick Start

```bash
git clone https://github.com/OneByJorah/VoiceCortex.git
cd VoiceCortex

cp .env.example .env
# Edit .env — set LLM_BACKEND, OLLAMA_MODEL (or API_KEY), etc.

docker compose up -d
```

### Local Development

```bash
pip install -r bot/requirements.txt
python3 bot/server.py
```

## Configuration

Copy `.env.example` to `.env`. Only the variables for your chosen `LLM_BACKEND` need to be set.

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_BACKEND` | `ollama` | LLM backend routing (ollama / llamacpp / api) |
| `OLLAMA_HOST` | `http://127.0.0.1:11434` | Ollama server URL |
| `OLLAMA_MODEL` | `llama3.1:8b-instruct-q4_K_M` | Ollama model name |
| `LLAMACPP_HOST` | `http://127.0.0.1:8080` | llama.cpp server URL |
| `API_PROVIDER` | `anthropic` | Cloud LLM provider (anthropic / openai) |
| `API_BASE_URL` | `https://api.anthropic.com/v1/messages` | Cloud LLM endpoint |
| `API_MODEL` | `claude-sonnet-4-6` | Cloud LLM model |
| `API_KEY` | — | Cloud LLM API key (keep server-side only) |
| `WHISPER_MODEL` | `small.en` | Whisper model variant (base.en / small.en / medium.en) |
| `WHISPER_DEVICE` | `cuda` | STT compute device (cuda / cpu) |
| `PIPER_VOICE` | `/piper-voices/en_US-amy-medium.onnx` | Piper voice model path |
| `BOT_WS_PORT` | `8765` | Bot WebSocket port (FreeSWITCH transport) |
| `SAMPLE_RATE` | `8000` (`server.py`) / `16000` (`server_asterisk.py`) | Audio sample rate |
| `VAD_BACKEND` | auto (silero if available) | Force amplitude fallback with `amplitude` |

Asterisk transport additionally supports: `ARI_HOST`, `ARI_USER`, `ARI_PASS`, `ARI_APP`, `EXTERNAL_MEDIA_HOST`, `EXTERNAL_MEDIA_PORT`.

## Project Structure

```
VoiceCortex/
├── bot/
│   ├── server.py              # FreeSWITCH WebSocket transport
│   ├── server_asterisk.py     # Asterisk HTTP transport
│   ├── hermes_brain.py        # STT → LLM → TTS core logic
│   ├── vad.py                 # Voice Activity Detection (silero-vad)
│   ├── persona.txt            # AI system prompt
│   └── requirements.txt
├── freeswitch/
│   └── conf/                  # FreeSWITCH configuration
├── docs/
│   └── assets/                # Brand assets, screenshots
├── dashboard/                 # Web UI landing page
├── docker-compose.yml         # Docker deployment
├── Dockerfile
├── .env.example               # Environment template
├── j1.yaml                    # Project metadata
├── LICENSE
└── README.md
```

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for community standards.

## Security

For security concerns, see [SECURITY.md](SECURITY.md). Report vulnerabilities to **info@jorahone.com** — do not use public issues.

## License

MIT © [Jhonattan L. Jimenez](https://github.com/OneByJorah)

---

<p align="center">
  Built with 🌴 by <a href="https://github.com/OneByJorah">OneByJorah</a> ·
  <a href="https://jorahone.com">jorahone.com</a> ·
  <a href="https://github.com/OneByJorah/VoiceCortex/issues">Issues</a>
</p>
