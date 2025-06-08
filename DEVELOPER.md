# MD-Updater 开发者文档

## 项目结构

```
MD-Updater/
├── build.gradle                    # Gradle构建脚本
├── gradle.properties              # Gradle属性配置
├── settings.gradle                 # Gradle设置
├── gradlew                        # Gradle Wrapper脚本
├── gradle/wrapper/                # Gradle Wrapper文件
├── src/main/
│   ├── java/com/mdupdater/md_updater/
│   │   ├── MDUpdater.java         # 主模组类
│   │   ├── UpdateChecker.java     # 更新检查逻辑
│   │   ├── UpdatePromptScreen.java # 提示界面
│   │   └── UpdateCompleteScreen.java # 完成界面
│   └── resources/
│       ├── META-INF/mods.toml     # 模组元数据
│       └── assets/md_updater/lang/
│           └── zh_cn.json         # 中文语言文件
├── README.md                      # 使用说明
├── SERVER_SETUP.md               # 服务器配置指南
├── FancyMenu_Integration_Guide.md # FancyMenu集成指南
└── example_mod_list.json         # 示例模组列表
```

## 核心类说明

### MDUpdater.java

主模组类，负责：
- 模组初始化和注册
- 按键绑定管理
- 客户端环境检测
- 更新检查触发

关键方法：
- `MDUpdater()`: 构造函数，注册按键映射
- `registerKeyMappings()`: 注册按键绑定
- `checkForUpdates()`: 静态方法，触发更新检查

### UpdateChecker.java

更新检查和下载逻辑，负责：
- 从服务器获取模组列表
- 比对本地文件
- 下载缺失模组

关键方法：
- `checkUpdates()`: 检查更新逻辑
- `downloadModList()`: 下载JSON列表
- `downloadMissingMods()`: 下载缺失模组

### UpdatePromptScreen.java

用户确认界面，负责：
- 显示发现的缺失模组
- 提供确认/取消选项
- 启动异步下载

### UpdateCompleteScreen.java

完成提示界面，负责：
- 显示下载完成信息
- 提示用户重启游戏

## 自定义配置

### 修改服务器URL

在 `UpdateChecker.java` 中修改：

```java
private static final String MOD_LIST_URL = "https://your-server.com/mod_list.json";
```

### 修改默认按键

在 `MDUpdater.java` 中修改：

```java
public static final KeyMapping CHECK_UPDATE_KEY = new KeyMapping(
    "key.md_updater.check_update",
    InputConstants.Type.KEYSYM,
    GLFW.GLFW_KEY_U,  // 修改这里的按键
    "key.categories.md_updater"
);
```

### 添加新的语言支持

1. 在 `src/main/resources/assets/md_updater/lang/` 目录下创建新的语言文件
2. 例如英文支持：`en_us.json`
3. 修改代码使用 `Component.translatable()` 而不是硬编码字符串

## 扩展功能

### 添加版本比较

当前模组只比较文件名，可以扩展为版本比较：

1. 修改 `mod_list.json` 格式，添加版本字段
2. 在 `UpdateChecker` 中添加版本解析逻辑
3. 实现版本比较算法

示例JSON格式：
```json
{
  "mods": [
    {
      "name": "example-mod.jar",
      "version": "1.2.3",
      "url": "https://server.com/mods/example-mod-1.2.3.jar"
    }
  ]
}
```

### 添加进度显示

为下载过程添加进度条：

1. 修改 `UpdatePromptScreen` 添加进度条组件
2. 在 `downloadMissingMods()` 中报告下载进度
3. 使用线程间通信更新UI

### 添加模组依赖检查

检查模组依赖关系：

1. 扩展JSON格式包含依赖信息
2. 在下载前检查依赖关系
3. 自动下载缺失的依赖模组

### 添加自动更新

实现定时自动检查：

1. 添加配置选项控制自动检查间隔
2. 使用定时器在后台检查更新
3. 在发现更新时显示通知

## 编译和构建

### 开发环境设置

1. 安装 JDK 17
2. 克隆项目到本地
3. 运行 `./gradlew build` 编译

### 调试运行

```bash
# 启动客户端调试
./gradlew runClient

# 启动服务器调试
./gradlew runServer
```

### 生成发布版本

```bash
# 清理并构建
./gradlew clean build

# 生成的jar文件位于 build/libs/
```

## API参考

### Forge事件

模组使用的主要Forge事件：
- `RegisterKeyMappingsEvent`: 注册按键绑定
- `InputEvent.Key`: 处理按键输入

### Minecraft API

使用的主要Minecraft API：
- `Minecraft.getInstance()`: 获取游戏实例
- `Screen`: 自定义GUI界面基类
- `Component`: 文本组件系统

### 文件系统API

- `FMLPaths.MODSDIR.get()`: 获取mods目录路径
- `File` 和 `Path`: 文件操作

## 测试指南

### 单元测试

虽然当前版本没有包含单元测试，但建议为以下组件添加测试：

1. `UpdateChecker` 的JSON解析逻辑
2. 文件比较逻辑
3. URL构建逻辑

### 集成测试

测试完整的更新流程：

1. 设置测试服务器
2. 准备测试模组文件
3. 验证下载和安装流程

### 手动测试清单

- [ ] 按键绑定正常工作
- [ ] FancyMenu按钮正常工作
- [ ] 网络连接失败时的错误处理
- [ ] JSON格式错误时的错误处理
- [ ] 下载失败时的错误处理
- [ ] 界面显示正确的中文文本
- [ ] 下载完成后的提示正确显示

## 性能优化

### 网络优化

1. 添加连接超时和重试机制
2. 使用连接池复用HTTP连接
3. 支持断点续传

### 内存优化

1. 使用流式下载避免大文件占用内存
2. 及时释放不需要的对象引用
3. 优化JSON解析内存使用

### 用户体验优化

1. 添加下载进度显示
2. 支持取消下载操作
3. 提供更详细的错误信息

## 安全考虑

### 输入验证

1. 验证服务器返回的JSON格式
2. 检查下载URL的合法性
3. 验证文件名的安全性

### 网络安全

1. 使用HTTPS协议
2. 验证服务器证书
3. 防止中间人攻击

### 文件安全

1. 检查下载文件的完整性
2. 防止路径遍历攻击
3. 限制下载文件大小

## 贡献指南

### 代码风格

1. 使用4个空格缩进
2. 类名使用PascalCase
3. 方法名使用camelCase
4. 常量使用UPPER_SNAKE_CASE

### 提交规范

1. 提交信息使用中文
2. 每个提交只包含一个功能或修复
3. 提供清晰的提交描述

### 分支策略

1. `main` 分支用于稳定版本
2. `develop` 分支用于开发
3. 功能分支命名：`feature/功能名称`
4. 修复分支命名：`fix/问题描述`

## 许可证

本项目采用 MIT 许可证，允许自由使用、修改和分发。

## 联系方式

如有技术问题或建议，请通过以下方式联系：
- 项目Issues页面
- 邮件联系
- 社区论坛

---

*本文档最后更新：2025年6月8日*

