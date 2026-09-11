---
name: hermes-stt-troubleshooting
description: "Fix Hermes STT: CUDA libs, language, model, GPU speed."
version: 0.1.0
author: y, Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [hermes, stt, transcription, voice, troubleshooting, cuda, gpu]
---

# Hermes Voice Transcription (STT) Troubleshooting

Fix Hermes speech-to-text: transcription failing, wrong language, poor accuracy, or slow speed. Covers the `stt.*` config, the missing-CUDA-library failure on Windows/NVIDIA, and GPU acceleration.

## When to Use

- Voice transcription always fails or errors out.
- Transcription is inaccurate for the user's language (e.g. Chinese garbled because `stt.language` is `en`).
- Transcription is slow on a machine that has an NVIDIA GPU.

Don't use for: TTS (text-to-speech) issues — that is a separate `tts.*` config.

## Prerequisites

- `faster-whisper` installed in the Hermes venv (`pip list | grep faster-whisper`).
- The `hermes` CLI and access to the Hermes home (`$HERMES_HOME`, usually `~/.hermes`).
- For GPU: an NVIDIA GPU and, on Windows, the `nvidia-cublas-cu12` pip package.

## Diagnosis

1. Read the STT config via `terminal`: `hermes config get stt`. Confirm `stt.enabled: true`, then note `stt.language`, `stt.local.model`, `stt.local.device`, `stt.local.compute_type`.
2. Find the real error in the log — the cause is in the log, not the symptom. Use `search_files` on `~/.hermes/logs/errors.log` for `transcri|stt|whisper|cublas`.
3. Match the error to a fix below.

## Common Failures and Fixes

### 1. `Library cublas64_12.dll is not found or cannot be loaded` (Windows + NVIDIA GPU)

Root cause: `stt.local.device: auto` selects CUDA, but the CUDA cuBLAS runtime DLL is absent, so encoding crashes at transcription time.

Proper fix (fast, uses GPU — ~0.3s/clip):

1. `hermes config set stt.local.device auto`
2. `hermes config set stt.local.compute_type float16`
3. Install cuBLAS into the Hermes venv: `<hermes-venv>/Scripts/pip.exe install nvidia-cublas-cu12`
4. Add the DLL dir to the user PATH (PowerShell), then restart Hermes:

```powershell
$d = '<venv>\Lib\site-packages\nvidia\cublas\bin'
$p = [Environment]::GetEnvironmentVariable('Path','User')
if ($p -notlike "*$d*") { [Environment]::SetEnvironmentVariable('Path', "$p;$d", 'User') }
```

Quick fix (CPU only, no download):

```
hermes config set stt.local.device cpu
hermes config set stt.local.compute_type int8
```

### 2. Wrong language (garbled output)

`stt.language` defaults to `en`. Set it to the user's language: `hermes config set stt.language zh` (or `ja`, `ko`, `es`, ...).

### 3. Poor accuracy for non-English speech

The `base` model is English-biased. Upgrade: `hermes config set stt.local.model medium`. `medium`/`large-v3` are multilingual; first load downloads ~1.5GB (medium) from Hugging Face.

### 4. Slow transcription

- On CPU: `medium` is slow; `small` is ~3x faster at slightly lower accuracy.
- On an NVIDIA GPU: install cuBLAS (fix #1 proper path) — GPU transcription is ~0.3s vs 5–10s on CPU.

## Verification

After any fix, restart Hermes (config, PATH, and the cached model are all read at process start), then confirm the model loads with the intended backend:

```
<hermes-venv>/Scripts/python.exe -c "from faster_whisper import WhisperModel; WhisperModel('medium', device='auto', compute_type='auto'); print('OK')"
```

Then transcribe a real voice message and check accuracy and speed.

## Pitfalls

- The CUDA→CPU fallback in `_load_local_whisper_model` only catches errors at model LOAD; the missing-cublas error fires at ENCODE time, so it is NOT auto-caught. Pin device/compute_type or install cuBLAS.
- `hermes config set` warns "not a recognized config key" for `stt.local.*` keys — the code reads them anyway; ignore the warning.
- The model is cached process-wide; config changes apply only after a restart.
- Whisper sometimes outputs traditional Chinese (天氣) for simplified-Chinese audio — cosmetic, not a failure.
- `beam_size` is hardcoded to 5 in `tools/transcription_local.py`; not configurable.
