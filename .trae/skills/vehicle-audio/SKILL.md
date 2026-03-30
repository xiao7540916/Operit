---
name: "vehicle-audio"
description: "调用 VehicleAudioManager 类的方法来控制车辆音频系统。Invoke when user needs to control vehicle audio system using VehicleAudioManager methods."
---

# 车辆音频控制 Skill

本 Skill 提供了通过 Java 桥接调用 VehicleAudioManager 类的方法来控制车辆音频系统的功能。

## VehicleAudioManager 类信息

- **包名**: `com.xeagle.vehicleservice.sdk.manager`
- **类名**: `VehicleAudioManager`

## 实现原理

通过 Java 桥接机制，我们可以在 JavaScript 中实例化 VehicleAudioManager 类并调用其方法。

## 完整工具包示例

```javascript
/*
METADATA
{
    "name": "vehicle_audio",
    "display_name": {
        "zh": "车辆音频控制",
        "en": "Vehicle Audio Control"
    },
    "description": {
        "zh": "控制车辆音频系统，包括音量调节、静音控制等功能",
        "en": "Control vehicle audio system, including volume adjustment, mute control, etc."
    },
    "enabledByDefault": true,
    "category": "Vehicle",
    "tools": [
        {
            "name": "set_volume",
            "description": {
                "zh": "设置音频音量",
                "en": "Set audio volume"
            },
            "parameters": [
                {
                    "name": "volume",
                    "description": {
                        "zh": "音量值（0-100）",
                        "en": "Volume value (0-100)"
                    },
                    "type": "number",
                    "required": true
                }
            ]
        },
        {
            "name": "mute_audio",
            "description": {
                "zh": "静音音频",
                "en": "Mute audio"
            },
            "parameters": []
        },
        {
            "name": "unmute_audio",
            "description": {
                "zh": "取消静音",
                "en": "Unmute audio"
            },
            "parameters": []
        },
        {
            "name": "get_current_volume",
            "description": {
                "zh": "获取当前音量",
                "en": "Get current volume"
            },
            "parameters": []
        },
        {
            "name": "is_muted",
            "description": {
                "zh": "检查是否静音",
                "en": "Check if audio is muted"
            },
            "parameters": []
        }
    ]
}
*/

const vehicleAudio = (function() {
    const VEHICLE_AUDIO_MANAGER_CLASS = 'com.xeagle.vehicleservice.sdk.manager.VehicleAudioManager';

    /**
     * 获取 VehicleAudioManager 实例
     */
    async function getVehicleAudioManager() {
        try {
            // 通过 Java 桥接创建 VehicleAudioManager 实例
            const VehicleAudioManager = Java.use(VEHICLE_AUDIO_MANAGER_CLASS);
            return VehicleAudioManager.getInstance();
        } catch (error) {
            throw new Error(`获取 VehicleAudioManager 实例失败: ${error.message}`);
        }
    }

    /**
     * 设置音量
     */
    async function set_volume(params) {
        try {
            const volume = Math.max(0, Math.min(100, params.volume));
            const audioManager = await getVehicleAudioManager();
            audioManager.setVolume(volume);
            
            return {
                success: true,
                message: `音量已设置为 ${volume}`
            };
        } catch (error) {
            return {
                success: false,
                message: `设置音量失败: ${error.message}`
            };
        }
    }

    /**
     * 静音音频
     */
    async function mute_audio() {
        try {
            const audioManager = await getVehicleAudioManager();
            audioManager.mute();
            
            return {
                success: true,
                message: "音频已静音"
            };
        } catch (error) {
            return {
                success: false,
                message: `静音失败: ${error.message}`
            };
        }
    }

    /**
     * 取消静音
     */
    async function unmute_audio() {
        try {
            const audioManager = await getVehicleAudioManager();
            audioManager.unmute();
            
            return {
                success: true,
                message: "音频已取消静音"
            };
        } catch (error) {
            return {
                success: false,
                message: `取消静音失败: ${error.message}`
            };
        }
    }

    /**
     * 获取当前音量
     */
    async function get_current_volume() {
        try {
            const audioManager = await getVehicleAudioManager();
            const volume = audioManager.getVolume();
            
            return {
                success: true,
                message: `当前音量: ${volume}`,
                data: {
                    volume: volume
                }
            };
        } catch (error) {
            return {
                success: false,
                message: `获取音量失败: ${error.message}`
            };
        }
    }

    /**
     * 检查是否静音
     */
    async function is_muted() {
        try {
            const audioManager = await getVehicleAudioManager();
            const muted = audioManager.isMuted();
            
            return {
                success: true,
                message: muted ? "音频已静音" : "音频未静音",
                data: {
                    muted: muted
                }
            };
        } catch (error) {
            return {
                success: false,
                message: `检查静音状态失败: ${error.message}`
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
        set_volume: wrap(set_volume),
        mute_audio: wrap(mute_audio),
        unmute_audio: wrap(unmute_audio),
        get_current_volume: wrap(get_current_volume),
        is_muted: wrap(is_muted)
    };
})();

exports.set_volume = vehicleAudio.set_volume;
exports.mute_audio = vehicleAudio.mute_audio;
exports.unmute_audio = vehicleAudio.unmute_audio;
exports.get_current_volume = vehicleAudio.get_current_volume;
exports.is_muted = vehicleAudio.is_muted;
```

## 注意事项

1. **依赖要求**: 需要确保设备上存在 `com.xeagle.vehicleservice.sdk.manager.VehicleAudioManager` 类
2. **权限要求**: 可能需要相应的权限来控制车辆音频系统
3. **错误处理**: 实现了完整的错误处理机制，确保在调用失败时能够返回友好的错误信息
4. **方法假设**: 假设 VehicleAudioManager 类具有以下方法：
   - `getInstance()`: 获取单例实例
   - `setVolume(int volume)`: 设置音量
   - `getVolume()`: 获取当前音量
   - `mute()`: 静音
   - `unmute()`: 取消静音
   - `isMuted()`: 检查是否静音

## 使用示例

### 设置音量
```javascript
const result = await vehicleAudio.set_volume({ volume: 50 });
console.log(result.message); // 音量已设置为 50
```

### 静音音频
```javascript
const result = await vehicleAudio.mute_audio();
console.log(result.message); // 音频已静音
```

### 获取当前音量
```javascript
const result = await vehicleAudio.get_current_volume();
console.log(`当前音量: ${result.data.volume}`);
```

### 检查是否静音
```javascript
const result = await vehicleAudio.is_muted();
console.log(`静音状态: ${result.data.muted}`);
```