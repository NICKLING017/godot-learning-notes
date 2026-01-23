# HealthComponent - 生命值管理

## 组件功能

管理游戏实体的生命值，处理伤害计算和治疗逻辑。

## 核心代码

```gdscript
extends Node
class_name HealthComponent

signal died                    # 死亡信号
signal health_changed          # 生命值变化信号
signal health_decreased        # 生命值降低信号

@export var max_health: float = 10.0
var current_health: float

func _ready() -> void:
    current_health = max_health

func damage(damage_amount: float) -> void:
    # clamp 防止血量越界
    current_health = clamp(current_health - damage_amount, 0, max_health)
    health_changed.emit()

    # 只在真正受到伤害时发射
    if damage_amount > 0:
        health_decreased.emit()

    # 延迟检查死亡，避免在信号回调链中直接 queue_free
    Callable(check_death).call_deferred()

func heal(heal_amount: float) -> void:
    # 治疗本质是负数伤害
    damage(-heal_amount)

func get_health_percent() -> float:
    if max_health <= 0:
        return 0
    return min(current_health / max_health, 1)

func check_death() -> void:
    if current_health == 0:
        died.emit()
        owner.queue_free()
```

## 关键设计点

### 1. 负数治疗

```gdscript
# 治疗 = 负数伤害
func heal(heal_amount: float) -> void:
    damage(-heal_amount)
```

### 2. 延迟死亡检查

```gdscript
# 为什么要用 call_deferred？
# 避免在信号回调链中直接销毁节点导致访问失效
Callable(check_death).call_deferred()
```

### 3. 多个信号分离

```gdscript
# health_changed - 任何变化（伤害或治疗）
# health_decreased - 仅伤害（用于受击反馈）
signal health_changed
signal health_decreased
```

## 使用方式

```
1. 将 HealthComponent.tscn 拖入实体场景
2. 挂载到实体节点
3. HurtBoxComponent 引用此组件进行扣血
```

## 信号连接示例

```gdscript
# 在实体脚本中
func _ready() -> void:
    $HealthComponent.died.connect(_on_died)
    $HealthComponent.health_changed.connect(_update_health_ui)

func _on_died() -> void:
    # 死亡特效、经验掉落等
    pass
```

## 最佳实践

1. **暴露接口** - 通过方法访问而非直接修改变量
2. **单一职责** - 只管理生命值，不处理死亡特效
3. **信号解耦** - 死亡后由外部监听器处理后续逻辑
