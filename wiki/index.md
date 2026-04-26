# Wiki Index

## Entities

_(Specific things: people, companies, products, tools)_

_No entities yet — add sources to populate._

## Concepts

_(Ideas, techniques, patterns, methodologies)_

| Page | Summary | Tags | Updated |
|------|---------|------|---------|
| [anker-solix-f3000-mqtt](./concepts/anker-solix-f3000-mqtt.md) | Anker SOLIX F3000 MQTT protocol — field mapping, decoding bugs, Grafana dashboard, 39 confirmed live fields | anker, f3000, mqtt, iot, grafana, power-station | 2026-04-26 |
|------|---------|------|---------|
| [camping-2026-trip](./concepts/camping-2026-trip.md) | Summer 2026 RV camping trip — 91 days, 4 provinces, 50 diesel stops, fully booked | camping, rv, travel, diesel, planning | 2026-04-19 |
| [anzen-egress-portal](./concepts/anzen-egress-portal.md) | Zero-CLI deployment portal for Anzen Egress WireGuard tunnels — 3-phase model, tech stack, DB schema, integrations | anzen-egress, mikrotik, wireguard, nextjs, postgresql, portal | 2026-04-23 |
| [anzen-egress-wireguard](./concepts/anzen-egress-wireguard.md) | WireGuard architecture — production/staging split, X25519 key gen, transit networks, proxy-ARP /30 handoff, tunnel lifecycle | anzen-egress, wireguard, x25519, mikrotik, proxy-arp | 2026-04-23 |
| [anzen-egress-config-generator](./concepts/anzen-egress-config-generator.md) | Dynamic RouterOS config generation — passthrough vs standard router, WAN/LAN/WiFi/firewall, no templates | anzen-egress, mikrotik, routeros, config-generation, wifi | 2026-04-23 |
| [anzen-egress-mikrowizard](./concepts/anzen-egress-mikrowizard.md) | Integration of MikroWizard features into Anzen Egress portal — health, firmware, backups, syslog, diagnostics | anzen-egress, mikrowizard, mikrotik, wireguard, device-management | 2026-04-19 |
| [voice-interface-web](./concepts/voice-interface-web.md) | Real-time voice interface for OpenClaw agents — Deepgram STT, ElevenLabs TTS, WebSocket | voice, stt, tts, deepgram, elevenlabs, openclaw, websocket | 2026-04-16 |
| [github-credentials](./concepts/github-credentials.md) | GitHub PAT and authentication patterns for crashf repositories | github, credentials, pat, authentication | 2026-04-10 |
| [cpanel-whm-skill](./concepts/cpanel-whm-skill.md) | cPanel/WHM API skill for account, email, database, DNS, SSL, and domain management | cpanel, whm, api, skill, infrastructure | 2026-04-10 |
| [google-chat-jwt-verification-patch](./concepts/google-chat-jwt-verification-patch.md) | Emergency bypass for `verifyGoogleChatRequest` — file hash changes every build | openclaw, googlechat, patch, security, fleet | 2026-04-22 |
| [openclaw-421-upgrade](./concepts/openclaw-421-upgrade.md) | Full upgrade procedure for .4.8 → .4.21 — JWT patch + plugins.entries fix | openclaw, fleet-management, upgrade, googlechat | 2026-04-22 |
| [llm-wiki-pattern](./concepts/llm-wiki-pattern.md) | Karpathy's pattern for LLM-maintained personal knowledge bases | knowledge-management, llms, automation | 2026-04-07 |
| [club-sixty-six](./concepts/club-sixty-six.md) | Club Sixty Six fitness subscription platform — Stripe, Next.js, PM2 on Ubuntu | clubsixty, stripe, nextjs, postgresql, pm2, vpn | 2026-04-10 |
| [stripe-automatic-tax-checkout](./concepts/stripe-automatic-tax-checkout.md) | Pattern for Stripe automatic tax with customer address collection | stripe, checkout, tax, payments, patterns | 2026-04-10 |
| [openclaw-fleet-management](./concepts/openclaw-fleet-management.md) | Managing distributed OpenClaw agents via central dashboard with SSH proxy | openclaw, fleet-management, distributed-agents, ssh-proxy | 2026-04-08 |
| [veeam-restore-automation](./concepts/veeam-restore-automation.md) | Fully automated VBR restore testing — Instant VM Recovery + health checks + auto-cleanup | veeam, backup, automation, powershell, rest-api, testing | 2026-04-08 |
| [openclaw-bot-debugging](./concepts/openclaw-bot-debugging.md) | Patterns for diagnosing and fixing unresponsive OpenClaw gateway instances | openclaw, debugging, troubleshooting, configuration, ssh | 2026-04-08 |
| [openclaw-421-ollama-provider-fix](./concepts/openclaw-421-ollama-provider-fix.md) | OpenClaw .4.21 upgrade causes "No API provider registered for api: ollama" in lossless-claw — fix is changing `api` from `"ollama"` to `"openai-completions"` in models.providers.ollama-local | openclaw, upgrade, .4.21, ollama, lossless-claw, bug-fix, pi-ai | 2026-04-22 |

## Summaries

_(Summary pages for ingested sources)_

| Page | Summary | Tags | Updated |
|------|---------|------|---------|
| [ssh-proxy-patterns](./concepts/ssh-proxy-patterns.md) | SSH fleet key pattern for centralized access to distributed infrastructure | ssh, fleet-key, proxy, security | 2026-04-07 |
| [anzen-egress-mvp](./summaries/anzen-egress-mvp.md) | MVP status — what's built, what's not, key decisions, lessons learned | anzen-egress, mvp, wireguard, mikrotik, proxy-arp | 2026-04-23 |

## Synthesis

_(High-level pages connecting topics)_

_No synthesis pages yet._

---

**Stats:** 2 sources | 11 concept pages | 2 summaries | Last updated: 2026-04-26
