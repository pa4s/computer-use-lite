# Computer Use Lite

Computer Use Lite 是一个 macOS GUI 自动化能力原型，目标是让 agent 通过 MCP 控制真实 macOS App，并在浏览器导航、窗口识别、点击输入、截图验证和失败诊断上做到更轻量、更可观测、更低打扰。

这个公开仓库只包含从零实现 Prompt 和可下载 DMG，不包含项目源代码。

## 能力概览

- MCP 工具接口：面向 Codex、Cursor、Claude Code 或自定义 agent 接入。
- App / Window 识别：读取运行中 App、窗口、标题、位置、尺寸和 Accessibility 信息。
- 浏览器导航：支持 Chrome / Safari 的 URL 输入、页面导航项点击和结果验证。
- 点击与输入：支持 AXPress、窗口坐标点击、按键输入、文本输入和低打扰事件投递。
- 截图验证：支持窗口/区域截图，并把截图路径、尺寸和验证结果写入 trace。
- Trace 记录：记录工具调用、窗口绑定、坐标转换、event delivery、截图和耗时。
- 失败诊断：覆盖 stale proxy、MCP transport、权限缺失、Gatekeeper/CoreServicesUIAgent、App open timeout 等场景。
- Benchmark：用于对比 Codex Computer Use 与 lite 实现的浏览器导航链路。

## 下载

从 Release 下载 DMG：

[ComputerUseLite-0.9.0.dmg](https://github.com/pa4s/computer-use-lite/releases/download/v0.9.0/ComputerUseLite-0.9.0.dmg)

校验：

```sh
shasum -a 256 ComputerUseLite-0.9.0.dmg
```

期望 SHA-256：

```text
c9387cc354df845b6912d0220509740e18ad26429227e6362de5cfe09c4eb8f5
```

注意：当前 DMG 使用本机自签证书签名，未做 Apple Developer ID notarization；首次打开时 macOS Gatekeeper 可能需要手动允许。

## 从零实现

查看 [PROMPT.md](PROMPT.md)，可以把它交给 Codex 或另一个工程代理，从空仓库实现同类能力。
