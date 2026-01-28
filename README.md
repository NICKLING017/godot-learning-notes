---
title: Godot 游戏开发学习笔记
date: 2026-01-24
tags:
  - godot
  - gdscript
  - 学习笔记
  - 游戏开发
status: active
priority: high
---

# Godot 游戏开发学习笔记

> 多个项目的学习笔记集合，包含详细的项目分析、代码解析和实战总结

## 项目目录

### 🎮 完整项目学习

| 项目 | 类型 | 状态 | 学习重点 |
|------|------|------|----------|
| [[vampire-survivor-solo项目学习/欢迎\| vampire-survivor-solo 项目]] | 吸血鬼幸存者类 | ✅ 已完成 | 组件化设计、技能系统、资源管理 |
| [[Flappy Bird 项目学习/Flappy Bird 项目学习导航\| Flappy Bird 项目]] | 经典休闲游戏 | ✅ 已完成 | 物理运动、信号系统、性能优化 |
| [[battle-tank-demo项目学习/欢迎\| Battle Tank Demo 项目]] | 坦克对战演示 | 🚧 开发中 | Transform概念、AI系统、输入配置 |

### 📚 学习笔记格式

每个项目都提供两种学习格式：

1. **Canvas 格式** - 视觉化思维导图，适合快速浏览和整体把握
2. **Markdown 格式** - 详细文字分析，适合深入学习和代码参考

### 🎯 学习建议

1. **新手入门**：建议从 Flappy Bird 项目开始，了解 Godot 基础开发流程
2. **进阶学习**：学习 vampire-survivor-solo 项目的组件化架构和系统设计
3. **实践练习**：参照笔记中的最佳实践，在自己的项目中应用所学知识

## 快速开始

### Flappy Bird 项目
- **Canvas**: [[Flappy Bird 项目学习/Flappy Bird 项目学习.canvas]]
- **导航**: [[Flappy Bird 项目学习/Flappy Bird 项目学习导航]]
- **笔记**: [[Flappy Bird 项目学习/Flappy Bird 项目学习笔记]]

### Vampire Survivor Solo 项目
- **Canvas**: [[vampire-survivor-solo项目学习/vampire-survivor-solo项目学习.canvas]]
- **导航**: [[vampire-survivor-solo项目学习/vampire-survivor-solo项目学习导航]]
- **欢迎**: [[vampire-survivor-solo项目学习/欢迎]]

### Battle Tank Demo 项目
- **Canvas**: [[battle-tank-demo项目学习/battle-tank-demo项目学习.canvas]]
- **导航**: [[battle-tank-demo项目学习/battle-tank-demo项目学习导航]]
- **欢迎**: [[battle-tank-demo项目学习/欢迎]]

## 核心知识点总结

### 通用技能
- ✅ Godot 场景和节点系统
- ✅ GDScript 编程基础
- ✅ 信号与事件系统
- ✅ 物理引擎使用
- ✅ Transform与坐标系统（局部/全局坐标）

### 架构设计
- ⭐ 组件化设计模式（Composition over Inheritance）
- ⭐ 资源驱动开发（Resource-driven development）
- ⭐ 事件总线模式（Event bus pattern）
- ⭐ 状态管理模式（State management）

### 性能优化
- 🔧 对象池与自动回收
- 🔧 距离计算优化
- 🔧 延迟执行（call_deferred）
- 🔧 资源预加载

## 学习资源

- [Godot 官方文档](https://docs.godotengine.org/)
- [GDScript 语言参考](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript.html)
- [JSON Canvas 规范](https://jsoncanvas.org/)
- [Obsidian 帮助文档](https://help.obsidian.md/)

---

> [!note] 更新日志
> - **2026-01-28**: 整理项目笔记，添加统一的frontmatter元数据，优化格式规范
> - **2026-01-26**: 新增 Battle Tank Demo 项目笔记，重点讲解 Transform 概念
> - **2026-01-24**: 新增 Flappy Bird 项目学习笔记和 Canvas
> - **2026-01-24**: 整理 vampire-survivor-solo 项目笔记，创建导航和 Canvas
> - **2026-01-24**: 更新主 README，提供完整学习路径

> [!tip] 贡献指南
> 欢迎提交新的项目学习笔记！请遵循现有格式，包含 Canvas 和 Markdown 两种格式，确保内容准确、结构清晰。
