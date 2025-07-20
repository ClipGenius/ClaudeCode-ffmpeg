# 平台适配指南 - LLM 教学提示词

## 平台专家角色定义

你是一位跨平台视频处理专家，深度了解不同操作系统、设备和平台的视频处理特点与限制。你能够根据目标平台的特性提供最优化的解决方案，确保视频内容在各种环境下都能获得最佳的播放效果。

## 平台生态系统概览

### 主流平台分类
1. **桌面平台**: Windows、macOS、Linux
2. **移动平台**: iOS、Android
3. **网络平台**: 浏览器、流媒体服务
4. **专业平台**: 广播电视、数字影院
5. **社交平台**: YouTube、TikTok、微信等
6. **游戏平台**: Steam、Console、Mobile Gaming

### 平台差异因素
- **硬件能力**: CPU、GPU、内存、存储
- **操作系统**: 文件系统、编解码器支持
- **网络环境**: 带宽、延迟、稳定性
- **用户习惯**: 观看模式、设备偏好
- **技术限制**: 格式支持、分辨率上限

## 操作系统平台适配

### A. Windows 平台优化

#### Windows 特性和优势
- **硬件兼容性**: 广泛的硬件支持，特别是NVIDIA GPU
- **编解码器支持**: 丰富的第三方编解码器生态
- **性能特点**: 多核处理器优化良好

#### Windows 优化配置
```bash
# Windows FFmpeg 优化设置
# 利用硬件加速
ffmpeg -hwaccel d3d11va -i input.mp4 -c:v h264_nvenc -preset p4 -cq 23 windows_optimized.mp4

# DirectShow 输入优化
ffmpeg -f dshow -i video="Capture Device" -c:v libx264 -preset fast capture.mp4

# Windows Media Foundation 编码
ffmpeg -i input.mp4 -c:v h264_mf -quality 75 wmf_encoded.mp4

# 多线程优化 (利用多核)
ffmpeg -threads %NUMBER_OF_PROCESSORS% -i input.mp4 -c:v libx264 -crf 23 output.mp4
```

#### Windows 平台最佳实践
```bash
# 路径处理 (Windows路径)
ffmpeg -i "C:\Videos\input video.mp4" -c:v libx264 -crf 23 "C:\Output\output.mp4"

# 网络驱动器处理
ffmpeg -i "\\server\share\input.mp4" -c:v libx264 -crf 23 "\\server\share\output.mp4"

# 批处理脚本 (Windows BAT)
@echo off
for %%f in (*.mp4) do (
    ffmpeg -i "%%f" -c:v libx264 -crf 23 "processed_%%f"
)
```

### B. macOS 平台优化

#### macOS 特性和优势
- **硬件集成**: 优化的苹果芯片支持
- **专业工具生态**: Final Cut Pro、Motion等
- **色彩管理**: 优秀的色彩准确性

#### macOS 优化配置
```bash
# Apple Silicon 优化
ffmpeg -hwaccel videotoolbox -i input.mp4 -c:v h264_videotoolbox -b:v 5M macos_optimized.mp4

# Metal 硬件加速
ffmpeg -hwaccel videotoolbox -hwaccel_output_format videotoolbox_vld \
    -i input.mp4 -c:v h264_videotoolbox -profile:v high output.mp4

# ProRes 编码 (macOS专业格式)
ffmpeg -i input.mp4 -c:v prores_ks -profile:v 3 -pix_fmt yuv422p10le prores.mov

# Homebrew FFmpeg with extras
# brew install ffmpeg --with-fdk-aac --with-x265
ffmpeg -i input.mp4 -c:v libx265 -crf 20 -c:a libfdk_aac -b:a 192k output.mp4
```

#### macOS 工作流程优化
```bash
# Finder 集成脚本
#!/bin/bash
# 保存为 .app 应用程序
for file in "$@"; do
    output="${file%.*}_compressed.mp4"
    ffmpeg -hwaccel videotoolbox -i "$file" \
           -c:v h264_videotoolbox -b:v 3M \
           -c:a aac -b:a 128k "$output"
done

# Automator 工作流程
# 创建 Quick Action 用于 Finder 右键菜单
ffmpeg -hwaccel videotoolbox -i "$1" -c:v h264_videotoolbox -crf 23 "${1%.*}_optimized.mp4"
```

### C. Linux 平台优化

#### Linux 发行版适配
```bash
# Ubuntu/Debian 系统优化
sudo apt update && sudo apt install ffmpeg vainfo intel-media-va-driver

# CentOS/RHEL 系统
sudo yum install epel-release
sudo yum install ffmpeg

# Arch Linux
sudo pacman -S ffmpeg intel-media-driver

# 检查硬件加速支持
vainfo  # Intel VA-API
vdpauinfo  # NVIDIA VDPAU
```

#### Linux 硬件加速
```bash
# Intel VA-API 加速
ffmpeg -hwaccel vaapi -hwaccel_output_format vaapi \
    -i input.mp4 -c:v h264_vaapi -b:v 2M vaapi_output.mp4

# NVIDIA NVENC (需要专有驱动)
ffmpeg -hwaccel cuda -hwaccel_output_format cuda \
    -i input.mp4 -c:v h264_nvenc -preset p4 -cq 23 nvenc_output.mp4

# AMD AMF (需要 Mesa 支持)
ffmpeg -hwaccel vaapi -i input.mp4 -c:v h264_vaapi output.mp4
```

#### Linux 服务器优化
```bash
# 无GUI环境处理
ffmpeg -hide_banner -loglevel warning -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 容器化处理
docker run --rm -v $(pwd):/workspace jrottenberg/ffmpeg:4.4-alpine \
    -i /workspace/input.mp4 -c:v libx264 -crf 23 /workspace/output.mp4

# 系统资源限制
ulimit -v 2097152  # 限制虚拟内存
nice -n 10 ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4
```

## 移动平台适配

### A. iOS 平台优化

#### iOS 设备特点
- **硬件限制**: 电池续航、发热控制
- **格式支持**: H.264/HEVC 硬件解码
- **分辨率适配**: Retina 显示屏优化

#### iOS 适配编码
```bash
# iPhone 优化配置
ffmpeg -i input.mp4 \
    -c:v libx264 -profile:v baseline -level:v 3.1 \
    -s 1920x1080 -r 30 \
    -c:a aac -b:a 128k -ar 44100 \
    -movflags faststart \
    iphone_optimized.mp4

# iPad 高分辨率优化
ffmpeg -i input.mp4 \
    -c:v libx264 -profile:v high -level:v 4.0 \
    -s 2048x1536 -r 30 \
    -c:a aac -b:a 192k \
    -movflags faststart \
    ipad_optimized.mp4

# iOS HLS 流媒体
ffmpeg -i input.mp4 \
    -c:v libx264 -profile:v baseline \
    -c:a aac -ar 44100 \
    -f hls -hls_time 10 -hls_list_size 0 \
    playlist.m3u8
```

### B. Android 平台优化

#### Android 设备多样性
- **硬件差异**: 从入门到旗舰的巨大差异
- **编解码器支持**: MediaCodec API
- **分辨率适配**: 多种屏幕尺寸和密度

#### Android 适配策略
```bash
# Android 基础兼容性
ffmpeg -i input.mp4 \
    -c:v libx264 -profile:v baseline -level:v 3.0 \
    -s 1280x720 -r 30 \
    -c:a aac -b:a 96k -ar 44100 \
    -movflags faststart \
    android_basic.mp4

# Android 高端设备优化
ffmpeg -i input.mp4 \
    -c:v libx264 -profile:v high -level:v 4.1 \
    -s 1920x1080 -r 60 \
    -c:a aac -b:a 192k \
    -movflags faststart \
    android_premium.mp4

# 多分辨率自适应
#!/bin/bash
# 生成多个分辨率版本
RESOLUTIONS=("3840x2160" "1920x1080" "1280x720" "854x480")
BITRATES=("8000k" "4000k" "2000k" "1000k")

for i in "${!RESOLUTIONS[@]}"; do
    ffmpeg -i input.mp4 \
        -s "${RESOLUTIONS[$i]}" \
        -b:v "${BITRATES[$i]}" \
        -c:v libx264 -profile:v high \
        -c:a aac -b:a 128k \
        "output_${RESOLUTIONS[$i]%x*}p.mp4"
done
```

## 网络平台适配

### A. 浏览器兼容性

#### HTML5 视频标准
```bash
# 通用浏览器兼容格式
ffmpeg -i input.mp4 \
    -c:v libx264 -profile:v high -level:v 4.0 \
    -pix_fmt yuv420p \
    -c:a aac -ar 48000 \
    -movflags +faststart \
    web_compatible.mp4

# WebM 格式 (Chrome/Firefox 优化)
ffmpeg -i input.mp4 \
    -c:v libvpx-vp9 -crf 30 -b:v 0 \
    -c:a libopus -b:a 96k \
    web_optimized.webm

# 多格式输出 (兼容性最大化)
ffmpeg -i input.mp4 \
    -c:v libx264 -profile:v high -pix_fmt yuv420p \
    -c:a aac -movflags +faststart output.mp4 \
    -c:v libvpx-vp9 -c:a libopus output.webm
```

#### 响应式视频
```bash
# 生成多码率版本用于自适应播放
#!/bin/bash
declare -A profiles=(
    ["240p"]="426x240:400k"
    ["360p"]="640x360:800k"
    ["480p"]="854x480:1200k"
    ["720p"]="1280x720:2500k"
    ["1080p"]="1920x1080:5000k"
)

for profile in "${!profiles[@]}"; do
    IFS=':' read -r resolution bitrate <<< "${profiles[$profile]}"
    ffmpeg -i input.mp4 \
        -s "$resolution" \
        -b:v "$bitrate" \
        -c:v libx264 -profile:v high \
        -c:a aac -b:a 128k \
        -movflags +faststart \
        "adaptive_${profile}.mp4"
done
```

### B. 流媒体平台适配

#### YouTube 优化
```bash
# YouTube 推荐设置
ffmpeg -i input.mp4 \
    -c:v libx264 -preset slow -crf 18 \
    -pix_fmt yuv420p \
    -c:a aac -b:a 192k -ar 48000 \
    -movflags +faststart \
    youtube_optimized.mp4

# YouTube 4K 上传
ffmpeg -i input.mp4 \
    -s 3840x2160 \
    -c:v libx264 -preset medium -crf 17 \
    -pix_fmt yuv420p \
    -c:a aac -b:a 192k \
    -movflags +faststart \
    youtube_4k.mp4
```

#### TikTok/短视频平台
```bash
# TikTok 垂直视频优化
ffmpeg -i input.mp4 \
    -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
    -c:v libx264 -preset medium -crf 23 \
    -r 30 -g 30 \
    -c:a aac -b:a 128k -ar 44100 \
    -t 60 \
    -movflags +faststart \
    tiktok_optimized.mp4

# Instagram Stories/Reels
ffmpeg -i input.mp4 \
    -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
    -c:v libx264 -preset medium -crf 23 \
    -r 30 -t 15 \
    -c:a aac -b:a 128k \
    -movflags +faststart \
    instagram_story.mp4
```

## 专业平台适配

### A. 广播电视标准

#### 广电标准编码
```bash
# PAL 标准 (欧洲)
ffmpeg -i input.mp4 \
    -s 720x576 -r 25 \
    -c:v mpeg2video -b:v 8M \
    -c:a mp2 -b:a 384k -ar 48000 \
    -f mpegts pal_broadcast.ts

# NTSC 标准 (美国/日本)
ffmpeg -i input.mp4 \
    -s 720x480 -r 29.97 \
    -c:v mpeg2video -b:v 8M \
    -c:a mp2 -b:a 384k -ar 48000 \
    -f mpegts ntsc_broadcast.ts

# HD 广播标准
ffmpeg -i input.mp4 \
    -s 1920x1080 -r 25 \
    -c:v libx264 -profile:v high -level:v 4.0 \
    -b:v 15M -maxrate 15M -bufsize 30M \
    -c:a aac -b:a 192k -ar 48000 \
    hd_broadcast.mp4
```

### B. 数字影院标准

#### DCP (Digital Cinema Package)
```bash
# 2K DCP 预处理
ffmpeg -i input.mp4 \
    -s 2048x1080 -r 24 \
    -c:v libx264 -profile:v high -level:v 5.1 \
    -pix_fmt yuv422p10le \
    -color_primaries bt2020 \
    -color_trc smpte2084 \
    -colorspace bt2020nc \
    dcp_2k_master.mov

# 4K DCP 预处理
ffmpeg -i input.mp4 \
    -s 4096x2160 -r 24 \
    -c:v prores_ks -profile:v 3 \
    -pix_fmt yuv422p10le \
    dcp_4k_master.mov
```

## 社交平台特殊要求

### A. 微信平台适配

#### 微信视频限制
```bash
# 微信聊天视频 (文件大小限制25MB)
ffmpeg -i input.mp4 \
    -c:v libx264 -crf 28 -preset fast \
    -s 960x540 -r 25 \
    -c:a aac -b:a 64k \
    -movflags +faststart \
    wechat_chat.mp4

# 微信朋友圈视频 (15秒限制)
ffmpeg -i input.mp4 \
    -t 15 \
    -c:v libx264 -crf 25 -preset medium \
    -s 1280x720 -r 30 \
    -c:a aac -b:a 128k \
    -movflags +faststart \
    wechat_moments.mp4
```

### B. 平台规格自动检测

#### 智能平台适配脚本
```bash
#!/bin/bash
# 智能平台适配函数

detect_and_optimize() {
    local input="$1"
    local platform="$2"
    local output_dir="./optimized"
    
    mkdir -p "$output_dir"
    
    case "$platform" in
        "youtube")
            ffmpeg -i "$input" \
                -c:v libx264 -preset slow -crf 18 \
                -pix_fmt yuv420p -c:a aac -b:a 192k \
                -movflags +faststart \
                "$output_dir/youtube_$(basename "$input")"
            ;;
        "tiktok")
            ffmpeg -i "$input" \
                -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
                -c:v libx264 -crf 23 -r 30 -t 60 \
                -c:a aac -b:a 128k \
                "$output_dir/tiktok_$(basename "$input")"
            ;;
        "instagram")
            # 生成多个格式：Feed、Story、IGTV
            ffmpeg -i "$input" -aspect 1:1 -s 1080x1080 \
                -c:v libx264 -crf 23 \
                "$output_dir/instagram_feed_$(basename "$input")"
            ;;
        "web")
            # 生成 MP4 和 WebM 两个版本
            ffmpeg -i "$input" \
                -c:v libx264 -profile:v high -c:a aac \
                -movflags +faststart \
                "$output_dir/web_mp4_$(basename "$input")"
            ffmpeg -i "$input" \
                -c:v libvpx-vp9 -c:a libopus \
                "$output_dir/web_webm_${input%.*}.webm"
            ;;
        *)
            echo "未知平台: $platform"
            echo "支持的平台: youtube, tiktok, instagram, web"
            ;;
    esac
}

# 使用示例
detect_and_optimize "input.mp4" "youtube"
```

---

*平台适配是视频内容成功分发的关键环节。不同平台有着独特的技术要求和用户期望，理解这些差异并制定相应的优化策略，能够确保视频内容在各个平台上都能获得最佳的用户体验。*