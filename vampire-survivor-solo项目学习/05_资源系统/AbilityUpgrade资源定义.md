# AbilityUpgrade 资源定义

## 升级资源结构

```gdscript
# AbilityUpgrade.gd
class_name AbilityUpgrade
extends Resource

@export var id: StringName                      # 唯一标识
@export var display_name: String                # 显示名称
@export var description: String                 # 描述文本
@export var icon: Texture2D                     # 图标
@export var max_quantity: int = 1               # 最大选择次数
@export var rarity: int = 1                     # 稀有度 (1=普通, 2=稀有, 3=传说)
@export var affected_ability: StringName = &""  # 关联技能ID
@export var effect_values: Dictionary = {}      # 效果数值 { "damage": 5, "cooldown": -0.1 }
```

## 升级资源示例

### 伤害提升

```
sword_damage.tres
├── id: &"sword_damage"
├── display_name: "锋利"
├── description: "剑攻击力+5"
├── max_quantity: 5
├── rarity: 1
└── effect_values: { "damage_bonus": 5 }
```

### 冷却缩减

```
sword_cooldown.tres
├── id: &"sword_cooldown"
├── display_name: "迅捷"
├── description: "剑冷却-10%"
├── max_quantity: 3
├── rarity: 2
└── effect_values: { "cooldown_reduction": 0.1 }
```

### 范围扩大

```
axe_range.tres
├── id: &"axe_range"
├── display_name: "回旋"
├── description: "斧头范围+20%"
├── max_quantity: 3
├── rarity: 2
└── effect_values: { "range_bonus": 0.2 }
```

## 升级资源配置

```
Resources/
└── upgrades/
    ├── sword_damage.tres
    ├── sword_cooldown.tres
    ├── sword_range.tres
    ├── axe_damage.tres
    ├── axe_cooldown.tres
    ├── knife_speed.tres
    └── ...
```

## Controller 读取升级效果

```gdscript
# SwordAbilityController.gd
func _on_ability_upgrade_added(upgrade: AbilityUpgrade, _current_upgrades: Dictionary) -> void:
    if upgrade.id == &"sword_damage":
        damage += upgrade.effect_values.get("damage_bonus", 5)
    elif upgrade.id == &"sword_cooldown":
        cooldown *= (1.0 - upgrade.effect_values.get("cooldown_reduction", 0.1))
    elif upgrade.id == &"sword_range":
        max_range *= (1.0 + upgrade.effect_values.get("range_bonus", 0.2))
```

## 升级应用流程

```
升级选择
        ↓
UpgradeManager.select_upgrade(upgrade)
        ↓
发送 GameEvents.ability_upgrade_added(upgrade, current_upgrades)
        ↓
各 AbilityController 监听此信号
        ↓
根据 upgrade.id 和 effect_values 修改属性
        ↓
后续使用技能时应用新数值
```

## 资源设计模式

### 1. 数据驱动

```gdscript
# 不写死在代码里
damage += upgrade.effect_values["damage_bonus"]

# 而是读取资源配置
```

### 2. 可扩展性

```gdscript
# 新增升级类型，无需修改 Controller 代码
func _on_ability_upgrade_added(upgrade: AbilityUpgrade, _current_upgrades: Dictionary) -> void:
    var effect = upgrade.effect_values
    if effect.has("damage"):
        damage += effect["damage"]
    if effect.has("cooldown"):
        cooldown *= (1 - effect["cooldown"])
    # 新的效果类型自动支持
```

## 最佳实践

1. **唯一 ID** - 使用 StringName (&) 保证类型安全
2. **效果字典** - effect_values 存储可配置的效果值
3. **默认参数** - get() 提供默认值避免崩溃
4. **可读命名** - 资源文件名与 id 保持一致
