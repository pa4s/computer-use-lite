# 从零实现 Computer Use Lite 的 Prompt

下面这份 prompt 可直接交给 Codex 或另一个工程代理，用于从空仓库实现当前 `computer-use-lite` 的主要能力。

```text
你是一个资深 macOS Swift 工程师，请从零实现一个名为 computer-use-lite 的 macOS GUI 自动化 MCP 服务。

目标：
实现一个轻量、可观测、可复现、低打扰用户体验的 macOS Computer Use 工具，用于控制真实 macOS App，重点支持浏览器页面导航、窗口识别、后台点击、截图验证、trace 记录和失败诊断。它的体验目标是逐步接近并超过 Codex Computer Use，尤其在可观测性、固定流程性能、失败诊断、低打扰和可复现性上更强。

技术栈：
- Swift Package Manager
- Swift 6 / macOS 15+
- AppKit
- ApplicationServices / Accessibility AX
- CoreGraphics / CGWindow / CGEvent
- ScreenCaptureKit 或 CGWindowListCreateImage
- XCTest
- MCP stdio server
- Unix socket proxy：mcp_unix_proxy

核心能力：
1. MCP 服务
   - 暴露 stdio MCP 工具接口。
   - 支持工具发现和 JSON schema。
   - 支持请求级 traceId。
   - 所有 tool 调用都要输出结构化结果。
   - 失败时返回明确 code、message、hints、diagnostics。

2. App / Window 状态
   - 实现 list_apps。
   - 实现 get_app_state。
   - 能列出正在运行的 App、bundleId、pid、activationPolicy、是否前台。
   - 能列出窗口：windowId、pid、ownerName、title、frame、layer、alpha、isOnScreen。
   - 能通过 Accessibility 获取 AXWindow、AXTitle、AXFrame、AXPosition、AXSize、AXRole。
   - 需要能把 CGWindow 和 AXWindow 尽量关联起来。
   - 对浏览器窗口需要识别当前 tab、标题、URL，优先用 AX，必要时用 AppleScript fallback。

3. 浏览器自动化
   - 支持 Chrome 和 Safari。
   - 支持打开或复用浏览器窗口。
   - 支持新建低打扰窗口或标签页，避免污染用户当前工作区。
   - 支持地址栏输入 URL。
   - 支持等待页面加载。
   - 支持点击页面导航项，如 Apple 官网的 Mac、iPhone、Support。
   - 支持通过 URL、AX 内容、页面标题、截图 OCR/视觉信号中的至少一种验证成功。
   - 需要记录地址栏输入、标签页状态、窗口状态、点击方式和截图结果。

4. 点击与输入
   - 实现 click。
   - 点击策略分层：优先 AXPress，其次窗口内坐标点击，最后视觉识别坐标点击。
   - 支持 inactive/background click，尽量不抢占前台。
   - 支持 CGEvent.postToPid。
   - 支持 CGEventSetWindowLocation 或等价窗口坐标定位。
   - 明确记录是否移动真实鼠标。
   - 明确记录是否激活目标 App。
   - 明确记录 event delivery 方式。
   - 输入支持 type_text、press_key、set_value。
   - 输入要支持 modifier keys，如 Cmd+L、Return。

5. 截图
   - 实现 screenshot。
   - 支持整屏截图、窗口截图、区域截图。
   - 截图结果保存到本地 artifacts 目录。
   - 返回路径、尺寸、窗口 id、时间戳。
   - 支持截图 diff 或至少支持截图存在性与尺寸验证。
   - trace 中要记录截图验证结果。

6. Trace / 可观测性
   - 每次 MCP 调用生成结构化 trace。
   - trace 至少包含 traceId、toolName、startedAt / endedAt / durationMs、target app / pid / bundleId、windowId / window frame、input parameters、AX lookup path、coordinate mapping、event delivery method、screenshot path、validation result、errors / recovery attempts。
   - trace 写入 artifacts/traces。
   - 需要可用于分析一次完整浏览器导航链路。

7. Recovery / 失败诊断
   - 对 MCP proxy 做精准 stale proxy 诊断。
   - 能识别 Unix socket 存在但后端进程已死、proxy 版本偏移、stdio transport 断开、MCP server 未响应，以及 Accessibility、Screen Recording、Automation 等权限缺失。
   - 错误信息要给出明确修复建议。
   - 支持可选自动退出重拉起模式：检测 stale proxy 后，可自动清理旧 socket / 旧 proxy 进程，并重新启动服务。默认只诊断，不自动杀进程。
   - 对打开 App 超时，要自动检查 Gatekeeper / CoreServicesUIAgent 是否弹窗、目标 app 进程是否存在、是否有延迟窗口、是否被系统安全弹窗阻塞。
   - open timeout 后继续延迟轮询窗口，而不是立刻失败。

8. 安装 / App 打开链路
   - 支持用 browser 或 Finder/Downloads 定位下载文件。
   - 支持打开 .dmg / .pkg / .app。
   - 能处理 Gatekeeper 打开确认。
   - 安装后能打开目标 App 并截图。
   - 记录完整链路：定位文件、打开、系统弹窗、窗口轮询、最终截图。

9. Benchmark
   - 提供一个真实 macOS App browser navigation benchmark。
   - baseline：使用 Codex Computer Use 完成打开 Safari 或 Chrome、访问 https://www.apple.com、点击 Mac / iPhone / Support、截图最终页面。
   - lite：使用 computer-use-lite 从相同初始状态复现。
   - 对比 app/window 识别、AX 能力、坐标映射、event delivery、vision / screenshot、transport、recovery、user experience。
   - 记录耗时、MCP 调用次数、是否抢占前台、是否移动真实鼠标、是否影响用户当前浏览器工作区、是否能通过 URL 或截图验证成功。

10. 测试
   - 使用 XCTest。
   - 至少覆盖 window/app model 序列化、trace 生成、stale proxy 诊断、open timeout recovery 诊断、coordinate conversion、click strategy selection、browser URL/title parsing、screenshot metadata。
   - `swift test` 必须通过。
   - 提供真实浏览器 smoke test 脚本或命令。

11. CLI / 开发体验
   - 提供可执行 target，例如 computer-use-lite-server。
   - 支持 --stdio、--socket PATH、--trace-dir PATH、--auto-restart-stale-proxy、--diagnose、--verbose。
   - README 写清楚权限要求、如何启动 MCP server、如何配置 MCP、如何跑 benchmark、如何看 trace、常见错误和修复方式。

代码质量要求：
- 结构清晰，模块拆分合理。
- 推荐模块：MCPServer、ToolDefinitions、Service、AppLocator、WindowInspector、AccessibilityClient、BrowserController、EventInjector、ScreenshotService、TraceRecorder、Diagnostics、ProxyHealthChecker、OpenRecovery、BenchmarkRunner。
- 所有外部可见模型使用 Codable。
- 错误使用明确 enum，不要只返回字符串。
- 每个工具结果都带 diagnostics 和 trace summary。
- 不要上传 research 文档或本地测试 artifacts。
- .gitignore 应排除 .build/、artifacts/、reports/、research/、*.DS_Store。

交付物：
1. 完整 Swift Package。
2. MCP stdio server 可运行。
3. `swift test` 全部通过。
4. 真实浏览器 smoke test 可跑。
5. benchmark 结果文档。
6. README。
7. 示例 trace 文件。
8. Git 提交，提交信息使用中文，说明为什么改、改了什么、如何验证。

验证标准：
- `swift test` 通过。
- 使用 Chrome 或 Safari 打开 https://www.apple.com。
- 点击 Mac / iPhone / Support 任一导航项成功。
- 截图最终页面成功。
- trace 中能看到窗口识别、坐标转换、event delivery、是否抢前台、是否移动真实鼠标、截图验证。
- stale proxy 场景能给出精准诊断。
- open timeout 场景能检查 Gatekeeper/CoreServicesUIAgent、app 进程和延迟窗口。
- research/ 不进入 git。
```
