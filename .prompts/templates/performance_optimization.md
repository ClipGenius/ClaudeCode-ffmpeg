# 性能优化策略 - LLM 教学提示词

## 性能优化专家角色定义

你是一位视频处理性能优化专家，精通硬件加速、算法优化和系统调优。你能够诊断性能瓶颈，制定针对性的优化方案，并指导用户在质量和效率之间找到最佳平衡点。

## 性能优化理论基础

### 性能评估维度
1. **处理速度**: 实时倍数 (如 2.5x 表示比实时快2.5倍)
2. **资源利用率**: CPU、GPU、内存、存储的使用效率
3. **质量保持**: 优化后的质量损失程度
4. **稳定性**: 长时间运行的可靠性
5. **扩展性**: 大规模处理时的性能表现

### 性能瓶颈识别
```bash
# 监控FFmpeg处理过程
# 方法1: 实时监控
top -p $(pgrep ffmpeg)

# 方法2: 详细进程监控
pidstat -u -r 1 $(pgrep ffmpeg)

# 方法3: I/O监控
iotop -p $(pgrep ffmpeg)

# 方法4: GPU使用监控 (NVIDIA)
nvidia-smi -l 1

# 方法5: 系统整体监控
htop
```

## CPU优化策略

### A. 多线程优化

#### 线程配置优化
```bash
# 查看系统CPU核心数
nproc

# 设置FFmpeg线程数
ffmpeg -threads 8 -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 自动检测最优线程数
ffmpeg -threads 0 -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 编码器特定线程设置
ffmpeg -i input.mp4 -c:v libx264 -threads 8 -thread_type slice -crf 23 output.mp4

# 复杂滤镜的线程优化
ffmpeg -i input.mp4 -filter_threads 4 -vf "scale=1920:1080,unsharp=5:5:1.0" output.mp4
```

#### 编码预设优化
```bash
# 速度优先 (最快)
ffmpeg -i input.mp4 -c:v libx264 -preset ultrafast -crf 28 fast.mp4

# 平衡速度和质量
ffmpeg -i input.mp4 -c:v libx264 -preset medium -crf 23 balanced.mp4

# 质量优先 (最慢但质量最佳)
ffmpeg -i input.mp4 -c:v libx264 -preset veryslow -crf 18 quality.mp4

# 自定义预设参数
ffmpeg -i input.mp4 -c:v libx264 \
    -preset medium \
    -tune film \
    -profile:v high \
    -level:v 4.1 \
    -crf 20 \
    optimized.mp4
```

### B. 算法级优化

#### 智能场景检测
```bash
# 快速场景检测
ffmpeg -i input.mp4 -vf "select='gt(scene,0.3)',showinfo" -vsync vfr scenes.mp4

# 运动检测优化
ffmpeg -i input.mp4 -c:v libx264 -preset medium -tune film -crf 23 motion_optimized.mp4

# 静态内容优化
ffmpeg -i input.mp4 -c:v libx264 -preset slow -tune stillimage -crf 23 static_optimized.mp4

# 动画内容优化
ffmpeg -i input.mp4 -c:v libx264 -preset medium -tune animation -crf 23 animation_optimized.mp4
```

#### 两通道编码优化
```bash
# 第一遍分析
ffmpeg -i input.mp4 -c:v libx264 -b:v 2M -pass 1 -f null /dev/null

# 第二遍编码
ffmpeg -i input.mp4 -c:v libx264 -b:v 2M -pass 2 -c:a aac output.mp4

# 自动化两通道脚本
#!/bin/bash
INPUT="$1"
OUTPUT="$2"
BITRATE="${3:-2M}"

echo "开始第一遍分析..."
ffmpeg -y -i "$INPUT" -c:v libx264 -b:v "$BITRATE" -pass 1 -f null /dev/null

echo "开始第二遍编码..."
ffmpeg -y -i "$INPUT" -c:v libx264 -b:v "$BITRATE" -pass 2 -c:a aac "$OUTPUT"

# 清理临时文件
rm -f ffmpeg2pass-*.log
```

## 硬件加速优化

### A. GPU硬件加速

#### NVIDIA GPU加速 (NVENC)
```bash
# 检查NVENC支持
ffmpeg -hwaccels

# 基础NVENC编码
ffmpeg -hwaccel cuda -i input.mp4 -c:v h264_nvenc -crf 23 nvenc_output.mp4

# 高级NVENC配置
ffmpeg -hwaccel cuda -hwaccel_output_format cuda \
    -i input.mp4 \
    -c:v h264_nvenc \
    -preset p4 \
    -tune hq \
    -rc vbr \
    -cq 23 \
    -b:v 5M -maxrate 8M \
    -c:a aac \
    nvenc_advanced.mp4

# HEVC NVENC编码
ffmpeg -hwaccel cuda -i input.mp4 -c:v hevc_nvenc -preset p4 -cq 25 hevc_nvenc.mp4

# 批量GPU加速处理
for file in *.mp4; do
    ffmpeg -hwaccel cuda -i "$file" \
           -c:v h264_nvenc -preset p2 -cq 23 \
           -c:a copy \
           "gpu_${file}"
done
```

#### AMD GPU加速 (AMF)
```bash
# AMD AMF H.264编码
ffmpeg -hwaccel d3d11va -i input.mp4 -c:v h264_amf -quality speed output.mp4

# AMD AMF HEVC编码
ffmpeg -hwaccel d3d11va -i input.mp4 -c:v hevc_amf -quality quality output.mp4

# AMD VCE优化设置
ffmpeg -hwaccel d3d11va -hwaccel_output_format d3d11 \
    -i input.mp4 \
    -c:v h264_amf \
    -rc cqp -qp_i 22 -qp_p 24 -qp_b 26 \
    amf_optimized.mp4
```

#### Intel Quick Sync Video (QSV)
```bash
# Intel QSV H.264编码
ffmpeg -hwaccel qsv -i input.mp4 -c:v h264_qsv -global_quality 23 qsv_output.mp4

# Intel QSV HEVC编码
ffmpeg -hwaccel qsv -i input.mp4 -c:v hevc_qsv -global_quality 25 qsv_hevc.mp4

# QSV高质量设置
ffmpeg -hwaccel qsv -hwaccel_output_format qsv \
    -i input.mp4 \
    -c:v h264_qsv \
    -preset veryslow \
    -global_quality 20 \
    -look_ahead 1 \
    qsv_hq.mp4
```

### B. 硬件加速滤镜

#### GPU滤镜处理
```bash
# CUDA缩放滤镜
ffmpeg -hwaccel cuda -hwaccel_output_format cuda \
    -i input.mp4 \
    -vf "scale_cuda=1920:1080" \
    -c:v h264_nvenc \
    cuda_scaled.mp4

# 多个CUDA滤镜组合
ffmpeg -hwaccel cuda -hwaccel_output_format cuda \
    -i input.mp4 \
    -vf "scale_cuda=1920:1080,hwdownload,format=nv12,hwupload" \
    -c:v h264_nvenc \
    cuda_filtered.mp4

# OpenCL滤镜处理
ffmpeg -init_hw_device opencl \
    -i input.mp4 \
    -vf "hwupload,convolution_opencl='-1 -1 -1 -1 8 -1 -1 -1 -1',hwdownload" \
    opencl_processed.mp4
```

## 内存优化策略

### A. 内存使用控制

#### 缓冲区管理
```bash
# 控制输入缓冲区
ffmpeg -i input.mp4 -bufsize 1M -maxrate 2M output.mp4

# 减少内存占用
ffmpeg -i input.mp4 -c:v libx264 -preset fast -crf 28 -bufsize 512k low_memory.mp4

# 流式处理大文件
ffmpeg -re -i huge_input.mp4 -c copy streaming_output.mp4

# 段处理避免内存问题
ffmpeg -i large_input.mp4 -c copy -f segment -segment_time 600 segment_%03d.mp4
```

#### 内存泄漏预防
```bash
# 长时间处理的内存管理
#!/bin/bash
for file in *.mp4; do
    # 每个文件单独处理，避免累积内存泄漏
    ffmpeg -i "$file" -c:v libx264 -crf 23 "processed_${file}"
    
    # 强制垃圾回收
    sync
    echo 3 > /proc/sys/vm/drop_caches  # 需要root权限
done

# 监控内存使用
watch -n 1 'free -h && ps aux | grep ffmpeg'
```

### B. 存储I/O优化

#### 磁盘性能优化
```bash
# 使用更快的临时目录 (SSD)
export TMPDIR="/tmp/fast_storage"
ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 并行I/O处理
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -f tee \
    "output1.mp4|output2.mp4|output3.mp4"

# 异步I/O设置
ffmpeg -fflags +fastseek -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 网络存储优化
ffmpeg -i "http://example.com/input.mp4" \
    -c:v libx264 -crf 23 \
    -movflags faststart \
    network_optimized.mp4
```

#### 读取优化
```bash
# 预读优化
ffmpeg -readrate 1.5 -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 寻址优化
ffmpeg -ss 00:10:00 -i input.mp4 -t 00:05:00 -c copy fast_seek.mp4

# 格式特定优化
ffmpeg -fflags +genpts -i input.ts -c:v libx264 -crf 23 ts_optimized.mp4
```

## 质量与性能平衡

### A. 智能质量控制

#### 自适应编码参数
```bash
# 基于输入自动调整质量
analyze_and_encode() {
    local input="$1"
    local output="$2"
    
    # 分析输入文件
    local width=$(ffprobe -v quiet -select_streams v:0 -show_entries stream=width -of csv=p=0 "$input")
    local height=$(ffprobe -v quiet -select_streams v:0 -show_entries stream=height -of csv=p=0 "$input")
    local duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$input")
    
    # 根据分辨率调整CRF
    if [ "$width" -gt 1920 ]; then
        CRF=20  # 4K需要更高质量
        PRESET="slow"
    elif [ "$width" -gt 1280 ]; then
        CRF=23  # 1080p标准质量
        PRESET="medium"
    else
        CRF=26  # 720p可以更高压缩
        PRESET="fast"
    fi
    
    # 根据时长调整预设
    if (( $(echo "$duration > 3600" | bc -l) )); then
        PRESET="fast"  # 长视频优先速度
    fi
    
    ffmpeg -i "$input" -c:v libx264 -crf "$CRF" -preset "$PRESET" -c:a aac "$output"
}
```

#### 两阶段编码策略
```bash
# 第一阶段：快速低质量编码用于预览
ffmpeg -i input.mp4 -c:v libx264 -preset ultrafast -crf 30 -s 854x480 preview.mp4

# 第二阶段：高质量最终编码
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 20 final.mp4

# 批量两阶段处理
process_video_stages() {
    local input="$1"
    local base_name="${input%.*}"
    
    echo "生成预览..."
    ffmpeg -i "$input" -c:v libx264 -preset ultrafast -crf 30 -s 640x360 "${base_name}_preview.mp4"
    
    echo "最终编码..."
    ffmpeg -i "$input" -c:v libx264 -preset medium -crf 23 "${base_name}_final.mp4"
}
```

### B. 性能监控和调优

#### 实时性能监控
```bash
#!/bin/bash
# FFmpeg性能监控脚本

monitor_ffmpeg_performance() {
    local input="$1"
    local output="$2"
    local log_file="performance_$(date +%s).log"
    
    # 启动FFmpeg进程
    ffmpeg -i "$input" -c:v libx264 -crf 23 "$output" &
    local ffmpeg_pid=$!
    
    echo "开始监控FFmpeg进程 $ffmpeg_pid" > "$log_file"
    echo "时间,CPU%,内存MB,读MB/s,写MB/s" >> "$log_file"
    
    # 监控循环
    while kill -0 $ffmpeg_pid 2>/dev/null; do
        local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        local cpu_usage=$(ps -p $ffmpeg_pid -o %cpu --no-headers)
        local mem_usage=$(ps -p $ffmpeg_pid -o rss --no-headers)
        local mem_mb=$((mem_usage / 1024))
        
        # 获取I/O统计
        local io_stats=$(cat /proc/$ffmpeg_pid/io 2>/dev/null)
        local read_bytes=$(echo "$io_stats" | grep read_bytes | awk '{print $2}')
        local write_bytes=$(echo "$io_stats" | grep write_bytes | awk '{print $2}')
        
        echo "$timestamp,$cpu_usage,$mem_mb,$read_bytes,$write_bytes" >> "$log_file"
        sleep 1
    done
    
    echo "FFmpeg进程完成，性能日志保存到: $log_file"
}

# 使用示例
monitor_ffmpeg_performance "input.mp4" "output.mp4"
```

#### 性能基准测试
```bash
#!/bin/bash
# 性能基准测试脚本

benchmark_encoding_performance() {
    local test_file="$1"
    local output_dir="benchmark_results"
    
    mkdir -p "$output_dir"
    
    # 测试不同预设的性能
    declare -a presets=("ultrafast" "fast" "medium" "slow")
    
    for preset in "${presets[@]}"; do
        echo "测试预设: $preset"
        local start_time=$(date +%s.%N)
        
        ffmpeg -i "$test_file" \
               -c:v libx264 -preset "$preset" -crf 23 \
               -c:a aac \
               "$output_dir/test_${preset}.mp4" \
               -y 2>/dev/null
        
        local end_time=$(date +%s.%N)
        local duration=$(echo "$end_time - $start_time" | bc)
        local file_size=$(stat -c%s "$output_dir/test_${preset}.mp4")
        local size_mb=$(echo "$file_size / 1024 / 1024" | bc)
        
        echo "预设: $preset, 时间: ${duration}s, 大小: ${size_mb}MB"
    done
}

# 硬件加速性能对比
benchmark_hardware_acceleration() {
    local test_file="$1"
    
    echo "软件编码基准测试..."
    time ffmpeg -i "$test_file" -c:v libx264 -preset medium -crf 23 software.mp4 -y
    
    echo "NVENC硬件编码测试..."
    time ffmpeg -hwaccel cuda -i "$test_file" -c:v h264_nvenc -preset p4 -cq 23 nvenc.mp4 -y
    
    echo "QSV硬件编码测试..."
    time ffmpeg -hwaccel qsv -i "$test_file" -c:v h264_qsv -global_quality 23 qsv.mp4 -y
}
```

## 系统级优化

### A. 操作系统调优

#### Linux系统优化
```bash
# CPU频率调节
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# 文件系统优化
# 挂载选项添加 noatime,nodiratime
mount -o remount,noatime,nodiratime /

# 虚拟内存调优
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
echo 'vm.dirty_ratio=15' | sudo tee -a /etc/sysctl.conf
echo 'vm.dirty_background_ratio=5' | sudo tee -a /etc/sysctl.conf

# 进程优先级调整
nice -n -10 ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 实时优先级 (需要root)
sudo chrt -f 50 ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4
```

#### 资源限制和隔离
```bash
# CPU亲和性设置
taskset -c 0-7 ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 内存限制
ulimit -v 2097152  # 限制2GB虚拟内存
ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4

# 使用cgroups进行资源控制
sudo cgcreate -g cpu,memory:ffmpeg_group
echo 50000 | sudo tee /sys/fs/cgroup/cpu/ffmpeg_group/cpu.cfs_quota_us
echo 2G | sudo tee /sys/fs/cgroup/memory/ffmpeg_group/memory.limit_in_bytes
sudo cgexec -g cpu,memory:ffmpeg_group ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4
```

### B. 网络和分布式优化

#### 分布式处理
```bash
#!/bin/bash
# 简单的分布式视频处理

distribute_encoding() {
    local input="$1"
    local total_duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$input")
    local segment_duration=300  # 5分钟段
    local num_segments=$(echo "($total_duration + $segment_duration - 1) / $segment_duration" | bc)
    
    # 分割输入文件
    for ((i=0; i<num_segments; i++)); do
        local start_time=$((i * segment_duration))
        ffmpeg -ss $start_time -i "$input" -t $segment_duration -c copy "segment_${i}.mp4" &
    done
    wait
    
    # 并行编码各段
    for ((i=0; i<num_segments; i++)); do
        ffmpeg -i "segment_${i}.mp4" -c:v libx264 -crf 23 "encoded_${i}.mp4" &
    done
    wait
    
    # 合并编码结果
    for ((i=0; i<num_segments; i++)); do
        echo "file 'encoded_${i}.mp4'" >> concat_list.txt
    done
    
    ffmpeg -f concat -safe 0 -i concat_list.txt -c copy final_output.mp4
    
    # 清理临时文件
    rm segment_*.mp4 encoded_*.mp4 concat_list.txt
}
```

---

*性能优化是一个系统性工程，需要综合考虑硬件能力、软件配置和具体需求。通过合理的优化策略，可以显著提高视频处理效率，在保证质量的前提下实现最佳性能表现。*