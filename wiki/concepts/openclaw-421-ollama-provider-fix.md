# OpenClaw .4.21 Upgrade Bug: Ollama Provider "No API provider registered"

## Symptom

After upgrading a bot to OpenClaw `.4.21`, the bot responds in Google Chat/Discord but returns errors like:

```
No API provider registered for api: ollama
```

This appears in the bot's response when lossless-claw attempts to make an LCM call (compaction, `lcm_grep`, `lcm_expand`, etc.).

## Root Cause

- **Library:** `lossless-claw` uses the `pi-ai` Node.js library for AI provider calls
- **pi-ai API registry:** Only registers OpenAI-compatible API types: `openai`, `openai-completions`, `anthropic`, `bedrock`, `gemini`, `azure`, etc.
- **Provider config conflict:** The `ollama-local` provider in `models.providers` has `api: "ollama"` set
- **pi-ai resolution order:** When calling `getApiProvider(api)`, pi-ai looks up by the `api` string. `"ollama"` is not in its registry → throws error

The gateway itself handles the Ollama provider correctly (via OpenClaw's own stock Ollama plugin), but lossless-claw's pi-ai dependency fails independently.

## Fix

Change the `api` field in the `ollama-local` provider config from `"ollama"` to `"openai-completions"`:

```bash
jq '.models.providers["ollama-local"].api = "openai-completions"' /root/.openclaw/openclaw.json | sudo tee /root/.openclaw/openclaw.json.new && sudo mv /root/.openclaw/openclaw.json.new /root/.openclaw/openclaw.json
sudo systemctl restart openclaw-gateway
```

### Why this works

| Component | Reads `api` from | How it interprets value |
|-----------|-----------------|------------------------|
| OpenClaw stock Ollama plugin | `models.providers[].api` | Uses it as the transport label; value is arbitrary |
| lossless-claw / pi-ai | `models.providers[].api` | Looks up in its own API registry; needs OpenAI-compatible value |

Changing to `"openai-completions"` makes pi-ai correctly use the OpenAI-compatible API type while OpenClaw's Ollama plugin continues to work unchanged.

## Affected Scope

- **Version:** OpenClaw `.4.21` (and likely forward)
- **Conditions:** Bot has `ollama-local` provider configured AND lossless-claw plugin enabled
- **Not affected:** Bots without lossless-claw, or bots using other provider types (Anthropic, OpenAI direct, etc.)

## Related Config

```json
{
  "models": {
    "providers": {
      "ollama-local": {
        "baseUrl": "http://localhost:11434",
        "api": "openai-completions",  // was "ollama" — change to this
        "models": [...]
      }
    }
  }
}
```

## Notes

- This is a config-only fix, no OpenClaw patch required
- The `.4.21` upgrade wizard may have reset or regenerated provider configs, introducing this mismatch
- Apply to all fleet bots when upgrading to `.4.21` if they use `ollama-local` with lossless-claw