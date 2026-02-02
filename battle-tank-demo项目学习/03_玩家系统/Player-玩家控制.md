---
title: Player 玩家控制
date: 2026-01-26
tags:
  - godot
  - player-control
  - movement
  - input-handling
category: 玩家系统
updated: 2026-01-26
related:
  - "[[Enemy-敌人AI]]"
  - "[[EnemyBullet-子弹运动]]"
---

# Player 玩家控制系统

> 坦克玩家的移动、瞄准与射击系统实现，展示 `Input.get_vector()` 和 `rotate_toward()` 的使用

## 目录
- [节点结构](#节点结构)
- [代码分析](#代码分析)
- [移动系统详解](#移动系统详解)
- [瞄准系统详解](#瞄准系统详解)
- [射击系统详解](#射击系统详解)
- [边界检测系统](#边界检测系统)
- [性能优化](#性能优化)
- [扩展功能](#扩展功能)

## 节点结构

> [!info] 场景树层级
```
Player (Area2D)
├── Sprite2D (玩家坦克车身)
├── Gun (Sprite2D)
│   └── Marker2D (子弹生成点)
└── VisibleOnScreenNotifier2D (可选)
```

### 组件说明
```
Player (Area2D)
├── Sprite2D (玩家坦克车身)
├── Gun (Sprite2D)
│   └── Marker2D (子弹生成点)
└── VisibleOnScreenNotifier2D (可选)
```

### 组件说明
| 节点 | 类型 | 功能 |
|------|------|------|
| **Player** | Area2D | 玩家根节点，碰撞检测 |
| **Sprite2D** | Sprite2D | 玩家坦克车身贴图 |
| **Gun** | Sprite2D | 炮管节点，可独立旋转 |
| **Marker2D** | Marker2D | 玩家子弹生成位置标记 |

## 代码分析

### 完整代码
```gdscript
# player.gd
extends Area2D

var direction: Vector2 = Vector2.ZERO # 方向
var target_pos: Vector2 = Vector2.ZERO # 鼠标指向位置
var speed: float = 0 # 移动速度

@export var max_speed: float = 300 ## 最大移动速度
@export var bullet_scene: PackedScene ## 炮弹场景

@onready var gun: Sprite2D = $Gun
@onready var marker_2d: Marker2D = $Gun/Marker2D

func _ready() -> void:
    pass

func _process(delta: float) -> void:
    move(delta)
    target()
    shoot()

func move(delta):
    direction = Input.get_vector("left", "right", "up", "down")
    if direction != Vector2.ZERO:
        var angle_rad = direction.angle()
        rotation = rotate_toward(rotation, angle_rad, 2 * PI * delta)
        speed = move_toward(speed, max_speed, max_speed * delta)
    else:
        speed = move_toward(speed, 0, 2 * max_speed * delta)
    
    position += transform.x * speed * delta
    check_border()

func check_border():
    var size = get_viewport_rect().size
    position = position.clamp(Vector2.ZERO, size)

func target():
    target_pos = get_global_mouse_position()
    gun.look_at(target_pos)

func shoot():
    if Input.is_action_just_pressed("shoot"):
        print("发射子弹" + str(target_pos))
        var bullet = bullet_scene.instantiate() as Area2D
        bullet.global_position = marker_2d.global_position
        bullet.look_at(target_pos)
        bullet.top_level = true
        add_child(bullet)
```

## 移动系统详解

### 1. 输入处理
```gdscript
direction = Input.get_vector("left", "right", "up", "down")
```

**Input.get_vector() 的优势**：
- 统一处理键盘、手柄、触摸输入
- 自动处理死区（deadzone）
- 返回归一化的方向向量（长度 ≤ 1）
- 支持8方向输入（对角线）

### 2. 坦克旋转
```gdscript
if direction != Vector2.ZERO:
    var angle_rad = direction.angle()
    rotation = rotate_toward(rotation, angle_rad, 2 * PI * delta)
```

**rotate_toward() 方法**：
- 参数1：当前角度
- 参数2：目标角度
- 参数3：最大旋转量（弧度/秒）
- 实现平滑旋转，避免瞬间转向

**数学原理**：
```gdscript
# rotate_toward 的近似实现
func simple_rotate_toward(current, target, max_delta):
    var diff = target - current
    # 处理角度环绕（-π 到 π）
    if diff > PI:
        diff -= 2 * PI
    elif diff < -PI:
        diff += 2 * PI
    
    if abs(diff) <= max_delta:
        return target
    else:
        return current + sign(diff) * max_delta
```

### 3. 速度控制
```gdscript
# 加速
speed = move_toward(speed, max_speed, max_speed * delta)

# 减速（减速速度是加速的2倍）
speed = move_toward(speed, 0, 2 * max_speed * delta)
```

**move_toward() 方法**：
- 线性插值，确保平滑加速/减速
- 减速更快（2倍），提高操控性
- 避免速度突变

### 4. 移动计算
```gdscript
position += transform.x * speed * delta
```

**使用 transform.x 的优势**：
- 自动适配坦克当前朝向
- 无需手动计算方向向量
- 性能更优

## 瞄准系统详解

### 鼠标瞄准
```gdscript
func target():
    target_pos = get_global_mouse_position()
    gun.look_at(target_pos)
```

**get_global_mouse_position()**：
- 返回鼠标在当前视口中的全局坐标
- 自动处理窗口缩放和偏移
- 与 `look_at()` 完美配合

### 炮管独立旋转
- 炮管（Gun）独立于坦克车身旋转
- 实现"坦克车体朝向移动方向，炮管朝向鼠标"的效果
- 提高游戏真实感和操作感

### 坐标系统
```gdscript
# 正确：使用全局坐标
gun.look_at(get_global_mouse_position())

# 错误：使用局部坐标（会导致瞄准错误）
# gun.look_at(get_local_mouse_position())
```

## 射击系统详解

### 射击触发
```gdscript
if Input.is_action_just_pressed("shoot"):
```

**just_pressed vs pressed**：
- `just_pressed`：单次触发，适合射击
- `pressed`：持续触发，适合连发（需修改）

### 子弹生成
```gdscript
var bullet = bullet_scene.instantiate() as Area2D
bullet.global_position = marker_2d.global_position
bullet.look_at(target_pos)
bullet.top_level = true
add_child(bullet)
```

**关键技术点**：
1. **全局位置**：`marker_2d.global_position` 确保生成位置准确
2. **子弹方向**：`look_at(target_pos)` 使子弹朝向鼠标位置
3. **独立坐标系**：`top_level = true` 避免子弹随坦克移动
4. **类型安全**：`as Area2D` 确保类型正确

### 射击反馈
```gdscript
print("发射子弹" + str(target_pos))
```
**可扩展为**：
- 播放射击音效
- 添加枪口火焰特效
- 屏幕震动效果
- 后坐力动画

## 边界检测系统

### 视口边界限制
```gdscript
func check_border():
    var size = get_viewport_rect().size
    position = position.clamp(Vector2.ZERO, size)
```

**get_viewport_rect().size**：
- 返回当前视口的大小（像素）
- 自动适应窗口缩放
- 动态更新（窗口大小改变时）

**clamp() 方法**：
- 限制位置在指定范围内
- `Vector2.ZERO` 到 `size` 确保坦克不会移出屏幕
- 简洁高效，无需复杂条件判断

### 边界处理策略对比
| 策略 | 实现方式 | 适用场景 |
|------|----------|----------|
| **位置限制** | `position.clamp()` | 固定边界，坦克不能移出 |
| **视口跟随** | 摄像机跟随玩家 | 大地图，玩家始终在中心 |
| **边界反弹** | 到达边界后反向移动 | 弹球类游戏 |
| **边界穿透** | 从一侧穿出从另一侧进入 | 太空射击游戏 |

## 性能优化

### 1. 输入处理优化
```gdscript
# 当前：每帧检测输入
func _process(delta):
    move(delta)
    target()
    shoot()

# 优化：分离输入检测频率
var input_accumulator = 0.0
const INPUT_INTERVAL = 0.05  # 20Hz

func _process(delta):
    input_accumulator += delta
    if input_accumulator >= INPUT_INTERVAL:
        process_input()
        input_accumulator = 0.0
    
    # 移动和瞄准每帧执行
    update_movement(delta)
    update_aiming()
```

### 2. 子弹对象池
```gdscript
# 减少实例化开销
class BulletPool:
    var pool: Array[Area2D] = []
    
    func get_bullet() -> Area2D:
        if pool.is_empty():
            return bullet_scene.instantiate()
        else:
            return pool.pop_back()
    
    func return_bullet(bullet: Area2D):
        bullet.hide()
        bullet.position = Vector2.ZERO
        pool.append(bullet)
```

### 3. 距离检查优化
```gdscript
# 只在敌人进入一定范围内时才精确瞄准
const AIM_RANGE = 500.0

func target():
    var mouse_pos = get_global_mouse_position()
    var distance = global_position.distance_to(mouse_pos)
    
    if distance > AIM_RANGE:
        # 简单瞄准：炮管朝向移动方向
        gun.rotation = rotation
    else:
        # 精确瞄准：炮管朝向鼠标
        gun.look_at(mouse_pos)
```

## 扩展功能

### 1. 多种移动模式
```gdscript
enum MoveMode { TANK, CAR, HOVER }
@export var move_mode: MoveMode = MoveMode.TANK

func move(delta):
    match move_mode:
        MoveMode.TANK:
            # 坦克模式：车体朝向移动方向
            tank_move(delta)
        MoveMode.CAR:
            # 汽车模式：车体独立于移动方向
            car_move(delta)
        MoveMode.HOVER:
            # 悬浮模式：惯性移动
            hover_move(delta)
```

### 2. 武器系统
```gdscript
class WeaponSystem:
    var weapons: Array[Weapon] = []
    var current_weapon_index = 0
    
    func shoot():
        var weapon = weapons[current_weapon_index]
        if weapon.can_shoot():
            weapon.shoot()
    
    func switch_weapon(index: int):
        if index >= 0 and index < weapons.size():
            current_weapon_index = index

class Weapon:
    var fire_rate: float = 0.5
    var damage: float = 10
    var bullet_scene: PackedScene
    var last_shot_time: float = 0.0
    
    func can_shoot() -> bool:
        return Time.get_ticks_msec() - last_shot_time >= fire_rate * 1000
    
    func shoot() -> Area2D:
        last_shot_time = Time.get_ticks_msec()
        var bullet = bullet_scene.instantiate()
        # 配置子弹属性
        return bullet
```

### 3. 技能系统
```gdscript
class SkillSystem:
    var skills: Dictionary = {}
    var cooldowns: Dictionary = {}
    
    func register_skill(name: String, cooldown: float, callback: Callable):
        skills[name] = callback
        cooldowns[name] = 0.0
    
    func use_skill(name: String):
        if name in skills and cooldowns[name] <= 0:
            skills[name].call()
            cooldowns[name] = get_cooldown(name)
    
    func _process(delta: float):
        for name in cooldowns.keys():
            if cooldowns[name] > 0:
                cooldowns[name] -= delta
```

### 4. 状态效果
```gdscript
class StatusEffect:
    var name: String
    var duration: float
    var start_time: float
    
    func apply(target: Node):
        # 应用效果
        pass
    
    func update(target: Node, delta: float):
        duration -= delta
        if duration <= 0:
            remove(target)
    
    func remove(target: Node):
        # 移除效果
        pass

class PlayerStatus:
    var effects: Array[StatusEffect] = []
    
    func add_effect(effect: StatusEffect):
        effects.append(effect)
        effect.apply(self)
    
    func _process(delta: float):
        for i in range(effects.size() - 1, -1, -1):
            effects[i].update(self, delta)
            if effects[i].duration <= 0:
                effects.remove_at(i)
```

> [!warning] 调试与测试
> 以下调试技巧可帮助你排查玩家控制系统的问题。

## 调试与测试

### 1. 输入可视化
```gdscript
func _draw():
    # 绘制移动方向
    var center = Vector2(50, 50)
    var dir_end = center + direction * 40
    draw_line(center, dir_end, Color.GREEN, 2)
    
    # 绘制速度条
    var speed_bar = Rect2(10, 10, speed / max_speed * 100, 10)
    draw_rect(speed_bar, Color.RED)
    
    # 绘制瞄准线
    var gun_pos = to_local(gun.global_position)
    var target_local = to_local(target_pos)
    draw_line(gun_pos, target_local, Color.YELLOW, 1)
```

### 2. 性能监控
```gdscript
var frame_times: Array[float] = []
const MAX_FRAMES = 60

func _process(delta: float):
    var start_time = Time.get_ticks_usec()
    
    # 正常逻辑...
    
    var end_time = Time.get_ticks_usec()
    frame_times.append((end_time - start_time) / 1000.0)  # 转换为毫秒
    
    if frame_times.size() > MAX_FRAMES:
        frame_times.remove_at(0)
    
    if Engine.get_frames_drawn() % 60 == 0:  # 每秒一次
        var avg_time = frame_times.reduce(func(a, b): return a + b) / frame_times.size()
        print("玩家控制平均帧时间: %.3f ms" % avg_time)
```

### 3. 碰撞调试
```gdscript
func _ready():
    # 启用碰撞形状可视化
    $CollisionShape2D.debug_color = Color(1, 0, 0, 0.3)
    
    # 连接碰撞信号
    area_entered.connect(_on_area_entered_debug)

func _on_area_entered_debug(area: Area2D):
    print("玩家碰撞: ", area.name)
    print("碰撞位置: ", global_position)
    print("碰撞对象位置: ", area.global_position)
```

## 最佳实践总结

### 1. 输入处理
- 使用 `Input.get_vector()` 统一多设备输入
- 区分 `just_pressed` 和 `pressed` 的使用场景
- 添加输入缓冲提高响应性

### 2. 移动系统
- `rotate_toward()` 实现平滑旋转
- `move_toward()` 实现平滑加速/减速
- `transform.x` 进行方向移动

### 3. 瞄准系统
- `get_global_mouse_position()` 获取鼠标位置
- `look_at()` 实现自动瞄准
- 炮管独立于车身旋转

### 4. 射击系统
- 使用 `global_position` 确保生成位置准确
- `top_level = true` 避免坐标系问题
- 对象池优化子弹实例化

### 5. 边界处理
- `get_viewport_rect().size` 获取视口大小
- `clamp()` 限制位置范围
- 考虑不同边界策略

### 6. 性能优化
- 分离输入检测频率
- 使用对象池减少实例化
- 距离检查避免不必要的计算

> [!success] 最佳实践总结
> - 使用 `Input.get_vector()` 统一多设备输入
> - `rotate_toward()` 实现平滑旋转
> - `move_toward()` 实现平滑加速/减速
> - `transform.x` 进行方向移动
> - `get_global_mouse_position()` 获取鼠标位置
> - `look_at()` 实现自动瞄准
> - 使用对象池优化子弹实例化

## 学习收获

### 核心技术
1. **输入处理**：多设备输入统一处理
2. **平滑移动**：rotate_toward 和 move_toward 的使用
3. **坐标系统**：全局坐标在游戏开发中的重要性
4. **组件设计**：玩家系统的模块化架构

### 设计模式
1. **状态模式**：移动、瞄准、射击状态分离
2. **对象池模式**：子弹实例化优化
3. **策略模式**：多种移动模式实现
4. **观察者模式**：碰撞检测与信号系统

### 调试技巧
1. **可视化调试**：_draw() 方法绘制调试信息
2. **性能监控**：帧时间统计与优化
3. **碰撞调试**：碰撞形状可视化与日志输出

---

## 相关笔记

- [[../欢迎|项目主页]]
- [[Enemy-敌人AI]]
- [[../04_项目实战/坦克运动战功能总结|坦克运动战功能总结]]

---

*最后更新：2026-01-26*

%% 
内部笔记：
- 考虑添加性能监控面板
- 补充视频教程链接
%%