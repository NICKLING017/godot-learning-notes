---
title: Procedural Map Generator 程序地图生成插件
date: 2026-02-04
tags:
  - godot
  - procedural-generation
  - map-generator
  - tilemap
status: active
priority: high
---

# Procedural Map Generator - 程序地图生成插件

> 通用程序地图生成工具，支持多种算法生成随机地图

## 简介

**Procedural Map Generator** 是一个用于 Godot 的程序化地图生成插件，可以自动修改 TileMap 使用各种算法生成随机地图。

### 核心特性

| 特性 | 说明 |
|------|------|
| **算法支持** | Cellular Automata, Wave Function Collapse, Gram-Elites |
| **目标对象** | 直接操作 TileMap |
| **配置灵活** | 可指定多种参数 |
| **示例演示** | 包含 Examples 文件夹 |

### 基本信息

- **版本**: 1.0
- **Godot版本**: 4.2+
- **许可证**: MIT
- **提交者**: mihalysarolta-ubb

## 支持的算法

### 1. Cellular Automata（元胞自动机）
适合生成洞穴、地牢等自然形状的地图。

**原理：**
- 随机填充初始格子
- 多次迭代，根据邻居数量决定生死
- 结果：自然、连续的地图

**参数：**
- `initial_fill_percent`: 初始填充率（通常 45-50%）
- `iterations`: 迭代次数（通常 5-10 次）
- `neighbor_threshold`: 邻居阈值

### 2. Wave Function Collapse（波函数坍缩）
适合生成有规则结构的地图（类似 Minecraft、Spelunky）。

**原理：**
- 每个格子有多种可能状态
- 随机选择一个，观察它的约束
- 逐步坍缩，直到所有格子确定

**参数：**
- `tile_constraints`: 瓦片约束
- `pattern_size`: 模式大小
- `symmetry`: 对称性

### 3. Gram-Elites
适合生成有特定风格的地图。

**原理：**
- 生成多个候选地图
- 选择最符合"精英"标准的
- 变异产生下一代

## 安装

### 方法1: Godot Asset Library
1. Godot 编辑器 → **AssetLib**
2. 搜索 "Procedural-Map-Generator"
3. 下载并安装

### 方法2: 手动安装
1. 下载：https://godotengine.org/asset-library/asset/2979
2. 解压到项目 `addons/procedural-map-generator/`
3. 启用插件

## 使用方法

### 1. 基本设置

```gdscript
extends Node2D

@onready var map_generator = $ProceduralMapGenerator
@onready var tilemap = $TileMap

func _ready():
    # 配置生成器
    map_generator.tilemap = tilemap
    map_generator.map_width = 50
    map_generator.map_height = 30
```

### 2. 使用 Cellular Automata

```gdscript
func generate_cave_map():
    map_generator.algorithm = map_generator.ALGORITHM_CELLULAR_AUTOMATA
    map_generator.initial_fill_percent = 0.47
    map_generator.iterations = 5
    map_generator.neighbor_threshold = 4
    map_generator.wall_tile_id = 1  # 墙壁瓦片ID
    map_generator.floor_tile_id = 0  # 地面瓦片ID
    map_generator.generate()
```

### 3. 使用 Wave Function Collapse

```gdscript
func generate_dungeon():
    map_generator.algorithm = map_generator.ALGORITHM_WAVE_FUNCTION_COLLAPSE
    map_generator.load_constraints_from_file("res://constraints.json")
    map_generator.pattern_size = 2
    map_generator.symmetry = 2
    map_generator.generate()
```

### 4. 生成后处理

```gdscript
func on_generate_complete():
    # 添加入口和出口
    var start_pos = find_free_position()
    var end_pos = find_farthest_position(start_pos)
    
    tilemap.set_cell(0, start_pos, 0, Vector2i(2, 0))  # 入口
    tilemap.set_cell(0, end_pos, 0, Vector2i(3, 0))    # 出口

func find_free_position() -> Vector2i:
    # 找到随机空闲位置
    var attempts = 0
    while attempts < 1000:
        var x = randi() % map_generator.map_width
        var y = randi() % map_generator.map_height
        if not is_wall(Vector2i(x, y)):
            return Vector2i(x, y)
        attempts += 1
    return Vector2i(0, 0)
```

## 算法对比

| 算法 | 适用场景 | 难度 | 控制粒度 |
|------|----------|------|----------|
| Cellular Automata | 洞穴、地牢、自然地形 | ⭐ | 低 |
| Wave Function Collapse | 关卡、房间布局 | ⭐⭐⭐ | 高 |
| Gram-Elites | 风格化地图 | ⭐⭐ | 中 |

## 实际应用案例

### 1. 随机地牢
```gdscript
# Cellular Automata + 后处理
func generate_dungeon():
    # 1. 生成基础洞穴
    cellular_automata_generate()
    
    # 2. 平滑边缘
    smooth_edges()
    
    # 3. 添加房间
    add_random_rooms()
    
    # 4. 连接房间
    connect_rooms_with_corridors()
    
    # 5. 放置道具
    place_items_and_enemies()
```

### 2. 随机关卡
```gdscript
# Wave Function Collapse
func generate_level():
    map_generator.algorithm = WFC
    map_generator.tile_constraints = load_constraints("level.json")
    
    # 约束配置
    # {
    #   "floor_north": ["floor", "empty"],
    #   "floor_south": ["floor", "wall"],
    #   ...
    # }
    
    map_generator.generate()
```

## 性能优化

### 1. 分块生成
```gdscript
func generate_chunked(chunk_size: int = 16):
    for y in range(0, map_height, chunk_size):
        for x in range(0, map_width, chunk_size):
            generate_chunk(x, y, chunk_size)
```

### 2. 使用对象池
```gdscript
var tile_pool: Array[Vector2i] = []

func _ready():
    # 预生成瓦片池
    for i in range(1000):
        tile_pool.append(Vector2i(randi() % 10, randi() % 10))
```

### 3. 增量生成
```gdscript
func generate_incremental():
    var todo: Array[Callable] = []
    
    # 每次生成一小部分
    for i in range(10):
        todo.append(generate_chunk.bind(i))
    
    call_functions_in_sequence(todo)
```

## 扩展插件

### 相关插件

| 插件 | 功能 |
|------|------|
| **Procedural World Map Generator** | 生成世界地图（地形、生物群系、河流） |
| **GDQuest Procedural Generation** | 包含多种算法的示例项目 |
| **Wave Function Collapse** | 专用的 WFC 插件 |

### 学习资源

- [Procedural Map Generator - Godot Asset Library](https://godotengine.org/asset-library/asset/2979)
- [Wave Function Collapse 论文](https://github.com/mxgmn/WaveFunctionCollapse)
- [Godot Procedural Generation - GDQuest](https://github.com/gdquest-demos/godot-procedural-generation)

## 优缺点

### 优点
- ✅ 多种算法可选
- ✅ MIT 开源许可证
- ✅ 易于集成
- ✅ 包含示例项目

### 缺点
- ❌ 文档较少
- ❌ WFC 算法需要手动配置约束
- ❌ 不支持 3D TileMap

## 总结

**适合：**
- ✅ 生成随机地牢、洞穴
- ✅ 生成随机关卡布局
- ✅ 2D 游戏地图生成

**不适合：**
- ❌ 需要复杂约束的关卡
- ❌ 3D 地图生成
- ❌ 实时生成大地图

> [!tip] 推荐组合
> 建议搭配 LimboAI（AI 行为树）使用，用 LimboAI 控制 AI 在程序生成的地图中行动！

---

> [!note] 更新日志
> - **2026-02-04**: 新增 Procedural Map Generator 插件笔记，整理算法和使用方法
