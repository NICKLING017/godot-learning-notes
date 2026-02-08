# MovementComponent 移动物理组件

> 学习来源：cat-knight 项目

## 概述

MovementComponent 是一个专门处理移动逻辑的组件，与状态机配合使用，将移动行为从角色中分离出来。

## 核心属性

```gdscript
class_name MovementComponent
extends Node

@export var max_speed: float = 200.0      # 最大速度
@export var acceleration: float = 50.0    # 加速度
@export var friction: float = 1200.0       # 摩擦力

var velocity: Vector2 = Vector2.ZERO      # 当前速度
```

## 核心方法

### accelerate_in_direction

沿指定方向加速：

```gdscript
func accelerate_in_direction(direction: Vector2, delta) -> void:
    var desired_velocity := direction * max_speed
    # 使用指数插值实现平滑加速
    velocity = velocity.lerp(desired_velocity, 1.0 - exp(-acceleration * delta))
```

**关键点**：使用指数衰减公式实现平滑加速，比线性插值更自然。

### move

执行移动：

```gdscript
func move(character_body: CharacterBody2D) -> void:
    character_body.velocity = velocity
    character_body.move_and_slide()
    # 同步实际速度（考虑碰撞后的速度变化）
    velocity = character_body.velocity
```

## 与状态机配合

```gdscript
# MoveState 中的使用
func update(delta: float) -> void:
    var direction = player.get_input_direction()
    player.movement_component.accelerate_in_direction(direction, delta)
    player.movement_component.move(player)
```

```gdscript
# DashState 中的使用
func enter() -> void:
    player.movement_component.velocity = _dash_direction * dash_speed

func update(_delta: float) -> void:
    player.movement_component.velocity = _dash_direction * dash_speed
    player.movement_component.move(player)

func exit() -> void:
    player.movement_component.velocity = Vector2.ZERO
```

## 设计优势

1. **解耦**：移动逻辑与角色逻辑分离
2. **复用**：多个状态可以共用同一个 MovementComponent
3. **统一管理**：速度、加速度等参数集中管理

## 相关笔记

- [[状态机架构]] - 使用 MovementComponent 的状态机系统
