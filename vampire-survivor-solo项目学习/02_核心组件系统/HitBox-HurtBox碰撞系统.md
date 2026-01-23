# HitBox 与 HurtBox 碰撞系统

## 概念

### HitBox（攻击盒）
- 定义伤害源
- 挂在投射物、技能、敌人本体上
- 携带伤害数值和攻击来源信息

### HurtBox（受击盒）
- 定义受击判定区域
- 挂在玩家、敌人身上
- 检测 HitBox 进入并结算伤害

## HitBoxComponent 代码

```gdscript
extends Area2D
class_name HitBoxComponent

signal hit(target: Node2D)

var damage: float = 0
var source_ability_id: StringName = &""
var source_variant: StringName = &""
var _ignored_targets: Array[Node2D] = []

# 添加忽略目标（防止重复伤害）
func add_ignored_target(target: Node2D) -> void:
    if target == null or _ignored_targets.has(target):
        return
    _ignored_targets.append(target)

# 检查是否应忽略该目标
func should_ignore_target(target: Node2D) -> bool:
    for i in range(_ignored_targets.size() - 1, -1, -1):
        var t := _ignored_targets[i]
        if not is_instance_valid(t):
            _ignored_targets.remove_at(i)  # 清理无效引用
            continue
        if t == target:
            _ignored_targets.remove_at(i)
            return true
    return false

func on_hit(target: Node2D = null):
    hit.emit(target)
```

## HurtBoxComponent 代码

```gdscript
extends Area2D
class_name HurtBoxComponent

signal hit

@export var health_component: HealthComponent

var floating_text_scene = preload("res://scenes/UI/floating_text.tscn")

func _on_area_entered(area: Area2D) -> void:
    # 只响应 HitBoxComponent
    if not area is HitBoxComponent:
        return
    if health_component == null:
        return

    var hitbox_component: HitBoxComponent = area as HitBoxComponent

    # 检查是否应忽略（防止重复伤害）
    if hitbox_component.should_ignore_target(self):
        return

    # 发送全局伤害事件
    GameEvents.emit_damage_dealt(
        hitbox_component.damage,
        hitbox_component.source_ability_id,
        hitbox_component.get_parent(),
        _resolve_enemy_root(),
        hitbox_component,
        self
    )

    # 扣血
    health_component.damage(hitbox_component.damage)

    # 生成伤害飘字
    var floating_text = floating_text_scene.instantiate() as FloatingText
    get_tree().get_first_node_in_group("foreground_layer").add_child(floating_text)
    floating_text.global_position = global_position + (Vector2.UP * 16)

    var format_string = "%0.1f"
    if round(hitbox_component.damage) == hitbox_component.damage:
        format_string = "%0.0f"
    floating_text.start(format_string % hitbox_component.damage)

    # 双方通知
    hit.emit()
    hitbox_component.on_hit(self)

# 向上查找实体根节点
func _resolve_enemy_root() -> Node:
    var node: Node = self
    while node != null:
        if node is Node2D and node.is_in_group("enemy"):
            return node
        node = node.get_parent()
    return owner
```

## 交互流程图

```
HitBox进入HurtBox区域
        ↓
    HurtBox检测到area_entered
        ↓
    检查是否HitBox类型
        ↓
    检查是否应忽略（防重复）
        ↓
    发送全局事件（可选）
        ↓
    调用HealthComponent.damage()
        ↓
    生成伤害飘字
        ↓
    发送hit信号 / 调用on_hit()
```

## 防重复伤害机制

```gdscript
# HitBox维护忽略列表
# 命中后将被攻击者加入忽略列表
# 后续碰撞检查时跳过
```

**为什么需要？**
- 持续伤害技能（如火焰）会持续触发碰撞
- 单次伤害技能可能在一帧内多次判定

## 飘字系统

```gdscript
# 添加到 foreground_layer 确保显示在最上层
var foreground = get_tree().get_first_node_in_group("foreground_layer")
foreground.add_child(floating_text)

# 位置偏移：击中部位向上偏移
floating_text.global_position = global_position + (Vector2.UP * 16)

# 格式化为整数或小数
var format_string = "%0.1f"
if round(damage) == damage:
    format_string = "%0.0f"
```

## 最佳实践

1. **信号分离** - hit 用于受击反馈，on_hit 用于攻击方确认
2. **忽略列表管理** - 及时清理无效引用
3. **全局事件** - 通过 GameEvents 解耦伤害统计
4. **飘字层级** - 使用独立层级确保可见性
