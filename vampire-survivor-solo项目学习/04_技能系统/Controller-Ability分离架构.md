# Controller-Ability 分离架构

## 架构设计

### 核心思想

将技能的**逻辑控制**与**视觉表现**分离：
- **Controller** - 挂载在玩家身上，负责CD、索敌、实例化
- **Ability** - 实际的视觉实体（动画、碰撞），生成后播放，结束销毁

## 架构图

```
Player
├── SwordAbilityController（控制器）
│   ├── 管理冷却
│   ├── 寻找目标
│   └── 实例化 SwordAbility
│
└── AxeAbilityController（控制器）
    ├── 管理冷却
    ├── 寻找目标
    └── 实例化 AxeAbility
```

```
SwordAbility（视觉实体）
├── HitBoxComponent（碰撞检测）
├── Sprite2D（剑的动画）
└── AnimationPlayer（动画播放）
```

## Controller 代码示例

```gdscript
# SwordAbilityController.gd
extends Node

@export var sword_ability: PackedScene
@export var damage: float = 10.0
@export var max_range: float = 100.0
@export var cooldown: float = 1.0

var _time_since_last_use: float = 0.0

func _ready() -> void:
    GameEvents.ability_upgrade_added.connect(_on_ability_upgrade_added)

func _process(delta: float) -> void:
    _time_since_last_use += delta

    if _time_since_last_use >= cooldown:
        var enemies = get_tree().get_nodes_in_group("enemy")
        var target = _find_closest_enemy(enemies)

        if target:
            _time_since_last_use = 0
            _use_ability(target)

func _find_closest_enemy(enemies: Array[Node]) -> Node2D:
    var valid_enemies = enemies.filter(func(enemy: Node2D):
        return enemy.global_position.distance_squared_to(owner.global_position) < pow(max_range, 2)
    )

    valid_enemies.sort_custom(func(a: Node2D, b: Node2D):
        var a_dist = a.global_position.distance_squared_to(owner.global_position)
        var b_dist = b.global_position.distance_squared_to(owner.global_position)
        return a_dist < b_dist
    )

    return valid_enemies[0] if not valid_enemies.is_empty() else null

func _use_ability(target: Node2D) -> void:
    var ability = sword_ability.instantiate() as Node2D
    get_tree().root.add_child(ability)

    ability.global_position = owner.global_position
    ability.look_at(target.global_position)

    # 配置伤害
    var hitbox = ability.get_node_or_null("HitBoxComponent")
    if hitbox:
        hitbox.damage = damage
        hitbox.source_ability_id = &"sword"

    # 动画结束后销毁
    ability.tree_exiting.connect(ability.queue_free)
```

## Ability 代码示例

```gdscript
# SwordAbility.gd
extends Node2D

@onready var animation_player: AnimationPlayer = $AnimationPlayer

func _ready() -> void:
    animation_player.play("swing")
    animation_player.animation_finished.connect(_on_animation_finished)

func _on_animation_finished(_anim_name: String) -> void:
    queue_free()
```

## 数据流

```
升级选择
        ↓
UpgradeManager.emit_ability_upgrade_added
        ↓
SwordAbilityController 监听
        ↓
修改 damage / cooldown 属性
        ↓
下次使用技能时应用新数值
```

## 扩展新技能模板

### 1. 创建 Ability 场景
```
scenes/ability/new_ability/
├── new_ability.tscn      # 视觉实体
├── new_ability.gd        # 动画控制
└── sprite.png            # 贴图
```

### 2. 创建 Controller
```gdscript
# NewAbilityController.gd
extends Node

@export var new_ability: PackedScene
# ... 配置属性 ...

func _use_ability(target: Node2D) -> void:
    var ability = new_ability.instantiate()
    get_tree().root.add_child(ability)
    # ... 设置位置、旋转 ...
```

### 3. 注册升级
```gdscript
# 在 Resource 中添加新升级定义
var new_upgrade = AbilityUpgrade.new()
new_upgrade.id = &"new_ability_damage"
new_upgrade.display_name = "新技能强化"
```

## 最佳实践

1. **职责分离** - Controller管逻辑，Ability管表现
2. **配置化** - 通过 @export 属性调整行为
3. **信号解耦** - 升级通过信号传递
4. **自动销毁** - Ability 动画结束后自动 queue_free
