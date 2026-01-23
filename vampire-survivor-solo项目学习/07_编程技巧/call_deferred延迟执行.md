# call_deferred 延迟执行

## 什么是 call_deferred

将函数调用推迟到当前帧结束、下一次 `_process` 之前执行。

```gdscript
# 立即执行
func damage(amount: float):
    current_health -= amount

# 延迟执行
func damage(amount: float):
    Callable(check_death).call_deferred()
```

## 为什么需要延迟

### 问题：信号链中的销毁

```gdscript
# HealthComponent.gd
func damage(amount: float) -> void:
    current_health -= amount
    if current_health <= 0:
        died.emit()
        owner.queue_free()  # 立即销毁

# Enemy.gd
func _ready() -> void:
    $HealthComponent.died.connect(_on_died)

func _on_died() -> void:
    # 此时 owner 已经被销毁
    print(get_parent().name)  # 可能出错！
    spawn_loot()  # 可能失败！
```

### 解决方案：延迟检查

```gdscript
# HealthComponent.gd
func damage(amount: float) -> void:
    current_health = clamp(current_health - amount, 0, max_health)
    health_changed.emit()
    if amount > 0:
        health_decreased.emit()

    # 延迟检查死亡
    Callable(check_death).call_deferred()

func check_death() -> void:
    if current_health == 0:
        died.emit()
        owner.queue_free()  # 这次销毁发生在下一帧
```

## 执行时机对比

```
当前帧
├─ damage() 调用
│  ├─ current_health 修改
│  ├─ health_changed.emit()
│  ├─ health_decreased.emit()
│  └─ Callable(check_death).call_deferred()
│
下一帧开始
├─ _process(delta)
├─ 渲染
└─ 延迟函数执行
   └─ check_death()
      ├─ died.emit()
      └─ owner.queue_free()
```

## 使用场景

### 1. 销毁前清理

```gdscript
func cleanup() -> void:
    # 断开所有连接
    for connection in get_signal_connection_list("some_signal"):
        disconnect(...)

func _on_health_zero() -> void:
    Callable(cleanup).call_deferred()
    queue_free()
```

### 2. 避免重复处理

```gdscript
var is_processing: bool = false

func on_event() -> void:
    if is_processing:
        return
    is_processing = true
    Callable(_process_event).call_deferred()

func _process_event() -> void:
    # 处理逻辑
    is_processing = false
```

### 3. UI 交互

```gdscript
func show_dialog() -> void:
    $DialogPanel.visible = true
    Callable(_grab_focus).call_deferred()

func _grab_focus() -> void:
    $DialogPanel/FirstButton.grab_focus()  # 确保UI已渲染
```

### 4. 跨场景引用

```gdscript
func _on_level_complete() -> void:
    # 延迟切换，避免当前场景正在清理时出错
    Callable(_transition_to_next_level).call_deferred()

func _transition_to_next_level() -> void:
    get_tree().change_scene_to_packed(next_level_scene)
```

## 与其他延迟方式对比

| 方式 | 延迟时间 | 适用场景 |
|------|----------|----------|
| `call_deferred` | 下一帧 | 信号链、销毁清理 |
| `await get_tree().process_frame` | 1帧后 | 等待一帧 |
| `await get_tree().create_timer(x).timeout` | x秒后 | 定时延迟 |
| `await signal` | 不确定 | 等待事件 |

## 注意事项

### 1. 参数传递

```gdscript
# 通过 bind 传递参数
Callable(check_death).call_deferred()

# 带参数
Callable(check_with_value).bind(damage_amount).call_deferred()
```

### 2. 返回值

```gdscript
# call_deferred 返回 Callable，不返回值
var result = Callable(func_with_return()).call_deferred()
# result 是空，不是 func_with_return 的返回值
```

### 3. 对象销毁后

```gdscript
# 如果对象在延迟期间被销毁，延迟调用不会执行
func _on_something() -> void:
    queue_free()  # 立即销毁
    Callable(cleanup).call_deferred()  # 不会执行！
```

## 最佳实践

1. **信号链中使用** - 避免在 emit 信号后立即销毁
2. **销毁前清理** - 断开信号连接、释放资源
3. **UI 焦点** - grab_focus 等 UI 操作延迟执行
4. **理解时机** - 明白它是在下一帧 _process 前执行
