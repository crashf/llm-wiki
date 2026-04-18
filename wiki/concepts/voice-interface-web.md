# Voice Interface Web

Real-time voice interface for OpenClaw agents enabling speech-to-text → agent → text-to-speech conversations.

## Overview

Browser-based voice interface that captures microphone audio, streams it to Deepgram for transcription, sends text to OpenClaw agent, and speaks back responses via ElevenLabs TTS.

## Architecture

```
Browser (React + Web Audio API)
    ↓ (16kHz PCM audio stream)
Deepgram (STT - Nova-3)
    ↓ (transcript text)
Voice Web Server (Node.js/WebSocket)
    ↓ (HTTP/WebSocket)
OpenClaw Gateway (10.255.245.167:3578)
    ↓ (agent response)
Voice Web Server
    ↓ (text)
ElevenLabs/VoiceForge (TTS)
    ↓ (audio)
Browser playback
```

## Components

| Component | Technology | Purpose |
|-----------|------------|---------|
| Frontend | Next.js 16 + React 19 + Tailwind | UI, mic capture, audio playback |
| STT | Deepgram Nova-3 | Real-time speech-to-text |
| TTS | ElevenLabs API / VoiceForge | Text-to-speech synthesis |
| Transport | WebSocket (Socket.io) | Bidirectional streaming |
| Hosting | Docker on 10.255.61.20 | Co-located with bot |
| Domain | voice.pund-it.ca | Via Caddy reverse proxy |

## Status

**Phase: MVP In Planning**
- GitHub repo created: https://github.com/crashf/voice-interface-web
- Project structure initialized
- Docker setup ready
- Infrastructure confirmed on 10.255.61.20

## Key Files

- `projects/voice-interface-web/PLAN.md` — Full implementation plan
- `projects/voice-interface-web/docker-compose.yml` — Deployment config
- `projects/voice-interface-web/Dockerfile` — Container build

## Related

- ElevenLabs TTS already integrated with VoiceForge (10.255.61.37)
- OpenClaw gateway WebSocket API at `ws://<bot>:3578/v1/chat`
- Cloudflare reference: `@cloudflare/voice` experimental SDK pattern

## Tags

voice, speech-to-text, text-to-speech, deepgram, elevenlabs, openclaw, websocket, nextjs, docker
