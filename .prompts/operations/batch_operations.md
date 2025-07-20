# 批处理和自动化脚本 - LLM 教学提示词

## 自动化专家角色定义

你是一位自动化和批处理专家，精通脚本编程和工作流程优化。你能够设计高效的批处理解决方案，实现大规模视频处理任务的自动化，并提供可靠的错误处理和进度监控机制。

## 批处理核心理念

### 自动化原则
1. **可重复性**: 相同输入产生相同输出
2. **可扩展性**: 支持从单个文件到大规模处理
3. **容错性**: 优雅处理错误和异常情况
4. **可监控性**: 提供清晰的进度和状态反馈
5. **可维护性**: 模块化设计，易于修改和扩展

### 批处理策略
- **并行处理**: 充分利用多核CPU资源
- **增量处理**: 只处理变更的文件
- **断点续传**: 支持中断后恢复处理
- **质量检查**: 自动验证输出结果
- **日志记录**: 详细记录处理过程和结果

## Shell脚本批处理

### A. 基础批处理模板

#### 通用批处理框架
```bash
#!/bin/bash
# 通用视频批处理脚本模板

# 配置部分
INPUT_DIR="${1:-./input}"
OUTPUT_DIR="${2:-./output}"
LOG_FILE="batch_process_$(date +%Y%m%d_%H%M%S).log"
PARALLEL_JOBS=4
BACKUP_ORIGINAL=true

# 函数定义
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

process_video() {
    local input_file="$1"
    local output_file="$2"
    
    log_message "处理文件: $input_file"
    
    # 检查输入文件
    if [[ ! -f "$input_file" ]]; then
        log_message "错误: 输入文件不存在 - $input_file"
        return 1
    fi
    
    # 创建输出目录
    mkdir -p "$(dirname "$output_file")"
    
    # 执行FFmpeg处理
    if ffmpeg -i "$input_file" \
               -c:v libx264 -crf 23 -preset medium \
               -c:a aac -b:a 128k \
               "$output_file" \
               -y 2>> "$LOG_FILE"; then
        log_message "成功: $output_file"
        return 0
    else
        log_message "失败: $input_file"
        return 1
    fi
}

# 主处理逻辑
main() {
    log_message "开始批处理任务"
    log_message "输入目录: $INPUT_DIR"
    log_message "输出目录: $OUTPUT_DIR"
    
    # 创建输出目录
    mkdir -p "$OUTPUT_DIR"
    
    # 处理计数器
    local total=0
    local success=0
    local failed=0
    
    # 处理所有视频文件
    find "$INPUT_DIR" -type f \( -name "*.mp4" -o -name "*.avi" -o -name "*.mov" \) | while read -r input_file; do
        # 生成输出文件路径
        relative_path="${input_file#$INPUT_DIR/}"
        output_file="$OUTPUT_DIR/${relative_path%.*}.mp4"
        
        # 检查是否已存在
        if [[ -f "$output_file" ]]; then
            log_message "跳过已存在文件: $output_file"
            continue
        fi
        
        ((total++))
        
        if process_video "$input_file" "$output_file"; then
            ((success++))
        else
            ((failed++))
        fi
        
        # 进度报告
        log_message "进度: 总计=$total, 成功=$success, 失败=$failed"
    done
    
    log_message "批处理完成"
}

# 执行主函数
main "$@"
```

#### 并行处理脚本
```bash
#!/bin/bash
# 并行批处理脚本

# 配置
MAX_PARALLEL=${1:-4}
INPUT_DIR="${2:-./input}"
OUTPUT_DIR="${3:-./output}"

# 工作函数
process_single_video() {
    local input="$1"
    local output="$2"
    local worker_id="$3"
    
    echo "[Worker $worker_id] 开始处理: $(basename "$input")"
    
    if ffmpeg -i "$input" \
              -c:v libx264 -crf 23 -preset fast \
              -c:a aac -b:a 128k \
              "$output" \
              -y 2>/dev/null; then
        echo "[Worker $worker_id] 完成: $(basename "$output")"
        return 0
    else
        echo "[Worker $worker_id] 失败: $(basename "$input")"
        return 1
    fi
}

export -f process_single_video

# 主处理逻辑
main() {
    mkdir -p "$OUTPUT_DIR"
    
    # 生成任务列表
    find "$INPUT_DIR" -type f \( -name "*.mp4" -o -name "*.avi" -o -name "*.mov" \) | \
    while read -r input_file; do
        relative_path="${input_file#$INPUT_DIR/}"
        output_file="$OUTPUT_DIR/${relative_path%.*}.mp4"
        echo "$input_file|$output_file"
    done > task_list.txt
    
    # 并行处理
    cat task_list.txt | parallel -j "$MAX_PARALLEL" --colsep '\\|' \
        'process_single_video {1} {2} {%}'
    
    # 清理
    rm -f task_list.txt
}

main
```

### B. 专用批处理脚本

#### 格式转换批处理
```bash
#!/bin/bash
# 批量格式转换脚本

convert_to_format() {
    local input_dir="$1"
    local output_dir="$2"
    local target_format="$3"
    local quality="${4:-23}"
    
    find "$input_dir" -type f \( -name "*.avi" -o -name "*.mov" -o -name "*.mkv" \) | \
    while read -r input_file; do
        # 生成输出文件名
        filename=$(basename "$input_file")
        name="${filename%.*}"
        output_file="$output_dir/$name.$target_format"
        
        echo "转换: $input_file -> $output_file"
        
        case "$target_format" in
            "mp4")
                ffmpeg -i "$input_file" \
                       -c:v libx264 -crf "$quality" -preset medium \
                       -c:a aac -b:a 128k \
                       "$output_file" -y
                ;;
            "webm")
                ffmpeg -i "$input_file" \
                       -c:v libvpx-vp9 -crf "$quality" \
                       -c:a libopus -b:a 96k \
                       "$output_file" -y
                ;;
            "mov")
                ffmpeg -i "$input_file" \
                       -c:v libx264 -crf "$quality" \
                       -c:a aac -b:a 192k \
                       "$output_file" -y
                ;;
        esac
    done
}

# 使用示例
convert_to_format "./source" "./converted" "mp4" "20"
```

#### 批量压缩脚本
```bash
#!/bin/bash
# 批量视频压缩脚本

compress_videos() {
    local input_dir="$1"
    local output_dir="$2"
    local target_size_mb="${3:-100}"  # 目标大小（MB）
    
    mkdir -p "$output_dir"
    
    find "$input_dir" -name "*.mp4" | while read -r input_file; do
        filename=$(basename "$input_file")
        output_file="$output_dir/compressed_$filename"
        
        # 获取视频时长
        duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$input_file")
        
        # 计算目标码率 (kbps)
        target_bitrate=$(echo "$target_size_mb * 8192 / $duration * 0.9" | bc)
        
        echo "压缩 $filename 到 ${target_size_mb}MB (目标码率: ${target_bitrate}kbps)"
        
        ffmpeg -i "$input_file" \
               -b:v "${target_bitrate}k" -maxrate "${target_bitrate}k" \
               -bufsize "$((target_bitrate*2))k" \
               -c:v libx264 -preset medium \
               -c:a aac -b:a 96k \
               "$output_file" -y
    done
}

# 使用示例
compress_videos "./input" "./compressed" "50"
```

## Python自动化脚本

### A. Python批处理框架

#### 面向对象批处理器
```python
#!/usr/bin/env python3
"""
Python视频批处理框架
"""

import os
import sys
import json
import logging
import subprocess
import multiprocessing
from pathlib import Path
from typing import List, Dict, Optional, Callable
from concurrent.futures import ProcessPoolExecutor, as_completed
import time

class VideoBatchProcessor:
    """视频批处理器基类"""
    
    def __init__(self, input_dir: str, output_dir: str, 
                 max_workers: Optional[int] = None):
        self.input_dir = Path(input_dir)
        self.output_dir = Path(output_dir)
        self.max_workers = max_workers or multiprocessing.cpu_count()
        
        # 设置日志
        self.setup_logging()
        
        # 支持的格式
        self.supported_formats = {'.mp4', '.avi', '.mov', '.mkv', '.webm'}
        
        # 处理统计
        self.stats = {
            'total': 0,
            'success': 0,
            'failed': 0,
            'skipped': 0
        }
    
    def setup_logging(self):
        """设置日志系统"""
        timestamp = time.strftime("%Y%m%d_%H%M%S")
        log_file = f"batch_process_{timestamp}.log"
        
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler(log_file),
                logging.StreamHandler(sys.stdout)
            ]
        )
        self.logger = logging.getLogger(__name__)
    
    def find_video_files(self) -> List[Path]:
        """查找所有视频文件"""
        video_files = []
        for ext in self.supported_formats:
            video_files.extend(self.input_dir.rglob(f"*{ext}"))
        return sorted(video_files)
    
    def get_output_path(self, input_path: Path, suffix: str = "") -> Path:
        """生成输出文件路径"""
        relative_path = input_path.relative_to(self.input_dir)
        output_path = self.output_dir / relative_path
        
        if suffix:
            name = output_path.stem + suffix
            output_path = output_path.with_name(name + output_path.suffix)
        
        return output_path
    
    def process_single_video(self, input_path: Path, 
                           processor_func: Callable) -> Dict:
        """处理单个视频文件"""
        try:
            output_path = self.get_output_path(input_path)
            
            # 检查输出文件是否已存在
            if output_path.exists():
                self.logger.info(f"跳过已存在文件: {output_path}")
                return {'status': 'skipped', 'input': input_path, 'output': output_path}
            
            # 创建输出目录
            output_path.parent.mkdir(parents=True, exist_ok=True)
            
            # 执行处理函数
            result = processor_func(input_path, output_path)
            
            if result:
                self.logger.info(f"成功处理: {input_path.name}")
                return {'status': 'success', 'input': input_path, 'output': output_path}
            else:
                self.logger.error(f"处理失败: {input_path.name}")
                return {'status': 'failed', 'input': input_path, 'output': output_path}
                
        except Exception as e:
            self.logger.error(f"处理异常 {input_path.name}: {str(e)}")
            return {'status': 'failed', 'input': input_path, 'output': None, 'error': str(e)}
    
    def batch_process(self, processor_func: Callable) -> Dict:
        """批量处理视频文件"""
        video_files = self.find_video_files()
        self.stats['total'] = len(video_files)
        
        self.logger.info(f"开始批处理: {self.stats['total']} 个文件")
        self.logger.info(f"使用 {self.max_workers} 个并行工作进程")
        
        start_time = time.time()
        results = []
        
        with ProcessPoolExecutor(max_workers=self.max_workers) as executor:
            # 提交所有任务
            future_to_file = {
                executor.submit(self.process_single_video, video_file, processor_func): video_file
                for video_file in video_files
            }
            
            # 处理完成的任务
            for future in as_completed(future_to_file):
                result = future.result()
                results.append(result)
                
                # 更新统计
                self.stats[result['status']] += 1
                
                # 进度报告
                completed = self.stats['success'] + self.stats['failed'] + self.stats['skipped']
                progress = (completed / self.stats['total']) * 100
                self.logger.info(f"进度: {progress:.1f}% ({completed}/{self.stats['total']})")
        
        end_time = time.time()
        duration = end_time - start_time
        
        # 最终报告
        self.logger.info(f"批处理完成，耗时: {duration:.2f}秒")
        self.logger.info(f"统计: {self.stats}")
        
        return {
            'stats': self.stats,
            'duration': duration,
            'results': results
        }

# FFmpeg处理函数示例
def standard_compression(input_path: Path, output_path: Path) -> bool:
    """标准压缩处理"""
    cmd = [
        'ffmpeg',
        '-i', str(input_path),
        '-c:v', 'libx264',
        '-crf', '23',
        '-preset', 'medium',
        '-c:a', 'aac',
        '-b:a', '128k',
        str(output_path),
        '-y'
    ]
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True)
        return result.returncode == 0
    except Exception:
        return False

# 使用示例
if __name__ == "__main__":
    processor = VideoBatchProcessor("./input", "./output", max_workers=4)
    result = processor.batch_process(standard_compression)
    print(f"处理完成: {result['stats']}")
```

### B. 专业化处理脚本

#### 智能质量压缩
```python
#!/usr/bin/env python3
"""
智能质量压缩脚本
根据文件大小和内容自动调整压缩参数
"""

import subprocess
import json
from pathlib import Path

class IntelligentCompressor:
    def __init__(self):
        self.size_thresholds = {
            'small': 100 * 1024 * 1024,    # 100MB
            'medium': 500 * 1024 * 1024,   # 500MB
            'large': 2 * 1024 * 1024 * 1024  # 2GB
        }
        
        self.quality_profiles = {
            'small': {'crf': 28, 'preset': 'fast'},
            'medium': {'crf': 23, 'preset': 'medium'},
            'large': {'crf': 20, 'preset': 'slow'}
        }
    
    def analyze_video(self, video_path: Path) -> Dict:
        """分析视频属性"""
        cmd = [
            'ffprobe',
            '-v', 'quiet',
            '-print_format', 'json',
            '-show_format',
            '-show_streams',
            str(video_path)
        ]
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        return json.loads(result.stdout)
    
    def determine_profile(self, video_path: Path) -> str:
        """确定压缩配置"""
        file_size = video_path.stat().st_size
        
        if file_size < self.size_thresholds['small']:
            return 'small'
        elif file_size < self.size_thresholds['medium']:
            return 'medium'
        else:
            return 'large'
    
    def compress_video(self, input_path: Path, output_path: Path) -> bool:
        """智能压缩视频"""
        # 分析视频
        analysis = self.analyze_video(input_path)
        profile_name = self.determine_profile(input_path)
        profile = self.quality_profiles[profile_name]
        
        # 获取视频信息
        video_stream = next(s for s in analysis['streams'] if s['codec_type'] == 'video')
        width = int(video_stream['width'])
        height = int(video_stream['height'])
        
        # 调整分辨率（如果太大）
        if width > 1920:
            scale_filter = "scale=1920:-2"
        elif width > 1280 and profile_name == 'small':
            scale_filter = "scale=1280:-2"
        else:
            scale_filter = None
        
        # 构建FFmpeg命令
        cmd = [
            'ffmpeg',
            '-i', str(input_path),
            '-c:v', 'libx264',
            '-crf', str(profile['crf']),
            '-preset', profile['preset'],
            '-c:a', 'aac',
            '-b:a', '128k'
        ]
        
        if scale_filter:
            cmd.extend(['-vf', scale_filter])
        
        cmd.extend([str(output_path), '-y'])
        
        # 执行压缩
        result = subprocess.run(cmd, capture_output=True)
        return result.returncode == 0

# 使用示例
compressor = IntelligentCompressor()
success = compressor.compress_video(
    Path("input_video.mp4"),
    Path("compressed_video.mp4")
)
```

## 高级自动化技术

### A. 工作流程自动化

#### 配置驱动的处理流程
```python
#!/usr/bin/env python3
"""
配置驱动的视频处理工作流程
"""

import yaml
from typing import Dict, List
from pathlib import Path

class WorkflowProcessor:
    def __init__(self, config_file: str):
        self.config = self.load_config(config_file)
        self.pipeline = self.build_pipeline()
    
    def load_config(self, config_file: str) -> Dict:
        """加载配置文件"""
        with open(config_file, 'r', encoding='utf-8') as f:
            return yaml.safe_load(f)
    
    def build_pipeline(self) -> List[Callable]:
        """构建处理管道"""
        pipeline = []
        for step in self.config['pipeline']:
            if step['type'] == 'convert':
                pipeline.append(self.create_converter(step['params']))
            elif step['type'] == 'compress':
                pipeline.append(self.create_compressor(step['params']))
            elif step['type'] == 'filter':
                pipeline.append(self.create_filter(step['params']))
        return pipeline
    
    def create_converter(self, params: Dict) -> Callable:
        """创建格式转换器"""
        def converter(input_path: Path, output_path: Path) -> bool:
            cmd = [
                'ffmpeg', '-i', str(input_path),
                '-c:v', params.get('video_codec', 'libx264'),
                '-c:a', params.get('audio_codec', 'aac'),
                str(output_path), '-y'
            ]
            result = subprocess.run(cmd, capture_output=True)
            return result.returncode == 0
        return converter
    
    def process_file(self, input_path: Path) -> bool:
        """处理单个文件"""
        current_file = input_path
        
        for i, processor in enumerate(self.pipeline):
            if i == len(self.pipeline) - 1:
                # 最后一步，输出到最终位置
                output_path = self.get_final_output_path(input_path)
            else:
                # 中间步骤，使用临时文件
                output_path = self.get_temp_path(input_path, i)
            
            if not processor(current_file, output_path):
                return False
            
            current_file = output_path
        
        return True

# 配置文件示例 (workflow.yaml)
workflow_config = """
pipeline:
  - type: convert
    params:
      video_codec: libx264
      audio_codec: aac
  - type: compress
    params:
      crf: 23
      preset: medium
  - type: filter
    params:
      filters:
        - "scale=1920:-2"
        - "fps=30"

input_patterns:
  - "*.mp4"
  - "*.avi"
  - "*.mov"

output:
  directory: "./processed"
  format: "mp4"
  naming: "{name}_processed.{ext}"
"""
```

### B. 监控和通知系统

#### 进度监控和报告
```python
#!/usr/bin/env python3
"""
进度监控和报告系统
"""

import time
import json
import smtplib
from email.mime.text import MimeText
from dataclasses import dataclass
from typing import Dict, List, Optional

@dataclass
class ProcessingTask:
    input_file: str
    output_file: str
    status: str  # pending, processing, completed, failed
    start_time: Optional[float] = None
    end_time: Optional[float] = None
    error_message: Optional[str] = None

class ProgressMonitor:
    def __init__(self, enable_notifications: bool = False):
        self.tasks: List[ProcessingTask] = []
        self.enable_notifications = enable_notifications
        self.start_time = time.time()
    
    def add_task(self, input_file: str, output_file: str):
        """添加任务"""
        task = ProcessingTask(input_file, output_file, "pending")
        self.tasks.append(task)
    
    def start_task(self, task_id: int):
        """开始任务"""
        self.tasks[task_id].status = "processing"
        self.tasks[task_id].start_time = time.time()
    
    def complete_task(self, task_id: int, success: bool = True, error: str = None):
        """完成任务"""
        task = self.tasks[task_id]
        task.status = "completed" if success else "failed"
        task.end_time = time.time()
        if error:
            task.error_message = error
    
    def get_progress_report(self) -> Dict:
        """生成进度报告"""
        total = len(self.tasks)
        completed = sum(1 for t in self.tasks if t.status == "completed")
        failed = sum(1 for t in self.tasks if t.status == "failed")
        processing = sum(1 for t in self.tasks if t.status == "processing")
        pending = sum(1 for t in self.tasks if t.status == "pending")
        
        elapsed_time = time.time() - self.start_time
        
        # 估算剩余时间
        if completed > 0:
            avg_time_per_task = elapsed_time / completed
            remaining_tasks = pending + processing
            estimated_remaining = avg_time_per_task * remaining_tasks
        else:
            estimated_remaining = 0
        
        return {
            'total_tasks': total,
            'completed': completed,
            'failed': failed,
            'processing': processing,
            'pending': pending,
            'progress_percentage': (completed / total * 100) if total > 0 else 0,
            'elapsed_time': elapsed_time,
            'estimated_remaining': estimated_remaining
        }
    
    def generate_report_html(self) -> str:
        """生成HTML报告"""
        report = self.get_progress_report()
        
        html = f"""
        <html>
        <head><title>批处理进度报告</title></head>
        <body>
            <h1>视频批处理进度报告</h1>
            <p>总任务数: {report['total_tasks']}</p>
            <p>已完成: {report['completed']}</p>
            <p>失败: {report['failed']}</p>
            <p>进行中: {report['processing']}</p>
            <p>等待中: {report['pending']}</p>
            <p>进度: {report['progress_percentage']:.1f}%</p>
            <p>已用时间: {report['elapsed_time']:.0f}秒</p>
            <p>预计剩余: {report['estimated_remaining']:.0f}秒</p>
            
            <h2>失败任务详情</h2>
            <ul>
        """
        
        for task in self.tasks:
            if task.status == "failed":
                html += f"<li>{task.input_file}: {task.error_message or '未知错误'}</li>"
        
        html += """
            </ul>
        </body>
        </html>
        """
        return html
    
    def send_notification(self, subject: str, message: str):
        """发送邮件通知"""
        if not self.enable_notifications:
            return
        
        # 这里实现邮件发送逻辑
        # 需要配置SMTP服务器信息
        pass

# 使用示例
monitor = ProgressMonitor(enable_notifications=True)

# 添加任务
for i in range(10):
    monitor.add_task(f"input_{i}.mp4", f"output_{i}.mp4")

# 模拟处理过程
for i in range(10):
    monitor.start_task(i)
    time.sleep(1)  # 模拟处理时间
    success = i < 8  # 模拟80%成功率
    monitor.complete_task(i, success, "示例错误" if not success else None)

# 生成报告
report = monitor.get_progress_report()
print(json.dumps(report, indent=2, ensure_ascii=False))
```

---

*批处理和自动化是提高视频处理效率的关键技术。通过合理的脚本设计和工作流程自动化，可以大幅度减少人工干预，提高处理质量的一致性。在实际应用中，要根据具体需求选择合适的技术方案和工具组合。*