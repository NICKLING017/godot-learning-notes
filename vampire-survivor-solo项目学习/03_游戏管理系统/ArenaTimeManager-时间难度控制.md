# ArenaTimeManager - 时间与难度控制

## 核心功能

管理游戏时间流逝、难度等级提升，驱动整个游戏的节奏变化。

## 核心代码

```gdscript
extends Node

const DIFFICULTY_INTERVAL: float = 5.0  # 难度提升间隔（秒）
const GAME_DURATION: float = 120.0       # 游戏总时长（秒）

signal arena_difficulty_increased(difficulty: int)
signal time_changed(time: float)
signal game_ended

@onready var timer: Timer = $Timer
var arena_difficulty: int = 0
var elapsed_time: float = 0.0

func _ready() -> void:
    timer.wait_time = DIFFICULTY_INTERVAL
    timer.start()

func _on_timer_timeout() -> void:
    elapsed_time += DIFFICULTY_INTERVAL
    arena_difficulty += 1

    # 发送难度提升信号
    arena_difficulty_increased.emit(arena_difficulty)

    # 检查游戏是否结束
    if elapsed_time >= GAME_DURATION:
        game_ended.emit()
        timer.stop()
```

## 难度系统设计

### 难度等级计算

```
难度等级 = floor(已用时间 / 难度提升间隔)
        = floor(elapsed_time / 5.0)

例如：
- 0-5秒：难度 0
- 5-10秒：难度 1
- 10-15秒：难度 2
```

### 难度信号传播

```
ArenaTimeManager
        ↓ emit
EnemyManager - 加快刷怪、解锁新敌人
SpawnManager - 改变生成策略
Environment - 改变场景效果
```

## 与其他系统联动

### EnemyManager 监听难度

```gdscript
func _ready() -> void:
    arena_time_manager.arena_difficulty_increased.connect(on_arena_difficulty_increased)

func on_arena_difficulty_increased(difficulty: int) -> void:
    # 加快刷怪速度
    var time_off = (0.1 / 12) * difficulty
    time_off = min(time_off, 0.7)
    timer.wait_time = base_spawn_time - time_off

    # 解锁新敌人
    if difficulty == 6:
        enemy_table.add_item(wizard_enemy_scene, 15)
    elif difficulty == 15:
        enemy_table.add_item(bat_enemy_scene, 5)
```

## 游戏节奏设计模板

| 时间段 | 难度 | 敌人类型 | 刷怪间隔 |
|--------|------|----------|----------|
| 0-30秒 | 0-5 | 基础怪 | 1.0秒 |
| 30-60秒 | 6-11 | 基础怪+巫师 | 0.8秒 |
| 60-90秒 | 12-17 | 基础怪+巫师+蝙蝠 | 0.6秒 |
| 90-120秒 | 18+ | 全部敌人 | 0.4秒 |

## 扩展功能

### 难度曲线配置

```gdscript
# 可配置难度的数据结构
var difficulty_config = {
    0: { "enemy_types": ["basic"], "spawn_interval": 1.0 },
    6: { "enemy_types": ["basic", "wizard"], "spawn_interval": 0.8 },
    15: { "enemy_types": ["basic", "wizard", "bat"], "spawn_interval": 0.6 }
}
```

### 时间暂停/加速

```gdscript
func set_time_scale(scale: float) -> void:
    timer.wait_time = DIFFICULTY_INTERVAL / scale
```

## 最佳实践

1. **信号驱动** - 难度变化通过信号通知各系统
2. **配置分离** - 难度表与逻辑分离，便于调整
3. **帧率无关** - 使用 Timer 而非 _process(delta) 计时
4. **单一数据源** - 所有难度相关计算以 ArenaTimeManager 为准
