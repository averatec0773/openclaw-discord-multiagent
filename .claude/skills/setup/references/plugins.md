# fish-audio — TTS Plugin Setup

Plugin: `@conan-scott/openclaw-fish-audio` v1.0.0
Requires: `ffmpeg` in image (already in `Dockerfile.custom`)

## Install

```bash
docker compose exec openclaw-gateway openclaw plugins install @conan-scott/openclaw-fish-audio
ssh openclaw "cd ~/openclaw && docker compose restart"
```

## Config (add to openclaw.json)

```json
"messages": {
  "tts": {
    "provider": "fish-audio",
    "providers": {
      "fish-audio": {
        "voiceId": "bf86e0ce9e1b47dd97efdad184608a67",
        "model": "s2-pro",
        "speed": 1.0
      }
    }
  }
},
"plugins": {
  "entries": {
    "fish-audio": {
      "enabled": true,
      "config": {
        "voiceId": "bf86e0ce9e1b47dd97efdad184608a67"
      }
    }
  }
}
```

Also add `FISH_AUDIO_API_KEY` to `env` block in `openclaw.json` (value in `_local/.env`).

## auto Mode Options

| Value | Behavior |
|---|---|
| `"off"` | Disabled; user can enable per-session with `/tts on` |
| `"always"` | Every reply converted to audio |
| `"tagged"` | Audio only when reply contains `[[tts:on]]` (current setup) |
| `"inbound"` | Audio only after user sends a voice message |

Current setup uses `"tagged"` as a per-agent workaround — see `conventions/server.md`.
