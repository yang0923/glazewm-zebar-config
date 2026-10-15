# glazewm-zebar-config

个人 [GlazeWM](https://github.com/glzr-io/glazewm) + [Zebar](https://github.com/glzr-io/zebar) 配置仓库。GlazeWM 负责平铺窗口管理，Zebar 提供状态栏，两者通过 GlazeWM 的 `startup_commands` 联动启动。

## 📁 目录结构

```
glazewm-zebar-config/
├── .gitignore
├── glazewm/
│   └── config.yaml          # GlazeWM 主配置：工作区、快捷键、窗口规则、间距与边框
└── zebar/
    ├── settings.json         # Zebar 全局设置：指定启动时加载的 widget pack
    └── .marketplace/         # Zebar 市场元数据
```

## 🖥️ GlazeWM 配置

配置文件位于 `glazewm/config.yaml`，部署到 `%userprofile%\.glzr\glazewm\config.yaml`。

### 启动与关闭联动

GlazeWM 启动时自动拉起 Zebar，关闭时终止 Zebar 进程：

```yaml
general:
  startup_commands: ["shell-exec zebar"]
  shutdown_commands: ["shell-exec taskkill /IM zebar.exe /F"]
  config_reload_commands: ["wm-redraw"]
```

> 如果启动时报 `failed to execute 'zebar'`，说明 Zebar 未加入系统 PATH。可以改用绝对路径，或确认 Zebar 安装时已勾选“添加到 PATH”[reference:0]。

### 工作区

定义了 9 个虚拟工作区（名称 "1" 到 "9"），配合 Zebar 的工作区切换器 widget 使用。

### 间距与视觉效果

```yaml
gaps:
  inner_gap: "8px"
  outer_gap:
    top: "42px" # 为顶部状态栏留出空间
    right: "6px"
    bottom: "2px"
    left: "6px"

window_effects:
  focused_window:
    border:
      enabled: true
      color: "#89b4fa"
      style: "rounded"
  other_windows:
    border:
      enabled: true
      color: "#9399b2"
    opacity: 0.95
```

### 快捷键体系

采用 Vim 风格的 HJKL 方向键：

| 操作         | 快捷键                  |
| ------------ | ----------------------- |
| 切换窗口焦点 | `Alt + H/J/K/L`         |
| 移动窗口     | `Alt + Shift + H/J/K/L` |
| 调整窗口大小 | `Alt + U/P/O/I`         |
| 进入调整模式 | `Alt + R`               |
| 切换窗口状态 | `Alt + Space`           |
| 关闭窗口     | `Alt + Shift + Q`       |
| 启动终端     | `Alt + Enter`           |
| 切换工作区   | `Alt + 1~9`             |
| 暂停窗口管理 | `Alt + Shift + P`       |

### 窗口规则

忽略以下工具的窗口，避免被 GlazeWM 接管平铺：

- Zebar 状态栏自身
- 画中画窗口
- Minecraft（Java / GLFW）
- 微信小程序
- PowerToys 命令面板
- Lively 壁纸引擎

```yaml
window_rules:
  - commands: ["ignore"]
    match:
      - window_process: { equals: "zebar" }
      - window_process: { equals: "PowerToys.PowerAccent" }
      # ... 其余规则
```

## 📊 Zebar 配置

### 全局设置

`zebar/settings.json` 指定启动时加载的 widget pack：

```json
{
  "$schema": "https://github.com/glzr-io/zebar/raw/v3.3.1/resources/settings-schema.json",
  "startupConfigs": [
    {
      "pack": "glzr-io.starter",
      "widget": "with-glazewm",
      "preset": "default"
    }
  ]
}
```

当前使用 Zebar 官方的 `glzr-io.starter` 包中的 `with-glazewm` widget，该 widget 内置了与 GlazeWM 集成的状态栏模板[reference:1]。

### Widget Pack 体系

Zebar v3 的配置以 **widget pack** 为单位组织。每个 pack 是一个包含 `zpack.json` 的目录，可以定义多个 widget[reference:2]。

如果你想自定义状态栏，可以从 starter 包复制一份作为基础：

1. 在 Zebar GUI 的 **My widgets** 标签页点击 **Create new pack**
2. 输入 pack 名称和描述
3. 在 pack 内点击 **Add widget** 创建 widget[reference:3]

### 与 GlazeWM 的集成

Zebar 通过 `glazewm` provider 获取窗口管理器的实时状态：

- 当前工作区（`currentWorkspaces`）
- 所有工作区列表（`workspaces`）
- 聚焦容器信息（`focusedContainer`）
- 绑定模式（`bindingMode`）
- 平铺方向（`tilingDirection`）

这些数据在 widget 的 JavaScript 逻辑中通过 `@zebar/client-api` 访问[reference:4]。

## 🚀 安装

### 前置条件

- Windows 10/11
- [GlazeWM](https://github.com/glzr-io/glazewm/releases) v3.x
- [Zebar](https://github.com/glzr-io/zebar/releases) v3.x
- 字体：[JetBrainsMono Nerd Font](https://www.nerdfonts.com/font-downloads)

### 部署 GlazeWM 配置

```powershell
# 备份现有配置
Copy-Item "$env:USERPROFILE\.glzr\glazewm\config.yaml" `
          "$env:USERPROFILE\.glzr\glazewm\config.yaml.bak" -ErrorAction SilentlyContinue

# 复制新配置
Copy-Item ".\glazewm\config.yaml" `
          "$env:USERPROFILE\.glzr\glazewm\config.yaml" -Force
```

### 部署 Zebar 配置

```powershell
# 备份
Copy-Item "$env:USERPROFILE\.glzr\zebar\settings.json" `
          "$env:USERPROFILE\.glzr\zebar\settings.json.bak" -ErrorAction SilentlyContinue

# 复制
Copy-Item ".\zebar\settings.json" `
          "$env:USERPROFILE\.glzr\zebar\settings.json" -Force
```

### 重启服务

```powershell
glazewm stop
glazewm start
```

GlazeWM 启动后会自动拉起 Zebar。

## 🔧 常用操作

### 修改快捷键

编辑 `glazewm/config.yaml` 的 `keybindings` 段，保存后执行：

```powershell
glazewm reload-config
```

### 换 Zebar widget pack

修改 `zebar/settings.json` 的 `startupConfigs`：

```json
{
  "startupConfigs": [
    {
      "pack": "your-pack-name",
      "widget": "your-widget-name",
      "preset": "default"
    }
  ]
}
```

保存后重启 Zebar 或 GlazeWM。

### 调整间距

修改 `glazewm/config.yaml` 的 `gaps` 段：

```yaml
gaps:
  inner_gap: "8px" # 窗口之间的间距
  outer_gap:
    top: "42px" # 顶部留白（状态栏高度 + 间距）
    right: "6px"
    bottom: "2px"
    left: "6px"
```

## 🐛 故障排查

### GlazeWM 启动报错 "failed to execute 'zebar'"

Zebar 不在系统 PATH 中。解决方法：

```yaml
startup_commands: ['shell-exec C:\path\to\zebar.exe']
```

或手动将 Zebar 安装目录加入系统环境变量[reference:5]。

### Zebar 不显示

1. 确认 GlazeWM 已启动
2. 检查 `settings.json` 中的 `pack` 和 `widget` 名称是否正确
3. 查看 Zebar 托盘图标是否在运行
4. 检查 `%userprofile%\.glzr\zebar\` 下是否存在对应 pack 目录

### 窗口被错误平铺

在 `glazewm/config.yaml` 的 `window_rules` 中添加 `ignore` 规则：

```yaml
window_rules:
  - commands: ["ignore"]
    match:
      - window_process: { equals: "你的应用进程名.exe" }
```

### GlazeWM 配置加载失败

```powershell
glazewm validate-config
```

根据输出定位 YAML 语法错误或字段名错误。

## 🔗 相关链接

- [GlazeWM 仓库](https://github.com/glzr-io/glazewm)
- [GlazeWM 文档](https://glazewm.org/docs/)
- [Zebar 仓库](https://github.com/glzr-io/zebar)
- [Zebar 文档](https://zebar.thedev.id/)
- [Catppuccin 配色方案](https://github.com/catppuccin/catppuccin)
- [Nerd Fonts](https://www.nerdfonts.com/)

## 📄 许可

个人配置文件，自由取用。
