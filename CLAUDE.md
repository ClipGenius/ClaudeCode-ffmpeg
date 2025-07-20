# AI 视频剪辑快速入门指南

## 🎯 核心目标

本指南专为LLM（Large Language Model）设计，旨在教授AI如何基于视觉和听觉理解进行专业级视频剪辑操作。通过系统性的提示词框架，让AI能够分析视频内容、理解用户需求，并生成精确的FFmpeg命令来完成各种视频编辑任务。

## ⚠️ 重要环境要求

**🐍 Python虚拟环境必须使用 `./.venv`**
- 所有Python相关操作必须在 `./.venv` 虚拟环境中执行
- 如果虚拟环境不存在，系统会自动创建
- 音频转录、图像处理等功能依赖虚拟环境隔离
- 这确保了环境的一致性和项目的可移植性

**📖 开始剪辑前请务必根据需求查看 `./.prompts` 下的提示词文档**
- 先撰写草稿
- 询问用户意见
- 最后生成内容

## 🎬 素材分析工作流程

### A. 视频内容理解

#### 1. 视频帧提取和分析
当用户提供视频文件时，首先使用以下命令提取关键帧进行视觉分析：

```bash
# 基础帧提取 - 每秒1帧
ffmpeg -i input.mp4 -vf "fps=1" frame_%04d.png

# 场景变化检测 - 提取关键转场点
ffmpeg -i input.mp4 -vf "select='gt(scene,0.3)',showinfo" -vsync vfr scene_changes_%04d.png

# 特定时间点提取 - 精确分析
ffmpeg -ss 00:01:30 -i input.mp4 -frames:v 1 specific_frame.png

# 缩略图网格 - 快速预览
ffmpeg -i input.mp4 -vf "fps=0.1,scale=160:90,tile=10x6" thumbnail_grid.png
```

#### 2. 视觉内容分析提示词模板

```
作为视频分析专家，请分析以下视频帧并提供详细的内容描述：

**分析维度**：
1. **场景类型**: 室内/户外/工作室/自然景观等
2. **人物情况**: 人数、位置、动作、表情、服装
3. **构图分析**: 景别（特写/中景/全景）、角度、焦点
4. **色彩特征**: 主色调、对比度、亮度、色彩饱和度
5. **技术质量**: 清晰度、曝光、色彩平衡、画面稳定性
6. **内容主题**: 活动类型、情绪氛围、故事元素
7. **剪辑线索**: 镜头运动、转场提示、节奏变化点

**输出格式**：
- 总体描述: [2-3句话概括]
- 技术参数: [分辨率、帧率、质量评估]
- 编辑建议: [基于内容特点的剪辑策略]
- 关键时间点: [重要场景的时间码]
```

### B. 音频内容理解

#### 1. 音频转录和分析

**前置检查 - Python环境验证**：
```python
import sys
import subprocess

def check_python_availability():
    """检查Python环境是否可用"""
    try:
        python_version = sys.version_info
        if python_version.major >= 3 and python_version.minor >= 8:
            print(f"✅ Python {python_version.major}.{python_version.minor} 可用")
            return True
        else:
            print(f"❌ Python版本过低 ({python_version.major}.{python_version.minor})，需要Python 3.8+")
            return False
    except:
        print("❌ Python环境不可用，音频转录功能将被禁用")
        return False

if not check_python_availability():
    print("\n⚠️  音频理解功能不可用")
    print("请先安装Python 3.8+：https://python.org/downloads/")
    print("安装后重新运行此程序以启用音频转录功能")
```

**Faster-Whisper安装和配置**：
```bash
# 必须先创建并激活 ./.venv 虚拟环境
# Windows环境
if not exist .\.venv (
    python -m venv .\.venv
    echo "✅ 虚拟环境已创建"
)
.\.venv\Scripts\activate
python -m pip install faster-whisper

# Unix-like环境 (macOS/Linux)
if [ ! -d "./.venv" ]; then
    python3 -m venv ./.venv
    echo "✅ 虚拟环境已创建"
fi
source ./.venv/bin/activate
python -m pip install faster-whisper
```

#### 2. 音频转录实现代码

```python
from faster_whisper import WhisperModel
import os
import json
from datetime import timedelta

class AudioAnalyzer:
    def __init__(self, device_preference="auto"):
        """
        音频分析器 - 必须在 ./.venv 虚拟环境中运行
        device_preference: "cpu", "cuda", "auto"
        """
        self.ensure_virtual_environment()
        self.ensure_dependencies()
        self.model = None
        self.device_config = self._setup_device(device_preference)
    
    def ensure_virtual_environment(self):
        """确保在虚拟环境中运行"""
        import sys
        if not hasattr(sys, 'real_prefix') and not (hasattr(sys, 'base_prefix') and sys.base_prefix != sys.prefix):
            raise RuntimeError("⚠️  AudioAnalyzer必须在./.venv虚拟环境中运行！请先激活虚拟环境。")
    
    def ensure_dependencies(self):
        """确保必要的依赖已安装"""
        required_packages = ['faster_whisper', 'torch']
        missing_packages = []
        
        for package in required_packages:
            try:
                __import__(package)
            except ImportError:
                missing_packages.append(package)
        
        if missing_packages:
            import subprocess
            import sys
            print(f"🔧 安装缺失的依赖: {missing_packages}")
            for package in missing_packages:
                subprocess.run([sys.executable, "-m", "pip", "install", package], check=True)
            print("✅ 依赖安装完成")
    
    def _setup_device(self, preference):
        """智能设备配置"""
        import torch
        
        if preference == "auto":
            if torch.cuda.is_available():
                return {"device": "cuda", "compute_type": "float16"}
            else:
                return {"device": "cpu", "compute_type": "int8"}
        elif preference == "cuda":
            return {"device": "cuda", "compute_type": "float16"}
        else:
            return {"device": "cpu", "compute_type": "int8"}
    
    def load_model(self, model_size="large-v2"):
        """加载Whisper模型"""
        try:
            self.model = WhisperModel(
                model_size, 
                device=self.device_config["device"],
                compute_type=self.device_config["compute_type"]
            )
            print(f"✅ 模型加载成功: {model_size} on {self.device_config['device']}")
            return True
        except Exception as e:
            print(f"❌ 模型加载失败: {e}")
            return False
    
    def transcribe_video(self, video_path, language="zh"):
        """转录视频中的音频"""
        if not self.model:
            raise ValueError("模型未加载，请先调用load_model()")
        
        # 首先提取音频
        audio_path = self._extract_audio(video_path)
        
        try:
            # 执行转录
            segments, info = self.model.transcribe(
                audio_path, 
                beam_size=5, 
                language=language,
                word_timestamps=True
            )
            
            # 处理结果
            results = list(segments)
            transcription_data = {
                "language": info.language,
                "duration": info.duration,
                "full_text": " ".join([segment.text for segment in results]),
                "segments": [
                    {
                        "start": segment.start,
                        "end": segment.end,
                        "text": segment.text.strip(),
                        "confidence": getattr(segment, 'avg_logprob', 0.0)
                    }
                    for segment in results
                ]
            }
            
            return transcription_data
            
        except Exception as e:
            print(f"转录失败: {e}")
            return None
        finally:
            # 清理临时音频文件
            if os.path.exists(audio_path):
                os.remove(audio_path)
    
    def _extract_audio(self, video_path):
        """从视频中提取音频"""
        import subprocess
        
        audio_path = "temp_audio.wav"
        cmd = [
            "ffmpeg", "-i", video_path,
            "-vn", "-acodec", "pcm_s16le",
            "-ar", "16000", "-ac", "1",
            audio_path, "-y"
        ]
        
        try:
            subprocess.run(cmd, check=True, capture_output=True)
            return audio_path
        except subprocess.CalledProcessError as e:
            raise Exception(f"音频提取失败: {e}")

# 使用示例
def analyze_video_audio(video_path, language="zh"):
    """完整的视频音频分析流程"""
    analyzer = AudioAnalyzer(device_preference="auto")
    
    if not analyzer.load_model("large-v2"):
        return None
    
    print(f"正在分析视频: {video_path}")
    transcription = analyzer.transcribe_video(video_path, language)
    
    if transcription:
        print(f"转录完成，时长: {transcription['duration']:.2f}秒")
        print(f"检测语言: {transcription['language']}")
        print(f"完整文本: {transcription['full_text']}")
        
        # 保存详细结果
        with open("transcription_result.json", "w", encoding="utf-8") as f:
            json.dump(transcription, f, ensure_ascii=False, indent=2)
        
        return transcription
    else:
        print("音频转录失败")
        return None
```

#### 3. 音频分析提示词模板

```
作为音频内容分析专家，请基于以下转录结果分析音频特征：

**转录数据**: {transcription_data}

**分析维度**：
1. **语言特征**: 语种、口音、语速、语调变化
2. **内容结构**: 
   - 开场方式、主要话题、结尾方式
   - 关键信息点和时间戳
   - 情感转折点
3. **说话者特征**: 
   - 说话者数量和身份
   - 对话模式（独白/对话/多人讨论）
   - 语气和情绪变化
4. **背景音频**: 
   - 背景音乐类型和情绪
   - 环境声音（室内/户外/噪音）
   - 音效和特殊声音
5. **技术质量**: 
   - 录音清晰度
   - 音量一致性
   - 需要音频处理的问题
6. **剪辑线索**: 
   - 自然停顿点
   - 话题转换时机
   - 可删除的冗余内容
   - 需要强调的关键段落

**输出格式**：
- 内容摘要: [核心主题和关键信息]
- 时间结构: [重要段落的时间范围]
- 剪辑建议: [基于内容的编辑策略]
- 音频处理需求: [技术优化建议]
```

## 🎨 LLM视频剪辑决策框架

### A. 任务类型识别

#### 1. 任务分类提示词

```
作为视频剪辑AI助手，请分析用户的需求并确定任务类型：

**用户需求**: "{user_request}"
**视频分析结果**: "{video_analysis}"
**音频分析结果**: "{audio_analysis}"

**任务类型分类**：
1. **基础剪辑** (参考: .prompts/operations/basic_editing.md)
   - 简单裁剪、合并、格式转换
   - 音量调整、基础色彩校正
   
2. **高级特效** (参考: .prompts/operations/advanced_effects.md)
   - 复杂滤镜、转场效果
   - 文字动画、视觉特效
   
3. **音频处理** (参考: .prompts/operations/audio_processing.md)
   - 音频清理、降噪、混音
   - 音视频同步、多轨处理
   
4. **批量处理** (参考: .prompts/operations/batch_operations.md)
   - 大量文件的统一处理
   - 自动化工作流程
   
5. **平台适配** (参考: .prompts/templates/platform_guide.md)
   - 社交媒体格式适配
   - 多平台版本输出

**输出判断**：
- 主要任务类型: [选择上述类型]
- 复杂度等级: [简单/中等/复杂/专家级]
- 推荐提示词文件: [具体文件路径]
- 预计处理时间: [时间估算]
```

### B. 技术方案制定

#### 2. FFmpeg命令生成提示词

```
作为FFmpeg技术专家，基于以下分析结果生成精确的处理命令：

**任务需求**: {task_description}
**输入文件分析**: {file_analysis}
**目标输出**: {output_requirements}

**命令生成规则**：
1. **参数优化**: 根据输入文件特征选择最佳参数
2. **质量平衡**: 在文件大小和质量间找到最佳平衡
3. **性能考虑**: 根据硬件能力优化处理速度
4. **错误预防**: 包含必要的验证和错误处理

**引用知识库**：
- 基础命令: .prompts/core/ffmpeg_guide.md
- 格式知识: .prompts/core/format_knowledge.md
- 性能优化: .prompts/templates/performance_optimization.md

**输出格式**：
```bash
# 处理目标: {简要说明}
# 预计处理时间: {时间估算}
# 质量等级: {质量评估}

{具体的FFmpeg命令}

# 验证命令
{验证输出质量的命令}
```

**备选方案**: 如果主方案不适用，提供2-3个备选命令
```

### C. 质量控制和验证

#### 3. 输出验证提示词

```
作为质量控制专家，请验证视频处理结果：

**原始需求**: {original_request}
**处理命令**: {executed_command}
**输出文件**: {output_file_path}

**验证检查清单** (参考: .prompts/templates/quality_guidelines.md):

1. **技术规格验证**:
   - [ ] 分辨率是否符合要求
   - [ ] 码率和质量是否达标
   - [ ] 音视频同步是否正确
   - [ ] 文件格式是否兼容

2. **内容质量验证**:
   - [ ] 视觉效果是否符合预期
   - [ ] 音频质量是否清晰
   - [ ] 剪辑衔接是否自然
   - [ ] 整体观感是否流畅

3. **用户需求匹配**:
   - [ ] 是否解决了用户的核心问题
   - [ ] 输出是否符合使用场景
   - [ ] 是否需要进一步优化

**验证命令**:
```bash
# 文件信息检查
ffprobe -v quiet -print_format json -show_format -show_streams output.mp4

# 质量分析
ffmpeg -i output.mp4 -vf "ssim=reference.mp4:stats_file=quality.log" -f null -

# 音频分析  
ffmpeg -i output.mp4 -af "ebur128" -f null -
```

**验证结果**:
- 技术质量: [通过/需要调整/不合格]
- 用户需求: [完全满足/部分满足/不满足]
- 改进建议: [具体的优化建议]
```

## 📚 知识库使用指南

### A. 模块化知识结构

#### 核心知识模块 (.prompts/core/)
```yaml
基础模块:
  - video_basics.md: 视频基础概念和术语
  - ffmpeg_guide.md: FFmpeg命令大全
  - format_knowledge.md: 格式和编码知识
  - terminology.md: 专业术语词典

使用场景:
  - 回答基础概念问题
  - 提供命令参考
  - 解释技术术语
  - 格式选择指导
```

#### 操作技能模块 (.prompts/operations/)
```yaml
操作模块:
  - basic_editing.md: 基础剪辑操作
  - advanced_effects.md: 高级特效制作
  - audio_processing.md: 音频后期处理
  - batch_operations.md: 批量自动化处理

使用场景:
  - 具体编辑任务执行
  - 特效技术实现
  - 音频问题解决
  - 批量处理方案
```

#### 模板工具模块 (.prompts/templates/)
```yaml
模板模块:
  - troubleshooting.md: 问题诊断和解决
  - quality_guidelines.md: 质量标准和检查
  - performance_optimization.md: 性能优化策略
  - platform_guide.md: 平台适配指南
  - workflow_optimization.md: 工作流程模板
  - case_studies.md: 实战案例参考
  - best_practices.md: 最佳实践总结

使用场景:
  - 问题排查和修复
  - 质量控制和验证
  - 性能调优指导
  - 平台特定优化
  - 工作流程设计
  - 案例学习参考
  - 最佳实践应用
```

### B. 智能知识检索

#### 1. 关键词映射表

```yaml
用户关键词到知识模块的映射:

# 基础操作
"剪切|裁剪|截取": .prompts/operations/basic_editing.md
"合并|拼接|连接": .prompts/operations/basic_editing.md  
"转换|格式|编码": .prompts/core/format_knowledge.md

# 质量问题
"画质|清晰度|模糊": .prompts/templates/quality_guidelines.md
"音质|噪音|杂音": .prompts/operations/audio_processing.md
"同步|延迟|不对齐": .prompts/operations/audio_processing.md

# 特效制作
"滤镜|特效|效果": .prompts/operations/advanced_effects.md
"文字|字幕|标题": .prompts/operations/advanced_effects.md
"转场|过渡": .prompts/operations/advanced_effects.md

# 性能优化
"慢|卡|优化": .prompts/templates/performance_optimization.md
"大文件|压缩": .prompts/templates/performance_optimization.md
"硬件加速|GPU": .prompts/templates/performance_optimization.md

# 平台适配
"TikTok|抖音|竖屏": .prompts/templates/platform_guide.md
"YouTube|横屏": .prompts/templates/platform_guide.md
"Instagram|正方形": .prompts/templates/platform_guide.md

# 批量处理
"批量|多个|自动化": .prompts/operations/batch_operations.md
"脚本|自动|一键": .prompts/operations/batch_operations.md

# 问题诊断
"错误|失败|报错": .prompts/templates/troubleshooting.md
"不能|无法|问题": .prompts/templates/troubleshooting.md
```

#### 2. 智能检索提示词

```
作为知识库检索专家，根据用户查询找到最相关的知识模块：

**用户查询**: "{user_query}"

**检索流程**:
1. 提取查询中的关键技术词汇
2. 识别任务类型和复杂度
3. 匹配最相关的知识模块
4. 确定学习路径优先级

**知识模块权重**:
- 直接匹配: 权重 1.0
- 相关概念: 权重 0.8  
- 支持知识: 权重 0.6
- 背景信息: 权重 0.4

**输出推荐**:
1. **主要参考**: [最相关的1-2个模块]
2. **辅助参考**: [支持性的2-3个模块]
3. **学习路径**: [建议的学习顺序]
4. **关键章节**: [具体的章节或示例]

**示例查询处理**:
- "如何给视频加字幕" → advanced_effects.md (文字叠加) + audio_processing.md (音频同步)
- "视频太大怎么压缩" → performance_optimization.md (压缩策略) + format_knowledge.md (编码选择)
- "批量转换格式" → batch_operations.md (批处理) + format_knowledge.md (格式知识)
```

## 📖 提示词使用指南和场景映射

### A. 提示词选择决策树

#### 1. 场景识别算法

```yaml
用户需求分析流程:
  
  第一层 - 主要任务类型:
    "基础编辑": 
      关键词: [剪切, 裁剪, 合并, 拼接, 转换, 格式]
      推荐模块: .prompts/operations/basic_editing.md
      提示词类型: 操作指导型
      
    "高级特效":
      关键词: [特效, 滤镜, 转场, 动画, 文字, 叠加]
      推荐模块: .prompts/operations/advanced_effects.md
      提示词类型: 创意实现型
      
    "音频处理":
      关键词: [音频, 声音, 降噪, 混音, 同步, 音量]
      推荐模块: .prompts/operations/audio_processing.md
      提示词类型: 技术优化型
      
    "批量处理":
      关键词: [批量, 多个, 自动化, 脚本, 一键]
      推荐模块: .prompts/operations/batch_operations.md
      提示词类型: 自动化流程型
  
  第二层 - 技术复杂度:
    "简单" (复杂度 0-2):
      特征: 单一操作, 标准参数, 常见格式
      提示词策略: 直接命令生成
      参考资源: .prompts/core/ffmpeg_guide.md
      
    "中等" (复杂度 3-5):
      特征: 多步操作, 参数优化, 质量平衡
      提示词策略: 结构化工作流程
      参考资源: basic_editing.md + quality_guidelines.md
      
    "复杂" (复杂度 6-8):
      特征: 创意实现, 性能优化, 多重约束
      提示词策略: 综合解决方案
      参考资源: advanced_effects.md + performance_optimization.md
      
    "专家级" (复杂度 9+):
      特征: 创新方案, 系统集成, 定制开发
      提示词策略: 研究探索型
      参考资源: case_studies.md + best_practices.md
  
  第三层 - 使用场景:
    "学习探索":
      用户特征: 初学者, 概念理解, 技能建立
      提示词风格: 教育引导型
      知识重点: 基础概念 + 逐步指导
      
    "问题解决":
      用户特征: 遇到具体问题, 需要快速解决
      提示词风格: 问题诊断型
      知识重点: troubleshooting.md + 错误处理
      
    "项目执行":
      用户特征: 有明确目标, 追求效率和质量
      提示词风格: 方案实施型
      知识重点: workflow_optimization.md + case_studies.md
      
    "创新实验":
      用户特征: 探索新可能, 突破技术限制
      提示词风格: 创新探索型
      知识重点: advanced_effects.md + 创意技术
```

### B. 多场景提示词库

#### 1. 基础操作场景提示词

```yaml
场景: 视频裁剪和分割
适用条件: 用户需要截取视频片段
触发关键词: [裁剪, 截取, 分割, 切段, 提取]

提示词模板:
  角色定义: "视频编辑基础操作专家"
  专业领域: "精确的时间轴操作和无损剪切"
  参考模块: ".prompts/operations/basic_editing.md"
  
  核心指导原则:
    - 保证时间精度到帧级别
    - 优先使用copy模式避免重编码
    - 提供多种时间格式支持
    - 包含批量处理选项
  
  输出结构:
    - 时间点验证和建议
    - 精确的FFmpeg命令
    - 质量保证措施
    - 效率优化提示

---

场景: 视频格式转换
适用条件: 不同平台或设备兼容性需求
触发关键词: [转换, 格式, 编码, 兼容, 压缩]

提示词模板:
  角色定义: "视频格式和编码专家"
  专业领域: "多平台兼容性和质量优化"
  参考模块: ".prompts/core/format_knowledge.md"
  
  核心指导原则:
    - 分析目标平台的技术规格
    - 平衡文件大小和质量
    - 考虑硬件加速可能性
    - 提供多种质量等级选择
  
  输出结构:
    - 格式选择理由说明
    - 参数优化建议
    - 兼容性验证方法
    - 性能优化策略
```

#### 2. 高级特效场景提示词

```yaml
场景: 视觉特效和滤镜应用
适用条件: 需要增强视觉效果和创意表达
触发关键词: [特效, 滤镜, 美化, 风格, 艺术]

提示词模板:
  角色定义: "视觉特效和创意实现专家"
  专业领域: "FFmpeg滤镜链设计和视觉创意实现"
  参考模块: ".prompts/operations/advanced_effects.md"
  
  核心指导原则:
    - 理解用户的创意意图
    - 选择合适的滤镜组合
    - 平衡效果强度和自然度
    - 考虑处理性能和时间
  
  输出结构:
    - 创意解读和效果建议
    - 滤镜链设计方案
    - 参数调优指导
    - 渲染性能优化

---

场景: 文字和图形叠加
适用条件: 添加标题、字幕、水印等元素
触发关键词: [文字, 字幕, 标题, 水印, 叠加]

提示词模板:
  角色定义: "视频文字设计和叠加专家"
  专业领域: "文字排版、动画效果和视觉层次"
  参考模块: ".prompts/operations/advanced_effects.md (文字部分)"
  
  核心指导原则:
    - 确保文字的可读性和美观性
    - 考虑不同分辨率的适配
    - 选择合适的字体和颜色
    - 设计合理的动画时序
    - ⚠️ 非ASCII文字必须检查字体支持
    - 🔤 自动检测文字类型并提供字体解决方案
  
  字体处理流程:
    - 检测文字是否包含非ASCII字符
    - 如有中文/日文/韩文等，必须指定字体文件
    - 提供字体安装命令(apt/yum/pacman)
    - 验证字体文件路径有效性
    - 生成包含fontfile参数的完整命令
  
  输出结构:
    - 文字类型分析 (ASCII/CJK/混合)
    - 字体要求和推荐路径
    - 技术实现方法 (包含字体处理)
    - 完整的FFmpeg命令
    - 样式参数建议
    - 可访问性考虑
    - 字体安装指导(如需要)
```

#### 3. 问题诊断场景提示词

```yaml
场景: 技术问题诊断和解决
适用条件: 用户遇到处理错误或质量问题
触发关键词: [错误, 失败, 问题, 不能, 异常]

提示词模板:
  角色定义: "视频处理故障诊断专家"
  专业领域: "系统性问题分析和解决方案设计"
  参考模块: ".prompts/templates/troubleshooting.md"
  
  核心指导原则:
    - 系统性地收集错误信息
    - 从简单到复杂逐步排查
    - 提供多种解决路径
    - 预防类似问题再次发生
  
  输出结构:
    - 问题分析和分类
    - 诊断步骤和命令
    - 解决方案优先级排序
    - 预防措施建议

---

场景: 性能优化和加速
适用条件: 处理速度慢或系统资源不足
触发关键词: [慢, 卡, 优化, 加速, 性能]

提示词模板:
  角色定义: "视频处理性能优化专家"
  专业领域: "硬件加速和处理流程优化"
  参考模块: ".prompts/templates/performance_optimization.md"
  
  核心指导原则:
    - 评估硬件能力和瓶颈
    - 选择最优的编码参数
    - 利用并行处理能力
    - 平衡速度和质量
  
  输出结构:
    - 性能瓶颈分析
    - 硬件加速方案
    - 参数优化建议
    - 工作流程改进
```

## 🎓 LLM应用流程

### A. 任务驱动应用策略

#### 1. 问题解决应用法
```
当遇到具体用户问题时：

第1步: 问题分析
- 解析用户需求的核心问题
- 识别涉及的技术领域
- 评估任务复杂度

第2步: 知识检索
- 使用关键词映射找到相关模块
- 按权重排序知识来源
- 构建知识应用序列

第3步: 方案制定
- 基于知识库制定技术方案
- 生成具体的实施步骤
- 准备验证和测试方法

第4步: 执行和验证
- 生成精确的FFmpeg命令
- 执行质量控制检查
- 提供改进和优化建议

第5步: 经验积累
- 总结解决方案的有效性
- 识别可复用的模式
- 更新个人知识库
```

#### 2. 案例应用法
```
基于 .prompts/templates/case_studies.md 中的实际案例：

应用模式:
1. **案例分析**: 深入理解案例背景和挑战
2. **方案研究**: 分析技术方案的设计思路
3. **代码理解**: 逐行分析关键代码实现
4. **效果评估**: 理解方案的效果和局限性
5. **变体练习**: 尝试解决类似但不同的问题

重点案例类型:
- TikTok短视频制作 → 社交媒体适配技能
- YouTube教程制作 → 长视频处理技能
- 产品宣传片制作 → 商业级质量控制
- 多机位后期制作 → 复杂项目管理
- 紧急修复处理 → 问题诊断和快速解决
```

## 🔧 技术实施指南

### B. 环境检查和设置

#### 1. 系统能力检测
```python
def check_system_capabilities():
    """检查系统的多媒体处理能力"""
    capabilities = {
        "ffmpeg": False,
        "python": False,
        "gpu_acceleration": False,
        "whisper": False,
        "storage_space": 0
    }
    
    # 检查FFmpeg
    try:
        import subprocess
        result = subprocess.run(['ffmpeg', '-version'], 
                              capture_output=True, text=True)
        if result.returncode == 0:
            capabilities["ffmpeg"] = True
            print("✅ FFmpeg 可用")
    except:
        print("❌ FFmpeg 未安装或不可用")
    
    # 检查Python环境
    import sys
    if sys.version_info >= (3, 8):
        capabilities["python"] = True
        print(f"✅ Python {sys.version_info.major}.{sys.version_info.minor} 可用")
    else:
        print(f"❌ Python版本过低: {sys.version_info.major}.{sys.version_info.minor}")
    
    # 检查GPU支持
    try:
        import torch
        if torch.cuda.is_available():
            capabilities["gpu_acceleration"] = True
            print(f"✅ CUDA GPU 可用: {torch.cuda.get_device_name()}")
    except:
        print("⚠️  GPU加速不可用，将使用CPU处理")
    
    # 检查Whisper
    if capabilities["python"]:
        try:
            import faster_whisper
            capabilities["whisper"] = True
            print("✅ Faster-Whisper 音频转录可用")
        except ImportError:
            print("⚠️  Faster-Whisper 未安装，音频转录功能将被禁用")
    
    # 检查存储空间
    import shutil
    total, used, free = shutil.disk_usage("/")
    capabilities["storage_space"] = free // (1024**3)  # GB
    print(f"💾 可用存储空间: {capabilities['storage_space']}GB")
    
    return capabilities
```

#### 2. 功能适配策略
```python
def adapt_functionality_to_capabilities(capabilities):
    """根据系统能力调整功能可用性"""
    
    available_features = {
        "basic_video_editing": capabilities["ffmpeg"],
        "audio_processing": capabilities["ffmpeg"],
        "visual_analysis": True,  # LLM原生能力
        "audio_transcription": capabilities["python"] and capabilities["whisper"],
        "gpu_acceleration": capabilities["gpu_acceleration"],
        "batch_processing": capabilities["ffmpeg"],
        "quality_analysis": capabilities["ffmpeg"]
    }
    
    # 生成功能报告
    feature_report = """
# 系统功能可用性报告

## 核心功能
- 视频基础编辑: {'✅ 可用' if available_features['basic_video_editing'] else '❌ 不可用 (需要FFmpeg)'}
- 音频处理: {'✅ 可用' if available_features['audio_processing'] else '❌ 不可用 (需要FFmpeg)'}
- 视觉内容分析: ✅ 可用 (LLM原生能力)
- 音频内容转录: {'✅ 可用' if available_features['audio_transcription'] else '❌ 不可用 (需要Python + Faster-Whisper)'}

## 性能功能  
- GPU硬件加速: {'✅ 可用' if available_features['gpu_acceleration'] else '⚠️ 不可用 (使用CPU处理)'}
- 批量处理: {'✅ 可用' if available_features['batch_processing'] else '❌ 不可用 (需要FFmpeg)'}
- 质量分析: {'✅ 可用' if available_features['quality_analysis'] else '❌ 不可用 (需要FFmpeg)'}

## 建议操作
"""
    
    if not available_features['basic_video_editing']:
        feature_report += "1. 安装FFmpeg: https://ffmpeg.org/download.html\n"
    
    if not available_features['audio_transcription']:
        feature_report += "2. 安装Python和Faster-Whisper以启用音频转录功能\n"
        
    if not available_features['gpu_acceleration']:
        feature_report += "3. 考虑使用GPU加速提升处理性能\n"
    
    if capabilities["storage_space"] < 10:
        feature_report += f"4. 建议清理磁盘空间，当前可用: {capabilities['storage_space']}GB\n"
    
    print(feature_report)
    return available_features
```

### B. 智能任务路由

#### 1. 任务复杂度评估
```python
def assess_task_complexity(user_request, video_analysis=None, audio_analysis=None):
    """评估任务复杂度并推荐处理策略"""
    
    complexity_factors = {
        "input_complexity": 0,    # 输入文件复杂度
        "processing_complexity": 0,  # 处理操作复杂度  
        "output_requirements": 0,    # 输出要求复杂度
        "technical_difficulty": 0    # 技术实现难度
    }
    
    # 分析输入复杂度
    if video_analysis:
        if video_analysis.get("resolution_4k", False):
            complexity_factors["input_complexity"] += 2
        if video_analysis.get("multiple_scenes", False):
            complexity_factors["input_complexity"] += 1
        if video_analysis.get("poor_quality", False):
            complexity_factors["input_complexity"] += 1
    
    # 分析处理复杂度
    processing_keywords = {
        "特效|滤镜|效果": 2,
        "转场|动画": 2, 
        "多轨|混音": 2,
        "批量|自动化": 1,
        "裁剪|合并": 0,
        "格式转换": 0
    }
    
    for keyword, weight in processing_keywords.items():
        if any(word in user_request for word in keyword.split("|")):
            complexity_factors["processing_complexity"] = max(
                complexity_factors["processing_complexity"], weight
            )
    
    # 计算总复杂度
    total_complexity = sum(complexity_factors.values())
    
    # 确定复杂度等级
    if total_complexity <= 2:
        complexity_level = "简单"
        recommended_modules = [".prompts/operations/basic_editing.md"]
        estimated_time = "< 5分钟"
    elif total_complexity <= 4:
        complexity_level = "中等"
        recommended_modules = [
            ".prompts/operations/basic_editing.md",
            ".prompts/operations/audio_processing.md"
        ]
        estimated_time = "5-15分钟"
    elif total_complexity <= 6:
        complexity_level = "复杂"
        recommended_modules = [
            ".prompts/operations/advanced_effects.md",
            ".prompts/templates/performance_optimization.md"
        ]
        estimated_time = "15-45分钟"
    else:
        complexity_level = "专家级"
        recommended_modules = [
            ".prompts/operations/advanced_effects.md",
            ".prompts/operations/batch_operations.md",
            ".prompts/templates/case_studies.md"
        ]
        estimated_time = "> 45分钟"
    
    return {
        "complexity_level": complexity_level,
        "factors": complexity_factors,
        "recommended_modules": recommended_modules,
        "estimated_time": estimated_time,
        "total_score": total_complexity
    }
```

#### 2. 智能方案推荐
```python
def generate_solution_recommendation(task_analysis, system_capabilities):
    """基于任务分析和系统能力生成解决方案推荐"""
    
    recommendation = {
        "primary_approach": "",
        "alternative_approaches": [],
        "required_modules": [],
        "performance_tips": [],
        "quality_checkpoints": []
    }
    
    # 主要方案选择
    if task_analysis["complexity_level"] == "简单":
        recommendation["primary_approach"] = "direct_ffmpeg"
        recommendation["required_modules"] = [".prompts/core/ffmpeg_guide.md"]
        
    elif task_analysis["complexity_level"] == "中等":
        recommendation["primary_approach"] = "structured_workflow"
        recommendation["required_modules"] = [
            ".prompts/operations/basic_editing.md",
            ".prompts/templates/quality_guidelines.md"
        ]
        
    else:  # 复杂或专家级
        recommendation["primary_approach"] = "comprehensive_solution"
        recommendation["required_modules"] = [
            ".prompts/operations/advanced_effects.md",
            ".prompts/templates/workflow_optimization.md",
            ".prompts/templates/performance_optimization.md"
        ]
    
    # 性能优化建议
    if not system_capabilities.get("gpu_acceleration", False):
        recommendation["performance_tips"].append(
            "使用CPU优化参数: -preset fast -threads auto"
        )
    else:
        recommendation["performance_tips"].append(
            "启用GPU加速: -hwaccel cuda -c:v h264_nvenc"
        )
    
    if system_capabilities.get("storage_space", 0) < 5:
        recommendation["performance_tips"].append(
            "使用临时文件清理策略避免空间不足"
        )
    
    # 质量检查点
    recommendation["quality_checkpoints"] = [
        "输入文件验证",
        "处理参数确认", 
        "输出质量检查",
        "用户需求匹配验证"
    ]
    
    return recommendation
```

## 📋 实际使用示例

### A. 完整工作流程示例

#### 示例1: 用户要求"给这个视频加字幕"

```python
# 第1步: 进行分析
def process_subtitle_request(video_file, subtitle_text=None):
    """处理字幕添加请求的完整流程"""
    
    print("🎬 开始视频字幕添加流程")
    
    # 视觉分析
    print("1️⃣ 提取视频帧进行分析...")
    extract_frames_cmd = f"ffmpeg -i {video_file} -vf 'fps=0.1' -frames:v 10 frame_%02d.png"
    
    # 音频转录 (如果用户没有提供字幕文本)
    if not subtitle_text:
        print("2️⃣ 进行音频转录...")
        analyzer = AudioAnalyzer()
        if analyzer.load_model("large-v2"):
            transcription = analyzer.transcribe_video(video_file, language="zh")
            subtitle_segments = transcription["segments"]
        else:
            print("❌ 无法进行音频转录，请手动提供字幕文本")
            return None
    
    # 视频特征分析
    print("3️⃣ 分析视频特征...")
    video_info = analyze_video_properties(video_file)
    
    # 根据分析结果确定字幕样式
    print("4️⃣ 确定字幕样式...")
    subtitle_style = determine_subtitle_style(video_info)
    
    # 生成FFmpeg命令
    print("5️⃣ 生成处理命令...")
    if subtitle_segments:
        # 使用转录结果生成时间轴字幕
        command = generate_subtitle_command_with_timing(
            video_file, subtitle_segments, subtitle_style
        )
    else:
        # 使用静态字幕
        command = generate_static_subtitle_command(
            video_file, subtitle_text, subtitle_style
        )
    
    # 执行处理
    print("6️⃣ 执行视频处理...")
    execute_command_with_monitoring(command)
    
    # 质量验证
    print("7️⃣ 验证输出质量...")
    quality_check_result = verify_subtitle_output("output_with_subtitles.mp4")
    
    return quality_check_result

def determine_subtitle_style(video_info):
    """根据视频特征确定字幕样式"""
    style = {
        "font_size": 24,
        "font_color": "white",
        "position": "bottom_center",
        "background": True
    }
    
    # 根据视频亮度调整字幕颜色
    if video_info.get("average_brightness", 128) > 180:
        style["font_color"] = "black"
        style["background_color"] = "white@0.8"
    else:
        style["font_color"] = "white"  
        style["background_color"] = "black@0.8"
    
    # 根据分辨率调整字体大小
    if video_info.get("width", 1920) >= 1920:
        style["font_size"] = 36
    elif video_info.get("width", 1920) <= 720:
        style["font_size"] = 18
    
    return style

def generate_subtitle_command_with_timing(video_file, segments, style):
    """生成带时间轴的字幕命令"""
    
    # 创建字幕文件
    subtitle_file = "subtitles.srt"
    with open(subtitle_file, "w", encoding="utf-8") as f:
        for i, segment in enumerate(segments, 1):
            start_time = format_time(segment["start"])
            end_time = format_time(segment["end"])
            text = segment["text"].strip()
            
            f.write(f"{i}\n")
            f.write(f"{start_time} --> {end_time}\n")
            f.write(f"{text}\n\n")
    
    # 生成FFmpeg命令
    command = f"""ffmpeg -i {video_file} -vf "subtitles={subtitle_file}:force_style='FontSize={style['font_size']},FontColour={style['font_color']},BackColour={style.get('background_color', 'black@0.8')}'" output_with_subtitles.mp4"""
    
    return command
```

#### 示例2: 用户要求"把这个视频压缩一下"

```python
def process_compression_request(video_file, target_size=None, quality_preference="balanced"):
    """处理视频压缩请求"""
    
    print("🗜️ 开始视频压缩流程")
    
    # 分析原始文件
    print("1️⃣ 分析原始视频...")
    original_info = analyze_video_file(video_file)
    original_size = original_info["file_size_mb"]
    duration = original_info["duration"]
    
    print(f"原始文件: {original_size:.1f}MB, 时长: {duration:.1f}秒")
    
    # 确定压缩策略
    print("2️⃣ 制定压缩策略...")
    if target_size:
        strategy = calculate_bitrate_for_target_size(duration, target_size)
        print(f"目标大小: {target_size}MB")
    else:
        # 默认压缩到原始大小的50%
        target_size = original_size * 0.5
        strategy = calculate_bitrate_for_target_size(duration, target_size)
        print(f"默认压缩到: {target_size:.1f}MB")
    
    # 根据质量偏好调整参数
    print("3️⃣ 调整质量参数...")
    compression_params = adjust_compression_params(strategy, quality_preference)
    
    # 生成压缩命令
    print("4️⃣ 生成压缩命令...")
    command = f"""ffmpeg -i {video_file} \\
        -c:v libx264 -crf {compression_params['crf']} \\
        -preset {compression_params['preset']} \\
        -maxrate {compression_params['maxrate']} \\
        -bufsize {compression_params['bufsize']} \\
        -c:a aac -b:a {compression_params['audio_bitrate']} \\
        compressed_output.mp4"""
    
    # 执行压缩
    print("5️⃣ 执行压缩...")
    execute_with_progress_monitoring(command)
    
    # 验证结果
    print("6️⃣ 验证压缩结果...")
    compressed_info = analyze_video_file("compressed_output.mp4")
    compression_ratio = (original_size - compressed_info["file_size_mb"]) / original_size * 100
    
    print(f"压缩完成!")
    print(f"压缩后大小: {compressed_info['file_size_mb']:.1f}MB")
    print(f"压缩率: {compression_ratio:.1f}%")
    print(f"质量保持: {estimate_quality_retention(original_info, compressed_info):.1f}%")
    
    return {
        "success": True,
        "original_size": original_size,
        "compressed_size": compressed_info["file_size_mb"],
        "compression_ratio": compression_ratio
    }

def calculate_bitrate_for_target_size(duration_seconds, target_size_mb):
    """根据目标文件大小计算码率"""
    target_size_bits = target_size_mb * 8 * 1024 * 1024
    target_bitrate_bps = target_size_bits / duration_seconds
    
    # 为音频预留空间 (通常128kbps)
    audio_bitrate_bps = 128 * 1000
    video_bitrate_bps = target_bitrate_bps - audio_bitrate_bps
    
    # 为容器开销预留10%空间
    video_bitrate_bps *= 0.9
    
    return {
        "video_bitrate": int(video_bitrate_bps / 1000),  # kbps
        "audio_bitrate": "128k"
    }
```

### B. 错误处理和恢复

#### 智能错误诊断
```python
def intelligent_error_diagnosis(error_message, command, input_file):
    """智能错误诊断和解决方案推荐"""
    
    # 参考 .prompts/templates/troubleshooting.md
    error_patterns = {
        "no such file or directory": {
            "cause": "文件路径错误或文件不存在",
            "solution": "使用pwd检查当前目录，然后定位文件",
            "special_handler": "handle_file_not_found",
            "requires_cwd_check": True,
            "diagnostic_steps": [
                "pwd",
                "ls -la",
                "find . -name '*video*' -type f",
                "find . -name '*.mp4' -type f | head -10"
            ]
        },
        "codec not found": {
            "cause": "编解码器不支持",
            "solution": "检查FFmpeg编译版本，使用支持的编解码器",
            "alternative_command": lambda cmd: cmd.replace("libx265", "libx264")
        },
        "permission denied": {
            "cause": "文件权限问题",
            "solution": "检查文件权限，确保有读写权限",
            "fix_command": "chmod 644 input_file"
        },
        "no space left": {
            "cause": "磁盘空间不足",
            "solution": "清理磁盘空间或选择其他输出目录",
            "prevention": "使用更高的压缩率"
        },
        "invalid argument": {
            "cause": "参数配置错误",
            "solution": "检查命令参数的兼容性",
            "debug_approach": "逐步简化命令参数"
        }
    }
    
    # 匹配错误模式
    for pattern, info in error_patterns.items():
        if pattern in error_message.lower():
            print(f"🔍 错误诊断: {info['cause']}")
            print(f"💡 解决方案: {info['solution']}")
            
            # 特殊处理：No such file or directory
            if info.get("special_handler") == "handle_file_not_found":
                return handle_file_not_found_error(input_file, command)
            
            if "alternative_command" in info:
                new_command = info["alternative_command"](command)
                print(f"🔄 建议命令: {new_command}")
                return new_command
            
            if "fix_command" in info:
                fix_cmd = info["fix_command"].replace("input_file", input_file)
                print(f"🔧 修复命令: {fix_cmd}")
    
    # 通用诊断流程
    print("🔍 执行通用错误诊断...")
    diagnostic_commands = [
        f"ffprobe -v error -show_format -show_streams {input_file}",
        "ffmpeg -encoders | grep h264",
        "df -h .",
        "ls -la input_file"
    ]
    
    return None

def handle_file_not_found_error(input_file, original_command):
    """专门处理"No such file or directory"错误的函数"""
    import os
    import subprocess
    
    print("🚨 检测到文件未找到错误，开始系统化诊断...")
    
    # 第1步：检查当前工作目录
    print("1️⃣ 检查当前工作目录...")
    try:
        current_dir = subprocess.check_output("pwd", shell=True, text=True).strip()
        print(f"📍 当前目录: {current_dir}")
    except:
        print("❌ 无法获取当前目录")
        return None
    
    # 第2步：列出当前目录内容
    print("2️⃣ 列出当前目录内容...")
    try:
        dir_contents = subprocess.check_output("ls -la", shell=True, text=True)
        print("📋 目录内容:")
        print(dir_contents)
    except:
        print("❌ 无法列出目录内容")
    
    # 第3步：尝试定位文件
    print("3️⃣ 搜索相关文件...")
    filename_base = os.path.splitext(os.path.basename(input_file))[0]
    extension = os.path.splitext(input_file)[1]
    
    search_commands = [
        f"find . -name '{os.path.basename(input_file)}' -type f 2>/dev/null",
        f"find . -name '*{filename_base}*' -type f 2>/dev/null",
        f"find . -name '*{extension}' -type f 2>/dev/null | head -10",
        "find . -name '*.mp4' -o -name '*.avi' -o -name '*.mov' -o -name '*.mkv' 2>/dev/null | head -10"
    ]
    
    found_files = []
    for cmd in search_commands:
        try:
            result = subprocess.check_output(cmd, shell=True, text=True).strip()
            if result:
                files = result.split('\n')
                found_files.extend([f for f in files if f])
                print(f"🔍 搜索命令: {cmd}")
                print(f"📁 找到文件: {files}")
        except:
            continue
    
    # 第4步：分析可能的解决方案
    print("4️⃣ 分析解决方案...")
    if found_files:
        # 去重并按相似度排序
        unique_files = list(set(found_files))
        
        print("🎯 可能的文件匹配:")
        for i, file_path in enumerate(unique_files[:5], 1):
            print(f"   {i}. {file_path}")
        
        # 推荐最佳匹配
        best_match = unique_files[0]
        print(f"💡 推荐使用: {best_match}")
        
        # 生成修正后的命令
        corrected_command = original_command.replace(input_file, best_match)
        print(f"🔄 修正后的命令:")
        print(f"   {corrected_command}")
        
        return {
            "status": "file_found",
            "recommended_file": best_match,
            "corrected_command": corrected_command,
            "all_candidates": unique_files[:5]
        }
    else:
        print("❌ 未找到匹配的文件")
        print("📝 建议检查项目:")
        print("   1. 确认文件是否在正确的目录中")
        print("   2. 检查文件名拼写是否正确")
        print("   3. 确认文件是否已被移动或删除")
        print("   4. 检查文件路径是否使用了正确的分隔符")
        
        return {
            "status": "file_not_found", 
            "current_directory": current_dir,
            "suggestions": [
                "检查文件名拼写",
                "确认文件路径",
                "使用绝对路径", 
                "从根目录 (./) 重新定位文件"
            ]
        }

def check_file_path_and_suggest(file_path):
    """检查文件路径并提供建议的便捷函数"""
    import os
    import subprocess
    
    print(f"🔍 检查文件路径: {file_path}")
    
    # 首先检查当前目录
    try:
        pwd_result = subprocess.check_output("pwd", shell=True, text=True).strip()
        print(f"📍 当前工作目录: {pwd_result}")
    except:
        print("❌ 无法获取当前目录")
    
    # 检查文件是否存在
    if os.path.exists(file_path):
        print(f"✅ 文件存在: {file_path}")
        return True
    else:
        print(f"❌ 文件不存在: {file_path}")
        
        # 自动搜索替代文件
        basename = os.path.basename(file_path)
        print(f"🔍 搜索类似文件: {basename}")
        
        search_cmd = f"find . -name '*{basename}*' -type f 2>/dev/null | head -5"
        try:
            found = subprocess.check_output(search_cmd, shell=True, text=True).strip()
            if found:
                print("📁 找到类似文件:")
                for f in found.split('\n'):
                    print(f"   - {f}")
                return found.split('\n')[0]  # 返回第一个匹配的文件
        except:
            pass
        
        return False
```

## 📚 学习场景和任务分类

### A. 学习场景分类体系

#### 1. 技能发展导向场景

```yaml
基础技能建立场景:
  目标: 建立扎实的基础操作能力
  适用人群: 视频编辑新手、技术概念薄弱者
  学习特征:
    - 概念理解优先于操作执行
    - 需要大量的示例和解释
    - 错误容忍度高，注重学习过程
  
  核心学习任务:
    - 视频基础概念掌握 (video_basics.md)
    - FFmpeg命令结构理解 (ffmpeg_guide.md)
    - 常见格式和编码认知 (format_knowledge.md)
    - 基础术语和概念映射 (terminology.md)
  
  学习策略:
    - 渐进式概念建立
    - 重复练习和强化
    - 即时反馈和纠错
    - 建立信心和兴趣
  
  成功指标:
    - 能够解释基本视频概念
    - 理解FFmpeg命令结构
    - 完成简单的编辑任务
    - 具备基础的问题诊断能力

---

技能深化场景:
  目标: 深化已有技能，扩展应用范围
  适用人群: 有基础经验，希望提升专业水平
  学习特征:
    - 注重技能的精细化和专业化
    - 关注效率和质量的平衡
    - 希望了解最佳实践
  
  核心学习任务:
    - 高级特效技术掌握 (advanced_effects.md)
    - 音频后期处理精通 (audio_processing.md)
    - 性能优化策略运用 (performance_optimization.md)
    - 工作流程标准化 (workflow_optimization.md)
  
  学习策略:
    - 案例驱动学习
    - 技术对比分析
    - 最佳实践应用
    - 创新探索尝试
  
  成功指标:
    - 独立设计复杂的编辑方案
    - 优化处理性能和质量
    - 建立个人工作流程
    - 指导他人解决问题

---

专业应用场景:
  目标: 达到专业级应用水平
  适用人群: 高级用户，专业需求导向
  学习特征:
    - 追求技术深度和创新
    - 关注行业标准和规范
    - 需要解决复杂实际问题
  
  核心学习任务:
    - 批量自动化处理 (batch_operations.md)
    - 复杂案例分析 (case_studies.md)
    - 最佳实践制定 (best_practices.md)
    - 自定义解决方案开发
  
  学习策略:
    - 项目驱动学习
    - 技术创新探索
    - 经验总结分享
    - 标准制定参与
  
  成功指标:
    - 解决复杂的实际项目
    - 创新技术解决方案
    - 制定行业最佳实践
    - 培养下一代专家
```

#### 2. 问题解决导向场景

```yaml
紧急修复场景:
  特征: 时间紧迫，需要快速解决关键问题
  用户状态: 焦虑、需要立即帮助
  任务复杂度: 通常中等，但时间压力大
  
  学习需求:
    - 快速问题诊断技能
    - 应急处理方案库
    - 风险评估和控制
  
  支持策略:
    - 提供快速诊断流程
    - 推荐最稳定的解决方案
    - 避免实验性技术
    - 准备备用方案
  
  关键资源:
    - troubleshooting.md (故障排除)
    - 常见问题解决方案库
    - 快速修复命令集合

---

质量提升场景:
  特征: 现有方案可用，但需要优化质量
  用户状态: 相对冷静，追求完美
  任务复杂度: 中等到高级
  
  学习需求:
    - 质量评估标准
    - 优化技术方法
    - 性能和质量平衡
  
  支持策略:
    - 提供多种优化方案
    - 详细的质量对比分析
    - 逐步改进建议
    - 测试验证方法
  
  关键资源:
    - quality_guidelines.md (质量标准)
    - performance_optimization.md (性能优化)
    - best_practices.md (最佳实践)

---

创新探索场景:
  特征: 追求新技术和创意实现
  用户状态: 充满好奇心，愿意尝试
  任务复杂度: 高级到专家级
  
  学习需求:
    - 前沿技术了解
    - 创意实现方法
    - 实验和测试技能
  
  支持策略:
    - 鼓励创新尝试
    - 提供实验指导
    - 分享成功案例
    - 建立试错机制
  
  关键资源:
    - advanced_effects.md (高级技术)
    - case_studies.md (创新案例)
    - 社区最新实践
```

### B. 任务分类和路由系统

#### 1. 智能任务分类算法

```python
def classify_learning_task(user_request, user_profile, context_analysis):
    """
    智能分类学习任务，提供个性化的学习路径
    """
    
    # 任务分类维度
    classification_result = {
        "skill_level_required": assess_required_skill_level(user_request),
        "knowledge_domains": identify_knowledge_domains(user_request),
        "learning_objective": determine_learning_objective(user_request),
        "urgency_level": assess_urgency(user_request),
        "complexity_score": calculate_complexity_score(user_request, context_analysis)
    }
    
    # 用户适配分析
    user_match_analysis = {
        "skill_gap": calculate_skill_gap(classification_result, user_profile),
        "interest_alignment": assess_interest_alignment(classification_result, user_profile),
        "learning_capacity": evaluate_learning_capacity(user_profile),
        "support_needs": identify_support_needs(classification_result, user_profile)
    }
    
    # 生成学习策略
    learning_strategy = generate_learning_strategy(classification_result, user_match_analysis)
    
    return {
        "task_classification": classification_result,
        "user_analysis": user_match_analysis,
        "recommended_strategy": learning_strategy,
        "resource_allocation": allocate_learning_resources(learning_strategy)
    }

def generate_learning_strategy(task_classification, user_analysis):
    """生成个性化学习策略"""
    
    # 基础策略选择
    if user_analysis["skill_gap"] > 2:
        base_strategy = "foundational_building"
    elif task_classification["urgency_level"] == "high":
        base_strategy = "rapid_solution"
    elif task_classification["complexity_score"] > 7:
        base_strategy = "guided_exploration"
    else:
        base_strategy = "direct_instruction"
    
    # 策略个性化调整
    personalization_factors = {
        "learning_pace": determine_optimal_pace(user_analysis),
        "support_intensity": calculate_support_intensity(user_analysis),
        "practice_frequency": recommend_practice_frequency(task_classification),
        "feedback_style": select_feedback_style(user_analysis)
    }
    
    # 生成完整策略
    complete_strategy = {
        "base_approach": base_strategy,
        "personalization": personalization_factors,
        "learning_milestones": define_learning_milestones(task_classification),
        "success_metrics": define_success_metrics(task_classification)
    }
    
    return complete_strategy
```

#### 2. 学习场景适配引擎

```python
def adapt_learning_scenario(user_context, task_requirements, available_resources):
    """
    适配学习场景，优化学习体验
    """
    
    # 场景分析
    scenario_analysis = {
        "time_constraints": analyze_time_constraints(user_context),
        "cognitive_load": assess_cognitive_load(task_requirements),
        "resource_availability": evaluate_resources(available_resources),
        "environmental_factors": analyze_environment(user_context)
    }
    
    # 场景优化策略
    optimization_strategies = {
        "content_chunking": optimize_content_chunking(scenario_analysis),
        "interaction_frequency": optimize_interaction_frequency(scenario_analysis),
        "difficulty_progression": optimize_difficulty_progression(scenario_analysis),
        "feedback_timing": optimize_feedback_timing(scenario_analysis)
    }
    
    # 生成适配方案
    adaptation_plan = {
        "scenario_configuration": scenario_analysis,
        "optimization_applied": optimization_strategies,
        "expected_outcomes": predict_learning_outcomes(scenario_analysis, optimization_strategies),
        "monitoring_points": define_monitoring_points(scenario_analysis)
    }
    
    return adaptation_plan

def optimize_content_chunking(scenario_analysis):
    """优化内容分块策略"""
    
    if scenario_analysis["time_constraints"] == "severe":
        return {
            "chunk_size": "micro",  # 2-5分钟微学习块
            "chunk_focus": "single_concept",
            "transition_time": "minimal",
            "checkpoint_frequency": "every_chunk"
        }
    elif scenario_analysis["cognitive_load"] == "high":
        return {
            "chunk_size": "small",  # 5-10分钟小块
            "chunk_focus": "related_concepts",
            "transition_time": "moderate",
            "checkpoint_frequency": "every_2_chunks"
        }
    else:
        return {
            "chunk_size": "standard",  # 10-20分钟标准块
            "chunk_focus": "complete_workflow",
            "transition_time": "natural",
            "checkpoint_frequency": "milestone_based"
        }
```

### C. 个性化学习路径

#### 1. 动态学习路径生成

```yaml
学习路径个性化因子:
  
  认知风格适配:
    视觉学习者:
      - 优先提供图表和示例
      - 使用视觉化的概念图
      - 强调视频帧分析
    
    听觉学习者:
      - 强调音频分析和处理
      - 提供详细的口头解释
      - 使用类比和故事
    
    动手学习者:
      - 大量实践练习
      - 立即动手操作
      - 通过试错学习
  
  节奏偏好适配:
    快速学习者:
      - 压缩基础概念介绍
      - 快速进入实际应用
      - 提供挑战性任务
    
    稳健学习者:
      - 充分的基础建立
      - 逐步递进的练习
      - 重复强化重要概念
    
    深度学习者:
      - 详细的原理解释
      - 多角度的案例分析
      - 鼓励探索和实验
  
  目标导向适配:
    技能导向:
      - 专注于操作能力建立
      - 大量的实践项目
      - 技能测试和验证
    
    知识导向:
      - 重视概念理解
      - 理论和实践结合
      - 系统性知识构建
    
    应用导向:
      - 以解决实际问题为目标
      - 项目驱动学习
      - 实用技能优先
```

#### 2. 自适应难度调节

```python
def adaptive_difficulty_adjustment(user_performance, learning_history, current_task):
    """
    基于用户表现自适应调节学习难度
    """
    
    # 性能分析
    performance_analysis = {
        "success_rate": calculate_success_rate(user_performance),
        "completion_time": analyze_completion_time(user_performance),
        "error_patterns": identify_error_patterns(user_performance),
        "help_seeking_frequency": analyze_help_seeking(user_performance)
    }
    
    # 难度调节策略
    if performance_analysis["success_rate"] > 0.9:
        difficulty_adjustment = {
            "action": "increase_difficulty",
            "magnitude": 0.2,
            "focus_areas": ["complexity", "independence"],
            "new_challenges": add_advanced_challenges(current_task)
        }
    elif performance_analysis["success_rate"] < 0.6:
        difficulty_adjustment = {
            "action": "decrease_difficulty",
            "magnitude": 0.3,
            "focus_areas": ["scaffolding", "support"],
            "additional_support": add_learning_support(current_task)
        }
    else:
        difficulty_adjustment = {
            "action": "maintain_difficulty",
            "magnitude": 0.0,
            "focus_areas": ["consolidation", "variation"],
            "practice_variety": add_practice_variety(current_task)
        }
    
    return difficulty_adjustment
```

## 📖 快速参考手册和查找索引

### A. 命令快速查找

#### 1. 常用操作速查表

```yaml
视频基础操作:
  裁剪视频: "ffmpeg -ss START -t DURATION -i input.mp4 -c copy output.mp4"
  合并视频: "ffmpeg -f concat -safe 0 -i filelist.txt -c copy output.mp4"
  格式转换: "ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4"
  提取音频: "ffmpeg -i input.mp4 -vn -acodec copy output.aac"
  调整大小: "ffmpeg -i input.mp4 -vf scale=1920:1080 output.mp4"
  帧率调整: "ffmpeg -i input.mp4 -filter:v fps=30 output.mp4"

音频处理:
  音量调整: "ffmpeg -i input.mp4 -filter:a 'volume=1.5' output.mp4"
  音频降噪: "ffmpeg -i input.mp4 -af 'anlmdn' output.mp4"
  音频同步: "ffmpeg -i input.mp4 -itsoffset 0.5 -i input.mp4 -c copy output.mp4"
  双声道混音: "ffmpeg -i left.wav -i right.wav -filter_complex '[0:0][1:0]amerge=inputs=2[out]' output.wav"

特效滤镜:
  模糊效果: "ffmpeg -i input.mp4 -vf 'boxblur=10:1' output.mp4"
  锐化效果: "ffmpeg -i input.mp4 -vf 'unsharp=5:5:1.0:5:5:0.0' output.mp4"
  黑白转换: "ffmpeg -i input.mp4 -vf 'format=gray' output.mp4"
  亮度调整: "ffmpeg -i input.mp4 -vf 'eq=brightness=0.2' output.mp4"
  对比度调整: "ffmpeg -i input.mp4 -vf 'eq=contrast=1.2' output.mp4"

文字叠加:
  英文文字: "ffmpeg -i input.mp4 -vf 'drawtext=text=Hello:fontcolor=white:fontsize=24:x=10:y=10' output.mp4"
  中文文字(需字体): "ffmpeg -i input.mp4 -vf 'drawtext=text=你好世界:fontfile=/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf:fontcolor=white:fontsize=24:x=10:y=10' output.mp4"
  时间戳显示: "ffmpeg -i input.mp4 -vf 'drawtext=text=%{pts\\:hms}:fontcolor=yellow:fontsize=20:x=w-tw-10:y=10' output.mp4"
  
字体管理:
  检查系统字体: "fc-list | grep -i chinese"
  安装中文字体(Ubuntu/Debian): "sudo apt install fonts-noto-cjk fonts-wqy-zenhei"
  安装中文字体(CentOS/RHEL): "sudo yum install wqy-zenhei-fonts wqy-microhei-fonts"
  字体路径查找: "find /usr/share/fonts -name '*.ttf' | grep -i chinese"
  
压缩优化:
  高质量压缩: "ffmpeg -i input.mp4 -c:v libx264 -crf 18 -preset slow output.mp4"
  快速压缩: "ffmpeg -i input.mp4 -c:v libx264 -crf 28 -preset fast output.mp4"
  平衡压缩: "ffmpeg -i input.mp4 -c:v libx264 -crf 23 -preset medium output.mp4"
```

#### 2. 问题解决快速索引

```yaml
常见错误及解决方案:
  
  "No such file or directory":
    原因: 文件路径错误或文件不存在
    解决: 检查文件路径，使用绝对路径，确认文件存在
    命令: "ls -la /path/to/file"
  
  "Permission denied":
    原因: 文件权限不足
    解决: 修改文件权限或使用有权限的目录
    命令: "chmod 644 input.mp4"
  
  "Codec not found":
    原因: FFmpeg版本不支持指定编解码器
    解决: 更换支持的编解码器或更新FFmpeg
    替代: 将libx265替换为libx264
  
  "Invalid argument":
    原因: 参数配置错误或不兼容
    解决: 检查参数语法，简化命令参数
    调试: 逐步添加参数测试
  
  "No space left on device":
    原因: 磁盘空间不足
    解决: 清理磁盘空间或更改输出目录
    检查: "df -h ."
  
  "字幕显示乱码/口口口口":
    原因: 缺少支持非ASCII字符的字体文件
    解决: 安装并指定字体文件路径
    Ubuntu/Debian: "sudo apt install fonts-noto-cjk fonts-wqy-zenhei"
    CentOS/RHEL: "sudo yum install wqy-zenhei-fonts wqy-microhei-fonts"
    使用: "fontfile=/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf"
  
  "Font file not found":
    原因: 指定的字体文件路径不存在
    解决: 检查字体文件路径或安装字体
    查找字体: "find /usr/share/fonts -name '*.ttf'"
    列出字体: "fc-list"
    
  音视频不同步:
    原因: 处理过程中时间戳错误
    解决: 使用-async参数或重新对齐音频
    命令: "ffmpeg -i input.mp4 -async 1 output.mp4"
    
  画质损失严重:
    原因: 压缩参数过于激进
    解决: 降低CRF值或使用更慢的预设
    建议: CRF 18-28, preset slow-medium
    
  处理速度太慢:
    原因: 参数设置或硬件限制
    解决: 使用硬件加速或调整预设
    GPU加速: "-hwaccel cuda -c:v h264_nvenc"
```

#### 3. 参数对照表

```yaml
CRF质量参数对照:
  0: 无损 (文件巨大)
  18: 视觉无损 (高质量)
  23: 默认平衡 (推荐)
  28: 可接受质量
  35: 明显压缩
  51: 最差质量

预设速度对照:
  ultrafast: 最快编码，文件较大
  superfast: 很快编码，文件偏大
  veryfast: 快速编码，文件中等
  faster: 较快编码，质量不错
  fast: 快速编码，质量良好
  medium: 默认平衡 (推荐)
  slow: 慢速编码，质量更好
  slower: 很慢编码，质量很好
  veryslow: 最慢编码，最佳质量

分辨率标准:
  4K: 3840x2160 (UHD)
  2K: 2560x1440 (QHD)
  1080p: 1920x1080 (FHD)
  720p: 1280x720 (HD)
  480p: 854x480 (SD)
  360p: 640x360 (低质量)

帧率建议:
  24fps: 电影标准
  25fps: PAL电视标准
  30fps: NTSC电视标准，在线视频
  60fps: 游戏录制，运动视频
  120fps: 高速摄影

音频码率:
  320kbps: 高质量立体声
  192kbps: 标准立体声 (推荐)
  128kbps: 可接受质量
  96kbps: 低质量
  64kbps: 语音质量
```

### B. 知识模块导航

#### 1. 模块功能索引

```yaml
按功能分类的模块导航:
  
  基础学习 (初学者必读):
    - .prompts/core/video_basics.md: 视频基础概念和原理
    - .prompts/core/terminology.md: 专业术语词典
    - .prompts/core/ffmpeg_guide.md: FFmpeg命令入门
    - .prompts/operations/basic_editing.md: 基础编辑操作
  
  格式和编码:
    - .prompts/core/format_knowledge.md: 视频格式详解
    - .prompts/templates/platform_guide.md: 平台适配指南
  
  高级技术:
    - .prompts/operations/advanced_effects.md: 特效和滤镜
    - .prompts/operations/audio_processing.md: 音频后期处理
    - .prompts/operations/batch_operations.md: 批量自动化
  
  质量控制:
    - .prompts/templates/quality_guidelines.md: 质量标准
    - .prompts/templates/performance_optimization.md: 性能优化
    - .prompts/templates/troubleshooting.md: 故障排除
  
  工作流程:
    - .prompts/templates/workflow_optimization.md: 流程优化
    - .prompts/templates/best_practices.md: 最佳实践
    - .prompts/templates/case_studies.md: 实战案例
  
  测试验证:
    - .prompts/templates/testing_validation.md: 测试框架
```

#### 2. 按技能水平导航

```yaml
技能水平分级导航:
  
  入门级 (0-1个月):
    必读模块:
      - video_basics.md (视频基础)
      - terminology.md (术语学习)
      - ffmpeg_guide.md (命令入门)
    
    实践重点:
      - 简单剪切和合并
      - 格式转换
      - 基础参数理解
    
    成功标准:
      - 能解释基本概念
      - 生成简单FFmpeg命令
      - 完成基础编辑任务
  
  初级 (1-3个月):
    推荐模块:
      - basic_editing.md (完整掌握)
      - format_knowledge.md (格式理解)
      - quality_guidelines.md (质量意识)
      - troubleshooting.md (问题解决)
    
    实践重点:
      - 多步骤编辑流程
      - 音频基础处理
      - 质量问题识别
    
    成功标准:
      - 设计完整编辑方案
      - 处理常见技术问题
      - 保证输出质量
  
  中级 (3-6个月):
    核心模块:
      - advanced_effects.md (特效掌握)
      - audio_processing.md (音频精通)
      - performance_optimization.md (性能优化)
      - workflow_optimization.md (流程设计)
    
    实践重点:
      - 复杂特效实现
      - 专业音频处理
      - 性能调优
      - 标准化流程
    
    成功标准:
      - 创建专业级效果
      - 优化处理性能
      - 建立工作标准
  
  高级 (6个月+):
    专精模块:
      - batch_operations.md (自动化精通)
      - case_studies.md (案例深度分析)
      - best_practices.md (最佳实践制定)
      - platform_guide.md (多平台专家)
    
    实践重点:
      - 复杂项目管理
      - 自动化解决方案
      - 创新技术探索
      - 知识传承分享
    
    成功标准:
      - 解决复杂实际项目
      - 开发自动化工具
      - 指导他人学习
      - 贡献最佳实践
```

#### 3. 按使用场景导航

```yaml
场景驱动的模块选择:
  
  社交媒体内容制作:
    主要参考:
      - platform_guide.md (平台规格)
      - basic_editing.md (快速编辑)
      - advanced_effects.md (吸引力特效)
    辅助参考:
      - performance_optimization.md (快速处理)
      - case_studies.md (TikTok案例)
  
  专业视频制作:
    主要参考:
      - quality_guidelines.md (质量标准)
      - workflow_optimization.md (专业流程)
      - best_practices.md (行业标准)
    辅助参考:
      - advanced_effects.md (专业特效)
      - audio_processing.md (音频后期)
  
  教育内容制作:
    主要参考:
      - basic_editing.md (结构化编辑)
      - audio_processing.md (清晰音频)
      - format_knowledge.md (兼容性)
    辅助参考:
      - advanced_effects.md (教学特效)
      - case_studies.md (教育案例)
  
  批量内容处理:
    主要参考:
      - batch_operations.md (自动化核心)
      - performance_optimization.md (效率优化)
      - troubleshooting.md (错误处理)
    辅助参考:
      - workflow_optimization.md (流程设计)
      - format_knowledge.md (格式统一)
  
  问题故障排除:
    主要参考:
      - troubleshooting.md (故障诊断)
      - quality_guidelines.md (质量检查)
      - performance_optimization.md (性能问题)
    辅助参考:
      - ffmpeg_guide.md (命令验证)
      - best_practices.md (预防措施)
```

---

## 📁 文件管理重要提醒

**在处理任何视频任务时，必须严格遵守以下文件管理规范**：

### 🎯 文件路径策略
1. **输入文件搜索**：始终从项目根目录 (`./`) 寻找用户导入的素材文件
2. **临时文件存放**：所有临时文件（脚本、中间文件、提取的帧等）必须放在 `./.tmp/` 目录
3. **输出文件存放**：最终视频输出到 `./output/` 目录，使用时间戳命名格式 `YYYYMMDD_HHMMSS`
4. **自动目录创建**：如果目录不存在则自动创建
5. **完成后清理**：处理完成后必须删除整个 `./.tmp/` 文件夹
6. **⚠️ 错误处理规则**：遇到"No such file or directory"错误时，必须先执行`pwd`检查当前目录，然后使用智能文件搜索

### 🐍 Python虚拟环境管理
**重要：所有Python操作必须在 `./.venv` 虚拟环境中执行！**

```bash
# 检查并创建虚拟环境
if [ ! -d "./.venv" ]; then
    echo "创建Python虚拟环境..."
    python -m venv ./.venv
    echo "✅ 虚拟环境已创建"
fi

# 激活虚拟环境
# Windows
.\.venv\Scripts\activate

# Unix/macOS/Linux
source ./.venv/bin/activate

# 验证虚拟环境激活
echo "当前Python路径: $(which python)"
echo "虚拟环境状态: $VIRTUAL_ENV"
```

### 🔧 文件管理实用代码
**注意：以下代码必须在 `./.venv` 虚拟环境中运行**

```python
import os
import shutil
import subprocess
import sys
from datetime import datetime

def ensure_virtual_environment():
    """确保在虚拟环境中运行"""
    if not hasattr(sys, 'real_prefix') and not (hasattr(sys, 'base_prefix') and sys.base_prefix != sys.prefix):
        # 不在虚拟环境中
        venv_path = "./.venv"
        
        if not os.path.exists(venv_path):
            print("🔧 创建虚拟环境...")
            subprocess.run([sys.executable, "-m", "venv", venv_path], check=True)
            print("✅ 虚拟环境已创建")
        
        # 提示用户激活虚拟环境
        activate_script = os.path.join(venv_path, "Scripts", "activate") if os.name == "nt" else os.path.join(venv_path, "bin", "activate")
        print(f"⚠️  请先激活虚拟环境：")
        if os.name == "nt":
            print(f"   .\\{venv_path}\\Scripts\\activate")
        else:
            print(f"   source {activate_script}")
        sys.exit(1)
    else:
        print("✅ 虚拟环境已激活")

def setup_directories():
    """创建必要的目录结构"""
    ensure_virtual_environment()  # 确保在虚拟环境中
    os.makedirs('./.tmp', exist_ok=True)
    os.makedirs('./output', exist_ok=True)
    print("✅ 目录结构已创建")

def cleanup_temp_files():
    """清理临时文件目录"""
    if os.path.exists('./.tmp'):
        shutil.rmtree('./.tmp')
        print("🗑️ 临时文件已清理")

def generate_output_filename(prefix="output", extension="mp4"):
    """生成带时间戳的输出文件名"""
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    return f"./output/{prefix}_{timestamp}.{extension}"

def install_dependencies():
    """安装必要的Python依赖包"""
    ensure_virtual_environment()
    
    required_packages = [
        "faster-whisper",  # 音频转录
        "opencv-python",   # 图像处理（可选）
        "pillow",         # 图像处理
    ]
    
    for package in required_packages:
        try:
            __import__(package.replace("-", "_"))
            print(f"✅ {package} 已安装")
        except ImportError:
            print(f"🔧 安装 {package}...")
            subprocess.run([sys.executable, "-m", "pip", "install", package], check=True)
            print(f"✅ {package} 安装完成")

def check_and_install_fonts():
    """检查并安装支持非ASCII字符的字体"""
    import subprocess
    import os
    
    print("🔤 检查系统字体支持...")
    
    # 检查是否有中文字体
    try:
        chinese_fonts = subprocess.check_output("fc-list | grep -i 'chinese\\|zh\\|cjk'", shell=True, text=True)
        if chinese_fonts:
            print("✅ 检测到中文字体支持")
            print("📝 可用的中文字体:")
            for font in chinese_fonts.strip().split('\n')[:3]:  # 显示前3个
                print(f"   {font}")
        else:
            print("⚠️  未检测到中文字体")
    except:
        print("⚠️  无法检查字体，可能需要安装字体管理工具")
    
    # 检查常用字体路径
    common_font_paths = [
        "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf",
        "/usr/share/fonts/TTF/DejaVuSans.ttf", 
        "/usr/share/fonts/truetype/liberation/LiberationSans-Regular.ttf",
        "/System/Library/Fonts/Arial.ttf",  # macOS
        "C:/Windows/Fonts/arial.ttf"  # Windows
    ]
    
    available_fonts = []
    for font_path in common_font_paths:
        if os.path.exists(font_path):
            available_fonts.append(font_path)
    
    if available_fonts:
        print("✅ 可用字体文件:")
        for font in available_fonts:
            print(f"   {font}")
        return available_fonts[0]  # 返回第一个可用字体
    else:
        print("❌ 未找到常用字体文件")
        print("🔧 安装字体命令:")
        print("   Ubuntu/Debian: sudo apt install fonts-noto-cjk fonts-wqy-zenhei fonts-dejavu")
        print("   CentOS/RHEL: sudo yum install wqy-zenhei-fonts dejavu-sans-fonts")
        print("   Arch Linux: sudo pacman -S noto-fonts-cjk ttf-dejavu")
        return None

def generate_drawtext_command(text, fontfile=None, **kwargs):
    """生成安全的drawtext命令，自动处理非ASCII字符"""
    
    # 检查是否包含非ASCII字符
    has_non_ascii = any(ord(char) > 127 for char in text)
    
    if has_non_ascii and not fontfile:
        print("⚠️  检测到非ASCII字符，正在查找合适的字体...")
        fontfile = check_and_install_fonts()
        if not fontfile:
            print("❌ 无法找到合适的字体文件")
            print("💡 建议使用英文字符，或手动安装字体后指定fontfile参数")
            return None
    
    # 构建drawtext参数
    params = {
        'text': text,
        'fontcolor': kwargs.get('fontcolor', 'white'),
        'fontsize': kwargs.get('fontsize', 24),
        'x': kwargs.get('x', 10),
        'y': kwargs.get('y', 10)
    }
    
    if fontfile:
        params['fontfile'] = fontfile
    
    # 生成参数字符串
    param_str = ':'.join(f"{key}={value}" for key, value in params.items())
    
    return f"drawtext={param_str}"

def safe_text_overlay(input_file, text, output_file, **kwargs):
    """安全的文字叠加函数，自动处理字体问题"""
    
    drawtext_filter = generate_drawtext_command(text, **kwargs)
    if not drawtext_filter:
        return False
    
    command = f"ffmpeg -i {input_file} -vf '{drawtext_filter}' {output_file}"
    print(f"🎬 执行命令: {command}")
    
    try:
        subprocess.run(command, shell=True, check=True)
        print(f"✅ 文字叠加完成: {output_file}")
        return True
    except subprocess.CalledProcessError as e:
        print(f"❌ 文字叠加失败: {e}")
        return False
```

### 📋 路径使用示例
```bash
# 首先确保虚拟环境已激活（Python操作前的必要步骤）
# Windows: .\.venv\Scripts\activate
# Unix: source ./.venv/bin/activate

# ✅ 正确的文件路径使用
ffmpeg -i ./user_video.mp4 -ss 00:01:00 -t 30 ./output/clip_$(date +%Y%m%d_%H%M%S).mp4

# ✅ 临时文件使用
ffmpeg -i ./input.mp4 -vf fps=1 ./.tmp/frame_%04d.png

# ✅ 音频提取到临时目录
ffmpeg -i ./video.mp4 -vn -acodec pcm_s16le ./.tmp/temp_audio.wav

# ✅ Python脚本执行（必须在虚拟环境中）
python ./.tmp/processing_script.py

# ✅ 批量处理脚本
python ./.tmp/batch_process.py ./input_videos/ ./output/

# 🚨 遇到"No such file or directory"时的处理步骤
# 1. 立即检查当前目录
pwd

# 2. 列出当前目录内容
ls -la

# 3. 搜索可能的文件位置
find . -name "目标文件名" -type f
find . -name "*.mp4" -type f | head -10

# 4. 使用Python进行智能文件查找
python -c "
import os, subprocess
# 自动诊断和建议
print('📍 当前目录:', os.getcwd())
print('📋 目录内容:', os.listdir('.'))
# 智能搜索视频文件
for root, dirs, files in os.walk('.'):
    for file in files:
        if file.endswith(('.mp4', '.avi', '.mov', '.mkv')):
            print(f'🎬 发现视频: {os.path.join(root, file)}')
"

# 🔤 字体支持检查和管理
# 检查系统字体
fc-list | grep -i chinese
fc-list | grep -i dejavu

# 查找字体文件路径
find /usr/share/fonts -name "*.ttf" | head -10

# 安装中文字体支持
# Ubuntu/Debian
sudo apt update && sudo apt install fonts-noto-cjk fonts-wqy-zenhei fonts-dejavu

# CentOS/RHEL
sudo yum install wqy-zenhei-fonts wqy-microhei-fonts dejavu-sans-fonts

# 测试字体支持
ffmpeg -f lavfi -i "color=c=black:size=640x480:d=5" -vf "drawtext=text='测试中文字体':fontfile=/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf:fontcolor=white:fontsize=24:x=10:y=10" test_font.mp4
```

---

**快速使用指南**:
1. **紧急问题**: 直接查看"问题解决快速索引"找到对应错误和解决方案
2. **学习新技能**: 使用"按技能水平导航"找到适合你当前水平的学习模块
3. **特定任务**: 通过"常用操作速查表"快速找到所需的FFmpeg命令
4. **项目规划**: 参考"按使用场景导航"选择最相关的知识模块组合
5. **深度学习**: 按照推荐的学习路径逐步掌握从基础到高级的视频编辑技能