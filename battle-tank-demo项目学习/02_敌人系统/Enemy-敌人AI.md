# Enemy 敌人AI系统

> 坦克敌人的自动瞄准与射击AI实现

## 目录
- [节点结构](#节点结构)
- [代码分析](#代码分析)
- [AI逻辑详解](#ai逻辑详解)
- [坐标系统处理](#坐标系统处理)
- [射击机制](#射击机制)
- [性能优化](#性能优化)
- [扩展功能](#扩展功能)

## 节点结构

### 场景树层级
```
Enemy (Area2D)
├── Sprite2D (敌人贴图)
├── Gun (Sprite2D)
│   └── Marker2D (子弹生成点)
└── Timer (射击冷却)
```

### 组件说明
| 节点 | 类型 | 功能 |
|------|------|------|
| **Enemy** | Area2D | 敌人根节点，碰撞检测 |
| **Sprite2D** | Sprite2D | 敌人坦克车身贴图 |
| **Gun** | Sprite2D | 炮管节点，可独立旋转 |
| **Marker2D** | Marker2D | 子弹生成位置标记 |
| **Timer** | Timer | 射击间隔控制 |

## 代码分析

### 完整代码
```gdscript
# enemy.gd
extends Area2D

@export var player: Node2D ## 玩家节点
@export var bullet_scene: PackedScene ## 子弹场景

@onready var gun: Sprite2D = $Gun
@onready var marker_2d: Marker2D = $Gun/Marker2D
@onready var timer: Timer = $Timer

func _ready() -> void:
    timer.start(1)

func _process(_delta: float) -> void:
    find_player()

func find_player():
    var player_pos = player.global_position # 在不同节点之间传递位置信息的时候，要用全局坐标
    if not player_pos: # 如果玩家节点为空
        printerr("玩家节点为空")
        return
    #var player_pos = get_global_mouse_position() # 暂时用鼠标位置
    gun.look_at(player_pos) # look_at旋转该节点，使其局部 X 轴的正方向指向 point，该参数应使用全局坐标。

func shoot():
    var bullet = bullet_scene.instantiate() as Area2D
    bullet.rotation = gun.rotation # 子弹方向和炮管方向一致
    bullet.global_position = marker_2d.global_position # 子弹场景在对应位置生成
    bullet.top_level = true # 将子弹提升为"世界级节点"
    add_child(bullet)

func _on_timer_timeout() -> void:
    shoot()
    timer.start(randf_range(1, 3)) # 随机冷却时间
```

## AI逻辑详解

### 1. 持续瞄准机制
```gdscript
func _process(_delta: float) -> void:
    find_player()  # 每帧执行
```

**设计选择**：
- 使用 `_process()` 而非 `_physics_process()`：瞄准不需要物理精度
- 每帧更新确保响应及时
- 简单的函数调用，易于维护

### 2. 玩家位置获取
```gdscript
var player_pos = player.global_position
```

**关键点**：
- 使用 `global_position` 而非 `position`
- 确保坐标系统一致性
- 添加空值检查避免崩溃

### 3. 自动瞄准实现
```gdscript
gun.look_at(player_pos)
```

**look_at() 工作原理**：
1. 计算从 `gun` 到 `player_pos` 的方向向量
2. 将该方向向量的角度设置为 `gun.rotation`
3. 使炮管的局部X轴正方向指向玩家

**数学等价**：
```gdscript
func manual_look_at(target_pos: Vector2):
    var direction = (target_pos - gun.global_position).normalized()
    gun.rotation = direction.angle()
```

## 坐标系统处理

### 全局坐标的重要性

**问题场景**：
- Enemy 节点可能位于复杂层级中
- Gun 是 Enemy 的子节点
- 玩家可能在不同的节点层级

**解决方案**：
```gdscript
# ❌ 错误：使用局部坐标
var player_local_pos = player.position

# ✅ 正确：使用全局坐标
var player_global_pos = player.global_position
gun.look_at(player_global_pos)
```

### 子弹生成位置

```gdscript
bullet.global_position = marker_2d.global_position
```

**为什么使用 Marker2D？**
1. **精确定位**：Marker2D 提供准确的局部位置
2. **跟随炮管**：Marker2D 作为 Gun 的子节点，随炮管旋转而移动
3. **易于调整**：在编辑器中可视化调整生成点

### top_level 属性的作用

```gdscript
bullet.top_level = true
```

**功能**：
- 使子弹脱离父节点（Enemy）的坐标系
- 子弹的 `position` 变为全局坐标
- 避免子弹随敌人移动而偏移

**对比实验**：
```gdscript
# top_level = false（默认）
# 子弹位置随敌人移动而改变

# top_level = true
# 子弹位置在世界坐标系中固定
```

## 射击机制

### 1. 定时器控制
```gdscript
func _ready() -> void:
    timer.start(1)  # 1秒后首次射击

func _on_timer_timeout() -> void:
    shoot()
    timer.start(randf_range(1, 3))  # 随机冷却
```

**设计特点**：
- 初始延迟：给玩家反应时间
- 随机间隔：增加游戏不确定性
- 可配置性：通过 @export 变量调整

### 2. 子弹实例化
```gdscript
func shoot():
    var bullet = bullet_scene.instantiate() as Area2D
    # 配置子弹属性
    bullet.rotation = gun.rotation
    bullet.global_position = marker_2d.global_position
    bullet.top_level = true
    add_child(bullet)
```

**实例化流程**：
1. **加载场景**：`bullet_scene.instantiate()`
2. **类型转换**：`as Area2D` 确保类型安全
3. **方向设置**：继承炮管旋转角度
4. **位置设置**：使用全局坐标
5. **层级设置**：独立于敌人坐标系
6. **添加到树**：`add_child(bullet)`

### 3. 随机冷却时间
```gdscript
timer.start(randf_range(1, 3))
```

**随机范围**：1-3秒
- 最小间隔：1秒，防止连续射击
- 最大间隔：3秒，给玩家喘息机会
- 随机性：增加游戏趣味性

## 性能优化

### 1. 优化查找频率
```gdscript
# 当前：每帧查找
func _process(_delta: float):
    find_player()

# 优化：降低频率
var search_interval = 0.1  # 100ms
var search_accumulator = 0.0

func _process(delta: float):
    search_accumulator += delta
    if search_accumulator >= search_interval:
        find_player()
        search_accumulator = 0.0
```

### 2. 距离检查优化
```gdscript
# 添加距离检查，避免远距离无效瞄准
const MAX_AIM_DISTANCE = 1000

func find_player():
    var player_pos = player.global_position
    var distance = global_position.distance_to(player_pos)
    
    if distance > MAX_AIM_DISTANCE:
        return  # 超出范围，不瞄准
    
    gun.look_at(player_pos)
```

### 3. 状态管理优化
```gdscript
enum EnemyState { IDLE, AIMING, SHOOTING, RELOADING }
var current_state = EnemyState.IDLE

func _process(delta: float):
    match current_state:
        EnemyState.IDLE:
            # 检查是否发现玩家
            if can_see_player():
                current_state = EnemyState.AIMING
        EnemyState.AIMING:
            find_player()
            if is_aimed_at_player():
                current_state = EnemyState.SHOOTING
        EnemyState.SHOOTING:
            shoot()
            current_state = EnemyState.RELOADING
            timer.start(reload_time)
        EnemyState.RELOADING:
            pass  # 等待计时器
```

## 扩展功能

### 1. 玩家检测范围
```gdscript
# 添加检测区域
@onready var detection_area: Area2D = $DetectionArea

func _on_detection_area_body_entered(body: Node2D):
    if body.is_in_group("Player"):
        player = body
        start_aiming()

func _on_detection_area_body_exited(body: Node2D):
    if body == player:
        player = null
        stop_aiming()
```

### 2. 瞄准预测系统
```gdscript
# 预测玩家移动轨迹
func predict_aim_position():
    if not player:
        return null
    
    var player_pos = player.global_position
    var player_velocity = player.velocity
    var bullet_speed = 100  # 子弹速度
    
    # 计算预测位置
    var distance = global_position.distance_to(player_pos)
    var time_to_hit = distance / bullet_speed
    var predicted_pos = player_pos + player_velocity * time_to_hit
    
    return predicted_pos
```

### 3. 多目标优先级
```gdscript
# 多个玩家时的目标选择
var players_in_range = []

func select_best_target():
    if players_in_range.is_empty():
        return null
    
    # 选择最近的玩家
    var best_target = null
    var min_distance = INF
    
    for p in players_in_range:
        var distance = global_position.distance_to(p.global_position)
        if distance < min_distance:
            min_distance = distance
            best_target = p
    
    return best_target
```

### 4. 难度调整系统
```gdscript
# 根据游戏进度调整AI难度
@export var difficulty_level = 1

func update_difficulty():
    match difficulty_level:
        1:  # 简单
            timer.wait_time = randf_range(2, 4)
            bullet_speed = 80
        2:  # 中等
            timer.wait_time = randf_range(1, 3)
            bullet_speed = 100
        3:  # 困难
            timer.wait_time = randf_range(0.5, 2)
            bullet_speed = 120
```

## 调试与测试

### 可视化调试
```gdscript
func _draw():
    if not player:
        return
    
    # 绘制瞄准线
    var start = gun.global_position
    var end = player.global_position
    var local_start = to_local(start)
    var local_end = to_local(end)
    
    draw_line(local_start, local_end, Color.RED, 2)
    
    # 绘制射击方向
    var direction = transform.x * 50
    draw_line(Vector2.ZERO, direction, Color.YELLOW, 3)
```

### 日志输出
```gdscript
func debug_info():
    print("=== 敌人AI调试信息 ===")
    print("玩家: ", player)
    print("玩家位置: ", player.global_position if player else "无")
    print("炮管旋转: ", gun.rotation)
    print("冷却时间: ", timer.time_left)
    print("====================")
```

## 总结

### 设计亮点
1. **简洁的AI逻辑**：`look_at()` 实现自动瞄准
2. **正确的坐标处理**：全局坐标确保精度
3. **灵活的射击机制**：随机间隔增加趣味性
4. **良好的扩展性**：模块化设计易于增强

### 改进方向
1. 添加状态机管理复杂行为
2. 实现瞄准预测提高难度
3. 增加多种攻击模式
4. 优化性能降低CPU占用

### 学习要点
1. 掌握 `look_at()` 的使用场景
2. 理解全局坐标在节点交互中的重要性
3. 学会使用 Timer 实现游戏循环
4. 实践实例化场景对象的完整流程

---

*返回 [[欢迎|项目主页]]*