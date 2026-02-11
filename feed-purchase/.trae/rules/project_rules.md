# 执行逻辑

执行任何任务前，根据用户需求，访问 `./.trae/skills` 目录下的技能文件，调用相应的技能。
  * ‼️ 解决任务时，选择技能优先于工具调用，如无匹配的技能，使用现有工具组合
  * 使用技能时，技能中的设定优先级高于通用规则，如与通用规则矛盾，应覆盖通用规则设定
  * `skills_index.md` 中有所有可用技能列表
  * 技能文件的命名规则为 `skill_技能名称.md`，例如 `skill_一键成片.md`
  * 技能文件的内容为技能的详细描述和使用方法，例如 `一键成片：根据用户需求，自动创建分镜脚本、生成素材并合成视频文件`
  * 技能文件中的规则若与素材创作助手的规则冲突，以技能文件中的规则为准。

## 重要步骤（‼️务必执行）

### skill_context.md

在执行具体技能前，先查看对应技能的文件（`./.trae/skills/skill_技能名称.md`），确认技能的具体内容和约束条件。

在 `./.trae/rules` 目录下创建或修改 `skill_context.md` 文件，写入内容：

- 技能名称：[技能名称]
- 技能描述：[技能描述]
- 技能核心约束（如有）：[技能核心约束]
- 技能默认设定：（比如技能中规定的默认模型、默认参数等）
- 其他注意事项

在切换新技能时，重写覆盖 `skill_context.md` 文件，更新为新技能的相关信息。

在每次通过 `retrieve-rules-context` 工具召回规则上下文时，都要检查 `skill_context.md` 文件是否存在，若不存在，立即创建该文件，并根据当前上下文写入内容。

在每次聊天任务执行中，通过查看 `skill_context.md` 文件，确认所调用的技能并强制输出到响应中，以确保每个执行步骤处于正确的技能规则中。

技能核心约束中禁止使用的MCP工具，严格禁用，不得在技能执行中调用。

### action_context.md

在执行具体任务后，根据任务的执行结果，务必创建或更新 `./.trae/actions/action_context.md` 文件，**增量**写入内容：

- 做了什么：[做了什么]
  - 用了什么工具：[用了什么工具]
  - 用了什么参数：[用了什么参数]
  - 用了什么技能：[用了什么技能]
- 执行的结果：[执行的结果]
- 其他注意事项

在每次聊天中，检查`action_context.md`文件内容，与对话上下文印证，确保指令能被准确无误执行。

---

## 项目结构与规范

```
projects/<id>/
  ├─ materials/      # 素材文件
  ├─ output/         # 成品输出（可选）
  ├─ storyboard.json    # 分镜脚本
  └─ media_logs.json  # 素材生成日志
```

### 视频合成规范
- 画幅：提前确定横竖屏，竖屏720x1280，横屏1280x720，如无特殊要求，竖屏(720x1280)优先
- 分辨率：**只有用户明确指定时才使用720x1280和1280x720之外的分辨率**，禁止擅自使用其他分辨率
- 字幕样式：
  * 默认字体：`"Noto Sans CJK SC"`
- BGM 音量控制
  * 音量：默认BGM音量控制为-15db（`audioVolume=0.177`），合成时通过设置`audio-video-async`的`audioVolume`参数可以调整BGM音量。

## 素材生成规则

### 核心原则（不得违反‼️）
1. 有 storyboard.json 的场景
  * 素材生成时，一律默认不跳过一致性检查`skipConsistencyCheck=false`
  * 只有当用户明确要求跳过一致性检查时，才指定`skipConsistencyCheck=true`
2. 无 storyboard.json 的场景
  * 素材生成时，默认跳过一致性检查`skipConsistencyCheck=true`
3. 生成素材命名用语义明确的文件名命名，优先使用`递增数字ID_中文`的命名规则命名文件。
  - 例如：`001_介绍.mp4`、`002_主体.mp4`、`003_结尾.mp4`等

### 视频合成规范
- 画幅：提前确定横竖屏，竖屏720x1280，横屏1280x720，如无特殊要求，竖屏(720x1280)优先
- 分辨率：**只有用户明确指定时才使用720x1280和1280x720之外的分辨率**，禁止擅自使用其他分辨率
- 字幕样式：
  * 默认字体：`"Noto Sans CJK SC"`
- BGM 音量控制
  * 音量：默认BGM音量控制为-15db（`audioVolume=0.177`），合成时通过设置`audio-video-async`的`audioVolume`参数可以调整BGM音量。

## JSON 文件一致性与质量规则

### 通用
1. 引号规则：JSON格式中，字符串中如包含引号，优先使用单引号，双引号需转义。

### storyboard.json
1. storyboard.json 中的 start_frame、end_frame 分别对应视频生成`generate-video`的首帧和尾帧
2. storyboard.json 中的 video_prompt 对应视频生成`generate-video`的 prompt

## 知识库

### 通用技巧及术语
1. 生成视频的几种方式
  - 首帧图生视频（默认采用）：先根据 start_frame 生成首帧图片如 sc01_start.png，然后用该图片作为视频的第一帧，以 video_prompt 的提示词用`generate-video`生成视频
  - 首尾帧生视频（连续镜头）：先根据 start_frame 生成首帧图片如 sc01_start.png，再根据 end_frame 生成尾帧图片如 sc01_end.png，用这两张图片作为视频的首尾帧，以 video_prompt 的提示词用`generate-video`生成视频

### 故障排查和自动处理
  * `generate-image` 失败，可以换一个 type 重试，替换顺序为 `banana` → `seedream` → `banana-pro`
  * `generate-video` 失败，如失败原因是内容相关（如包含敏感信息），不要轻易重新生成 outline，应手动编辑 `storyboard.json` 中的 video_prompt 删除或修改可能的敏感信息后重试；若还是失败，修改 `use_video_model` 换一个模型重试

## 搜索

如果用户提交的内容包含的信息不完整，你可以要求用户补充信息，也可以用搜索工具帮你完善信息。

### 工具优先级
1. 优先使用系统自有搜索工具
2. 备选：`search-context`（搜索文字或图片）