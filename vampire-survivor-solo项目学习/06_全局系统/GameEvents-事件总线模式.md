# GameEvents - 事件总线模式

## 什么是事件总线

全局单例，作为中央事件调度器，让"发出事件的人"和"响应事件的人"完全解耦。

## 实现代码

```gdscript
# game_events.gd (Autoload)
extends Node

signal experience_vial_collected(number: float)
signal ability_upgrade_added(upgrade: AbilityUpgrade, current_upgrades: Dictionary)
signal player_damaged
signal damage_dealt(amount: float, ability_id: StringName, attacker: Node2D, target: Node2D, hitbox: HitBoxComponent, hurtbox: HurtBoxComponent)

func emit_experience_vial_collected(number: float):
    experience_vial_collected.emit(number)

func emit_ability_upgrade_added(upgrade: AbilityUpgrade, current_upgrades: Dictionary):
    ability_upgrade_added.emit(upgrade, current_upgrades)

func emit_player_damaged():
    player_damaged.emit()

func emit_damage_dealt(amount: float, ability_id: StringName, attacker: Node2D, target: Node2D, hitbox: HitBoxComponent, hurtbox: HurtBoxComponent) -> void:
    damage_dealt.emit(amount, ability_id, attacker, target, hitbox, hurtbox)
```

## 事件流示例

### 经验收集

```
ExperienceVial（玩家触碰）
        ↓ emit_experience_vial_collected(10)
        ↓
GameEvents
        ↓ emit
        ↓
ExperienceManager（监听）
        ↓ 增加经验
```

**代码实现**：

```gdscript
# ExperienceVial.gd
func _on_body_entered(body: Node2D):
    if body.is_in_group("player"):
        GameEvents.emit_experience_viol_collected(experience_value)
        queue_free()

# ExperienceManager.gd
func _ready() -> void:
    GameEvents.experience_vial_collected.connect(_on_experience_vial_collected)
```

### 伤害处理

```
HurtBox（检测到HitBox）
        ↓ emit_damage_dealt(...)
        ↓
GameEvents
        ↓ emit
        ↓
HurtBoxComponent（处理）
        ↓ health_component.damage()
        ↓ 生成飘字
```

## 事件总线优势

| 传统方式 | 事件总线 |
|----------|----------|
| A 直接调用 B 的方法 | A 发送事件 |
| A 需要知道 B 的存在 | A 不需要知道谁接收 |
| 新增接收者需要改 A | 新增接收者只连信号 |

## 常用事件

```gdscript
# 游戏状态
signal game_started
signal game_paused
signal game_resumed
signal game_ended

# 玩家状态
signal player_damaged
signal player_healed
signal player_level_up
signal player_died

# 战斗
signal enemy_spawned(enemy: Node2D)
signal enemy_killed(enemy: Node2D)
signal damage_dealt(amount: float, ...)

# 资源
signal experience_vial_collected(amount: float)
signal gold_collected(amount: float)

# 升级
signal ability_upgrade_added(upgrade: AbilityUpgrade, current_upgrades: Dictionary)
```

## 使用注意事项

### 1. 避免循环

```gdscript
# 不好：A 触发 B，B 又触发 A
func _on_event_a():
    GameEvents.emit_event_b()
    # 可能在 event_b 回调中又触发 event_a

# 好：明确事件流向
```

### 2. 断开连接

```gdscript
func _ready() -> void:
    GameEvents.player_damaged.connect(_on_player_damaged)

func _exit_tree() -> void:
    GameEvents.player_damaged.disconnect(_on_player_damaged)
```

### 3. 参数完整

```gdscript
# 传递足够信息，让接收者不需要额外查找
signal damage_dealt(amount, ability_id, attacker, target, hitbox, hurtbox)
```

## 最佳实践

1. **单一职责** - GameEvents 只负责发送，不管谁接收
2. **信号封装** - 用 emit_xxx 方法封装，方便日后扩展（如加日志）
3. **参数完整** - 事件携带足够上下文
4. **命名规范** - emit_ 开头的方法发送，信号本身是名词
