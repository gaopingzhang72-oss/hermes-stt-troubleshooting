# hermes-stt-troubleshooting

A [Hermes Agent](https://hermes-agent.nousresearch.com) skill for fixing voice transcription (STT / speech-to-text) issues.

## What it fixes

- Transcription always failing — e.g. `Library cublas64_12.dll is not found or cannot be loaded` on Windows + NVIDIA GPU
- Garbled output for non-English speech (wrong `stt.language`, which defaults to `en`)
- Poor accuracy for Chinese / multilingual speech (the `base` model is English-biased)
- Slow transcription (CPU vs GPU)

## Example: the missing-CUDA case

**Symptom** — every transcription fails. The error log (`~/.hermes/logs/errors.log`) shows:

```
Local transcription failed: Library cublas64_12.dll is not found or cannot be loaded
```

**Root cause** — `stt.local.device: auto` selects CUDA because an NVIDIA GPU is present, but the CUDA cuBLAS runtime DLL is missing, so encoding crashes at transcription time.

**Fix (use the GPU — ~0.3s per clip):**

```bash
hermes config set stt.local.device auto
hermes config set stt.local.compute_type float16
# install the missing DLL into the Hermes venv (Windows)
<hermes-venv>/Scripts/pip.exe install nvidia-cublas-cu12
# add the DLL dir to the user PATH, then restart Hermes
```

**Result** — transcription goes from failing (or 5–10s on CPU) to ~0.3s on an RTX 5060.

## Install

```bash
mkdir -p ~/.hermes/skills/autonomous-ai-agents/hermes-stt-troubleshooting
cp SKILL.md ~/.hermes/skills/autonomous-ai-agents/hermes-stt-troubleshooting/
```

Windows: `%APPDATA%\hermes\skills\autonomous-ai-agents\hermes-stt-troubleshooting\`

Restart Hermes (or start a new session) to load the skill.

## Background

This skill was distilled from a real debugging session: voice transcription went from *totally broken* → *inaccurate* → *slow*, and the root causes turned out to be three hidden issues — a missing CUDA library, a wrong language setting, and an under-sized model — fixed all the way to GPU acceleration.

## 中文说明

这是一个 [Hermes Agent](https://hermes-agent.nousresearch.com) 技能，用于修复语音转写（STT）问题：转写失败（Windows + NVIDIA 显卡上缺 `cublas64_12.dll`）、非英语语音乱码（`stt.language` 默认 `en`）、中文/多语言不准（`base` 模型偏英文）、以及转写太慢（CPU vs GPU）。

安装：把 `SKILL.md` 复制到 `~/.hermes/skills/autonomous-ai-agents/hermes-stt-troubleshooting/`（Windows 为 `%APPDATA%\hermes\skills\...`），然后重启 Hermes。

## License

MIT
