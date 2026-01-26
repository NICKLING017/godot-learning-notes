# Transform 变换详解

> 上帝引擎空间变换的核心概念，理解 2D/3D 游戏开发的数学基础

## 目录
- [什么是 Transform？](#什么是-transform)
- [Transform2D 结构分析](#transform2d-结构分析)
- [局部坐标 vs 全局坐标](#局部坐标-vs-全局坐标)
- [transform.x 的实际应用](#transformx-的实际应用)
- [look_at() 瞄准系统](#lookat-瞄准系统)
- [常见错误与调试技巧](#常见错误与调试技巧)
- [实战代码分析](#实战代码分析)
- [扩展应用场景](#扩展应用场景)

## 什么是 Transform？

Transform（变换）是 Godot 中**描述物体在空间中状态**的核心数据结构。它包含了三个关键信息：

| 属性 | 描述 | 数学表示 |
|------|------|----------|
| **位置 (Position)** | 物体在空间中的坐标 | Vector2 (x, y) |
| **旋转 (Rotation)** | 物体朝向的角度 | 弧度 (radians) |
| **缩放 (Scale)** | 物体的大小比例 | Vector2 (x, y) |

### 为什么需要 Transform？

在游戏开发中，我们经常需要：
1. 移动物体到指定位置
2. 旋转物体使其面向目标
3. 缩放物体调整大小
4. 组合上述变换实现复杂效果

Transform 将这些操作**统一封装**，提供一致的数学接口。

## Transform2D 结构分析

### 数学定义
`Transform2D` 是一个 2×3 矩阵：
```
[ x.x  x.y  origin.x ]
[ y.x  y.y  origin.y ]
```

### 关键属性
```gdscript
var transform: Transform2D

# 三个核心向量
transform.x      # 局部 X 轴方向向量 (归一化)
transform.y      # 局部 Y 轴方向向量 (归一化)
transform.origin # 位置坐标 (等同于 position)
```

### 代码验证
```gdscript
# 打印 transform 信息
func print_transform_info():
    print("transform.x = ", transform.x)      # 例如: (1, 0)
    print("transform.y = ", transform.y)      # 例如: (0, 1)
    print("transform.origin = ", transform.origin)  # 例如: (100, 200)
    print("length of x = ", transform.x.length())   # 应该为 1
    print("rotation = ", rotation)                  # 弧度值
```

### 重要关系
1. **transform.x** = `Vector2(cos(rotation), sin(rotation))`
2. **transform.y** = `Vector2(-sin(rotation), cos(rotation))`
3. 当 rotation = 0 时：`transform.x = (1, 0)`, `transform.y = (0, 1)`

## 局部坐标 vs 全局坐标

### 坐标系统对比

| 特性 | 局部坐标 (Local) | 全局坐标 (Global) |
|------|------------------|-------------------|
| **参考点** | 父节点原点 | 世界原点 (0, 0) |
| **表示方法** | `position` | `global_position` |
| **变换矩阵** | `transform` | `global_transform` |
| **使用场景** | 节点相对布局 | 跨节点交互 |

### 转换方法
```gdscript
# 1. 局部 → 全局
var global_pos = to_global(local_pos)
var global_pos = global_position  # 快捷方式

# 2. 全局 → 局部
var local_pos = to_local(global_pos)

# 3. 获取全局变换
var world_transform = global_transform
```

### 实战经验：为什么必须用全局坐标？

**错误案例**：
```gdscript
# enemy.gd 中获取玩家位置
var player_pos = player.position  # ❌ 这是局部坐标！

func find_player():
    gun.look_at(player_pos)  # ❌ 可能瞄准错误位置
```

**正确做法**：
```gdscript
# 在不同节点之间传递位置时
var player_pos = player.global_position  # ✅ 全局坐标

func find_player():
    gun.look_at(player_pos)  # ✅ 准确瞄准
```

**原因分析**：
- `player.position` 是相对于 player 父节点的坐标
- 如果 player 节点有复杂的层级结构，`position` 不是世界坐标
- `look_at()` 方法要求传入**全局坐标**

## transform.x 的实际应用

### 子弹运动系统（项目核心代码）

```gdscript
# enemy_bullet.gd 第7行
func _process(delta: float) -> void:
    position += transform.x * bullet_speed * delta
```

### 数学原理分解

1. **transform.x** = 归一化的方向向量
   - 长度 = 1
   - 方向 = 物体的局部 X 轴方向

2. **位移计算**：
   ```
   位移 = 方向 × 速度 × 时间
   position += transform.x * bullet_speed * delta
   ```

3. **与传统方法的对比**：

```gdscript
# ❌ 传统方法（需要三角函数）
func move_traditional(delta):
    var direction = Vector2(cos(rotation), sin(rotation))
    position += direction * speed * delta

# ✅ Transform 方法（高效简洁）
func move_transform(delta):
    position += transform.x * speed * delta
```

### 性能优势

| 方法 | 计算复杂度 | 代码简洁度 | 可读性 |
|------|------------|------------|--------|
| 三角函数法 | 高 (cos/sin) | 低 | 一般 |
| transform.x | 低 (直接读取) | 高 | 优秀 |

### 扩展应用：多方向移动

```gdscript
# 沿局部 X 轴正方向移动
position += transform.x * speed * delta

# 沿局部 X 轴负方向移动
position -= transform.x * speed * delta

# 沿局部 Y 轴移动
position += transform.y * speed * delta

# 对角线移动
position += (transform.x + transform.y).normalized() * speed * delta
```

## look_at() 瞄准系统

### 敌人瞄准代码分析

```gdscript
# enemy.gd 第20-26行
func find_player():
    var player_pos = player.global_position  # 获取全局坐标
    if not player_pos:
        printerr("玩家节点为空")
        return
    gun.look_at(player_pos)  # 关键方法
```

### look_at() 方法详解

**功能**：旋转节点，使其**局部 X 轴正方向**指向目标点。

**参数要求**：
- 必须传入**全局坐标** (Vector2)
- 目标点在世界空间中的位置

**数学等价实现**：
```gdscript
func manual_look_at(target_global_pos: Vector2):
    # 计算方向向量
    var direction = (target_global_pos - global_position).normalized()
    
    # 计算角度（弧度）
    var angle = direction.angle()
    
    # 设置旋转
    rotation = angle
```

### 使用要点

1. **坐标一致性**：
   ```gdscript
   # ❌ 错误：混合坐标系统
   gun.look_at(player.position)  # 局部坐标
   
   # ✅ 正确：统一全局坐标
   gun.look_at(player.global_position)  # 全局坐标
   ```

2. **节点层级影响**：
   ```gdscript
   # gun 是 enemy 的子节点
   # gun.look_at() 使用 gun 的全局坐标系
   # 但旋转是相对于 gun 的父节点（enemy）
   ```

3. **立即生效**：
   - `look_at()` 是立即执行的
   - 如需平滑旋转，需要自行实现插值

### 实际应用：炮塔旋转

```gdscript
# 完整的炮塔瞄准系统
class_name Turret extends Node2D

@export var target: Node2D
@export var rotation_speed: float = 5.0  # 弧度/秒

func _process(delta: float):
    if not target:
        return
    
    # 获取目标全局位置
    var target_pos = target.global_position
    
    # 计算当前朝向与目标方向的夹角
    var current_direction = transform.x
    var target_direction = (target_pos - global_position).normalized()
    var angle_diff = current_direction.angle_to(target_direction)
    
    # 平滑旋转（可选）
    if abs(angle_diff) > 0.01:
        rotation += sign(angle_diff) * min(abs(angle_diff), rotation_speed * delta)
    else:
        # 直接瞄准（项目中使用的方法）
        look_at(target_pos)
```

## 常见错误与调试技巧

### 错误1：坐标系统混淆

**症状**：物体移动到奇怪的位置，瞄准不准确。

**调试方法**：
```gdscript
func debug_coordinates():
    print("=== 坐标调试 ===")
    print("local position: ", position)
    print("global position: ", global_position)
    print("parent position: ", get_parent().global_position if get_parent() else "无父节点")
    print("=================")
```

**解决方案**：
- 在不同节点间交互时，**始终使用全局坐标**
- 使用 `global_position` 而非 `position`
- `look_at()` 参数必须是全局坐标

### 错误2：忽略节点层级

**症状**：子节点旋转不正常，位置偏移。

**解决方案**：
```gdscript
# 方法1：设置 top_level
bullet.top_level = true  # 使子弹独立于父节点坐标系
bullet.global_position = spawn_point.global_position

# 方法2：手动计算全局变换
var global_spawn_pos = to_global(spawn_point.position)
```

### 错误3：误解 transform.x 的方向

**验证方法**：
```gdscript
func validate_transform():
    # 绘制方向线（调试用）
    var start = global_position
    var end = start + transform.x * 50  # 50像素长的X轴方向
    draw_line(start, end, Color.RED, 2)
    
    # 验证向量长度
    print("transform.x length: ", transform.x.length())  # 应该≈1
```

### 调试可视化工具

```gdscript
# 在 _draw() 方法中可视化变换
func _draw():
    # 绘制局部X轴（红色）
    var x_end = transform.x * 30
    draw_line(Vector2.ZERO, x_end, Color.RED, 2)
    
    # 绘制局部Y轴（绿色）
    var y_end = transform.y * 30
    draw_line(Vector2.ZERO, y_end, Color.GREEN, 2)
    
    # 绘制朝向指示器
    var direction = transform.x * 50
    draw_line(Vector2.ZERO, direction, Color.YELLOW, 3)
```

## 实战代码分析

### 敌人系统完整实现

```gdscript
# enemy.gd - 敌人AI与射击系统
extends Area2D

@export var player: Node2D  # 玩家节点引用
@export var bullet_scene: PackedScene  # 子弹场景

@onready var gun: Sprite2D = $Gun
@onready var marker_2d: Marker2D = $Gun/Marker2D
@onready var timer: Timer = $Timer

func _ready() -> void:
    timer.start(1)  # 1秒后开始射击

func _process(_delta: float) -> void:
    find_player()  # 持续瞄准玩家

func find_player():
    # 关键：使用全局坐标进行交互
    var player_pos = player.global_position
    if not player_pos:
        printerr("玩家节点为空")
        return
    
    # 核心：look_at 实现自动瞄准
    gun.look_at(player_pos)

func shoot():
    # 实例化子弹
    var bullet = bullet_scene.instantiate() as Area2D
    
    # 继承炮管的旋转方向
    bullet.rotation = gun.rotation
    
    # 使用全局坐标设置生成位置
    bullet.global_position = marker_2d.global_position
    
    # 重要：使子弹独立于敌人坐标系
    bullet.top_level = true
    
    # 添加到场景
    add_child(bullet)

func _on_timer_timeout() -> void:
    shoot()
    timer.start(randf_range(1, 3))  # 随机冷却时间
```

### 子弹运动系统

```gdscript
# enemy_bullet.gd - 子弹物理运动
extends Area2D

@export var bullet_speed: float = 100

func _process(delta: float) -> void:
    # 核心：使用 transform.x 进行方向移动
    position += transform.x * bullet_speed * delta

func _on_visible_on_screen_notifier_2d_screen_exited() -> void:
    # 离开屏幕时自动回收
    queue_free()

func _on_area_entered(area: Area2D) -> void:
    # 碰撞检测
    if area.is_in_group("Player"):
        print_debug("子弹命中玩家")
        queue_free()  # 命中后销毁
```

### 代码设计亮点

1. **清晰的坐标处理**：
   - 明确区分局部/全局坐标
   - 跨节点交互使用 `global_position`

2. **高效的移动计算**：
   - 使用 `transform.x` 避免三角函数
   - 利用内置向量运算

3. **合理的节点层级**：
   - `bullet.top_level = true` 解决坐标系问题
   - 独立的碰撞检测逻辑

## 扩展应用场景

### 1. 玩家坦克移动

```gdscript
# player.gd - 坦克移动控制
extends CharacterBody2D

@export var move_speed: float = 200
@export var rotation_speed: float = 3

func _physics_process(delta: float):
    # 获取输入
    var move_input = Input.get_vector("left", "right", "up", "down")
    
    # 旋转坦克车身
    if move_input.x != 0:
        rotation += move_input.x * rotation_speed * delta
    
    # 前后移动（沿局部X轴）
    if move_input.y != 0:
        velocity = transform.x * move_input.y * move_speed
    else:
        velocity = Vector2.ZERO
    
    move_and_slide()
```

### 2. 相机跟随系统

```gdscript
# camera_follow.gd - 平滑相机跟随
extends Camera2D

@export var target: Node2D
@export var follow_speed: float = 5.0

func _process(delta: float):
    if not target:
        return
    
    # 使用全局坐标计算相机位置
    var target_pos = target.global_position
    
    # 平滑插值
    global_position = global_position.lerp(target_pos, follow_speed * delta)
    
    # 保持与目标相同的旋转（可选）
    # global_rotation = lerp_angle(global_rotation, target.global_rotation, follow_speed * delta)
```

### 3. 弹道预测系统

```gdscript
# trajectory_prediction.gd - 弹道预测
func predict_hit_position(
    shooter_pos: Vector2,
    target_pos: Vector2,
    target_velocity: Vector2,
    bullet_speed: float
) -> Vector2:
    # 计算相对位置和速度
    var relative_pos = target_pos - shooter_pos
    var relative_vel = target_velocity
    
    # 解运动方程（简化版）
    # 实际实现需要解二次方程
    var a = relative_vel.length_squared() - bullet_speed * bullet_speed
    var b = 2 * relative_pos.dot(relative_vel)
    var c = relative_pos.length_squared()
    
    # 返回预测位置（简化）
    var time_to_hit = relative_pos.length() / bullet_speed
    return target_pos + target_velocity * time_to_hit
```

## 总结

Transform 是 Godot 游戏开发中的**核心数学工具**，理解它对于掌握 2D/3D 游戏编程至关重要。

### 关键要点
1. **坐标系统**：明确区分局部坐标与全局坐标
2. **方向向量**：善用 `transform.x` 和 `transform.y` 进行方向计算
3. **瞄准系统**：`look_at()` 配合全局坐标实现自动瞄准
4. **性能优化**：优先使用向量运算而非三角函数

### 学习建议
1. 在实际项目中练习 Transform 的各种用法
2. 使用调试绘图可视化变换信息
3. 阅读 Godot 官方文档中的数学相关章节
4. 尝试实现更复杂的变换组合（如缩放+旋转+位移）

### 下一步学习
- 学习 **Transform3D** 在 3D 项目中的应用
- 探索 **InterpolatedTransform** 实现平滑过渡
- 研究 **Transform** 在物理引擎中的使用

---

*返回 [[欢迎|项目主页]]*