# UpgradeManager - 升级选择系统

## 核心功能

管理升级选项池、随机抽取、玩家选择和升级应用。

## 核心代码

```gdscript
extends Node

signal upgrade_screen_opened
signal upgrade_screen_closed

@export var upgrade_screens: Array[PackedScene] = []

var available_upgrades: Array[AbilityUpgrade] = []
var current_upgrades: Dictionary = {}  # { upgrade_id: { "quantity": 0, "max_quantity": 3 } }
var _upgrade_pool: Array[AbilityUpgrade] = []

func _ready() -> void:
    GameEvents.experience_vial_collected.connect(_on_experience_collected)

func _on_experience_collected(_amount: float) -> void:
    var experience_manager = get_tree().get_first_node_in_group("experience_manager")
    if experience_manager and experience_manager.is_leveling_up:
        show_upgrade_screen()

func generate_upgrade_options() -> Array[AbilityUpgrade]:
    var options: Array[AbilityUpgrade] = []
    var usable_upgrades = _upgrade_pool.filter(func(u): return can_use_upgrade(u))

    for i in range(3):
        if usable_upgrades.is_empty():
            break
        var random_index = randi() % usable_upgrades.size()
        options.append(usable_upgrades[random_index])
        usable_upgrades.remove_at(random_index)

    return options

func select_upgrade(upgrade: AbilityUpgrade) -> void:
    if not current_upgrades.has(upgrade.id):
        current_upgrades[upgrade.id] = { "quantity": 0, "max_quantity": upgrade.max_quantity }

    current_upgrades[upgrade.id].quantity += 1

    GameEvents.emit_ability_upgrade_added(upgrade, current_upgrades)

func can_use_upgrade(upgrade: AbilityUpgrade) -> bool:
    if current_upgrades.has(upgrade.id):
        return current_upgrades[upgrade.id].quantity < upgrade.max_quantity
    return true

func show_upgrade_screen() -> void:
    var options = generate_upgrade_options()
    var screen = upgrade_screens[0].instantiate()
    screen.upgrades = options
    screen.upgrade_selected.connect(select_upgrade)

    get_tree().root.add_child(screen)
    upgrade_screen_opened.emit()
```

## 升级资源定义

```gdscript
# AbilityUpgrade.gd (Resource)
extends Resource
class_name AbilityUpgrade

@export var id: StringName
@export var display_name: String
@export var description: String
@export var icon: Texture2D
@export var max_quantity: int = 1
@export var rarity: int = 1  # 1=普通, 2=稀有, 3=传说
```

## 升级卡 UI

```gdscript
# ability_upgrade_card.gd
extends Control

signal selected(upgrade: AbilityUpgrade)

@export var upgrade: AbilityUpgrade

func _ready() -> void:
    $Button.pressed.connect(_on_button_pressed)
    $NameLabel.text = upgrade.display_name
    $DescriptionLabel.text = upgrade.description

func _on_button_pressed() -> void:
    selected.emit(upgrade)
```

## 升级流程图

```
经验达到要求
        ↓
ExperienceManager 发送 level_up
        ↓
UpgradeManager 生成3个随机升级选项
        ↓
暂停游戏，显示 UpgradeScreen
        ↓
玩家选择
        ↓
UpgradeManager 应用升级
        ↓
发送 ability_upgrade_added 信号
        ↓
各技能 Controller 响应升级
        ↓
恢复游戏
```

## Controller 响应升级

```gdscript
# SwordAbilityController.gd
func _ready() -> void:
    GameEvents.ability_upgrade_added.connect(_on_ability_upgrade_added)

func _on_ability_upgrade_added(upgrade: AbilityUpgrade, _current_upgrades: Dictionary) -> void:
    if upgrade.id == &"sword_damage":
        damage += 5
    elif upgrade.id == &"sword_cooldown":
        cooldown -= 0.1
```

## 最佳实践

1. **资源驱动** - 升级数据定义为 Resource，便于编辑
2. **信号解耦** - 升级选择与升级应用分离
3. **数量限制** - 支持升级次数限制（max_quantity）
4. **随机池** - 从可用升级池中随机抽取
