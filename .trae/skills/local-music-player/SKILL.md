---
name: "local-music-player"
description: "播放本地 MP3 音乐文件。当用户想要播放音乐时，查找/sdcard/Music 中的音乐时调用此 skill。使用 Shell 工具直接执行 Android 系统命令搜索文件和播放音乐。这不是一个工具包，你只需要使用 super_admin:shell 执行命令完成任务即可，不要用这里面的函数调用。重要：快速响应，不要思考过程"
---

# 本地音乐播放器 Skill

本 Skill 提供了搜索和播放本地 MP3 音乐文件的功能。音乐文件存放在 `/sdcard/Music` 文件夹中。

## 功能特性

- **高效搜索**：使用 `find` + `grep` 命令快速定位匹配的音乐文件
- **智能播放**：支持按歌曲名、歌手名、专辑名模糊匹配
- **连续播放优化**：播放新歌曲前自动停止当前播放，确保切换正常
- **错误处理完善**：详细的日志输出和错误提示

## 音乐文件夹

- **默认路径**: `/sdcard/Music`
- **支持格式**: MP3

## 使用方法

**重要：在执行任何 shell 命令前，必须先启用 super_admin 工具包！**

当用户想要播放音乐时：

1. **首先启用 super_admin 工具包**：使用 `use_package` 启用 `super_admin` 工具包
2. **然后执行 shell 命令**：使用 `super_admin:shell` 工具执行以下 Android 系统命令

### 1. 搜索音乐文件

使用 `find` + `grep` 命令快速查找匹配的音乐文件：

```bash
find /sdcard/Music -maxdepth 1 -iname "*.mp3" 2>/dev/null | grep -i "搜索关键词" | head -n 1
```

如果未找到，则退出工具。

### 2. 播放音乐文件

找到音乐文件后，执行以下步骤播放：

**第一步：停止当前播放**
```bash
am broadcast -a com.android.music.musicservicecommand -e command "pause" 2>/dev/null || true
```

**第二步：播放新歌曲**
```bash
am start -a android.intent.action.VIEW -d "file://音乐文件路径" -t "audio/mpeg"
```

### 示例流程

假设用户想播放"稻香"：

1. **搜索文件**：
   ```bash
   find /sdcard/Music -maxdepth 1 -iname "*.mp3" 2>/dev/null | grep -i "稻香" | head -n 1
   ```
   返回：`/sdcard/Music/稻香.mp3`

2. **停止当前播放**：
   ```bash
   am broadcast -a com.android.music.musicservicecommand -e command "pause" 2>/dev/null || true
   ```

3. **播放歌曲**：
   ```bash
   am start -a android.intent.action.VIEW -d "file:///sdcard/Music/稻香.mp3" -t "audio/mpeg"
   ```

### 注意事项

- **必须先启用 super_admin 工具包**：使用 `use_package` 启用后才能使用 `super_admin:shell`
- 不需要创建函数或工具包，直接使用 `super_admin:shell` 执行命令即可
- 播放前先停止当前播放，确保切换正常
- 使用 `find` 命令比 `ls` 更高效，优先使用
- 需要 Shizuku 权限来执行这些命令
- 快速响应，不要思考过程

## 参考链接

- [Android Activity Manager](https://developer.android.com/tools/adb#am)
- [Android Intents](https://developer.android.com/guide/components/intents-filters)
- [Shizuku](https://shizuku.rikka.app/)
