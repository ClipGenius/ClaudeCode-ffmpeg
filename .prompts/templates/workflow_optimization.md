# 工作流程优化模板 - LLM 教学提示词

## 工作流程专家角色定义

你是一位视频制作工作流程优化专家，精通项目管理、效率提升和质量控制。你能够设计高效的制作流程，建立标准化操作规范，并提供针对不同项目类型的最佳实践指导。

## 工作流程设计原理

### 效率优化原则
1. **标准化流程**: 建立可重复的操作步骤
2. **并行处理**: 最大化资源利用率
3. **质量检查点**: 在关键节点进行验证
4. **错误预防**: 从源头避免常见问题
5. **持续改进**: 基于反馈优化流程

### 项目分类体系
- **短视频制作**: 社交媒体、广告片段
- **长视频制作**: 纪录片、教程、直播录制
- **专业制作**: 电影、电视、商业广告
- **批量处理**: 大规模内容转换和优化
- **实时制作**: 直播、在线会议录制

## 通用工作流程框架

### A. 项目前期规划模板

#### 需求分析检查清单
```yaml
项目基本信息:
  - 项目类型: [短视频/长视频/专业制作/批量处理]
  - 目标平台: [YouTube/TikTok/Instagram/broadcast/web]
  - 预计时长: [分钟数]
  - 质量要求: [标清/高清/4K/8K]
  - 交付时间: [日期]

技术规格:
  - 分辨率: [1920x1080/3840x2160]
  - 帧率: [24/30/60 fps]
  - 音频: [立体声/5.1环绕声]
  - 编码格式: [H.264/H.265/ProRes]
  - 文件大小限制: [MB/GB]

资源评估:
  - 原始素材大小: [GB]
  - 可用存储空间: [GB]
  - 处理设备性能: [CPU/GPU规格]
  - 预计处理时间: [小时]
```

#### 文件组织结构模板
```bash
# 标准项目目录结构
project_name/
├── 01_原始素材/
│   ├── video/
│   ├── audio/
│   ├── images/
│   └── documents/
├── 02_工作文件/
│   ├── proxy/          # 代理文件
│   ├── temp/           # 临时文件
│   └── backup/         # 备份文件
├── 03_输出文件/
│   ├── drafts/         # 草稿版本
│   ├── final/          # 最终版本
│   └── deliverables/   # 交付文件
├── 04_项目文档/
│   ├── scripts/        # 脚本文件
│   ├── notes/          # 制作笔记
│   └── logs/           # 处理日志
└── 05_资源管理/
    ├── fonts/          # 字体文件
    ├── graphics/       # 图形资源
    └── music/          # 音乐素材
```

### B. 制作流程优化模板

#### 短视频制作工作流程
```bash
#!/bin/bash
# 短视频制作优化流程

# 第1步：素材预处理
echo "=== 短视频制作流程 ==="
echo "步骤1: 素材预处理"

# 创建代理文件以提高编辑效率
create_proxy() {
    local input="$1"
    local output="02_工作文件/proxy/$(basename "$input" .mp4)_proxy.mp4"
    
    ffmpeg -i "$input" \
           -s 960x540 \
           -c:v libx264 -crf 30 -preset fast \
           -c:a aac -b:a 96k \
           "$output"
    echo "代理文件创建完成: $output"
}

# 批量创建代理文件
for video in 01_原始素材/video/*.mp4; do
    create_proxy "$video"
done

# 第2步：快速剪辑
echo "步骤2: 快速剪辑处理"

# 自动生成时间轴模板
generate_timeline_template() {
    cat > 04_项目文档/timeline_template.txt << EOF
# 短视频时间轴模板 (60秒)
00:00-00:05: 开场hook (吸引注意力)
00:05-00:15: 主题介绍
00:15-00:45: 核心内容
00:45-00:55: 总结要点
00:55-01:00: 结尾CTA (行动召唤)
EOF
    echo "时间轴模板已生成"
}

# 第3步：质量检查
quality_check() {
    local input="$1"
    
    echo "检查视频规格..."
    ffprobe -v quiet -print_format json -show_format -show_streams "$input" \
        > "04_项目文档/$(basename "$input")_analysis.json"
    
    # 检查分辨率
    resolution=$(ffprobe -v quiet -select_streams v:0 -show_entries stream=width,height -of csv=p=0 "$input")
    echo "分辨率: $resolution"
    
    # 检查时长
    duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$input")
    echo "时长: ${duration}秒"
    
    # 检查文件大小
    size=$(du -h "$input" | cut -f1)
    echo "文件大小: $size"
}

# 第4步：平台优化输出
platform_export() {
    local input="$1"
    local platform="$2"
    
    case "$platform" in
        "tiktok")
            ffmpeg -i "$input" \
                   -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
                   -c:v libx264 -crf 23 -r 30 \
                   -c:a aac -b:a 128k \
                   -t 60 \
                   "03_输出文件/final/${platform}_$(basename "$input")"
            ;;
        "youtube")
            ffmpeg -i "$input" \
                   -c:v libx264 -crf 20 -preset slow \
                   -c:a aac -b:a 192k \
                   "03_输出文件/final/${platform}_$(basename "$input")"
            ;;
        "instagram")
            ffmpeg -i "$input" \
                   -vf "scale=1080:1080:force_original_aspect_ratio=increase,crop=1080:1080" \
                   -c:v libx264 -crf 23 \
                   -c:a aac -b:a 128k \
                   -t 60 \
                   "03_输出文件/final/${platform}_$(basename "$input")"
            ;;
    esac
}
```

#### 长视频制作工作流程
```bash
#!/bin/bash
# 长视频制作优化流程

# 第1步：分段处理策略
segment_processing() {
    local input="$1"
    local segment_duration=600  # 10分钟段
    
    echo "=== 长视频分段处理 ==="
    
    # 获取总时长
    total_duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$input")
    num_segments=$(echo "($total_duration + $segment_duration - 1) / $segment_duration" | bc)
    
    echo "总时长: ${total_duration}秒"
    echo "分段数量: $num_segments"
    
    # 分段切割
    for ((i=0; i<num_segments; i++)); do
        start_time=$((i * segment_duration))
        output_file="02_工作文件/temp/segment_$(printf "%03d" $i).mp4"
        
        ffmpeg -ss $start_time -i "$input" -t $segment_duration -c copy "$output_file"
        echo "分段 $i 处理完成: $output_file"
    done
}

# 第2步：并行处理管理
parallel_processing() {
    local max_jobs=4
    local job_count=0
    
    echo "=== 并行处理任务 ==="
    
    for segment in 02_工作文件/temp/segment_*.mp4; do
        # 控制并发数量
        while [ $job_count -ge $max_jobs ]; do
            wait -n  # 等待任何一个后台任务完成
            ((job_count--))
        done
        
        # 启动后台处理任务
        (
            echo "处理分段: $segment"
            ffmpeg -i "$segment" \
                   -c:v libx264 -crf 23 -preset medium \
                   -c:a aac -b:a 192k \
                   "${segment%.mp4}_processed.mp4"
            echo "完成处理: $segment"
        ) &
        
        ((job_count++))
    done
    
    # 等待所有任务完成
    wait
    echo "所有分段处理完成"
}

# 第3步：无缝合并
seamless_concat() {
    echo "=== 无缝合并分段 ==="
    
    # 生成合并列表
    for file in 02_工作文件/temp/segment_*_processed.mp4; do
        echo "file '$file'" >> 02_工作文件/temp/concat_list.txt
    done
    
    # 执行无缝合并
    ffmpeg -f concat -safe 0 -i 02_工作文件/temp/concat_list.txt \
           -c copy \
           "03_输出文件/drafts/merged_draft.mp4"
    
    echo "合并完成: merged_draft.mp4"
}

# 第4步：质量验证
long_video_qa() {
    local input="03_输出文件/drafts/merged_draft.mp4"
    
    echo "=== 长视频质量验证 ==="
    
    # 检查音视频同步
    ffmpeg -i "$input" -vf "showinfo" -af "ashowinfo" -f null - 2>&1 | \
        grep -E "(pts_time|best_effort_timestamp_time)" | head -20 > \
        "04_项目文档/sync_analysis.log"
    
    # 检查音频电平
    ffmpeg -i "$input" -af "ebur128=peak=true" -f null - 2>&1 | \
        grep "Integrated loudness" > "04_项目文档/loudness_report.txt"
    
    # 生成质量报告
    cat > "04_项目文档/quality_report.md" << EOF
# 长视频质量报告

## 基本信息
- 文件: $(basename "$input")
- 大小: $(du -h "$input" | cut -f1)
- 时长: $(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$input" | xargs printf "%.2f秒\n")

## 技术规格
- 分辨率: $(ffprobe -v quiet -select_streams v:0 -show_entries stream=width,height -of csv=p=0 "$input")
- 编码: $(ffprobe -v quiet -select_streams v:0 -show_entries stream=codec_name -of csv=p=0 "$input")
- 音频: $(ffprobe -v quiet -select_streams a:0 -show_entries stream=codec_name,sample_rate -of csv=p=0 "$input")

## 质量检查
- 音视频同步: 查看 sync_analysis.log
- 音频电平: 查看 loudness_report.txt
- 状态: 待人工审查
EOF
    
    echo "质量报告已生成: quality_report.md"
}
```

### C. 专业制作工作流程

#### 色彩管理工作流程
```bash
#!/bin/bash
# 专业色彩管理流程

color_management_workflow() {
    local input="$1"
    local color_profile="$2"  # rec709, dci-p3, rec2020
    
    echo "=== 专业色彩管理流程 ==="
    
    # 第1步：色彩空间分析
    echo "步骤1: 色彩空间分析"
    ffprobe -v quiet -select_streams v:0 \
            -show_entries stream=color_space,color_primaries,color_transfer \
            "$input" > "04_项目文档/color_analysis.txt"
    
    # 第2步：色彩校正
    echo "步骤2: 色彩校正处理"
    case "$color_profile" in
        "rec709")
            ffmpeg -i "$input" \
                   -vf "colorspace=bt709:iall=bt601-6-625:fast=1" \
                   -color_primaries bt709 \
                   -color_trc bt709 \
                   -colorspace bt709 \
                   "02_工作文件/temp/color_corrected_rec709.mp4"
            ;;
        "dci-p3")
            ffmpeg -i "$input" \
                   -vf "colorspace=bt709:iall=bt2020nc:fast=1" \
                   -color_primaries bt2020 \
                   -color_trc smpte2084 \
                   -colorspace bt2020nc \
                   "02_工作文件/temp/color_corrected_dci-p3.mp4"
            ;;
        "rec2020")
            ffmpeg -i "$input" \
                   -vf "zscale=tin=bt709:min=bt709:pin=bt709:tout=bt2020nc:mout=bt2020nc:pout=bt2020" \
                   -color_primaries bt2020 \
                   -color_trc smpte2084 \
                   -colorspace bt2020nc \
                   "02_工作文件/temp/color_corrected_rec2020.mp4"
            ;;
    esac
    
    # 第3步：色彩验证
    echo "步骤3: 色彩验证"
    ffmpeg -i "02_工作文件/temp/color_corrected_${color_profile}.mp4" \
           -vf "vectorscope=m=color3:i=0.04:envelope=peak" \
           -frames:v 1 \
           "04_项目文档/vectorscope_${color_profile}.png"
    
    echo "色彩管理流程完成"
}

# 音频后期制作流程
audio_post_workflow() {
    local video_input="$1"
    local audio_input="$2"
    
    echo "=== 专业音频后期流程 ==="
    
    # 第1步：音频分析
    echo "步骤1: 音频质量分析"
    ffmpeg -i "$audio_input" -af "astats=metadata=1" -f null - 2>&1 | \
        grep -E "(RMS|Peak|Dynamic)" > "04_项目文档/audio_analysis.txt"
    
    # 第2步：音频清理
    echo "步骤2: 音频清理处理"
    ffmpeg -i "$audio_input" \
           -af "afftdn=nr=20,agate=threshold=-40dB:ratio=2,eq=type=highpass:frequency=80" \
           "02_工作文件/temp/audio_cleaned.wav"
    
    # 第3步：动态处理
    echo "步骤3: 动态范围处理"
    ffmpeg -i "02_工作文件/temp/audio_cleaned.wav" \
           -af "acompressor=threshold=-18dB:ratio=3:attack=5:release=50,alimiter=level_in=1:level_out=0.95" \
           "02_工作文件/temp/audio_processed.wav"
    
    # 第4步：音频同步
    echo "步骤4: 音视频同步"
    ffmpeg -i "$video_input" -i "02_工作文件/temp/audio_processed.wav" \
           -c:v copy -c:a aac -b:a 192k \
           -map 0:v:0 -map 1:a:0 \
           "02_工作文件/temp/synced_output.mp4"
    
    # 第5步：最终母带处理
    echo "步骤5: 母带处理"
    ffmpeg -i "02_工作文件/temp/synced_output.mp4" \
           -af "loudnorm=I=-16:LRA=11:TP=-1.5" \
           "03_输出文件/final/mastered_output.mp4"
    
    echo "音频后期流程完成"
}
```

## 自动化脚本模板

### A. 项目初始化自动化

#### 智能项目创建器
```python
#!/usr/bin/env python3
"""
智能项目创建器
根据项目类型自动创建标准化目录结构和配置文件
"""

import os
import json
import yaml
from pathlib import Path
from datetime import datetime
from typing import Dict, List

class ProjectCreator:
    def __init__(self):
        self.project_templates = {
            "short_video": {
                "description": "短视频项目模板",
                "directories": [
                    "01_原始素材/video",
                    "01_原始素材/audio", 
                    "01_原始素材/images",
                    "02_工作文件/proxy",
                    "02_工作文件/temp",
                    "03_输出文件/drafts",
                    "03_输出文件/final",
                    "04_项目文档/scripts",
                    "04_项目文档/logs"
                ],
                "config": {
                    "target_duration": 60,
                    "resolution": "1920x1080",
                    "framerate": 30,
                    "platforms": ["tiktok", "youtube", "instagram"]
                }
            },
            "long_video": {
                "description": "长视频项目模板", 
                "directories": [
                    "01_原始素材/video",
                    "01_原始素材/audio",
                    "01_原始素材/images",
                    "01_原始素材/documents",
                    "02_工作文件/proxy",
                    "02_工作文件/temp",
                    "02_工作文件/backup",
                    "03_输出文件/drafts",
                    "03_输出文件/final",
                    "03_输出文件/deliverables",
                    "04_项目文档/scripts",
                    "04_项目文档/notes",
                    "04_项目文档/logs",
                    "05_资源管理/fonts",
                    "05_资源管理/graphics",
                    "05_资源管理/music"
                ],
                "config": {
                    "target_duration": 1800,  # 30分钟
                    "resolution": "1920x1080",
                    "framerate": 25,
                    "platforms": ["youtube", "vimeo", "web"]
                }
            },
            "professional": {
                "description": "专业制作项目模板",
                "directories": [
                    "01_原始素材/camera_a",
                    "01_原始素材/camera_b", 
                    "01_原始素材/audio/location",
                    "01_原始素材/audio/studio",
                    "01_原始素材/graphics",
                    "02_工作文件/proxy",
                    "02_工作文件/temp",
                    "02_工作文件/backup",
                    "02_工作文件/color",
                    "03_输出文件/previews",
                    "03_输出文件/drafts",
                    "03_输出文件/final",
                    "03_输出文件/masters",
                    "04_项目文档/scripts",
                    "04_项目文档/notes",
                    "04_项目文档/logs",
                    "04_项目文档/reports",
                    "05_资源管理/fonts",
                    "05_资源管理/graphics",
                    "05_资源管理/music",
                    "05_资源管理/luts"
                ],
                "config": {
                    "resolution": "3840x2160",
                    "framerate": 24,
                    "color_space": "rec2020",
                    "platforms": ["cinema", "broadcast", "streaming"]
                }
            }
        }
    
    def create_project(self, project_name: str, project_type: str, 
                      base_path: str = "./") -> bool:
        """创建新项目"""
        if project_type not in self.project_templates:
            print(f"错误: 未知项目类型 {project_type}")
            return False
        
        template = self.project_templates[project_type]
        project_path = Path(base_path) / project_name
        
        # 创建项目根目录
        project_path.mkdir(exist_ok=True)
        
        # 创建目录结构
        for directory in template["directories"]:
            dir_path = project_path / directory
            dir_path.mkdir(parents=True, exist_ok=True)
            print(f"创建目录: {dir_path}")
        
        # 生成项目配置文件
        config = {
            "project_name": project_name,
            "project_type": project_type,
            "created_date": datetime.now().isoformat(),
            "template_config": template["config"]
        }
        
        config_file = project_path / "project_config.yaml"
        with open(config_file, 'w', encoding='utf-8') as f:
            yaml.dump(config, f, default_flow_style=False, allow_unicode=True)
        
        # 生成README文件
        readme_content = f"""# {project_name}

## 项目信息
- 项目类型: {template["description"]}
- 创建日期: {datetime.now().strftime("%Y-%m-%d")}
- 配置文件: project_config.yaml

## 目录结构
"""
        for directory in template["directories"]:
            readme_content += f"- {directory}/\n"
        
        readme_content += """
## 工作流程
1. 将原始素材放入 01_原始素材/ 目录
2. 在 02_工作文件/ 中进行编辑工作
3. 输出文件保存到 03_输出文件/ 目录
4. 项目文档记录在 04_项目文档/ 中

## 注意事项
- 定期备份重要工作文件
- 遵循命名规范
- 记录处理日志
"""
        
        readme_file = project_path / "README.md"
        with open(readme_file, 'w', encoding='utf-8') as f:
            f.write(readme_content)
        
        print(f"项目创建完成: {project_path}")
        return True
    
    def list_templates(self):
        """列出可用模板"""
        print("可用项目模板:")
        for template_name, template_info in self.project_templates.items():
            print(f"- {template_name}: {template_info['description']}")

# 使用示例
if __name__ == "__main__":
    creator = ProjectCreator()
    
    # 列出模板
    creator.list_templates()
    
    # 创建项目
    creator.create_project("我的短视频项目", "short_video", "./projects")
```

### B. 质量控制自动化

#### 自动质量检查器
```python
#!/usr/bin/env python3
"""
自动质量检查器
对输出文件进行全面的质量验证
"""

import subprocess
import json
import logging
from pathlib import Path
from typing import Dict, List, Tuple

class QualityChecker:
    def __init__(self, config_file: str = None):
        self.setup_logging()
        self.quality_standards = self.load_standards(config_file)
        
    def setup_logging(self):
        """设置日志系统"""
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler('quality_check.log'),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def load_standards(self, config_file: str) -> Dict:
        """加载质量标准"""
        default_standards = {
            "video": {
                "min_bitrate": 1000,  # kbps
                "max_bitrate": 50000,
                "min_framerate": 23.98,
                "max_framerate": 60,
                "required_codecs": ["h264", "hevc"],
                "max_duration_variance": 2.0  # 秒
            },
            "audio": {
                "min_bitrate": 96,  # kbps
                "max_bitrate": 320,
                "required_sample_rates": [44100, 48000],
                "max_loudness": -14,  # LUFS
                "min_loudness": -30,
                "required_codecs": ["aac", "mp3"]
            },
            "file": {
                "max_size_gb": 10,
                "min_size_mb": 1
            }
        }
        
        if config_file and Path(config_file).exists():
            # 这里可以加载自定义配置
            pass
            
        return default_standards
    
    def analyze_video_file(self, file_path: Path) -> Dict:
        """分析视频文件"""
        cmd = [
            'ffprobe',
            '-v', 'quiet',
            '-print_format', 'json',
            '-show_format',
            '-show_streams',
            str(file_path)
        ]
        
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, check=True)
            return json.loads(result.stdout)
        except subprocess.CalledProcessError as e:
            self.logger.error(f"分析文件失败 {file_path}: {e}")
            return {}
    
    def check_video_quality(self, analysis: Dict) -> List[str]:
        """检查视频质量"""
        issues = []
        
        # 查找视频流
        video_stream = None
        for stream in analysis.get('streams', []):
            if stream.get('codec_type') == 'video':
                video_stream = stream
                break
        
        if not video_stream:
            issues.append("未找到视频流")
            return issues
        
        # 检查编解码器
        codec = video_stream.get('codec_name', '').lower()
        if codec not in self.quality_standards['video']['required_codecs']:
            issues.append(f"不支持的视频编解码器: {codec}")
        
        # 检查比特率
        bit_rate = int(video_stream.get('bit_rate', 0)) // 1000  # 转换为kbps
        if bit_rate > 0:
            if bit_rate < self.quality_standards['video']['min_bitrate']:
                issues.append(f"视频比特率过低: {bit_rate}kbps")
            elif bit_rate > self.quality_standards['video']['max_bitrate']:
                issues.append(f"视频比特率过高: {bit_rate}kbps")
        
        # 检查帧率
        framerate_str = video_stream.get('r_frame_rate', '0/1')
        if '/' in framerate_str:
            num, den = map(int, framerate_str.split('/'))
            framerate = num / den if den != 0 else 0
            
            if framerate < self.quality_standards['video']['min_framerate']:
                issues.append(f"帧率过低: {framerate}fps")
            elif framerate > self.quality_standards['video']['max_framerate']:
                issues.append(f"帧率过高: {framerate}fps")
        
        return issues
    
    def check_audio_quality(self, analysis: Dict) -> List[str]:
        """检查音频质量"""
        issues = []
        
        # 查找音频流
        audio_stream = None
        for stream in analysis.get('streams', []):
            if stream.get('codec_type') == 'audio':
                audio_stream = stream
                break
        
        if not audio_stream:
            issues.append("未找到音频流")
            return issues
        
        # 检查编解码器
        codec = audio_stream.get('codec_name', '').lower()
        if codec not in self.quality_standards['audio']['required_codecs']:
            issues.append(f"不支持的音频编解码器: {codec}")
        
        # 检查比特率
        bit_rate = int(audio_stream.get('bit_rate', 0)) // 1000  # 转换为kbps
        if bit_rate > 0:
            if bit_rate < self.quality_standards['audio']['min_bitrate']:
                issues.append(f"音频比特率过低: {bit_rate}kbps")
            elif bit_rate > self.quality_standards['audio']['max_bitrate']:
                issues.append(f"音频比特率过高: {bit_rate}kbps")
        
        # 检查采样率
        sample_rate = int(audio_stream.get('sample_rate', 0))
        if sample_rate not in self.quality_standards['audio']['required_sample_rates']:
            issues.append(f"不支持的采样率: {sample_rate}Hz")
        
        return issues
    
    def check_file_quality(self, file_path: Path, analysis: Dict) -> List[str]:
        """检查文件质量"""
        issues = []
        
        # 检查文件大小
        file_size = file_path.stat().st_size
        size_gb = file_size / (1024**3)
        size_mb = file_size / (1024**2)
        
        if size_gb > self.quality_standards['file']['max_size_gb']:
            issues.append(f"文件过大: {size_gb:.2f}GB")
        elif size_mb < self.quality_standards['file']['min_size_mb']:
            issues.append(f"文件过小: {size_mb:.2f}MB")
        
        return issues
    
    def generate_quality_report(self, file_path: Path) -> Dict:
        """生成质量报告"""
        self.logger.info(f"检查文件质量: {file_path}")
        
        # 分析文件
        analysis = self.analyze_video_file(file_path)
        if not analysis:
            return {"status": "error", "message": "文件分析失败"}
        
        # 执行各项检查
        video_issues = self.check_video_quality(analysis)
        audio_issues = self.check_audio_quality(analysis)
        file_issues = self.check_file_quality(file_path, analysis)
        
        all_issues = video_issues + audio_issues + file_issues
        
        # 生成报告
        report = {
            "file_path": str(file_path),
            "check_time": datetime.now().isoformat(),
            "status": "pass" if not all_issues else "fail",
            "issues": {
                "video": video_issues,
                "audio": audio_issues,
                "file": file_issues
            },
            "technical_info": {
                "duration": float(analysis.get('format', {}).get('duration', 0)),
                "size_mb": file_path.stat().st_size / (1024**2),
                "format": analysis.get('format', {}).get('format_name', 'unknown')
            }
        }
        
        self.logger.info(f"质量检查完成: {report['status']}")
        if all_issues:
            self.logger.warning(f"发现 {len(all_issues)} 个问题")
        
        return report
    
    def batch_check(self, directory: Path, pattern: str = "*.mp4") -> List[Dict]:
        """批量质量检查"""
        reports = []
        files = list(directory.glob(pattern))
        
        self.logger.info(f"开始批量检查: {len(files)} 个文件")
        
        for file_path in files:
            report = self.generate_quality_report(file_path)
            reports.append(report)
        
        # 生成汇总报告
        passed = sum(1 for r in reports if r['status'] == 'pass')
        failed = len(reports) - passed
        
        self.logger.info(f"批量检查完成: {passed} 通过, {failed} 失败")
        
        return reports

# 使用示例
checker = QualityChecker()
report = checker.generate_quality_report(Path("03_输出文件/final/output.mp4"))
print(json.dumps(report, indent=2, ensure_ascii=False))
```

## 工作流程监控和优化

### A. 性能监控系统

#### 处理时间分析器
```bash
#!/bin/bash
# 处理时间分析和优化建议

analyze_processing_time() {
    local input_file="$1"
    local start_time=$(date +%s.%N)
    
    echo "=== 处理时间分析 ==="
    echo "开始时间: $(date)"
    
    # 获取文件信息
    file_size=$(du -m "$input_file" | cut -f1)
    duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$input_file")
    resolution=$(ffprobe -v quiet -select_streams v:0 -show_entries stream=width,height -of csv=p=0 "$input_file")
    
    echo "文件大小: ${file_size}MB"
    echo "视频时长: ${duration}秒"
    echo "分辨率: $resolution"
    
    # 执行测试处理
    ffmpeg -i "$input_file" \
           -c:v libx264 -crf 23 -preset medium \
           -c:a aac -b:a 128k \
           "temp_output.mp4" \
           2>&1 | tee processing.log
    
    local end_time=$(date +%s.%N)
    local processing_time=$(echo "$end_time - $start_time" | bc)
    local speed_ratio=$(echo "$duration / $processing_time" | bc -l)
    
    echo "处理耗时: ${processing_time}秒"
    echo "处理速度: ${speed_ratio}x (相对于实时)"
    
    # 性能分析
    if (( $(echo "$speed_ratio < 0.5" | bc -l) )); then
        echo "性能状态: 较慢 (建议优化)"
        suggest_optimization "$input_file"
    elif (( $(echo "$speed_ratio < 1.0" | bc -l) )); then
        echo "性能状态: 正常"
    else
        echo "性能状态: 快速"
    fi
    
    # 清理临时文件
    rm -f temp_output.mp4 processing.log
}

suggest_optimization() {
    local input_file="$1"
    
    echo "=== 优化建议 ==="
    
    # 检查分辨率
    width=$(ffprobe -v quiet -select_streams v:0 -show_entries stream=width -of csv=p=0 "$input_file")
    if [ "$width" -gt 1920 ]; then
        echo "1. 考虑使用代理文件进行编辑 (降低分辨率到1920x1080)"
    fi
    
    # 检查系统资源
    cpu_cores=$(nproc)
    echo "2. 当前CPU核心数: $cpu_cores"
    echo "   建议使用 -threads $cpu_cores 参数"
    
    # 检查硬件加速
    if nvidia-smi &>/dev/null; then
        echo "3. 检测到NVIDIA GPU，建议使用硬件加速:"
        echo "   -hwaccel cuda -c:v h264_nvenc"
    fi
    
    # 检查预设选择
    echo "4. 预设建议:"
    echo "   - 预览: ultrafast"
    echo "   - 草稿: fast"
    echo "   - 最终: medium"
    echo "   - 质量优先: slow"
}
```

### B. 自动化报告生成

#### 项目完成报告生成器
```python
#!/usr/bin/env python3
"""
项目完成报告生成器
自动生成项目总结和统计报告
"""

import os
import json
import yaml
from pathlib import Path
from datetime import datetime
import matplotlib.pyplot as plt

class ProjectReporter:
    def __init__(self, project_path: str):
        self.project_path = Path(project_path)
        self.config_file = self.project_path / "project_config.yaml"
        self.load_project_config()
    
    def load_project_config(self):
        """加载项目配置"""
        if self.config_file.exists():
            with open(self.config_file, 'r', encoding='utf-8') as f:
                self.config = yaml.safe_load(f)
        else:
            self.config = {}
    
    def analyze_project_structure(self) -> Dict:
        """分析项目结构"""
        structure_info = {
            "directories": [],
            "file_counts": {},
            "total_files": 0,
            "total_size_gb": 0
        }
        
        for root, dirs, files in os.walk(self.project_path):
            rel_path = Path(root).relative_to(self.project_path)
            structure_info["directories"].append(str(rel_path))
            
            # 统计文件数量和大小
            for file in files:
                file_path = Path(root) / file
                try:
                    size = file_path.stat().st_size
                    structure_info["total_size_gb"] += size / (1024**3)
                    structure_info["total_files"] += 1
                    
                    # 按扩展名分类
                    ext = file_path.suffix.lower()
                    structure_info["file_counts"][ext] = structure_info["file_counts"].get(ext, 0) + 1
                except:
                    pass
        
        return structure_info
    
    def analyze_output_files(self) -> Dict:
        """分析输出文件"""
        output_dir = self.project_path / "03_输出文件"
        if not output_dir.exists():
            return {}
        
        output_info = {
            "final_files": [],
            "draft_files": [],
            "total_output_size": 0
        }
        
        # 分析最终文件
        final_dir = output_dir / "final"
        if final_dir.exists():
            for file in final_dir.iterdir():
                if file.is_file():
                    size_mb = file.stat().st_size / (1024**2)
                    output_info["final_files"].append({
                        "name": file.name,
                        "size_mb": round(size_mb, 2),
                        "modified": datetime.fromtimestamp(file.stat().st_mtime).isoformat()
                    })
                    output_info["total_output_size"] += size_mb
        
        # 分析草稿文件
        draft_dir = output_dir / "drafts"
        if draft_dir.exists():
            for file in draft_dir.iterdir():
                if file.is_file():
                    size_mb = file.stat().st_size / (1024**2)
                    output_info["draft_files"].append({
                        "name": file.name,
                        "size_mb": round(size_mb, 2),
                        "modified": datetime.fromtimestamp(file.stat().st_mtime).isoformat()
                    })
        
        return output_info
    
    def generate_timeline_chart(self, output_path: Path):
        """生成项目时间轴图表"""
        logs_dir = self.project_path / "04_项目文档" / "logs"
        if not logs_dir.exists():
            return
        
        # 这里可以分析日志文件生成时间轴
        # 简化版本：显示文件创建时间分布
        dates = []
        for root, dirs, files in os.walk(self.project_path):
            for file in files:
                file_path = Path(root) / file
                try:
                    mtime = datetime.fromtimestamp(file_path.stat().st_mtime)
                    dates.append(mtime.date())
                except:
                    pass
        
        # 统计每天的文件数量
        from collections import Counter
        date_counts = Counter(dates)
        
        if date_counts:
            plt.figure(figsize=(12, 6))
            dates_list = sorted(date_counts.keys())
            counts_list = [date_counts[date] for date in dates_list]
            
            plt.plot(dates_list, counts_list, marker='o')
            plt.title('项目活动时间轴')
            plt.xlabel('日期')
            plt.ylabel('文件操作数量')
            plt.xticks(rotation=45)
            plt.tight_layout()
            plt.savefig(output_path / 'project_timeline.png', dpi=300)
            plt.close()
    
    def generate_comprehensive_report(self) -> str:
        """生成综合报告"""
        structure_info = self.analyze_project_structure()
        output_info = self.analyze_output_files()
        
        report = f"""# 项目完成报告

## 项目基本信息
- 项目名称: {self.config.get('project_name', '未知')}
- 项目类型: {self.config.get('project_type', '未知')}
- 创建日期: {self.config.get('created_date', '未知')}
- 报告生成时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

## 项目统计
- 总文件数: {structure_info['total_files']}
- 项目总大小: {structure_info['total_size_gb']:.2f} GB
- 目录数量: {len(structure_info['directories'])}

### 文件类型分布
"""
        
        for ext, count in sorted(structure_info['file_counts'].items()):
            if ext:
                report += f"- {ext}: {count} 个文件\n"
        
        report += f"""
## 输出文件分析
- 最终文件数量: {len(output_info.get('final_files', []))}
- 草稿文件数量: {len(output_info.get('draft_files', []))}
- 输出文件总大小: {output_info.get('total_output_size', 0):.2f} MB

### 最终输出文件
"""
        
        for file_info in output_info.get('final_files', []):
            report += f"- {file_info['name']} ({file_info['size_mb']} MB)\n"
        
        report += """
## 项目结构
"""
        
        for directory in sorted(structure_info['directories']):
            if directory != '.':
                report += f"- {directory}/\n"
        
        report += f"""
## 总结
项目已完成，包含 {len(output_info.get('final_files', []))} 个最终输出文件。
总处理数据量: {structure_info['total_size_gb']:.2f} GB

## 建议
1. 定期清理临时文件以释放存储空间
2. 备份重要的项目文件
3. 归档项目文档以便后续参考
"""
        
        return report
    
    def save_report(self, output_file: str = None):
        """保存报告"""
        if not output_file:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            output_file = f"project_report_{timestamp}.md"
        
        report_path = self.project_path / "04_项目文档" / output_file
        report_path.parent.mkdir(exist_ok=True)
        
        report_content = self.generate_comprehensive_report()
        
        with open(report_path, 'w', encoding='utf-8') as f:
            f.write(report_content)
        
        # 生成图表
        self.generate_timeline_chart(report_path.parent)
        
        print(f"项目报告已生成: {report_path}")
        return report_path

# 使用示例
reporter = ProjectReporter("./我的项目")
reporter.save_report()
```

---

*高效的工作流程是成功项目的基础。通过建立标准化的流程模板和自动化工具，可以显著提高制作效率、保证质量一致性，并减少人为错误。在实际应用中，要根据项目特点和团队规模选择合适的工作流程策略。*