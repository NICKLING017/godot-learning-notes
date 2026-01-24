---
title: Flappy Bird 项目学习笔记
date: 2026-01-24
tags:
  - godot
  - gdscript
  - 学习笔记
  - flappy-bird
  - 游戏开发
status: completed
priority: high
---

# Flappy Bird 项目学习笔记

## 项目概述

本项目是一个基于 Godot 4 引擎实现的 Flappy Bird 游戏克隆。通过这个项目，学习了 Godot 的基本开发流程、物理系统、信号机制、场景管理和游戏状态控制。

**项目特点**：
- 使用 `CharacterBody2D` 实现小鸟物理运动
- 使用 `Parallax2D` 实现背景滚动
- 基于信号 (`Signal`) 的游戏事件系统
- 使用 `VisibleOnScreenNotifier2D` 自动回收管道
- 模块化的场景设计（小鸟、管道、背景、HUD 分离）

## 项目结构

```plaintext
C:\Godot\flappy-bird\
├── assets\                    # 游戏资源（图片、音效、字体）
├── scenes\                   # 游戏场景和脚本
│   ├── bird.tscn            # 小鸟场景
│   ├── bird.gd             # 小鸟控制脚本
│   ├── main.tscn           # 主场景
│   ├── main.gd            # 主场景控制脚本
│   ├── pipes.tscn         # 管道组场景
│   ├── pipes.gd          # 管道控制脚本
│   ├── bg.tscn           # 背景场景
│   ├── bg.gd            # 背景滚动脚本
│   ├── hud.tscn         # HUD界面
│   └── hud.gd          # HUD控制脚本
├── game_manager\
│   └── game_manager.gd    # 游戏状态管理器
├── addons\                # 编辑器插件
└── project.godot         # 项目配置文件
```

## 关键代码分析

### 小鸟控制 (`bird.gd`)

```gdscript
extends CharacterBody2D

const JUMP_VELOCITY = -500.0  # 向上飞行的瞬时速度
const GRAVITY = 1500          # 模拟重力加速度

func _physics_process(delta: float) -> void:
	if not is_dead:
		velocity.y += GRAVITY * delta
		
		# 关键点：使用 just_pressed 而非 pressed
		if Input.is_action_just_pressed("fly"):
			velocity.y = JUMP_VELOCITY  # 直接设置速度，而非累加
			fly_sound.stream = WING
			fly_sound.play()
		
		# 根据速度计算旋转角度
		rot_degree = clampf(-30.0 * velocity.y/JUMP_VELOCITY, -30.0, 30.0)
		rotation_degrees = rot_degree
		velocity.y = clampf(velocity.y, -max_speed, max_speed)
		move_and_slide()
```

### 游戏管理器 (`game_manager.gd`)

```gdscript
extends Node

signal GameStart
signal GameOver
signal UpdateScore

var score: int = 0

func add_score():
	score += 1

func on_game_over():
	score = 0
```

### 管道生成与回收 (`pipes.gd`)

```gdscript
extends Node2D
class_name PipesScene

const SPEED: float = -150 # 管道向左移动速度

func _process(delta: float) -> void:
	position.x += delta * SPEED

func _on_visible_on_screen_notifier_2d_screen_exited() -> void:
	queue_free()  # 离开屏幕后自动销毁
```

### 背景滚动 (`bg.gd`)

```gdscript
extends Parallax2D

@export var speed: float = -150.0  # 可导出变量，便于编辑器调整

func _process(delta: float) -> void:
	scroll_offset.x += speed * delta
```

## 踩坑记录

### 1. `just_pressed` vs `pressed` 的区别

> [!warning] 重要区别
> 这是本项目中最关键的学习点之一，通过提交历史可以清楚地看到这个问题的解决过程。

**问题描述**：
在最初的实现中，使用了 `Input.is_action_pressed("fly")` 来检测跳跃输入：

```gdscript
if Input.is_action_pressed("fly"):
	velocity.y += JUMP_VELOCITY  # 给一个向上的速度
```

**问题**：
- `pressed` 会在**每一帧**都触发，导致玩家按住空格键时小鸟会持续获得向上的速度
- 这会导致小鸟飞行高度不可控，游戏体验极差
- `velocity.y += JUMP_VELOCITY` 的累加方式会使速度不断增大

**解决方案**：
修改为 `Input.is_action_just_pressed("fly")` 并直接设置速度：

```gdscript
if Input.is_action_just_pressed("fly"):
	velocity.y = JUMP_VELOCITY  # 直接设置速度
```

**核心区别**：
| 方法 | 触发时机 | 适用场景 |
|------|----------|----------|
| `is_action_pressed()` | 动作被按住的**每一帧**都返回 `true` | 需要持续检测的输入（如移动、射击连发） |
| `is_action_just_pressed()` | 只在动作**刚刚被按下**的那一帧返回 `true` | 需要单次触发的输入（如跳跃、射击单发） |

**提交记录证明**：
```
commit ec9768e: 使用 Input.is_action_pressed("fly")
commit dbdb9c2: 修改为 Input.is_action_just_pressed("fly")
提交信息: "修改跳跃检测为 just_pressed 以避免连续触发，并直接设置速度而非累加"
```

### 2. 速度设置方式

**问题**：最初使用 `velocity.y += JUMP_VELOCITY` 会导致速度累加
**解决**：改为 `velocity.y = JUMP_VELOCITY` 直接设置瞬时速度

### 3. 背景速度可配置化

**学习点**：将背景滚动速度定义为 `@export` 变量

```gdscript
@export var speed: float = -150.0
```

这样可以在 Godot 编辑器中直接调整数值，无需修改代码，提高了迭代效率。

### 4. 对象自动回收

**学习点**：使用 `VisibleOnScreenNotifier2D` 节点自动回收离开屏幕的管道

```gdscript
func _on_visible_on_screen_notifier_2d_screen_exited() -> void:
	queue_free()
```

避免了手动管理对象生命周期，减少了内存泄漏的风险。

### 5. 信号通信模式

**学习点**：使用 Godot 的信号系统进行组件间通信

```gdscript
# 定义信号
signal GameStart
signal GameOver
signal UpdateScore

# 发射信号
GameManager.GameOver.emit()

# 连接信号
GameManager.GameOver.connect(on_game_over)
```

这种松耦合的设计使得游戏状态管理更加清晰。

### 6. 动画等待机制

**学习点**：使用 `await` 等待动画完成后再销毁对象

```gdscript
animation_player.play("coin")
await animation_player.animation_finished
coin.queue_free()
```

确保了动画播放完整，提升了游戏体验。

### 7. 触屏输入支持

> [!info] 新增功能
> 在最新版本中，项目添加了触摸屏输入支持，使游戏可以在移动设备上运行。

**实现方式**：在 `bird.gd` 中动态添加 `InputEventScreenTouch` 事件到 "fly" 动作

```gdscript
func _setup_touch_input() -> void:
    # 动态添加触摸支持，确保手机浏览器上点屏幕能跳
    if InputMap.has_action("fly"):
        var touch_event = InputEventScreenTouch.new()
        touch_event.index = -1 # -1 通常用于匹配任意手指
        touch_event.pressed = true # 只有按下时才触发
        # 注意：在 Godot 4 中，InputMap 的匹配通常比较严格。
        # 实际上，最稳妥的方式是依赖 Godot 的 "Emulate Mouse From Touch" (默认开启)。
        # 但为了保险，我们显式添加一个默认的触摸事件。
        InputMap.action_add_event("fly", touch_event)
```

**关键设计点**：

1. **运行时添加输入事件**：
   - 在 `_ready()` 中调用 `_setup_touch_input()`
   - 动态添加触摸事件，而不是在编辑器中静态配置

2. **触摸事件参数**：
   - `index = -1`：匹配任意手指，支持多点触控
   - `pressed = true`：只在触摸按下时触发，避免按住时持续跳跃

3. **多平台兼容性**：
   - Godot 默认开启 "Emulate Mouse From Touch"（触摸模拟鼠标）
   - 添加显式触摸事件作为额外保障

4. **用户提示更新**：
   - HUD界面提示从 "Welcome" 改为 "Tap to make the bird fly"
   - 更清晰地指导移动设备用户如何操作

**提交记录**：
```
commit 00c3ae2: feat: 更新游戏提示并添加触摸屏支持
- 将HUD的欢迎消息改为更具体的操作提示"点击让小鸟飞起来"
- 在bird.gd中添加触摸屏输入支持，确保在手机浏览器上点击屏幕也能控制小鸟跳跃
```

**输入配置总结**：
| 输入类型 | 事件类 | 触发条件 | 设备 |
|----------|--------|----------|------|
| 键盘 | `InputEventKey` | 空格键 (keycode: 32) | PC |
| 鼠标 | `InputEventMouseButton` | 左键点击 (button_index: 1) | PC |
| 触摸屏 | `InputEventScreenTouch` | 屏幕触摸 (index: -1) | 移动设备 |

## 学习总结

### 核心技术点

1. **物理运动**：使用 `CharacterBody2D` + `move_and_slide()` 实现物理运动
2. **输入处理**：区分 `just_pressed`（单次触发）和 `pressed`（持续触发）
3. **多平台输入**：支持键盘、鼠标和触摸屏输入，适配PC和移动设备
4. **场景管理**：模块化场景设计，各司其职
5. **信号系统**：使用信号实现松耦合的组件通信
6. **性能优化**：自动回收离开屏幕的对象，避免内存泄漏
7. **编辑器集成**：使用 `@export` 变量提高开发效率

### 最佳实践

1. **输入处理**：对于跳跃类操作，始终使用 `just_pressed`
2. **速度控制**：直接设置速度值，而非累加速度
3. **资源管理**：离开屏幕的对象及时销毁
4. **代码组织**：使用信号而非直接函数调用进行组件通信
5. **开发效率**：将可调参数暴露为 `@export` 变量

### 扩展思考

1. **游戏难度调整**：可以通过修改管道生成间隔、移动速度等参数调整难度
2. **特效增强**：可以添加粒子系统、屏幕震动等增强游戏反馈
3. **多平台支持**：✅ 已完成 - 已添加触摸输入支持，适配移动设备
4. **数据持久化**：可以添加高分记录功能

## 参考资料

1. [Godot 官方文档：Input 类](https://docs.godotengine.org/en/stable/classes/class_input.html)
2. [Godot 官方文档：CharacterBody2D](https://docs.godotengine.org/en/stable/classes/class_characterbody2d.html)
3. [Godot 官方文档：信号系统](https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html)
4. [Git 提交记录分析](./.git/logs/HEAD)

---

> [!note] 学习状态
> 本项目已完成核心功能开发，掌握了 Godot 基础开发流程和关键编程技巧。建议下一步可以尝试添加更多游戏特性，如粒子特效、音效系统、关卡设计等。

*最后更新：2026年1月25日*