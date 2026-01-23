# MetaProgression - 存档与持久化

## 核心功能

管理局外成长数据（永久升级、金币等），实现存档/读档。

## 实现代码

```gdscript
# meta_progression.gd (Autoload)
extends Node

const SAVE_FILE_PATH = "user://game.save"

var save_data: Dictionary = {
    "coins": 0,
    "meta_upgrades": {},        # { "upgrade_id": quantity }
    "unlocked_abilities": [],   # ["sword", "axe", "knife"]
    "play_time": 0,
    "high_score": 0
}

func _ready() -> void:
    load()

func save() -> void:
    var file = FileAccess.open(SAVE_FILE_PATH, FileAccess.WRITE)
    file.store_var(save_data)

func load() -> void:
    if FileAccess.file_exists(SAVE_FILE_PATH):
        var file = FileAccess.open(SAVE_FILE_PATH, FileAccess.READ)
        save_data = file.get_var()
    else:
        save_data = {
            "coins": 0,
            "meta_upgrades": {},
            "unlocked_abilities": [],
            "play_time": 0,
            "high_score": 0
        }

func add_coins(amount: int) -> void:
    save_data["coins"] += amount
    save()

func buy_meta_upgrade(upgrade_id: StringName, cost: int) -> bool:
    if save_data["coins"] < cost:
        return false

    if not save_data["meta_upgrades"].has(upgrade_id):
        save_data["meta_upgrades"][upgrade_id] = 0

    save_data["meta_upgrades"][upgrade_id] += 1
    save_data["coins"] -= cost
    save()
    return true

func get_meta_upgrade_count(upgrade_id: StringName) -> int:
    return save_data["meta_upgrades"].get(upgrade_id, 0)

func update_high_score(score: int) -> void:
    if score > save_data["high_score"]:
        save_data["high_score"] = score
        save()
```

## 数据结构

```gdscript
# save_data 内容示例
{
    "coins": 150,
    "meta_upgrades": {
        "starting_gold": 2,
        "damage_boost": 1,
        "cooldown_reduction": 3
    },
    "unlocked_abilities": ["sword", "axe"],
    "play_time": 3600,
    "high_score": 99999
}
```

## 使用示例

```gdscript
# 获得金币（敌人掉落）
func _on_enemy_killed() -> void:
    var coins = 10
    MetaProgression.add_coins(coins)

# 购买永久升级
func _on_buy_upgrade(upgrade_id: StringName, cost: int) -> void:
    if MetaProgression.buy_meta_upgrade(upgrade_id, cost):
        print("购买成功")

# 局内使用永久升级效果
func _ready() -> void:
    var starting_gold = MetaProgression.get_meta_upgrade_count(&"starting_gold")
    gold = base_gold + starting_gold * 10
```

## 存档位置

| 平台 | 路径 |
|------|------|
| Windows | `%APPDATA%/Godot/app_userdata/[项目名]/game.save` |
| macOS | `~/Library/Application Support/Godot/app_userdata/[项目名]/game.save` |
| Linux | `~/.local/share/godot/app_userdata/[项目名]/game.save` |

## 自动存档策略

```gdscript
# 重要时机自动保存
func _on_player_died() -> void:
    MetaProgression.save()

func _on_game_paused() -> void:
    MetaProgression.save()

# 定期自动保存
func _process(_delta: float) -> void:
    _time_since_last_save += _delta
    if _time_since_last_save > 60:  # 每60秒
        MetaProgression.save()
```

## 数据迁移

```gdscript
func migrate_save_data(old_version: int) -> void:
    if old_version < 2:
        # v2 新增 meta_upgrades 字段
        save_data["meta_upgrades"] = {}
    if old_version < 3:
        # v3 新增 play_time
        save_data["play_time"] = 0
```

## 最佳实践

1. **user://** - 使用 user:// 路径确保跨平台可写
2. **FileAccess** - Godot 4 使用 FileAccess API
3. **版本控制** - 考虑存档版本号
4. **备份** - 重要存档前先备份
5. **错误处理** - 检查文件是否存在
