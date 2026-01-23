# VelocityComponent - 移动物理系统

## 组件功能

处理游戏实体的移动逻辑，包括加速度、速度限制和平滑移动。

## 核心代码

```gdscript
extends Node
class_name VelocityComponent

@export var max_speed: float = 40      # 最大速度
@export var acceleration: float = 5    # 加速度

var velocity: Vector2 = Vector2.ZERO

# 朝玩家方向加速（敌人追踪）
func accelerate_to_player() -> void:
    var owner_node2d = owner as Node2D
    if owner_node2d == null:
        return

    var player = get_tree().get_first_node_in_group("player") as Node2D
    if player == null:
        return

    var direction = (player.global_position - owner_node2d.global_position).normalized()
    accelerate_in_direction(direction)

# 沿指定方向加速
func accelerate_in_direction(direction: Vector2) -> void:
    var desired_velocity = direction * max_speed
    # 指数插值：帧率无关的平滑移动
    velocity = velocity.lerp(desired_velocity, 1 - exp(-acceleration * get_process_delta_time()))

# 减速停止
func decelerate() -> void:
    accelerate_in_direction(Vector2.ZERO)

# 执行移动
func move(character_body: CharacterBody2D) -> void:
    # 注意：move_and_slide 可能会修改 velocity
    character_body.velocity = velocity
    character_body.move_and_slide()
    velocity = character_body.velocity  # 回写修正后的速度
```

## 关键设计点

### 1. 指数插值实现平滑加速

```gdscript
# 公式：velocity.lerp(desired, 1 - exp(-acceleration * delta))
velocity = velocity.lerp(desired_velocity, 1 - exp(-acceleration * get_process_delta_time()))
```

**优点**：
- 帧率无关
- 加速度越大，达到最大速度越快
- 自然平滑的加减速效果

### 2. 速度回写机制

```gdscript
# move_and_slide 会因碰撞修改速度
character_body.velocity = velocity
character_body.move_and_slide()
velocity = character_body.velocity  # 必须回写
```

### 3. Group 获取玩家

```gdscript
var player = get_tree().get_first_node_in_group("player")
```

## 应用场景

| 场景 | 配置 |
|------|------|
| 玩家 | 高加速度，高最大速度 |
| 追踪敌人 | 中等加速度，中等最大速度 |
| 笨重敌人 | 低加速度，低最大速度 |

## 使用示例

```gdscript
# 玩家移动（Player.gd）
func _physics_process(_delta: float) -> void:
    var movement_vector = get_movement_vector()
    direction = movement_vector.normalized()
    velocity_component.accelerate_in_direction(direction)
    velocity_component.move(self)

# 敌人追踪
func _physics_process(_delta: float) -> void:
    velocity_component.accelerate_to_player()
    velocity_component.move(self)
```

## 参数调节建议

| 参数 | 效果 | 调节建议 |
|------|------|----------|
| max_speed | 最大速度 | 根据游戏节奏调整 |
| acceleration | 加速度 | 大=跟手，小=惯性大 |

## 最佳实践

1. **组件独立** - 移动逻辑与实体逻辑分离
2. **帧率无关** - 使用 delta 进行插值计算
3. **碰撞兼容** - 正确处理 move_and_slide 的速度修改
