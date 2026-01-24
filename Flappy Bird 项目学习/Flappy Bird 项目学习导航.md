---
title: Flappy Bird 项目学习导航
date: 2026-01-24
tags:
  - godot
  - gdscript
  - flappy-bird
  - 学习笔记
  - canvas
---

# Flappy Bird 项目学习导航

## 两种格式的学习笔记

我为您准备了两种格式的Flappy Bird项目学习笔记，您可以根据需要选择使用：

### 1. Canvas格式（视觉化）
**文件**：[[Flappy Bird 项目学习.canvas]]

**特点**：
- 视觉化布局，便于快速浏览
- 节点间有连接线展示关系
- 颜色编码区分不同内容类型
- 适合脑图式学习和展示

**包含内容**：
- 项目概览（中央节点）
- 项目结构（左侧）
- 关键代码分析（右侧）
- 踩坑记录（底部）
- 学习总结（右上角）
- 核心洞见（底部中央）

**打开方式**：在Obsidian中双击 `.canvas` 文件即可打开画布视图。

### 2. Markdown格式（传统）
**文件**：[[Flappy Bird 项目学习笔记]]

**特点**：
- 详细文字描述，适合深入学习
- 完整的代码示例和解释
- 包含Git提交历史分析
- 适合打印或文本搜索

**包含内容**：
- 完整的项目结构分析
- 关键代码逐行解析
- 详细的踩坑记录（含Git提交证明）
- 学习总结与最佳实践
- 参考资料

## 学习路径建议

### 快速了解（5分钟）
1. 打开 [[Flappy Bird 项目学习.canvas]]
2. 浏览各节点，了解项目概况
3. 重点关注"踩坑记录"节点

### 深入学习（30分钟）
1. 阅读 [[Flappy Bird 项目学习笔记]]
2. 查看代码示例，理解实现细节
3. 学习Git提交历史中的问题解决过程

### 实践练习
1. 打开原项目：`C:\Godot\flappy-bird`
2. 尝试修改 `@export` 变量，观察效果
3. 将 `just_pressed` 改回 `pressed`，体验问题
4. 添加新的游戏特性

## 核心知识点

### 必须掌握的
- ✅ `just_pressed` 与 `pressed` 的区别
- ✅ `CharacterBody2D` 物理运动
- ✅ Godot 信号系统

### 推荐的
- ⭐ `@export` 变量的使用
- ⭐ `VisibleOnScreenNotifier2D` 自动回收
- ⭐ 模块化场景设计

### 拓展的
- 🔧 粒子系统添加
- 🔧 音效系统完善
- 🔧 高分记录功能

## 常见问题

### Q: Canvas文件打不开？
A: 确保使用 Obsidian 1.4+ 版本。如果无法打开，可以：
1. 使用文本编辑器查看JSON结构
2. 将文件重命名为 `.canvas.txt` 查看内容
3. 在Obsidian设置中检查是否启用canvas功能

### Q: 如何修改Canvas布局？
A: 在Obsidian中可以直接拖拽节点，或编辑JSON文件调整 `x`, `y`, `width`, `height` 参数。

### Q: 两种格式如何选择？
A: 
- **教学/展示** → Canvas格式
- **查阅/搜索** → Markdown格式  
- **代码参考** → Markdown格式
- **思维整理** → Canvas格式

## 相关链接

### 项目文件
- [[.git/COMMIT_EDITMSG]] - Git提交历史
- [[project.godot]] - Godot项目配置

### 关键脚本
- [[scenes/bird.gd]] - 小鸟控制
- [[scenes/main.gd]] - 主场景控制  
- [[scenes/pipes.gd]] - 管道生成
- [[game_manager/game_manager.gd]] - 游戏管理器

### 学习资源
- [Godot官方文档](https://docs.godotengine.org/)
- [JSON Canvas规范](https://jsoncanvas.org/)
- [Obsidian帮助文档](https://help.obsidian.md/)

---

> [!note] 更新日志
> - **2026-01-24**: 创建两种格式的学习笔记
> - **Canvas格式**: 视觉化展示项目结构
> - **Markdown格式**: 详细文字分析

> [!tip] 学习建议
> 建议先浏览Canvas获得整体认识，再深入阅读Markdown文档学习细节，最后动手修改代码实践。