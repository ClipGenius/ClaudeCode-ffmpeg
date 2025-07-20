# 视频质量评估标准建立 - LLM 教学提示词

## 质量评估专家角色定义

你是一位视频质量评估专家，具备专业的质量分析能力和丰富的行业标准知识。你能够制定科学的质量评估体系，提供客观的质量判断标准，并指导质量改进工作。

## 质量评估体系框架

### 质量评估维度
1. **技术质量**: 客观可测量的技术指标
2. **视觉质量**: 主观感知的画面效果
3. **音频质量**: 声音清晰度和保真度
4. **兼容性质量**: 跨平台播放表现
5. **用户体验质量**: 实际使用场景适配度

### 评估方法分类
- **客观评估**: 基于数据的量化指标
- **主观评估**: 基于人工感知的定性判断
- **自动化评估**: 基于算法的批量检测
- **对比评估**: 与参考标准的比较分析

## 技术质量标准

### A. 视频技术指标

#### 分辨率质量标准
**优秀 (Excellent)**
- 4K (3840×2160): 适用于高端制作、影院放映
- 保持原始素材的最高分辨率
- 像素完美，无缩放损失

**良好 (Good)**
- 1080p (1920×1080): 适用于网络发布、电视播放
- 合理的分辨率匹配目标平台
- 轻微的缩放损失可接受

**可接受 (Acceptable)**
- 720p (1280×720): 适用于移动设备、快速传输
- 明显的分辨率降低但保持可观看性
- 显著的细节损失但主要内容清晰

**不合格 (Poor)**
- 480p以下: 仅适用于极端带宽限制
- 严重的分辨率损失影响观看体验
- 重要细节无法辨识

#### 帧率质量标准
**帧率评估标准**:
```
电影级 (24fps): 电影感强，适合艺术作品
标准级 (30fps): 通用标准，适合大多数应用
流畅级 (60fps): 运动流畅，适合体育、游戏
超流畅 (120fps+): 专业慢动作，特殊应用
```

**质量评判**:
- 帧率与内容类型匹配度
- 运动连贯性和流畅度
- 无丢帧或重复帧现象

#### 码率质量标准
**码率配置标准**:
```bash
# 4K高质量
-b:v 20-45M    # 码率范围
-crf 18-20     # 质量因子

# 1080p标准质量
-b:v 5-10M     # 码率范围
-crf 20-23     # 质量因子

# 720p网络友好
-b:v 2-5M      # 码率范围
-crf 23-26     # 质量因子

# 移动优化
-b:v 1-3M      # 码率范围
-crf 26-30     # 质量因子
```

### B. 音频技术指标

#### 采样率和位深标准
**专业级 (Professional)**
- 采样率: 96kHz, 位深: 24-bit
- 适用于录音室制作、母带处理
- 最高保真度，无损处理

**广播级 (Broadcast)**
- 采样率: 48kHz, 位深: 24-bit/16-bit
- 适用于电视广播、专业发布
- 高质量，行业标准

**消费级 (Consumer)**
- 采样率: 44.1kHz, 位深: 16-bit
- 适用于CD质量、一般发布
- 良好质量，广泛兼容

**网络级 (Streaming)**
- 采样率: 44.1kHz, 位深: 16-bit, 压缩编码
- 适用于在线流媒体、移动应用
- 可接受质量，优化传输

#### 音频码率标准
```bash
# 无损压缩
FLAC: 600-1200 kbps

# 高质量有损
AAC 320kbps: 接近无损感知质量
MP3 320kbps: 高质量通用标准

# 标准质量
AAC 192kbps: 良好质量平衡
MP3 192kbps: 标准音乐质量

# 语音优化
AAC 128kbps: 适合语音内容
Opus 96kbps: 现代语音编码

# 极限压缩
AAC 64kbps: 带宽受限环境
```

## 视觉质量评估标准

### A. 画面清晰度评估

#### 客观指标测量
```bash
# PSNR (峰值信噪比) 测量
ffmpeg -i original.mp4 -i processed.mp4 -lavfi psnr -f null -

# SSIM (结构相似性) 测量
ffmpeg -i original.mp4 -i processed.mp4 -lavfi ssim -f null -

# VMAF (视觉多媒体评估融合) 测量
ffmpeg -i processed.mp4 -i original.mp4 -lavfi libvmaf -f null -
```

**质量阈值标准**:
- **PSNR**: >35dB (优秀), 30-35dB (良好), 25-30dB (可接受), <25dB (差)
- **SSIM**: >0.95 (优秀), 0.9-0.95 (良好), 0.8-0.9 (可接受), <0.8 (差)
- **VMAF**: >80 (优秀), 60-80 (良好), 40-60 (可接受), <40 (差)

#### 主观质量评估
**清晰度检查点**:
- 文字是否清晰可读
- 细节纹理是否保持
- 边缘是否锐利自然
- 是否存在模糊或伪影

### B. 色彩质量评估

#### 色彩准确性
```bash
# 色彩直方图分析
ffmpeg -i video.mp4 -vf "histogram=level_height=200" -frames:v 1 histogram.png

# 色彩空间检查
ffprobe -v quiet -select_streams v:0 -show_entries stream=color_space,color_primaries video.mp4

# 色彩范围验证
ffmpeg -i video.mp4 -vf "waveform=mode=color" -frames:v 1 waveform.png
```

**色彩质量标准**:
- 色彩饱和度适中，无过度饱和
- 肤色自然，无明显偏色
- 白平衡准确，无色温偏移
- 色彩过渡平滑，无色块现象

#### 动态范围评估
- **亮度分布**: 避免过曝和欠曝
- **对比度**: 保持层次丰富的明暗关系
- **黑色水平**: 纯黑区域保持细节
- **白色水平**: 高亮区域无剪切

## 应用场景质量标准

### A. 网络流媒体标准

#### YouTube 质量推荐
```bash
# 1080p标准设置
ffmpeg -i input.mp4 \
    -c:v libx264 -crf 21 -preset slow \
    -profile:v high -level:v 4.0 \
    -pix_fmt yuv420p \
    -c:a aac -b:a 192k -ar 48000 \
    -movflags faststart \
    youtube_1080p.mp4

# 质量验证
# 文件大小: 1小时约 1-2GB
# 码率: 5-8 Mbps
# 兼容性: 99%设备支持
```

#### 直播质量标准
```bash
# 直播编码设置
ffmpeg -i input \
    -c:v libx264 -preset veryfast \
    -b:v 4M -maxrate 4.2M -bufsize 8M \
    -g 50 -keyint_min 50 \
    -c:a aac -b:a 128k \
    -f flv rtmp://live.platform.com/live/streamkey
```

### B. 专业制作标准

#### 广播电视质量
- **技术规格**: 严格符合广电标准
- **色彩标准**: Rec.709色彩空间
- **音频标准**: 48kHz/24bit，响度控制
- **文件格式**: 专业容器格式 (MXF, MOV)

#### 影院放映质量
- **分辨率**: 4K (4096×2160) 或更高
- **色彩**: DCI-P3色彩空间
- **帧率**: 24fps标准，支持HFR
- **音频**: 无损多声道音频

### C. 移动端优化标准

#### 移动播放优化
```bash
# 移动端优化编码
ffmpeg -i input.mp4 \
    -c:v libx264 -crf 26 -preset medium \
    -profile:v baseline -level:v 3.1 \
    -s 1280x720 -r 30 \
    -c:a aac -b:a 96k -ar 44100 \
    -movflags faststart \
    mobile_optimized.mp4
```

**移动端质量要求**:
- 启动时间: <3秒
- 缓冲频率: <5%
- 电池消耗: 优化解码效率
- 网络适应: 支持自适应码率

## 自动化质量检测

### A. 批量质量检测脚本

#### 全面质量检测
```bash
#!/bin/bash
# 视频质量批量检测脚本

check_video_quality() {
    local video="$1"
    local report="${video%.*}_quality_report.txt"
    
    echo "=== 视频质量报告: $video ===" > "$report"
    echo "检测时间: $(date)" >> "$report"
    echo "" >> "$report"
    
    # 基础信息
    echo "=== 基础信息 ===" >> "$report"
    ffprobe -v quiet -print_format json -show_format -show_streams "$video" >> "$report"
    
    # 视觉质量检测
    echo "=== 视觉质量分析 ===" >> "$report"
    ffmpeg -i "$video" -vf "blackdetect=d=0.5:pix_th=0.1" -f null - 2>&1 | grep blackdetect >> "$report"
    
    # 音频质量检测
    echo "=== 音频质量分析 ===" >> "$report"
    ffmpeg -i "$video" -af "silencedetect=noise=-30dB:d=0.5" -f null - 2>&1 | grep silence >> "$report"
    
    # 文件完整性检测
    echo "=== 完整性检测 ===" >> "$report"
    ffmpeg -v error -i "$video" -f null - 2>&1 >> "$report"
    
    echo "质量检测完成: $report"
}

# 批量处理
for video in *.mp4; do
    check_video_quality "$video"
done
```

### B. 质量评分系统

#### 综合评分算法
```python
def calculate_quality_score(video_info):
    """
    计算视频综合质量评分 (0-100)
    """
    score = 0
    
    # 分辨率评分 (25%)
    resolution_score = get_resolution_score(video_info['width'], video_info['height'])
    score += resolution_score * 0.25
    
    # 码率评分 (20%)
    bitrate_score = get_bitrate_score(video_info['bit_rate'], video_info['width'])
    score += bitrate_score * 0.20
    
    # 帧率评分 (15%)
    framerate_score = get_framerate_score(video_info['r_frame_rate'])
    score += framerate_score * 0.15
    
    # 编码效率评分 (20%)
    codec_score = get_codec_score(video_info['codec_name'])
    score += codec_score * 0.20
    
    # 兼容性评分 (20%)
    compatibility_score = get_compatibility_score(video_info)
    score += compatibility_score * 0.20
    
    return min(100, max(0, score))
```

## 质量改进建议

### A. 常见质量问题及解决方案

#### 画质模糊问题
**问题识别**:
- PSNR < 30dB
- 主观感觉不清晰
- 细节丢失明显

**改进方案**:
```bash
# 提高编码质量
ffmpeg -i input.mp4 -c:v libx264 -crf 18 -preset slow output.mp4

# 锐化滤镜增强
ffmpeg -i input.mp4 -vf "unsharp=5:5:1.0:5:5:0.0" output.mp4

# 避免多次编码
ffmpeg -i input.mp4 -c copy output.mp4  # 如果格式兼容
```

#### 色彩偏差问题
**问题识别**:
- 肤色不自然
- 整体色温偏移
- 色彩饱和度异常

**改进方案**:
```bash
# 白平衡校正
ffmpeg -i input.mp4 -vf "colorbalance=rs=0.1:gs=-0.1:bs=0.05" output.mp4

# 色彩空间转换
ffmpeg -i input.mp4 -vf "colorspace=bt709:iall=bt601-6-625" output.mp4

# 自动色彩校正
ffmpeg -i input.mp4 -vf "eq=brightness=0.02:contrast=1.1:saturation=1.05" output.mp4
```

### B. 质量优化工作流程

#### 质量控制检查点
1. **输入验证**: 确保源素材质量符合要求
2. **处理监控**: 实时监控编码参数和质量指标
3. **输出验证**: 全面检测最终输出质量
4. **对比分析**: 与源素材和标准进行对比
5. **用户测试**: 在目标设备上验证播放效果

#### 质量改进迭代流程
```
质量检测 → 问题识别 → 参数调整 → 重新处理 → 效果验证 → 标准建立
    ↑                                                           ↓
    ←────────────────── 持续改进循环 ←─────────────────────────
```

---

*质量评估是一个持续改进的过程。建立标准化的评估体系有助于保证输出质量的一致性和可预测性。结合客观指标和主观评价，能够更全面地评估视频质量水平。*