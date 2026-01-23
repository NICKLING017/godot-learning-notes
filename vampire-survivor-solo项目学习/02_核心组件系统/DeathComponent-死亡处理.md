# DeathComponent - 死亡处理

## 组件功能

处理游戏实体的死亡逻辑，包括经验掉落、死亡特效等。

## 核心代码

```gdscript
extends Node
class_name DeathComponent

@export var experience_vial_scene: PackedScene
@export var vial_drop_component: Node

func _ready() -> void:
    # 连接到 HealthComponent 的 died 信号
    var health_component = owner.get_node_or_null("HealthComponent")
    if health_component:
        health_component.died.connect(_on_died)

func _on_died() -> void:
    # 死亡时掉落经验球
    if experience_vial_scene:
        spawn_experience_vial()

    # 延迟销毁，避免在信号链中出问题
    Callable(_delayed_free).call_deferred()

func spawn_experience_vial() -> void:
    var vial = experience_vial_scene.instantiate()
    var entities_layer = get_tree().get_first_node_in_group("entities_layer")
    if entities_layer:
        entities_layer.add_child(vial)
        vial.global_position = owner.global_position

func _delayed_free() -> void:
    owner.queue_free()
```

## VialDropComponent 代码

```gdscript
extends Node
class_name VialDropComponent

@export var vial_scene: PackedScene
var _dropped: bool = false

func drop() -> void:
    if _dropped:
        return
    _dropped = true

    var vial = vial_scene.instantiate()
    var entities_layer = get_tree().get_first_node_in_group("entities_layer")
    entities_layer.add_child(vial)
    vial.global_position = owner.global_position
```

## 死亡流程

```
HealthComponent.current_health == 0
        ↓
    HealthComponent.check_death()
        ↓
    HealthComponent.died.emit()
        ↓
    DeathComponent._on_died()
        ↓
    生成经验球（敌人死亡时）
        ↓
    queue_free() 销毁实体
```

## 使用方式

```
1. DeathComponent 挂载到敌人场景
2. 引用经验球 PackedScene
3. HealthComponent 的 died 信号自动连接
4. 敌人死亡时自动掉落经验球
```

## 扩展死亡逻辑

```gdscript
func _on_died() -> void:
    # 1. 掉落物品
    spawn_experience_vial()
    spawn_loot()

    # 2. 播放死亡特效
    play_death_effect()

    # 3. 播放音效
    play_death_sound()

    # 4. 通知管理器（击杀计数）
    GameEvents.emit_enemy_killed(owner)

    # 5. 延迟销毁
    Callable(_delayed_free).call_deferred()
```

## 最佳实践

1. **延迟销毁** - 使用 call_deferred 避免信号链问题
2. **单一职责** - 只处理死亡逻辑，不处理掉落概率
3. **引用验证** - 使用 get_node_or_null 避免空引用崩溃
4. **层级管理** - 经验球添加到 entities_layer 而非直接添加到世界
