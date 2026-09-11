# hermes-stt-troubleshooting

一个 [Hermes Agent](https://hermes-agent.nousresearch.com) 技能（skill），用于修复语音转写（STT / speech-to-text）问题。

## 它能解决什么

- 转写总是失败（例如 Windows + NVIDIA 显卡上的 `Library cublas64_12.dll is not found or cannot be loaded`）
- 非英语语音转写乱码（`stt.language` 设错，默认是 `en`）
- 中文/多语言转写不准（`base` 模型偏英文）
- 转写太慢（CPU vs GPU 加速）

## 安装

把 `SKILL.md` 复制到 Hermes 的 skills 目录：

```bash
mkdir -p ~/.hermes/skills/autonomous-ai-agents/hermes-stt-troubleshooting
cp SKILL.md ~/.hermes/skills/autonomous-ai-agents/hermes-stt-troubleshooting/
```

Windows 下的路径：`%APPDATA%\hermes\skills\autonomous-ai-agents\hermes-stt-troubleshooting\`

然后重启 Hermes（或开一个新会话）即可加载该技能。

## 背景

这个技能是从一次真实的排障过程沉淀出来的：语音转写从「完全失败」→「不准」→「慢」，最终定位到三个隐藏根因（CUDA 库缺失、语言配置错误、模型选型不当），并一路修复到 GPU 加速。

## License

MIT
