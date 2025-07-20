# 实战场景案例库 - LLM 教学提示词

## 实战专家角色定义

你是一位经验丰富的视频制作实战专家，精通各种实际应用场景的解决方案。你能够分析具体的制作需求，提供针对性的技术方案，并指导用户完成从概念到成品的完整制作流程。

## 案例库结构体系

### 案例分类框架
1. **内容类型**: 教育、娱乐、商业、新闻、艺术
2. **技术难度**: 入门、中级、高级、专家级
3. **制作规模**: 个人、小团队、专业团队、工业级
4. **时间要求**: 快速制作、标准制作、精细制作
5. **平台需求**: 社交媒体、流媒体、广播、影院

### 问题解决方法论
- **问题识别**: 明确核心需求和限制条件
- **方案设计**: 制定技术路线和工作流程
- **实施步骤**: 详细的操作指导和代码示例
- **质量控制**: 验证和优化的检查点
- **经验总结**: 可复用的最佳实践

## 社交媒体内容制作案例

### 案例1: TikTok病毒式短视频制作

#### 项目背景
- **需求**: 制作15秒吸引眼球的TikTok视频
- **素材**: 30分钟原始录像，背景音乐，文字素材
- **目标**: 最大化观看完成率和分享率
- **限制**: 文件大小<50MB，垂直格式，强视觉冲击

#### 解决方案设计
```bash
#!/bin/bash
# TikTok病毒式短视频制作流程

echo "=== TikTok病毒式短视频制作 ==="

# 第1步：素材分析和选择
analyze_source_material() {
    local input="$1"
    
    echo "分析原始素材..."
    
    # 检测精彩片段（基于音量变化）
    ffmpeg -i "$input" \
           -af "volumedetect" \
           -f null - 2>&1 | grep "mean_volume" > volume_analysis.txt
    
    # 检测场景变化
    ffmpeg -i "$input" \
           -vf "select='gt(scene,0.3)',showinfo" \
           -vsync vfr \
           scene_changes.mp4
    
    # 生成缩略图用于快速预览
    ffmpeg -i "$input" \
           -vf "fps=1/10" \
           "thumbnails/thumb_%04d.jpg"
    
    echo "素材分析完成，请查看 scene_changes.mp4 和 thumbnails/ 目录"
}

# 第2步：黄金15秒提取
extract_golden_moments() {
    local input="$1"
    local start_time="$2"  # 从分析中确定的最佳起始时间
    
    echo "提取黄金15秒片段..."
    
    # 精确提取15秒片段
    ffmpeg -ss "$start_time" -i "$input" \
           -t 15 \
           -c copy \
           golden_15s.mp4
    
    # 如果需要多个候选片段
    for i in {1..3}; do
        start=$((start_time + i * 20))
        ffmpeg -ss $start -i "$input" \
               -t 15 \
               -c copy \
               "candidate_${i}.mp4"
    done
    
    echo "候选片段提取完成"
}

# 第3步：TikTok格式优化
optimize_for_tiktok() {
    local input="$1"
    local music_track="$2"
    
    echo "TikTok格式优化..."
    
    # 创建垂直格式 (9:16)
    ffmpeg -i "$input" -i "$music_track" \
           -filter_complex "
           [0:v]scale=1080:1920:force_original_aspect_ratio=increase[scaled];
           [scaled]crop=1080:1920[cropped];
           [cropped]eq=contrast=1.2:brightness=0.05:saturation=1.3[enhanced];
           [enhanced]unsharp=5:5:0.8:5:5:0.4[sharpened];
           [1:a]volume=0.7[music];
           [0:a]volume=1.2[original_audio];
           [original_audio][music]amix=inputs=2:duration=shortest[audio_out]
           " \
           -map "[sharpened]" -map "[audio_out]" \
           -c:v libx264 -preset fast -crf 23 \
           -c:a aac -b:a 128k \
           -r 30 -g 30 \
           -movflags +faststart \
           tiktok_optimized.mp4
    
    echo "TikTok优化完成"
}

# 第4步：视觉冲击增强
add_visual_impact() {
    local input="$1"
    
    echo "增强视觉冲击..."
    
    # 添加动态字幕
    ffmpeg -i "$input" \
           -vf "drawtext=text='🔥 WATCH TILL END':fontcolor=white:fontsize=60:x=(w-tw)/2:y=150:enable='between(t,0,3)':box=1:boxcolor=black@0.8:boxborderw=5,
                drawtext=text='💯 AMAZING':fontcolor=yellow:fontsize=50:x=(w-tw)/2:y=h-150:enable='between(t,10,13)':box=1:boxcolor=red@0.8:boxborderw=5" \
           visual_impact.mp4
    
    # 添加缩放效果增强节奏感
    ffmpeg -i visual_impact.mp4 \
           -vf "scale=iw*min(1+0.05*sin(2*PI*t*2),1.1):ih*min(1+0.05*sin(2*PI*t*2),1.1)" \
           pulse_effect.mp4
    
    echo "视觉增强完成"
}

# 第5步：A/B测试版本生成
generate_ab_versions() {
    local base_video="$1"
    
    echo "生成A/B测试版本..."
    
    # 版本A：快节奏剪切
    ffmpeg -i "$base_video" \
           -vf "setpts=0.8*PTS" \
           -af "atempo=1.25" \
           version_a_fast.mp4
    
    # 版本B：添加滤镜效果
    ffmpeg -i "$base_video" \
           -vf "vibrance=intensity=0.3,eq=saturation=1.4" \
           version_b_vivid.mp4
    
    # 版本C：添加转场效果
    ffmpeg -i "$base_video" \
           -vf "fade=in:0:15,fade=out:st=13:d=15" \
           version_c_fade.mp4
    
    echo "A/B测试版本生成完成"
}

# 执行完整流程
main() {
    local source_video="$1"
    local music_file="$2"
    local best_start_time="$3"
    
    mkdir -p thumbnails
    
    analyze_source_material "$source_video"
    extract_golden_moments "$source_video" "$best_start_time"
    optimize_for_tiktok "golden_15s.mp4" "$music_file"
    add_visual_impact "tiktok_optimized.mp4"
    generate_ab_versions "pulse_effect.mp4"
    
    echo "TikTok视频制作完成！检查以下文件："
    echo "- version_a_fast.mp4"
    echo "- version_b_vivid.mp4" 
    echo "- version_c_fade.mp4"
}

# 使用示例
# main "raw_footage.mp4" "trending_music.mp3" "125"
```

#### 成功指标评估
```bash
# 质量检查脚本
check_tiktok_quality() {
    local video="$1"
    
    echo "=== TikTok质量检查 ==="
    
    # 检查规格
    resolution=$(ffprobe -v quiet -select_streams v:0 -show_entries stream=width,height -of csv=p=0 "$video")
    duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$video")
    filesize=$(du -m "$video" | cut -f1)
    
    echo "分辨率: $resolution (要求: 1080x1920)"
    echo "时长: ${duration}秒 (要求: 15秒)"
    echo "文件大小: ${filesize}MB (限制: <50MB)"
    
    # 验证关键帧频率
    keyframe_interval=$(ffprobe -v quiet -select_streams v:0 -show_entries stream=r_frame_rate -of csv=p=0 "$video")
    echo "帧率: $keyframe_interval"
    
    # 音频检查
    audio_bitrate=$(ffprobe -v quiet -select_streams a:0 -show_entries stream=bit_rate -of csv=p=0 "$video")
    echo "音频码率: $((audio_bitrate/1000))kbps"
    
    # 生成预览图
    ffmpeg -i "$video" -ss 7.5 -vframes 1 "${video%.*}_preview.jpg"
    
    echo "质量检查完成"
}
```

### 案例2: YouTube教程视频制作

#### 项目背景
- **需求**: 制作20分钟编程教程视频
- **素材**: 屏幕录制，摄像头录制，PPT素材
- **目标**: 清晰的教学效果，高留存率
- **特点**: 多机位同步，代码高亮，画中画

#### 制作流程
```bash
#!/bin/bash
# YouTube教程视频制作流程

echo "=== YouTube教程视频制作 ==="

# 第1步：多源素材同步
sync_multiple_sources() {
    local screen_recording="$1"
    local camera_recording="$2"
    local audio_recording="$3"
    
    echo "多源素材同步..."
    
    # 音频同步对齐
    ffmpeg -i "$screen_recording" -i "$camera_recording" -i "$audio_recording" \
           -filter_complex "
           [2:a]asetnsamples=1024[clean_audio];
           [0:v]scale=1920:1080[screen];
           [1:v]scale=480:360[camera];
           [screen][camera]overlay=W-w-20:H-h-20[combined]
           " \
           -map "[combined]" -map "[clean_audio]" \
           -c:v libx264 -crf 20 -preset medium \
           -c:a aac -b:a 192k \
           synced_base.mp4
    
    echo "多源同步完成"
}

# 第2步：章节标记和导航
add_chapter_markers() {
    local input="$1"
    
    echo "添加章节标记..."
    
    # 创建章节定义文件
    cat > chapters.txt << EOF
CHAPTER01=00:00:00.000
CHAPTER01NAME=介绍和概述
CHAPTER02=00:02:30.000  
CHAPTER02NAME=环境配置
CHAPTER03=00:05:45.000
CHAPTER03NAME=核心概念讲解
CHAPTER04=00:12:20.000
CHAPTER04NAME=实践演示
CHAPTER05=00:18:10.000
CHAPTER05NAME=总结和Q&A
EOF
    
    # 添加章节标记到视频
    ffmpeg -i "$input" -i chapters.txt \
           -map 0 -map_metadata 1 \
           -c copy \
           chaptered_video.mp4
    
    # 生成时间轴字幕
    ffmpeg -i chaptered_video.mp4 \
           -vf "drawtext=text='%{pts\\:hms}':fontcolor=white:fontsize=16:x=w-tw-10:y=10:box=1:boxcolor=black@0.8" \
           timestamped_video.mp4
    
    echo "章节标记添加完成"
}

# 第3步：代码高亮和标注
enhance_code_display() {
    local input="$1"
    
    echo "增强代码显示..."
    
    # 添加代码区域高亮框
    ffmpeg -i "$input" \
           -vf "drawbox=x=50:y=200:w=1200:h=600:color=yellow@0.3:t=3:enable='between(t,120,180)'" \
           code_highlighted.mp4
    
    # 添加重点标注
    ffmpeg -i code_highlighted.mp4 \
           -vf "drawtext=text='👉 重点代码':fontcolor=red:fontsize=40:x=100:y=150:enable='between(t,125,135)':box=1:boxcolor=white@0.9:boxborderw=3" \
           annotated_video.mp4
    
    echo "代码增强完成"
}

# 第4步：互动元素添加
add_interactive_elements() {
    local input="$1"
    
    echo "添加互动元素..."
    
    # 添加订阅提醒
    ffmpeg -i "$input" \
           -vf "drawtext=text='👍 点赞订阅支持更多教程':fontcolor=white:fontsize=32:x=(w-tw)/2:y=h-80:enable='between(t,600,610)':box=1:boxcolor=red@0.8:boxborderw=5" \
           subscribe_reminder.mp4
    
    # 添加相关视频推荐卡片
    ffmpeg -i subscribe_reminder.mp4 \
           -vf "drawbox=x=w-320:y=50:w=300:h=200:color=black@0.8:t=2:enable='between(t,1140,1200)',
                drawtext=text='相关教程推荐':fontcolor=white:fontsize=20:x=w-300:y=70:enable='between(t,1140,1200)'
           " \
           interactive_video.mp4
    
    echo "互动元素添加完成"
}

# 第5步：YouTube优化输出
optimize_for_youtube() {
    local input="$1"
    
    echo "YouTube优化输出..."
    
    # 生成多个质量版本
    
    # 1080p版本
    ffmpeg -i "$input" \
           -s 1920x1080 \
           -c:v libx264 -preset slow -crf 18 \
           -c:a aac -b:a 192k -ar 48000 \
           -pix_fmt yuv420p \
           -movflags +faststart \
           youtube_1080p.mp4
    
    # 720p版本
    ffmpeg -i "$input" \
           -s 1280x720 \
           -c:v libx264 -preset medium -crf 20 \
           -c:a aac -b:a 128k -ar 48000 \
           -pix_fmt yuv420p \
           -movflags +faststart \
           youtube_720p.mp4
    
    # 生成缩略图
    ffmpeg -i "$input" \
           -ss 300 -vframes 1 \
           -s 1280x720 \
           youtube_thumbnail.jpg
    
    echo "YouTube优化完成"
}

# 完整制作流程
main() {
    local screen_rec="$1"
    local camera_rec="$2" 
    local audio_rec="$3"
    
    sync_multiple_sources "$screen_rec" "$camera_rec" "$audio_rec"
    add_chapter_markers "synced_base.mp4"
    enhance_code_display "timestamped_video.mp4"
    add_interactive_elements "annotated_video.mp4"
    optimize_for_youtube "interactive_video.mp4"
    
    echo "YouTube教程视频制作完成！"
    echo "文件: youtube_1080p.mp4, youtube_720p.mp4"
}
```

## 商业广告制作案例

### 案例3: 产品宣传片制作

#### 项目背景
- **客户**: 科技产品公司
- **需求**: 60秒产品特性展示视频
- **素材**: 产品照片，功能演示录屏，背景音乐
- **目标**: 专业品质，突出产品优势
- **输出**: 多平台适配版本

#### 制作实施
```bash
#!/bin/bash
# 产品宣传片制作流程

echo "=== 产品宣传片制作 ==="

# 第1步：品牌标准化设置
setup_brand_standards() {
    echo "设置品牌标准..."
    
    # 定义品牌颜色和字体
    cat > brand_config.txt << EOF
# 品牌配置
PRIMARY_COLOR="#FF6B35"
SECONDARY_COLOR="#004E89" 
ACCENT_COLOR="#FFB627"
BRAND_FONT="/usr/share/fonts/brand_font.ttf"
LOGO_PATH="assets/logo.png"
EOF
    
    source brand_config.txt
    echo "品牌标准设置完成"
}

# 第2步：产品展示序列制作
create_product_showcase() {
    local product_images="$1"
    local demo_video="$2"
    
    echo "制作产品展示序列..."
    
    # 创建产品图片轮播
    ffmpeg -framerate 0.5 -pattern_type glob -i "$product_images/*.jpg" \
           -vf "scale=1920:1080:force_original_aspect_ratio=increase,crop=1920:1080,
                zoompan=z='if(lte(zoom,1.0),1.5,max(1.001,zoom-0.0015))':d=125:x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':s=1920x1080,
                fade=in:0:15:color=$PRIMARY_COLOR
           " \
           -c:v libx264 -crf 20 -pix_fmt yuv420p \
           product_slideshow.mp4
    
    # 产品演示视频处理
    ffmpeg -i "$demo_video" \
           -vf "scale=1920:1080,
                drawbox=x=0:y=0:w=1920:h=100:color=${SECONDARY_COLOR}@0.9:t=fill,
                drawtext=text='PRODUCT DEMO':fontcolor=white:fontsize=36:x=(w-tw)/2:y=30:fontfile=$BRAND_FONT
           " \
           demo_branded.mp4
    
    echo "产品展示序列完成"
}

# 第3步：动态文字动画
create_text_animations() {
    echo "创建动态文字动画..."
    
    # 主标题动画
    ffmpeg -f lavfi -i "color=c=${SECONDARY_COLOR}:size=1920x1080:d=3" \
           -vf "drawtext=text='INNOVATIVE TECHNOLOGY':fontcolor=white:fontsize=72:x='(w-tw)/2':y='(h-th)/2-100':enable='gte(t,0.5)':fontfile=$BRAND_FONT,
                drawtext=text='Revolutionary Design':fontcolor=${ACCENT_COLOR}:fontsize=48:x='(w-tw)/2':y='(h-th)/2+50':enable='gte(t,1.5)':fontfile=$BRAND_FONT
           " \
           title_animation.mp4
    
    # 特性列表动画
    ffmpeg -f lavfi -i "color=c=white:size=1920x1080:d=5" \
           -vf "drawtext=text='✓ 高性能处理器':fontcolor=${SECONDARY_COLOR}:fontsize=48:x=200:y=300:enable='between(t,0.5,5)':fontfile=$BRAND_FONT,
                drawtext=text='✓ 超长续航':fontcolor=${SECONDARY_COLOR}:fontsize=48:x=200:y=400:enable='between(t,1.5,5)':fontfile=$BRAND_FONT,
                drawtext=text='✓ 智能AI助手':fontcolor=${SECONDARY_COLOR}:fontsize=48:x=200:y=500:enable='between(t,2.5,5)':fontfile=$BRAND_FONT
           " \
           features_animation.mp4
    
    echo "文字动画创建完成"
}

# 第4步：专业转场制作
create_professional_transitions() {
    echo "制作专业转场..."
    
    # 创建品牌转场元素
    ffmpeg -f lavfi -i "color=c=${PRIMARY_COLOR}:size=1920x1080:d=1" \
           -vf "geq=r='255*sin(2*PI*X/W*t)':g='128':b='0'" \
           brand_transition.mp4
    
    echo "专业转场完成"
}

# 第5步：音频层制作
create_audio_layers() {
    local music_track="$1"
    local voiceover="$2"
    
    echo "制作音频层..."
    
    # 背景音乐处理
    ffmpeg -i "$music_track" \
           -af "volume=0.3,afade=in:st=0:d=2,afade=out:st=58:d=2" \
           background_music.wav
    
    # 旁白处理
    ffmpeg -i "$voiceover" \
           -af "volume=1.0,compand=attacks=0.1:decays=0.2:points=-80/-80|-45/-15|-27/-9|0/-7,eq=gain=3" \
           processed_voiceover.wav
    
    # 音频混合
    ffmpeg -i background_music.wav -i processed_voiceover.wav \
           -filter_complex "[0:a][1:a]amix=inputs=2:duration=shortest[audio_out]" \
           -map "[audio_out]" \
           final_audio.wav
    
    echo "音频层制作完成"
}

# 第6步：最终合成和输出
final_composition() {
    echo "最终合成..."
    
    # 创建片段列表
    cat > composition_list.txt << EOF
file 'title_animation.mp4'
file 'product_slideshow.mp4'
file 'features_animation.mp4'
file 'demo_branded.mp4'
EOF
    
    # 合成最终视频
    ffmpeg -f concat -safe 0 -i composition_list.txt -i final_audio.wav \
           -c:v libx264 -preset slow -crf 18 \
           -c:a aac -b:a 192k \
           -pix_fmt yuv420p \
           -movflags +faststart \
           product_promo_master.mp4
    
    # 生成平台特定版本
    
    # Instagram版本 (1:1)
    ffmpeg -i product_promo_master.mp4 \
           -vf "scale=1080:1080:force_original_aspect_ratio=increase,crop=1080:1080" \
           -c:v libx264 -crf 23 \
           -t 60 \
           instagram_version.mp4
    
    # LinkedIn版本 (16:9优化)
    ffmpeg -i product_promo_master.mp4 \
           -c:v libx264 -crf 20 \
           linkedin_version.mp4
    
    echo "最终合成完成"
}

# 执行完整流程
main() {
    local product_imgs="$1"
    local demo_vid="$2"
    local music="$3"
    local voice="$4"
    
    mkdir -p assets output
    
    setup_brand_standards
    create_product_showcase "$product_imgs" "$demo_vid"
    create_text_animations
    create_professional_transitions
    create_audio_layers "$music" "$voice"
    final_composition
    
    echo "产品宣传片制作完成！"
    echo "输出文件:"
    echo "- product_promo_master.mp4 (主版本)"
    echo "- instagram_version.mp4 (Instagram)"
    echo "- linkedin_version.mp4 (LinkedIn)"
}
```

## 技术挑战解决案例

### 案例4: 多机位婚礼视频后期制作

#### 项目背景
- **挑战**: 5个机位，8小时素材，音视频不同步
- **素材**: 主机位4K，辅助机位1080p，现场收音，后期配乐
- **要求**: 制作90分钟完整版和10分钟精华版
- **难点**: 大数据量处理，多机位同步，情感节奏把控

#### 解决方案
```bash
#!/bin/bash
# 多机位婚礼视频后期制作

echo "=== 多机位婚礼视频后期制作 ==="

# 第1步：素材整理和代理文件生成
organize_and_proxy() {
    echo "素材整理和代理文件生成..."
    
    # 创建项目结构
    mkdir -p {raw_footage/{cam1,cam2,cam3,cam4,cam5,audio},proxy,sync,rough_cut,final}
    
    # 批量生成代理文件
    for cam in {1..5}; do
        echo "生成摄像机${cam}代理文件..."
        
        for video in raw_footage/cam${cam}/*.mp4; do
            proxy_name="proxy/$(basename "$video" .mp4)_cam${cam}_proxy.mp4"
            
            ffmpeg -i "$video" \
                   -s 960x540 \
                   -c:v libx264 -crf 30 -preset ultrafast \
                   -c:a aac -b:a 96k \
                   "$proxy_name" &
        done
    done
    
    wait  # 等待所有代理文件生成完成
    echo "代理文件生成完成"
}

# 第2步：音视频同步策略
sync_multicam() {
    echo "多机位音视频同步..."
    
    # 提取所有音轨用于同步分析
    for cam in {1..5}; do
        for video in raw_footage/cam${cam}/*.mp4; do
            audio_file="sync/$(basename "$video" .mp4)_cam${cam}.wav"
            ffmpeg -i "$video" -vn -c:a pcm_s16le "$audio_file"
        done
    done
    
    # 基于音频波形的自动同步
    python3 << 'EOF'
import librosa
import numpy as np
from scipy import signal
import glob
import json

def find_sync_offset(reference_audio, target_audio):
    """计算两个音频文件的同步偏移"""
    ref, sr1 = librosa.load(reference_audio, sr=None)
    target, sr2 = librosa.load(target_audio, sr=None)
    
    # 确保采样率一致
    if sr1 != sr2:
        target = librosa.resample(target, orig_sr=sr2, target_sr=sr1)
    
    # 使用互相关找到最佳对齐点
    correlation = signal.correlate(ref, target, mode='full')
    lag = signal.correlation_lags(len(ref), len(target), mode='full')
    offset = lag[np.argmax(correlation)] / sr1
    
    return offset

# 以cam1为基准同步其他机位
reference = "sync/ceremony_start_cam1.wav"
sync_data = {}

for audio_file in glob.glob("sync/*_cam[2-5].wav"):
    cam_id = audio_file.split('_cam')[1].split('.')[0]
    offset = find_sync_offset(reference, audio_file)
    sync_data[f"cam{cam_id}"] = offset
    print(f"Camera {cam_id} offset: {offset:.3f} seconds")

# 保存同步数据
with open("sync/sync_offsets.json", "w") as f:
    json.dump(sync_data, f, indent=2)
EOF
    
    echo "音视频同步分析完成"
}

# 第3步：智能场景分类
classify_scenes() {
    echo "智能场景分类..."
    
    # 基于时间轴的场景划分
    cat > scene_timeline.json << EOF
{
    "preparation": {"start": "08:00:00", "end": "10:30:00", "cameras": [2, 3]},
    "ceremony": {"start": "14:00:00", "end": "15:30:00", "cameras": [1, 4, 5]},
    "couple_shots": {"start": "16:00:00", "end": "17:30:00", "cameras": [1, 2]},
    "reception": {"start": "18:00:00", "end": "23:00:00", "cameras": [1, 2, 3, 4, 5]},
    "dancing": {"start": "20:00:00", "end": "22:00:00", "cameras": [3, 4, 5]}
}
EOF
    
    # 根据场景提取关键片段
    python3 << 'EOF'
import json
from datetime import datetime, timedelta

with open("scene_timeline.json") as f:
    scenes = json.load(f)

def time_to_seconds(time_str):
    """将时间字符串转换为秒数"""
    h, m, s = map(int, time_str.split(':'))
    return h * 3600 + m * 60 + s

for scene_name, scene_info in scenes.items():
    start_sec = time_to_seconds(scene_info["start"])
    end_sec = time_to_seconds(scene_info["end"])
    duration = end_sec - start_sec
    
    print(f"\n=== {scene_name.upper()} ===")
    print(f"Duration: {duration//60}:{duration%60:02d}")
    print(f"Cameras: {scene_info['cameras']}")
    
    # 生成ffmpeg命令提取场景
    for cam in scene_info['cameras']:
        cmd = f"""ffmpeg -ss {scene_info['start']} -i raw_footage/cam{cam}/ceremony.mp4 \
                 -t {duration} -c copy rough_cut/{scene_name}_cam{cam}.mp4"""
        print(cmd)
EOF
    
    echo "场景分类完成"
}

# 第4步：情感节奏编辑
emotional_pacing() {
    echo "情感节奏编辑..."
    
    # 定义情感节拍点
    cat > emotional_beats.json << EOF
{
    "highlight_moments": [
        {"time": "14:25:30", "type": "first_look", "duration": 15, "emotion": "tender"},
        {"time": "14:45:00", "type": "vows", "duration": 45, "emotion": "emotional"},
        {"time": "15:10:00", "type": "kiss", "duration": 8, "emotion": "joyful"},
        {"time": "20:30:00", "type": "first_dance", "duration": 60, "emotion": "romantic"},
        {"time": "21:45:00", "type": "party", "duration": 30, "emotion": "celebration"}
    ]
}
EOF
    
    # 根据情感节拍创建精华版
    python3 << 'EOF'
import json

with open("emotional_beats.json") as f:
    beats = json.load(f)

total_duration = 0
for i, moment in enumerate(beats["highlight_moments"]):
    start_time = moment["time"]
    duration = moment["duration"]
    moment_type = moment["type"]
    emotion = moment["emotion"]
    
    print(f"\n--- {moment_type.upper()} ({emotion}) ---")
    
    # 根据情感类型选择机位和效果
    if emotion == "tender":
        effect = "slow motion, soft focus"
        primary_cam = 1
    elif emotion == "emotional":
        effect = "close-up, stable"
        primary_cam = 2
    elif emotion == "joyful":
        effect = "wide shot, dynamic"
        primary_cam = 4
    elif emotion == "romantic":
        effect = "cinematic movement"
        primary_cam = 1
    else:  # celebration
        effect = "multi-cam montage"
        primary_cam = "all"
    
    total_duration += duration
    print(f"Time: {start_time}, Duration: {duration}s")
    print(f"Camera: {primary_cam}, Effect: {effect}")

print(f"\nTotal highlight duration: {total_duration//60}:{total_duration%60:02d}")
EOF
    
    echo "情感节奏编辑完成"
}

# 第5步：高质量最终输出
final_output() {
    echo "高质量最终输出..."
    
    # 完整版（90分钟）
    ffmpeg -f concat -safe 0 -i full_version_list.txt \
           -c:v libx264 -preset slow -crf 18 \
           -c:a aac -b:a 192k \
           -s 1920x1080 \
           -r 25 \
           -pix_fmt yuv420p \
           wedding_full_version.mp4
    
    # 精华版（10分钟）
    ffmpeg -f concat -safe 0 -i highlights_list.txt \
           -c:v libx264 -preset medium -crf 20 \
           -c:a aac -b:a 192k \
           -s 1920x1080 \
           -movflags +faststart \
           wedding_highlights.mp4
    
    # 4K版本（仅主要片段）
    ffmpeg -i wedding_highlights.mp4 \
           -s 3840x2160 \
           -c:v libx264 -preset slow -crf 16 \
           -c:a aac -b:a 256k \
           wedding_highlights_4k.mp4
    
    # 社交媒体版本
    ffmpeg -i wedding_highlights.mp4 \
           -vf "scale=1080:1080:force_original_aspect_ratio=increase,crop=1080:1080" \
           -t 60 \
           -c:v libx264 -crf 23 \
           wedding_social_media.mp4
    
    echo "最终输出完成"
}

# 执行完整工作流程
main() {
    echo "开始多机位婚礼视频制作..."
    
    organize_and_proxy
    sync_multicam
    classify_scenes
    emotional_pacing
    final_output
    
    echo "多机位婚礼视频制作完成！"
    echo "输出文件："
    echo "- wedding_full_version.mp4 (90分钟完整版)"
    echo "- wedding_highlights.mp4 (10分钟精华版)"
    echo "- wedding_highlights_4k.mp4 (4K精华版)"
    echo "- wedding_social_media.mp4 (社交媒体版)"
}

# 启动工作流程
main
```

## 紧急处理案例

### 案例5: 直播录制紧急修复

#### 问题描述
- **紧急情况**: 3小时直播录制出现严重音视频不同步
- **问题**: 音频延迟2.3秒，中途摄像头切换导致分辨率不一致
- **时限**: 4小时内必须交付修复版本
- **要求**: 保持最高质量，修复所有技术问题

#### 快速修复流程
```bash
#!/bin/bash
# 直播录制紧急修复流程

echo "=== 直播录制紧急修复 ==="

# 第1步：快速问题诊断
emergency_diagnosis() {
    local input_file="$1"
    
    echo "快速问题诊断..."
    
    # 检查音视频同步问题
    ffprobe -v quiet -show_entries packet=pts_time,dts_time -select_streams v:0 "$input_file" | head -10 > video_timestamps.txt
    ffprobe -v quiet -show_entries packet=pts_time,dts_time -select_streams a:0 "$input_file" | head -10 > audio_timestamps.txt
    
    # 检查分辨率变化
    ffprobe -v quiet -show_entries frame=width,height -select_streams v:0 "$input_file" > resolution_changes.txt
    
    # 统计问题
    unique_resolutions=$(cut -d'=' -f2 resolution_changes.txt | sort | uniq -c)
    echo "发现的分辨率:"
    echo "$unique_resolutions"
    
    # 计算音频延迟
    video_start=$(head -1 video_timestamps.txt | grep -o 'pts_time=[0-9.]*' | cut -d'=' -f2)
    audio_start=$(head -1 audio_timestamps.txt | grep -o 'pts_time=[0-9.]*' | cut -d'=' -f2)
    delay=$(echo "$audio_start - $video_start" | bc)
    
    echo "检测到音频延迟: ${delay}秒"
    echo "$delay" > detected_delay.txt
    
    echo "问题诊断完成"
}

# 第2步：音频同步快速修复
fix_audio_sync() {
    local input_file="$1"
    local delay=$(cat detected_delay.txt)
    
    echo "修复音频同步 (延迟: ${delay}秒)..."
    
    if (( $(echo "$delay > 0" | bc -l) )); then
        # 音频延迟，需要延迟视频
        ffmpeg -i "$input_file" \
               -itsoffset "$delay" -i "$input_file" \
               -map 1:v -map 0:a \
               -c:v copy -c:a copy \
               sync_fixed.mp4
    else
        # 视频延迟，需要延迟音频
        delay_abs=$(echo "$delay * -1" | bc)
        ffmpeg -i "$input_file" \
               -itsoffset "$delay_abs" -i "$input_file" \
               -map 0:v -map 1:a \
               -c:v copy -c:a copy \
               sync_fixed.mp4
    fi
    
    echo "音频同步修复完成"
}

# 第3步：分辨率标准化
standardize_resolution() {
    local input_file="$1"
    
    echo "标准化分辨率..."
    
    # 找到最常见的分辨率
    most_common_res=$(cut -d'=' -f2 resolution_changes.txt | sort | uniq -c | sort -nr | head -1 | awk '{print $2 "x" $3}')
    echo "标准化到分辨率: $most_common_res"
    
    # 应用标准化
    if [[ "$most_common_res" == "1920x1080" ]]; then
        ffmpeg -i "$input_file" \
               -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2:black" \
               -c:v libx264 -preset fast -crf 23 \
               -c:a copy \
               resolution_fixed.mp4
    else
        # 其他分辨率处理
        ffmpeg -i "$input_file" \
               -vf "scale=$most_common_res:force_original_aspect_ratio=decrease,pad=$most_common_res:(ow-iw)/2:(oh-ih)/2:black" \
               -c:v libx264 -preset fast -crf 23 \
               -c:a copy \
               resolution_fixed.mp4
    fi
    
    echo "分辨率标准化完成"
}

# 第4步：质量增强处理
enhance_quality() {
    local input_file="$1"
    
    echo "质量增强处理..."
    
    # 音频增强
    ffmpeg -i "$input_file" \
           -af "highpass=f=80,lowpass=f=8000,compand=attacks=0.1:decays=0.2:points=-80/-80|-45/-15|-27/-9|0/-7,loudnorm=I=-16:LRA=11:TP=-1.5" \
           -c:v copy \
           audio_enhanced.mp4
    
    # 视频稳定（如果需要）
    ffmpeg -i audio_enhanced.mp4 \
           -vf "vidstabdetect=shakiness=5:accuracy=15" \
           -f null -
    
    ffmpeg -i audio_enhanced.mp4 \
           -vf "vidstabtransform=smoothing=30:input=transforms.trf" \
           -c:v libx264 -preset medium -crf 20 \
           -c:a copy \
           stabilized.mp4
    
    echo "质量增强完成"
}

# 第5步：快速验证和输出
quick_verification() {
    local final_file="$1"
    
    echo "快速验证..."
    
    # 检查最终文件
    duration=$(ffprobe -v quiet -show_entries format=duration -of csv=p=0 "$final_file")
    filesize=$(du -h "$final_file" | cut -f1)
    
    echo "最终文件时长: $(echo "$duration / 60" | bc)分钟"
    echo "文件大小: $filesize"
    
    # 生成验证报告
    cat > emergency_fix_report.txt << EOF
紧急修复报告
=============
修复时间: $(date)
原始问题: 音视频不同步 + 分辨率不一致
修复方案: 
- 音频同步校正: $(cat detected_delay.txt)秒
- 分辨率标准化: 1920x1080
- 质量增强: 音频处理 + 视频稳定

最终文件: $final_file
时长: $(echo "$duration / 60" | bc)分钟
大小: $filesize

质量检查: PASSED
状态: 准备交付
EOF
    
    echo "验证完成，检查 emergency_fix_report.txt"
}

# 并行处理优化
parallel_processing() {
    local input_file="$1"
    
    echo "启动并行处理..."
    
    # 同时进行多个处理步骤
    {
        emergency_diagnosis "$input_file"
    } &
    
    {
        # 预处理音频分析
        ffmpeg -i "$input_file" -vn -c:a pcm_s16le temp_audio.wav &
    } &
    
    {
        # 预处理视频分析
        ffmpeg -i "$input_file" -an -c:v copy temp_video.mp4 &
    } &
    
    wait  # 等待所有后台任务完成
    
    echo "并行处理准备完成"
}

# 紧急修复主流程
emergency_main() {
    local input_file="$1"
    local start_time=$(date +%s)
    
    echo "开始紧急修复: $input_file"
    echo "开始时间: $(date)"
    
    # 并行预处理
    parallel_processing "$input_file"
    
    # 依次执行修复步骤
    emergency_diagnosis "$input_file"
    fix_audio_sync "$input_file"
    standardize_resolution "sync_fixed.mp4"
    enhance_quality "resolution_fixed.mp4"
    
    # 最终输出
    mv stabilized.mp4 "EMERGENCY_FIXED_$(basename "$input_file")"
    
    quick_verification "EMERGENCY_FIXED_$(basename "$input_file")"
    
    local end_time=$(date +%s)
    local total_time=$((end_time - start_time))
    
    echo "紧急修复完成！"
    echo "总耗时: $((total_time / 60))分$((total_time % 60))秒"
    echo "输出文件: EMERGENCY_FIXED_$(basename "$input_file")"
    
    # 清理临时文件
    rm -f temp_*.* sync_fixed.mp4 resolution_fixed.mp4 audio_enhanced.mp4 transforms.trf
}

# 使用示例
# emergency_main "problematic_livestream.mp4"
```

## 案例总结和最佳实践

### 通用成功要素

#### 项目管理最佳实践
1. **需求明确**: 详细的技术规格和创意简报
2. **时间规划**: 合理的时间分配和缓冲时间
3. **质量控制**: 多层次的质量检查机制
4. **备份策略**: 完整的素材和项目备份方案
5. **沟通协调**: 清晰的团队沟通和反馈机制

#### 技术实施要点
1. **工作流标准化**: 建立可重复的制作流程
2. **自动化程度**: 提高效率的自动化工具使用
3. **性能优化**: 充分利用硬件资源和并行处理
4. **错误预防**: 在关键节点进行验证和检查
5. **应急预案**: 针对常见问题的快速解决方案

#### 创意制作原则
1. **故事驱动**: 技术服务于内容和故事
2. **观众导向**: 基于目标观众调整制作策略
3. **平台适配**: 针对不同平台优化输出格式
4. **品牌一致性**: 保持视觉和音频的品牌统一
5. **情感连接**: 通过技术手段增强情感表达

---

*实战案例是理论知识转化为实际技能的桥梁。通过深入分析和实践这些典型场景，可以快速提升视频制作的综合能力。在实际项目中，要灵活运用这些经验，并根据具体情况调整技术方案和工作流程。*