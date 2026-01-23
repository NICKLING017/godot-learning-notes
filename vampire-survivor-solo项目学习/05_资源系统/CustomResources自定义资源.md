# Custom Resources - 自定义资源

## 什么是 Resource

Godot 中的资源文件（.tres）是存储数据的方式，类似 JSON 但有类型检查。

## 定义自定义 Resource

```gdscript
# AbilityUpgrade.gd
class_name AbilityUpgrade
extends Resource

@export var id: StringName
@export var display_name: String
@export var description: String
@export var icon: Texture2D
@export var max_quantity: int = 1
@export var rarity: int = 1
@export var affected_ability: StringName = &""
```

## 创建资源文件

```
1. 在 FileSystem 中右键
2. New Resource
3. 选择 AbilityUpgrade
4. 填写属性
5. 保存为 sword_damage.tres
```

## 资源文件示例

```
# sword_damage.tres (tres 文本格式)
[gd_resource type="Resource" load_steps=2 format=3 uid="uid://..."]

[ext_resource type="Script" path="res://scenes/ability/sword_ability/sword_upgrade.gd" id="1_..."]

[resource]
id = &"sword_damage"
display_name = "剑刃强化"
description = "攻击力+5"
icon = null
max_quantity = 5
rarity = 1
affected_ability = &"sword"
```

## 使用自定义资源

```gdscript
# 加载资源
var upgrade = preload("res://Resources/upgrades/sword_damage.tres")

# 读取属性
print(upgrade.id)           # &"sword_damage"
print(upgrade.display_name) # "剑刃强化"
print(upgrade.max_quantity) # 5

# 修改属性（内存中）
upgrade.max_quantity = 10
```

## Resource 数组

```gdscript
# 定义升级池
@export var upgrade_pool: Array[AbilityUpgrade] = []

func _ready() -> void:
    for upgrade in upgrade_pool:
        print(upgrade.display_name)
```

## 资源管理优势

| 对比项 | 硬编码 | Resource |
|--------|--------|----------|
| 修改数据 | 改代码 | 改资源文件 |
| 类型安全 | 无 | 有 |
| 美术/策划编辑 | 程序员 | 可独立编辑 |
| 版本控制 | 代码diff | 二进制/文本 |

## 常用 Resource 类型

| 类型 | 用途 |
|------|------|
| Resource | 基类 |
| Texture2D | 图片资源 |
| AudioStream | 音频资源 |
| PackedScene | 场景文件 |
| Theme | UI主题 |

## 最佳实践

1. **class_name** - 给自定义资源定义类名
2. **@export** - 暴露属性到编辑器
3. **类型注解** - 数组声明时指定类型
4. **UID** - 使用 uid://... 而非路径（跨移动兼容）
