# ClipGenius FFmpeg 版 · Claude Code 集成

在 Claude Code 中通过 Claude 4 Opus 模型 *(推荐)* 即可创建一个简洁的短片。

## 快速开始

```bash
git clone https://github.com/ClipGenius/ClaudeCode-ffmpeg ClipGenius-ClaudeCode-ffmpeg && cd ClipGenius-ClaudeCode-ffmpeg && rm README.md && claude
```

💡 移除 README.md 有助于保持 LLM 的稳定性。

然后，将素材文件也放入 `git clone` 下来的仓库 `ClipGenius-ClaudeCode-ffmpeg` 中，对 Claude 提出一个要求 (强烈建议要求包含**剪视频**、**视频类型**、**横屏/竖屏**、**你期望看到的效果**、**要求先设计剪辑方案**、**提供详细额外信息**)，即可开始剪辑！

提出的要求应当包含: 

- 要求先设计剪辑方案
- 视频类型
- 横屏/竖屏
- 你期望看到的效果
- 提供详细额外信息

Claude 会将输出的内容放入 `./output` 文件夹，并在结束剪辑时删除临时文件 `./.tmp`。如果您还有需要调整的内容，切勿同意删除该文件夹！否则剪辑过程将从头开始。

## 环境要求

- 类 Unix 系统，例如 macOS、Linux、WSL (适用于 Linux 的 Windows 子系统) 等
- 带有 FFmpeg、Python 以及 Claude Code 的软件环境 (请确保它们均位于 PATH 中)
- 可以流畅运行 FFmpeg 的硬件环境

## 存在缺陷

⚠️ 这个项目目前仍然处于 Alpha 阶段，生成的视频质量不稳定，需要未来长期的开发才可以逐渐解决。因此，我们目前还不推荐将其用于生产用途。

## 支持我们

您可以: 

- 给我们点个 Star
- 打开一个 [Issue](https://github.com/ClipGenius/ClaudeCode-ffmpeg/issues)
- 发起一个 [Pull request](https://github.com/ClipGenius/ClaudeCode-ffmpeg/pulls)
- 捐赠我们，让我们能够继续使用 Claude 加速开发此项目。(如果您想要捐赠，请打开一个 Issue)

## 关于 ClipGenius

ClipGenius 是一个由 [@xiaoheiCat](https://github.com/xiaoheiCat) 发起的、使用 LLM 来剪辑视频的以提示词工程为主的项目。

截至目前为止，该项目 100% 由 Claude 开发。
