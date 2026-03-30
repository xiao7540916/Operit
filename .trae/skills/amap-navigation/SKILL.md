---
name: "amap-navigation"
description: "通过Intent启动高德地图APP进行路线规划导航。Invoke when user needs to launch Amap (Gaode Map) for driving directions, route planning, or navigation with start and destination addresses."
---

# 高德地图导航 Skill

本 Skill 提供了在安卓端通过 Intent 启动高德地图 APP，并传入起始地址和目的地生成驾驶路线规划的功能。

## 高德地图包名

- **主包名**: `com.autonavi.minimap`

## 启动高德地图的 Intent 方式

### 1. 基础路线规划（URI Scheme 方式）

高德地图支持通过 `android.intent.action.VIEW` 配合 URI 来启动路线规划：

```javascript
const intent = new Intent(IntentAction.ACTION_VIEW);
intent.setData(`amapuri://route/plan/?sid=&did=&dlat=${destLat}&dlon=${destLng}&dname=${encodedDestName}&dev=0&t=0`);
intent.addFlag(IntentFlag.ACTIVITY_NEW_TASK);
await intent.start();
```

### 2. 详细参数说明

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `sid` | 起点ID（可选） | 空字符串表示使用当前位置 |
| `did` | 终点ID（可选） | 空字符串 |
| `dlat` | 终点纬度 | 39.9042 |
| `dlon` | 终点经度 | 116.4074 |
| `dname` | 终点名称（需URL编码） | encodeURIComponent("天安门") |
| `dev` | 是否偏移（0:使用经纬度，1:偏移） | 0 |
| `t` | 交通方式（0:驾车，1:公交，2:步行，3:骑行） | 0 |

### 3. 完整示例代码

```javascript
/**
 * 启动高德地图进行驾车路线规划
 * @param {Object} params - 参数对象
 * @param {string} params.destination - 目的地名称
 * @param {number} params.destLat - 目的地纬度
 * @param {number} params.destLng - 目的地经度
 * @param {string} [params.startName] - 起点名称（可选，默认使用当前位置）
 * @param {number} [params.startLat] - 起点纬度（可选）
 * @param {number} [params.startLng] - 起点经度（可选）
 * @returns {Promise<Object>} - 启动结果
 */
async function startAmapNavigation(params) {
    try {
        // 检查高德地图是否已安装
        const android = new Android();
        const isInstalled = await android.packageManager.isInstalled('com.autonavi.minimap');
        
        if (!isInstalled) {
            return {
                success: false,
                message: "高德地图未安装，请先安装高德地图APP"
            };
        }

        // 构建URI
        const destName = encodeURIComponent(params.destination || "");
        const startName = params.startName ? encodeURIComponent(params.startName) : "";
        
        // 如果有起点坐标
        let uri;
        if (params.startLat && params.startLng) {
            // 指定起点和终点
            uri = `amapuri://route/plan/?sid=&did=&slat=${params.startLat}&slon=${params.startLng}&sname=${startName}&dlat=${params.destLat}&dlon=${params.destLng}&dname=${destName}&dev=0&t=0`;
        } else {
            // 只指定终点，起点使用当前位置
            uri = `amapuri://route/plan/?sid=&did=&dlat=${params.destLat}&dlon=${params.destLng}&dname=${destName}&dev=0&t=0`;
        }

        // 创建并启动Intent
        const intent = new Intent(IntentAction.ACTION_VIEW);
        intent.setData(uri);
        intent.addFlag(IntentFlag.ACTIVITY_NEW_TASK);
        
        await intent.start();
        
        return {
            success: true,
            message: `已启动高德地图，正在规划到 ${params.destination} 的路线`
        };
    } catch (error) {
        return {
            success: false,
            message: `启动高德地图失败: ${error.message}`
        };
    }
}
```

### 4. 通过地址名称搜索启动（POI搜索方式）

如果只有地址名称没有坐标，可以先使用地理编码获取坐标：

```javascript
/**
 * 使用地址名称启动高德地图导航
 * @param {Object} params - 参数对象
 * @param {string} params.destination - 目的地名称
 * @param {string} [params.city] - 城市名称（用于提高搜索准确性）
 * @param {string} [params.startAddress] - 起点地址（可选）
 */
async function startAmapNavigationByAddress(params) {
    try {
        // 检查高德地图是否已安装
        const android = new Android();
        const isInstalled = await android.packageManager.isInstalled('com.autonavi.minimap');
        
        if (!isInstalled) {
            return {
                success: false,
                message: "高德地图未安装，请先安装高德地图APP"
            };
        }

        // 使用POI搜索URI
        const destName = encodeURIComponent(params.destination || "");
        const city = encodeURIComponent(params.city || "");
        
        // 通过关键词搜索并导航
        const uri = `amapuri://route/plan/?sid=&did=&dname=${destName}&dev=0&t=0`;
        
        const intent = new Intent(IntentAction.ACTION_VIEW);
        intent.setData(uri);
        intent.addFlag(IntentFlag.ACTIVITY_NEW_TASK);
        
        await intent.start();
        
        return {
            success: true,
            message: `已启动高德地图，正在搜索 ${params.destination} 并规划路线`
        };
    } catch (error) {
        return {
            success: false,
            message: `启动高德地图失败: ${error.message}`
        };
    }
}
```

### 5. 其他常用高德地图URI Scheme

| 功能 | URI Scheme | 说明 |
|------|------------|------|
| 显示地图 | `amapuri://map/show` | 打开地图显示指定位置 |
| POI搜索 | `amapuri://search/keyword/?keyword=${keyword}` | 搜索POI |
| 导航 | `amapuri://navi/navi/?lat=${lat}&lon=${lng}&name=${name}` | 直接开始导航 |
| 我的位置 | `amapuri://map/my_location` | 显示我的位置 |

### 6. 完整工具包示例

```javascript
/*
METADATA
{
    "name": "amap_navigation",
    "display_name": {
        "zh": "高德地图导航",
        "en": "Amap Navigation"
    },
    "description": {
        "zh": "启动高德地图APP进行路线规划和导航",
        "en": "Launch Amap app for route planning and navigation"
    },
    "enabledByDefault": true,
    "category": "Navigation",
    "tools": [
        {
            "name": "navigate_to",
            "description": {
                "zh": "启动高德地图导航到指定目的地",
                "en": "Launch Amap to navigate to destination"
            },
            "parameters": [
                {
                    "name": "destination",
                    "description": {
                        "zh": "目的地名称",
                        "en": "Destination name"
                    },
                    "type": "string",
                    "required": true
                },
                {
                    "name": "city",
                    "description": {
                        "zh": "城市名称，用于提高搜索准确性",
                        "en": "City name for better search accuracy"
                    },
                    "type": "string",
                    "required": false
                }
            ]
        },
        {
            "name": "navigate_with_coords",
            "description": {
                "zh": "使用坐标启动高德地图导航",
                "en": "Launch Amap navigation with coordinates"
            },
            "parameters": [
                {
                    "name": "dest_lat",
                    "description": {
                        "zh": "目的地纬度",
                        "en": "Destination latitude"
                    },
                    "type": "number",
                    "required": true
                },
                {
                    "name": "dest_lng",
                    "description": {
                        "zh": "目的地经度",
                        "en": "Destination longitude"
                    },
                    "type": "number",
                    "required": true
                },
                {
                    "name": "dest_name",
                    "description": {
                        "zh": "目的地名称",
                        "en": "Destination name"
                    },
                    "type": "string",
                    "required": false
                },
                {
                    "name": "start_lat",
                    "description": {
                        "zh": "起点纬度（可选，默认当前位置）",
                        "en": "Start latitude (optional, default current location)"
                    },
                    "type": "number",
                    "required": false
                },
                {
                    "name": "start_lng",
                    "description": {
                        "zh": "起点经度（可选，默认当前位置）",
                        "en": "Start longitude (optional, default current location)"
                    },
                    "type": "number",
                    "required": false
                }
            ]
        }
    ]
}
*/

const amapNavigation = (function() {
    const AMAP_PACKAGE = 'com.autonavi.minimap';

    /**
     * 检查高德地图是否安装
     */
    async function checkAmapInstalled() {
        const android = new Android();
        return await android.packageManager.isInstalled(AMAP_PACKAGE);
    }

    /**
     * 使用地址名称导航
     */
    async function navigate_to(params) {
        try {
            const isInstalled = await checkAmapInstalled();
            if (!isInstalled) {
                return {
                    success: false,
                    message: "高德地图未安装，请先安装高德地图APP"
                };
            }

            const destName = encodeURIComponent(params.destination || "");
            const uri = `amapuri://route/plan/?sid=&did=&dname=${destName}&dev=0&t=0`;

            const intent = new Intent(IntentAction.ACTION_VIEW);
            intent.setData(uri);
            intent.addFlag(IntentFlag.ACTIVITY_NEW_TASK);

            await intent.start();

            return {
                success: true,
                message: `已启动高德地图，正在规划到 ${params.destination} 的路线`
            };
        } catch (error) {
            return {
                success: false,
                message: `导航启动失败: ${error.message}`
            };
        }
    }

    /**
     * 使用坐标导航
     */
    async function navigate_with_coords(params) {
        try {
            const isInstalled = await checkAmapInstalled();
            if (!isInstalled) {
                return {
                    success: false,
                    message: "高德地图未安装，请先安装高德地图APP"
                };
            }

            const destName = encodeURIComponent(params.dest_name || "");
            let uri;

            if (params.start_lat && params.start_lng) {
                uri = `amapuri://route/plan/?sid=&did=&slat=${params.start_lat}&slon=${params.start_lng}&dlat=${params.dest_lat}&dlon=${params.dest_lng}&dname=${destName}&dev=0&t=0`;
            } else {
                uri = `amapuri://route/plan/?sid=&did=&dlat=${params.dest_lat}&dlon=${params.dest_lng}&dname=${destName}&dev=0&t=0`;
            }

            const intent = new Intent(IntentAction.ACTION_VIEW);
            intent.setData(uri);
            intent.addFlag(IntentFlag.ACTIVITY_NEW_TASK);

            await intent.start();

            return {
                success: true,
                message: `已启动高德地图进行路线规划`
            };
        } catch (error) {
            return {
                success: false,
                message: `导航启动失败: ${error.message}`
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
        navigate_to: wrap(navigate_to),
        navigate_with_coords: wrap(navigate_with_coords)
    };
})();

exports.navigate_to = amapNavigation.navigate_to;
exports.navigate_with_coords = amapNavigation.navigate_with_coords;
```

## 注意事项

1. **权限要求**: 启动高德地图不需要特殊权限，但需要设备已安装高德地图APP
2. **坐标系**: 高德地图使用GCJ-02坐标系（火星坐标系），传入WGS-84坐标可能会有偏差
3. **URL编码**: 所有中文参数都需要进行URL编码
4. **Intent标志**: 建议添加 `ACTIVITY_NEW_TASK` 标志以确保在新任务栈中启动
5. **错误处理**: 始终检查高德地图是否已安装，避免崩溃

## 参考链接

- [高德地图URI Scheme文档](https://lbs.amap.com/api/amap-mobile/guide/android/route)
