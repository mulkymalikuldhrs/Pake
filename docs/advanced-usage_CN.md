# 高级用法

<h4 align="right"><strong><a href="advanced-usage.md">English</a></strong> | 简体中文</h4>

通过样式修改、JavaScript 注入和容器通信等方式自定义 Pake 应用。

## 样式自定义

通过修改 CSS 移除广告或自定义外观。

**快速流程：**

1. 运行 `pnpm run dev` 进入开发模式
2. 使用开发者工具找到要修改的元素
3. 编辑 `src-tauri/src/inject/style.js`：

   ```javascript
   const css = `
     .ads-banner { display: none !important; }
     .header { background: #1a1a1a !important; }
   `;
   ```

## JavaScript 注入

添加自定义功能，如键盘快捷键。

**实现方式：**

1. 编辑 `src-tauri/src/inject/event.js`
2. 添加事件监听器：

```javascript
document.addEventListener("keydown", (e) => {
  if (e.ctrlKey && e.key === "k") {
    // 自定义操作
  }
});
```

## 内置功能

### 下载错误通知

Pake 自动提供用户友好的下载错误通知：

**功能特性：**

- **双语支持**：自动检测浏览器语言（中文/英文）
- **系统通知**：在授予权限后使用原生操作系统通知
- **优雅降级**：如果通知不可用则降级到控制台日志
- **全面覆盖**：处理所有下载类型（HTTP、Data URI、Blob）

**用户体验：**

当下载失败时，用户将看到通知：

- 英文："Download Error - Download failed: filename.pdf"
- 中文："下载错误 - 下载失败: filename.pdf"

**请求通知权限：**

要启用通知，请在注入的 JavaScript 中添加：

```javascript
// 在应用启动时请求通知权限
if (window.Notification && Notification.permission === "default") {
  Notification.requestPermission();
}
```

下载系统自动处理：

- 常规 HTTP(S) 下载
- Data URI 下载（base64 编码文件）
- Blob URL 下载（动态生成的文件）
- 右键菜单发起的下载

## 容器通信

在网页内容和 Pake 容器之间发送消息。

**网页端（JavaScript）：**

```javascript
window.__TAURI__.invoke("handle_scroll", {
  scrollY: window.scrollY,
  scrollX: window.scrollX,
});
```

**容器端（Rust）：**

```rust
#[tauri::command]
fn handle_scroll(scroll_y: f64, scroll_x: f64) {
  println!("滚动位置: {}, {}", scroll_x, scroll_y);
}
```

## 窗口配置

在 `pake.json` 中配置窗口属性：

```json
{
  "windows": {
    "width": 1200,
    "height": 780,
    "fullscreen": false,
    "resizable": true
  },
  "hideTitleBar": true
}
```

## 静态文件打包

打包本地 HTML/CSS/JS 文件：

```bash
pake ./my-app/index.html --name my-static-app --use-local-file
```

要求：Pake CLI >= 3.0.0

## 项目结构

了解 Pake 的代码库结构将帮助您有效地进行导航和贡献：

```tree
├── bin/                    # CLI 源代码 (TypeScript)
│   ├── builders/          # 平台特定的构建器
│   ├── helpers/           # 实用函数
│   └── options/           # CLI 选项处理
├── docs/                  # 项目文档
├── src-tauri/             # Tauri 应用核心
│   ├── src/
│   │   ├── app/           # 核心模块（窗口、托盘、快捷键）
│   │   ├── inject/        # 网页注入逻辑
│   │   └── lib.rs         # 应用程序入口点
│   ├── icons/             # macOS 图标 (.icns)
│   ├── png/               # Windows/Linux 图标 (.ico, .png)
│   ├── pake.json          # 应用配置
│   └── tauri.*.conf.json  # 平台特定配置
├── scripts/               # 构建和实用脚本
└── tests/                 # 测试套件
```

### 关键组件

- **CLI 工具** (`bin/`): 基于 TypeScript 的命令接口，用于打包应用
- **Tauri 应用** (`src-tauri/`): 基于 Rust 的桌面框架
- **注入系统** (`src-tauri/src/inject/`): 用于网页的自定义 CSS/JS 注入
- **配置**: 多平台应用设置和构建配置

## 开发工作流

### 前置条件

- Node.js ≥22.0.0 (推荐 LTS，较旧版本 ≥18.0.0 可能可用)
- Rust ≥1.85.0 (推荐稳定版)

#### 平台特定要求

**macOS:**

- Xcode 命令行工具：`xcode-select --install`

**Windows:**

- **重要**：请先参阅 [Tauri 依赖项指南](https://tauri.app/start/prerequisites/)
- Windows 10 SDK (10.0.19041.0) 和 Visual Studio Build Tools 2022 (≥17.2)
- 必需的运行库：
  1. Microsoft Visual C++ 2015-2022 Redistributable (x64)
  2. Microsoft Visual C++ 2015-2022 Redistributable (x86)
  3. Microsoft Visual C++ 2012 Redistributable (x86)（可选）
  4. Microsoft Visual C++ 2013 Redistributable (x86)（可选）
  5. Microsoft Visual C++ 2008 Redistributable (x86)（可选）

- **Windows ARM (ARM64) 支持**：在 Visual Studio Installer 中的"单个组件"下安装"MSVC v143 - VS 2022 C++ ARM64 构建工具"

**Linux (Ubuntu):**

```bash
sudo apt install libdbus-1-dev \
    libsoup-3.0-dev \
    libjavascriptcoregtk-4.1-dev \
    libwebkit2gtk-4.1-dev \
    build-essential \
    curl \
    wget \
    file \
    libxdo-dev \
    libssl-dev \
    libgtk-3-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev \
    gnome-video-effects \
    gnome-video-effects-extra \
    libglib2.0-dev \
    pkg-config
```

### 安装

```bash
# 克隆仓库
git clone https://github.com/tw93/Pake.git
cd Pake

# 安装依赖
pnpm install

# 开始开发
pnpm run dev
```

### 开发命令

1. **CLI 更改**: 编辑 `bin/` 中的文件，然后运行 `pnpm run cli:build`
2. **核心应用更改**: 编辑 `src-tauri/src/` 中的文件，然后运行 `pnpm run dev`
3. **注入逻辑**: 修改 `src-tauri/src/inject/` 中的文件以进行网页自定义
4. **测试**: 运行 `pnpm test` 进行综合验证

#### 命令参考

- **开发模式**：`pnpm run dev`（热重载）
- **构建**：`pnpm run build`
- **调试构建**：`pnpm run build:debug`
- **CLI 构建**：`pnpm run cli:build`

#### CLI 开发调试

对于需要热重载的 CLI 开发，可修改 `bin/defaults.ts` 中的 `DEFAULT_DEV_PAKE_OPTIONS` 配置：

```typescript
export const DEFAULT_DEV_PAKE_OPTIONS: PakeCliOptions & { url: string } = {
  ...DEFAULT_PAKE_OPTIONS,
  url: "https://weekly.tw93.fun/en",
  name: "Weekly",
};
```

然后运行：

```bash
pnpm run cli:dev
```

此脚本会读取上述配置并使用 watch 模式打包指定的应用，对 `pake-cli` 代码修改可实时热更新。

### 测试指南

统一的 CLI 构建测试套件，用于验证多平台打包功能。

#### 运行测试

```bash
# 完整测试套件（推荐）
pnpm test                   # 运行完整测试套件，包含真实构建测试（8-12分钟）

# 开发时快速测试
pnpm test -- --no-build     # 跳过构建测试，仅验证核心功能（30秒）

# 构建 CLI 以供测试
pnpm run cli:build
```

#### 🚀 完整测试套件包含

- ✅ **单元测试**：CLI命令、参数验证、响应时间
- ✅ **集成测试**：进程管理、文件权限、依赖解析
- ✅ **构建器测试**：平台检测、架构检测、文件命名
- ✅ **真实构建测试**：完整的GitHub.com应用打包验证

#### 测试内容详情

**单元测试（6个）**

- 版本命令 (`--version`)
- 帮助命令 (`--help`)
- URL 验证（有效/无效链接）
- 参数验证（数字类型检查）
- CLI 响应时间（<2秒）
- Weekly URL 可访问性

**集成测试（3个）**

- 进程生成和管理
- 文件系统权限检查
- 依赖包解析验证

**构建测试（3个）**

- 平台检测（macOS/Windows/Linux）
- 架构检测（Intel/ARM64）
- 文件命名模式验证

**真实构建测试（重点）**

_macOS_: 🔥 多架构构建（通用二进制）

- 编译 Intel + Apple Silicon 双架构
- 检测 `.app` 文件生成：`GitHubMultiArch.app`
- 备用检测：`src-tauri/target/universal-apple-darwin/release/bundle/macos/`
- 验证通用二进制：`file` 命令检查架构

_Windows_: 单架构构建

- 检测 EXE 文件：`src-tauri/target/x86_64-pc-windows-msvc/release/pake.exe`
- 检测 MSI 安装包：`src-tauri/target/x86_64-pc-windows-msvc/release/bundle/msi/*.msi`

_Linux_: 单架构构建

- 检测 DEB 包：`src-tauri/target/release/bundle/deb/*.deb`
- 检测 AppImage：`src-tauri/target/release/bundle/appimage/*.AppImage`

#### 发布构建测试

```bash
# 实际构建测试（固定测试 weread + twitter 两个应用）
node ./tests/release.js
```

真实构建2个应用包，验证完整的打包流程和 release.yml 逻辑是否正常工作。

#### 故障排除

- **CLI 文件不存在**：运行 `pnpm run cli:build`
- **测试超时**：构建测试需要较长时间完成
- **构建失败**：检查 Rust 工具链 `rustup update`
- **权限错误**：确保有写入权限

总计：**13 个测试**，全部通过表示 CLI 功能正常。提交代码前建议运行 `pnpm test` 确保所有平台构建正常。

### 常见构建问题

- **Rust 编译错误**: 在 `src-tauri/` 目录中运行 `cargo clean`
- **Node 依赖问题**: 删除 `node_modules` 并运行 `pnpm install`
- **macOS 权限错误**: 运行 `sudo xcode-select --reset`

## 链接

- [CLI 文档](cli-usage_CN.md)
- [GitHub 讨论区](https://github.com/tw93/Pake/discussions)

---

> **Contact:** Mulky Malikul Dhaher — [mulkymalikuldhaher@email.com](mailto:mulkymalikuldhaher@email.com)
>
> **Disclaimer:** This project is for Education Purpose only. Risiko apapun tidak kita tanggung. (We are not responsible for any risks or damages.)
