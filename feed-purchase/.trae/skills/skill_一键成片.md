# 通用视频规则

你通过 Zerocut 根据用户需求创作通用视频

## 标准流水线

### 全局配置和注意事项

1. 音画同步：除非用户明确指定，否则生成视频时一律**不静音**（默认`mute=false`）
2. 一致性检查
  - ‼️ 除非用户明确要求不传，否则为分镜生成图(generate-image)和视频(generate-video)时，都务必传 `sceneIndex` 参数，从 1 开始编号
3. 分镜自动化：在第一次创建视频时，根据用户需求，使用`generate-video-outlines`生成分镜草稿 storyboard.json，生成后直接使用，不需要进行任何修改。
4. 故障排查和自动处理
  - 一般的错误，除非用户明确许可，否则禁止自行跳过一致性检查，而是修改 storyboard.json 中的相关内容，以保持一致性
5. 调用 `audio-video-sync` 合成视频时，如有对话或旁白，同步合成字幕(参数`addSubtitles=true`)
6. 在执行任务的过程中，任何工具返回`Not enough credits`错误时，必须暂停任务，等待用户充值后由用户手动触发继续执行

### 新建

1. 确保项目已正确开启：`project-open` 已被调用
2. 使用`retrieve-rules-context`，召回规则上下文
3. 分镜创作：根据用户需求，使用`generate-video-outlines`生成分镜草稿 storyboard.json
  * 使用`generate-video-outlines`时，prompt 请直接转述用户描述，无需自行分析整理或创作
4. 使用  `generate-image` 生成各分镜图片
  - 生成图片时，必须用`referenceImages`参数引用`outline_sheet.png`这张图，这张图已经由`generate-video-outlines`生成
  - 引用 `outline_sheet.png` 时，参考图片的类型(type)为 `normal`
  - 生成图片时，必须用`sceneIndex`参数指定分镜编号，从 1 开始编号
5. 使用  `generate-video` 生成各分镜视频
  - 生成视频时，必须用`sceneIndex`参数指定分镜编号，从 1 开始编号
  - 生成视频时，必须不静音（参数`mute=false`）
6. 使用 `generate-music-or-mv` 生成背景音乐
  - 背景音乐时长为分镜视频总时长，若不足30秒应补足30秒
7. 合成视频：调用`audio-video-sync`输出视频并根据情况合成字幕(参数`addSubtitles=true`)，自动下载到本地
  - 视频默认不包含字幕，如有对话或旁白，务必在调用`audio-video-sync`时设置参数`addSubtitles=true`
8. 关闭项目 → `project-close`

### 修改

1. 确保项目已启动 → `project-open`
2. 修改脚本 → 按用户要求直接手动修改 storyboard.json （⚠️ 不再使用`generate-video-outlines`重新生成）
3. 更新素材 → 重新生成需要修改的素材
4. 重新合成视频 → `audio-video-sync` 输出视频并合成字幕(参数`addSubtitles=true`)，自动下载到本地
5. 关闭项目 → `project-close`

---

## 角色参考图
* 如需保持主要角色形象一致，你可以先用`generate-character-image`生成角色三视图，然后参考三视图创建分镜草稿 storyboard.json

---

## 质量建议

### materials 资源命名规范

- 场景素材：`sc01_bg.png`、`sc01_motion.mp4`、`sc01_vo.mp3`
- 通用素材：`main_bgm_60s.mp3`
- 合成视频：`<主题名>.v<版本>.mp4`

### 工作流管理
* 规划先行：先分析制定执行计划
* 工作流顺序：规划→搜索→分镜→图片→视频→BGM→合成视频
* 视频生成策略：
  - 先使用`generate-image`生成首帧图片
  - 再使用`generate-video`生成动态视频
* 统一命名：`scXX_*`、`main_bgm_*`、`*_vo.*`
* 时长控制：单镜头3-16s

### 图生视频技巧
* 运动导向：提示词=主体运动+背景变化+镜头运动
* 特征定位：突出主体特征(老人、戴墨镜的女人)便于识别
* 环境一致性：确保场景间环境元素一致
  - 时间：保持时间段一致(白天、夜晚)，避免无故突变
  - 天气：保持天气状况一致(晴天、雨天)
  - 地点：场景转换符合空间逻辑
  - 光线：保持光源方向和强度一致

### BGM 音量控制
* 音量：默认BGM音量控制为-15db（`audioVolume=0.177`），通过设置`audio-video-async`的`audioVolume`参数可以调整BGM音量。

---

## 规划与搜索规则

### 需求分析
- 理解核心需求：明确视频主题、目标受众、预期效果
- 确定视频类型：科普解说、产品介绍、故事叙述等
- 分析技术要求：视频时长、画幅比例、风格偏好
- 识别素材需求：需要的图片、视频、音频素材

### 搜索内容
- 特定领域知识、热点话题、视觉参考、事实验证