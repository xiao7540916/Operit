---
name: "local-music-player"
description: "播放本地 MP3 音乐文件。当用户想要播放/sdcard/Music 文件夹中的音乐时调用此 skill。必须首先启用 super_admin 工具包。使用 Shizuku 权限执行 shell 命令直接播放指定歌曲。"
---

# 本地音乐播放器 Skill

本 Skill 提供了搜索和播放本地 MP3 音乐文件的功能。音乐文件存放在 `/sdcard/Music` 文件夹中。

## 功能特性

- 搜索并播放本地音乐文件（支持按歌曲名、歌手名模糊匹配）
- 使用 Shizuku 权限执行 shell 命令直接播放指定的 MP3 文件

## 音乐文件夹

- **默认路径**: `/sdcard/Music`
- **支持格式**: MP3

## 完整工具包示例

```javascript
/*
METADATA
{
    "name": "local_music_player",
    "display_name": {
        "zh": "本地音乐播放器",
        "en": "Local Music Player"
    },
    "description": {
        "zh": "播放本地 MP3 音乐文件",
        "en": "Play local MP3 music files"
    },
    "enabledByDefault": true,
    "category": "Media",
    "tools": [
    ],
    "requiredSkills": ["super_admin"]
}
*/

const localMusicPlayer = (function() {
    const MUSIC_FOLDER = '/sdcard/Music';

    async function play_by_search(params) {
        try {
            const query = params.query.toLowerCase().trim();
            
            // 第一步：使用 super_admin 工具包的 terminal 命令到/sdcard/Music 目录查找音乐
            console.log(`使用 super_admin 工具包查找音乐文件：${query}`);
            
            // 使用 ls 命令列出/sdcard/Music 目录下的所有 MP3 文件
            const listCommand = `ls /sdcard/Music/*.mp3 2>/dev/null || echo "NO_FILES"`;
            const listResult = await Tools.SuperAdmin.terminal({
                command: listCommand,
                background: 'false',
                timeoutMs: '10000'
            });
            
            console.log(`文件列表命令输出：${listResult.output}`);
            
            if (!listResult.output || listResult.output.includes('NO_FILES') || listResult.output.trim() === '') {
                return {
                    success: false,
                    message: '未找到任何 MP3 音乐文件'
                };
            }
            
            // 解析文件列表，查找匹配的文件
            const files = listResult.output.split('\n').filter(line => line.trim() !== '');
            const matchedFiles = files.filter(file => {
                const fileName = file.split('/').pop().toLowerCase();
                return fileName.includes(query);
            });
            
            if (matchedFiles.length === 0) {
                return {
                    success: false,
                    message: `未找到匹配 "${params.query}" 的音乐`
                };
            }
            
            // 获取第一个匹配的文件
            const filePath = matchedFiles[0].trim();
            const fileName = filePath.split('/').pop();
            
            console.log(`找到匹配文件：${filePath}`);
            
            // 第二步：直接调用 shell 命令播放音乐
            const shellCommand = `am start -a android.intent.action.VIEW -d "file://${filePath}" -t "audio/mpeg"`;
            
            console.log(`执行 shell 命令：${shellCommand}`);
            
            const shellResult = await Tools.System.shell(shellCommand);
            
            if (shellResult.exitCode === 0) {
                return {
                    success: true,
                    message: `正在播放：${fileName}`
                };
            } else {
                return {
                    success: false,
                    message: `播放音乐失败：${shellResult.output || '未知错误'}`
                };
            }
        } catch (error) {
            return {
                success: false,
                message: `播放音乐失败：${error.message}`
            };
        }
    }

    function wrap(fn) {
        return async (params) => {
            const result = await fn(params);
            complete(result);
            return result;
        };
    }

    return {
        play_by_search: wrap(play_by_search)
    };
})();

exports.play_by_search = localMusicPlayer.play_by_search;
```

## 使用示例

### 搜索并播放
```javascript
const result = await localMusicPlayer.play_by_search({ query: "稻香" });
console.log(result.message);
// 正在播放：稻香.mp3
```

### 按歌手搜索播放
```javascript
const result = await localMusicPlayer.play_by_search({ query: "周杰伦" });
console.log(result.message);
// 正在播放：周杰伦 - 稻香.mp3
```

## 注意事项

1. **权限要求**: 需要存储权限来读取 `/sdcard/Music` 文件夹，需要 Shizuku 权限来执行 shell 命令
2. **文件格式**: 目前仅支持 MP3 格式
3. **播放方式**: 使用 Shizuku 权限执行 shell 命令直接播放音乐文件
4. **Shizuku 要求**: 需要设备已安装并激活 Shizuku，授予应用 Shizuku 权限
5. **文件夹**: 默认从 `/sdcard/Music` 文件夹搜索音乐文件
6. **唯一方案**: 本技能只有这一种播放方案，不提供其他备选方案

## 技术说明

### 工作流程

1. **第一步：使用 super_admin 工具包查找音乐文件**
   - 调用 `Tools.SuperAdmin.terminal` 执行 `ls /sdcard/Music/*.mp3` 命令
   - 获取 `/sdcard/Music` 目录下所有 MP3 文件列表
   - 根据用户输入的关键词过滤匹配的文件

2. **第二步：调用 shell 命令播放音乐**
   - 使用 `Tools.System.shell` 执行 `am start` 命令
   - 直接播放找到的第一个匹配文件

### 使用的 Shell 命令

**查找文件：**
```bash
ls /sdcard/Music/*.mp3 2>/dev/null || echo "NO_FILES"
```

**播放音乐：**
```bash
am start -a android.intent.action.VIEW -d "file:///sdcard/Music/歌曲名.mp3" -t "audio/mpeg"
```

### 为什么需要 super_admin 工具包？

- ✅ **可靠访问**：super_admin 的 terminal 工具运行在 Ubuntu 环境中，已正确挂载 sdcard
- ✅ **权限保障**：通过 super_admin 可以可靠地访问 `/sdcard/Music` 目录
- ✅ **避免失败**：直接使用 Tools.Files.list 可能会因为权限问题失败

## 参考链接

- [Android Activity Manager](https://developer.android.com/tools/adb#am)
- [Android Intents](https://developer.android.com/guide/components/intents-filters)
- [Shizuku](https://shizuku.rikka.app/)
