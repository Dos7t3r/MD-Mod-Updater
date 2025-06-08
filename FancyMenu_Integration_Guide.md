# FancyMenu 集成配置指南

## 概述
MD-Updater 模组通过按键绑定（默认U键）触发更新检查功能。要在主菜单添加"检查更新"按钮，需要配合 FancyMenu 模组使用。

## 前置要求
1. 安装 FancyMenu 模组（适用于 Minecraft 1.20.1 的版本）
2. 确保 MD-Updater 模组已正确安装

## 配置方法一：使用 FancyMenu 内置编辑器

### 步骤：
1. 启动游戏，进入主菜单
2. 点击顶部菜单栏的 "FM" 按钮（FancyMenu 菜单）
3. 选择 "启用当前界面自定义"
4. 右键背景打开菜单，选择 "新建元素 -> 按钮 (Button)"
5. 将新按钮拖动到合适位置（建议主菜单右下角）
6. 编辑按钮属性：
   - 显示文本：设置为 "检查更新"
   - 动作类型：选择 "按键 (Press Key)"
   - 按键：选择 U 键
7. 调整按钮外观（字体、颜色、大小）
8. 保存配置并退出编辑模式

## 配置方法二：手动编辑配置文件

### 配置文件位置：
`.minecraft/config/fancymenu/layouts/`

### 示例配置（添加到主菜单布局文件中）：
```json
{
  "elements": [
    {
      "id": "update_button",
      "type": "button",
      "text": "检查更新",
      "action": {
        "type": "PRESS_KEY",
        "key": "key.keyboard.u"
      },
      "position": { "x": 0.8, "y": 0.9 },
      "size": { "width": 80, "height": 20 },
      "alignment": { "anchor": "TOP_LEFT" }
    }
  ]
}
```

## 使用说明
1. 配置完成后，主菜单将显示"检查更新"按钮
2. 点击按钮将触发 MD-Updater 的更新检查功能
3. 如果发现缺失模组，会弹出确认对话框
4. 确认下载后，模组将在后台下载到 mods 文件夹
5. 下载完成后需要重启游戏以加载新模组

## 注意事项
- 确保服务器上的 mod_list.json 文件可访问
- 按钮位置应避免与其他UI元素冲突
- 建议在整合包分发时一并提供 FancyMenu 配置文件

