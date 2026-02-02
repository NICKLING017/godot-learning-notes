---
title: battle-tank-demo 项目学习笔记
date: 2026-02-01
tags:
  - godot
  - game-dev
  - battle-tank-demo
  - project-notes
category: Godot 项目学习
updated: 2026-02-03
aliases:
  - battle-tank-demo学习笔记
  - battle-tank-demo项目笔记
---

# battle-tank-demo 项目学习笔记

> 基于 Git 提交记录的更新：2026年2月

## 项目概述

基于 Godot 4 的坦克对战游戏demo，实现了玩家控制、敌人AI、子弹系统、碰撞检测和UI显示等核心机制。

---

## Git 提交历史

### 2026-02-03: feat: 添加升级参数并调整游戏平衡性

**优化内容：**
- **参数默认值**：为武器和生命值升级方法添加默认参数，提高代码灵活性
- **视觉效果**：启用闪光着色器效果，增强视觉反馈
- **子弹系统**：设置子弹速度，区分敌我子弹速度差异
- **道具系统**：为拾取物品添加类型标识，便于游戏逻辑处理
- **地图优化**：优化地图物品布局，减少冗余元素
- **轨迹特效**：为玩家子弹添加轨迹效果，提升视觉表现

**修改文件：**
- `components/health_component.gd`
- `components/weapon_component.gd`
- `scenes/bullet/enemy_bullet.tscn`
- `scenes/bullet/player_bullet.tscn`
- `scenes/map/map.tscn`
- `scenes/pick_up/pick_up_gun.tscn`
- `scenes/pick_up/pick_up_health.tscn`
- `scenes/player/player.gd`
- `tres/flash.tres`

---

### 2026-02-03: feat: 重构游戏系统为组件化架构并添加新功能

**架构升级：**
- **组件化系统**：引入可复用组件架构
  - `HealthComponent`：生命值管理组件
  - `WeaponComponent`：武器系统组件
  - `HitBoxComponent`：受击框组件
  - `HurtBoxComponent`：伤害框组件
  - `DetectComponent`：检测组件
  - `TrailComponent`：轨迹特效组件

- **子弹系统重构**：
  - 创建通用 `BulletBase` 类，统一玩家和敌人子弹逻辑
  - 移除重复的 `player_bullet.gd` 和 `enemy_bullet.gd`
  - 新增 `bullet_trail` 子弹拖尾效果

- **管理器扩展**：
  - `EnemyManager`：敌人生成与管理
  - `PickupManager`：道具拾取管理
  - 增强 `VFXManager` 和 `GameManager` 功能

**新增内容：**
- **敌人类型**：
  - `EnemyTank`：移动坦克敌人
  - `EnemyTower`：固定炮塔敌人

- **道具系统**：
  - `PickUp`：基础拾取道具
  - `PickUpGun`：武器道具
  - `PickUpHealth`：生命恢复道具

- **视觉效果**：
  - 坦克履带轨迹 (`tank_trail`)
  - 子弹拖尾效果 (`bullet_trail`)
  - 玩家受击闪烁 Shader
  - 爆炸粒子效果 (`exp_particle`)

- **游戏主场景**：新增完整 `game.tscn`
- **UI系统**：扩展 `hud.tscn`，新增 `font35.tres` 和 `theme.tres`

---

### 2026-02-03: docs: 添加 monitoring 属性与 set_deferred 安全设置笔记

**新增笔记：**
- **monitoring 属性与 set_deferred 安全设置**
  - 详解 Area2D 碰撞检测开关 monitoring 的使用场景
  - 讲解 set_deferred() 延迟设置属性的原理和时机
  - 对比单次命中 vs 穿透群伤策略
  - 完整子弹脚本和对象池管理器示例

**内容更新：**
- 完善 EnemyBullet-子弹运动笔记，添加碰撞优化章节

---

### 2026-02-02: docs: 完善坦克项目笔记格式与内容

**格式优化：**
- **Enemy-敌人AI.md**：添加 frontmatter 和 Obsidian callout 格式
- **Player-玩家控制.md**：添加 frontmatter、组件说明和最佳实践总结
- **battle-tank-demo项目学习.canvas**：整理项目结构

---

## 项目结构

```
battle-tank-demo/
├── assets/                   # 资源文件
│   ├── sound/               # 音效
│   ├── VFX/                 # 视觉特效
│   ├── tank/                # 坦克素材
│   └── enemy/               # 敌人素材
├── components/              # 组件化系统
│   ├── health_component.gd  # 生命值组件
│   ├── weapon_component.gd  # 武器组件
│   ├── hit_box_component.gd # 受击框组件
│   ├── hurt_box_component.gd# 伤害框组件
│   ├── detect_component.gd  # 检测组件
│   └── trail_component.gd   # 轨迹组件
├── manager/                 # 管理器
│   ├── gamemanager.gd       # 游戏管理器
│   ├── enemy_manager.gd     # 敌人管理器
│   ├── pickup_manager.gd    # 道具管理器
│   └── vfx_manager.gd       # 特效管理器
├── scenes/                  # 游戏场景
│   ├── player/              # 玩家
│   ├── enemy/               # 敌人
│   ├── bullet/              # 子弹系统
│   │   ├── bullet_base.gd   # 通用子弹基类
│   │   ├── player_bullet.tscn
│   │   └── enemy_bullet.tscn
│   ├── bullet_trail/        # 子弹轨迹
│   ├── enemy_tank/          # 坦克敌人
│   ├── enemy_tower/         # 炮塔敌人
│   ├── pick_up/             # 道具系统
│   ├── tank_trail/          # 坦克轨迹
│   ├── game/                # 主游戏场景
│   ├── map/                 # 地图
│   ├── ui/                  # UI界面
│   │   ├── hud.gd           # HUD控制器
│   │   └── hud.tscn         # HUD界面
│   ├── vfx/                 # 特效
│   │   └── exp_anim.tscn    # 爆炸动画
│   └── exp_particle/        # 爆炸粒子
├── tres/                    # 资源文件
│   ├── map.tres             # 地图瓦片集
│   ├── theme.tres           # 主题样式
│   ├── flash.tres           # 闪光效果
│   ├── panel.tres           # 面板样式
│   └── font35.tres          # 字体资源
└── project.godot
```

---

## 组件化架构详解

### HealthComponent 生命值组件

```gdscript
class_name HealthComponent
extends Node

@export var max_health: float = 100.0
@export var current_health: float = 100.0
@export var default_damage: float = 10.0

signal health_changed(new_health: float)
signal health_depleted()

func take_damage(amount: float):
    current_health -= amount
    health_changed.emit(current_health)
    if current_health <= 0:
        health_depleted.emit()

func heal(amount: float):
    current_health = min(current_health + amount, max_health)
    health_changed.emit(current_health)
```

### WeaponComponent 武器组件

```gdscript
class_name WeaponComponent
extends Node

@export var bullet_scene: PackedScene
@export var fire_rate: float = 0.5
@export var bullet_speed: float = 300.0
@export var damage: float = 10.0
@export var is_player_bullet: bool = true

var _can_fire: bool = true
var _fire_cooldown: float = 0.0

func fire():
    if not _can_fire:
        return

    var bullet = bullet_scene.instantiate()
    # 配置子弹参数...
    get_parent().get_parent().add_child(bullet)

    _start_cooldown()

func _start_cooldown():
    _can_fire = false
    await get_tree().create_timer(fire_rate).timeout
    _can_fire = true
```

### TrailComponent 轨迹组件

```gdscript
class_name TrailComponent
extends Node2D

@export var trail_color: Color = Color.WHITE
@export var trail_width: float = 2.0
@export var trail_length: int = 20
@export var fade_speed: float = 2.0

var _points: Array[Vector2] = []

func _process(delta: float):
    _points.append(get_parent().global_position)
    if _points.size() > trail_length:
        _points.pop_front()
    queue_redraw()

func _draw():
    for i in range(1, _points.size()):
        draw_line(_points[i-1], _points[i], trail_color, trail_width)
```

---

## 碰撞层配置

| 层名称 | 层号 | 用途 |
|--------|------|------|
| Player | 1 | 玩家实体 |
| Enemy | 2 | 敌人实体 |
| PlayerBullet | 4 | 玩家子弹 |
| EnemyBullet | 8 | 敌人子弹 |
| Wall | 16 | 墙壁障碍 |
| Pickup | 32 | 道具拾取物 |

---

> [!tip] 学习要点
> 1. **组件化设计**：将通用功能提取为可复用组件
> 2. **管理器模式**：使用管理器处理全局逻辑
> 3. **资源管理**：善用 .tres 资源文件管理样式
> 4. **视觉效果**：Shader 和粒子特效提升游戏体验
> 5. **代码复用**：通用子弹基类减少重复代码

---

## 相关笔记

- [[Player-玩家控制]]
- [[Enemy-敌人AI]]
- [[EnemyBullet-子弹运动]]
- [[Transform变换详解]]
- [[monitoring属性与set_deferred安全设置]]
- [[游戏核心机制与UI]]

---

*最后更新：基于最新提交 e9db56a*

%% 
内部笔记：
- 考虑添加组件化架构设计模式详解
- 补充 Shader 编写教程
- 添加道具系统实现笔记
%%
