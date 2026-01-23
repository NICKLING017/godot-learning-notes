# EnemyManager - 敌人生成系统

## 核心功能

按计划生成敌人，支持动态难度调整和智能生成位置。

## 核心代码

```gdscript
extends Node

const SPAWN_RADIUS = 320 + 30  # 生成半径（320是窗口宽度的一半）

@export var basic_enemy_scene: PackedScene
@export var wizard_enemy_scene: PackedScene
@export var bat_enemy_scene: PackedScene
@export var arena_time_manager: Node

@onready var timer: Timer = $Timer

var base_spawn_time: float = 0
var enemy_table = WeightedTable.new()  # 敌人权重表

func _ready() -> void:
    # 初始敌人：基础怪，权重10
    enemy_table.add_item(basic_enemy_scene, 10)

    # 保存初始刷怪间隔
    base_spawn_time = timer.wait_time

    # 监听难度提升
    arena_time_manager.arena_difficulty_increased.connect(on_arena_difficulty_increased)

func get_spawn_position() -> Vector2:
    var player = get_tree().get_first_node_in_group("player") as Node2D
    if player == null:
        return Vector2.ZERO

    var spawn_position = Vector2.ZERO
    var random_direction = Vector2.RIGHT.rotated(randf_range(0, TAU))

    # 最多尝试4次找到有效生成点
    for i in 4:
        spawn_position = player.global_position + random_direction * SPAWN_RADIUS
        var additional_check_offset = random_direction * 20

        # 射线检测：确保生成点与玩家之间没有墙体
        var query_parameters = PhysicsRayQueryParameters2D.create(
            player.global_position,
            spawn_position + additional_check_offset,
            1 << 0  # 碰撞层0
        )
        var result = get_tree().root.world_2d.direct_space_state.intersect_ray(query_parameters)

        if result.is_empty():
            break  # 找到有效位置

        random_direction = random_direction.rotated(deg_to_rad(90))  # 旋转90度重试

    return spawn_position

func _on_timer_timeout() -> void:
    timer.start()

    var player = get_tree().get_first_node_in_group("player") as Node2D
    if player == null:
        return

    # 按权重随机选择敌人并生成
    var enemy_scene = enemy_table.pick_item()
    var enemy = enemy_scene.instantiate() as Node2D
    var entities_layer = get_tree().get_first_node_in_group("entities_layer")
    entities_layer.add_child(enemy)
    enemy.global_position = get_spawn_position()

func on_arena_difficulty_increased(arena_difficulty: int) -> void:
    # 加快刷怪速度
    var time_off = (0.1 / 12) * arena_difficulty
    time_off = min(time_off, 0.7)
    timer.wait_time = base_spawn_time - time_off

    # 难度6：解锁巫师
    if arena_difficulty == 6:
        enemy_table.add_item(wizard_enemy_scene, 15)
    # 难度15：解锁蝙蝠
    elif arena_difficulty == 15:
        enemy_table.add_item(bat_enemy_scene, 5)
```

## 智能生成位置算法

```
1. 随机方向向量
2. 在SPAWN_RADIUS距离上取点
3. 射线检测：玩家 -> 出生点
4. 如有阻挡，旋转90度重试
5. 最多4次尝试
```

### 射线检测代码

```gdscript
var query_parameters = PhysicsRayQueryParameters2D.create(
    player.global_position,      # 起点：玩家位置
    spawn_position + offset,     # 终点：生成位置
    1 << 0                       # 碰撞层0
)
var result = get_tree().root.world_2d.direct_space_state.intersect_ray(query_parameters)
```

## 权重表系统

```gdscript
class_name WeightedTable

var items: Array[PackedScene] = []
var weights: Array[float] = []

func add_item(item: PackedScene, weight: float) -> void:
    items.append(item)
    weights.append(weight)

func pick_item() -> PackedScene:
    var total_weight = weights.reduce(func(acc, w): return acc + w, 0)
    var random_value = randf() * total_weight

    var current_weight = 0.0
    for i in range(items.size()):
        current_weight += weights[i]
        if random_value <= current_weight:
            return items[i]
    return items[0]
```

## 难度联动效果

| 难度 | 刷怪间隔 | 新增敌人 |
|------|----------|----------|
| 0 | 1.0秒 | 基础怪 |
| 6 | 0.85秒 | +巫师 |
| 12 | 0.7秒 | - |
| 15 | 0.55秒 | +蝙蝠 |

## 最佳实践

1. **射线检测** - 防止生成在墙体后面
2. **权重表** - 灵活控制敌人种类分布
3. **难度信号** - 解耦难度系统与生成系统
4. **entities_layer** - 统一管理游戏实体层级
