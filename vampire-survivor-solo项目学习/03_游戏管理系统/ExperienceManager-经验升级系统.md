# ExperienceManager - 经验与升级系统

## 核心功能

管理玩家经验获取、等级计算，触发升级流程。

## 核心代码

```gdscript
extends Node

signal experience_vial_collected(amount: float)
signal experience_gained(amount: float)
signal level_up(new_level: int)

@export var experience_per_level: Array[float] = [
    0,      # 1级
    10,     # 2级需要10经验
    20,     # 3级需要20
    40,     # 4级需要40
    80,     # 5级需要80
    160,    # 6级需要160
    320,    # 7级需要320
    640,    # 8级需要640
    1280,   # 9级需要1280
    2560    # 10级需要2560
]

var current_experience: float = 0
var current_level: int = 1
var experience_needed: float = 0

func _ready() -> void:
    experience_needed = experience_per_level[1]
    GameEvents.experience_vial_collected.connect(_on_experience_vial_collected)

func _on_experience_vial_collected(amount: float) -> void:
    experience_gained.emit(amount)
    add_experience(amount)

func add_experience(amount: float) -> void:
    current_experience += amount

    while current_experience >= experience_needed:
        current_experience -= experience_needed
        level_up.emit(current_level)

        current_level += 1
        if current_level < experience_per_level.size():
            experience_needed = experience_per_level[current_level]
        else:
            experience_needed *= 2  # 满级后指数增长

func get_experience_progress() -> float:
    if experience_needed == 0:
        return 0
    return current_experience / experience_needed
```

## 经验获取流程

```
敌人死亡
    ↓
DeathComponent 生成 ExperienceVial
    ↓
玩家触碰 ExperienceVial
    ↓
ExperienceVial 发送 experience_vial_collected 信号
    ↓
ExperienceManager 接收并累加经验
    ↓
检查是否升级
    ↓
发射 level_up 信号（暂停游戏，显示升级界面）
```

## 经验球实现

```gdscript
# ExperienceVial.gd
extends Area2D

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D) -> void:
    if body.is_in_group("player"):
        GameEvents.emit_experience_vial_collected(experience_value)
        queue_free()
```

## 等级曲线设计

```gdscript
# 线性增长
experience_per_level = [0, 10, 20, 30, 40, 50...]

# 指数增长（推荐）
experience_per_level = [0, 10, 20, 40, 80, 160, 320...]

# 混合增长
experience_per_level = [0, 10, 25, 50, 100, 150, 250...]
```

## UI 进度条

```gdscript
# experience_bar.gd
func _process(_delta: float) -> void:
    var experience_manager = get_tree().get_first_node_in_group("experience_manager")
    if experience_manager:
        value = experience_manager.get_experience_progress() * 100
```

## 最佳实践

1. **信号解耦** - 经验球只负责发送信号，不关心谁接收
2. **while 循环** - 支持一次获得大量经验连升多级
3. **可配置曲线** - 等级经验数组便于策划调整
4. **经验溢出** - 升级后多余经验保留
