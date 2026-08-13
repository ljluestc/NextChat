## feat: 支持多窗口，每个窗口拥有独立的服务器配置

关闭 #4886

#### 💻 变更类型 | Change Type

- [x] feat    <!-- 引入新功能 | Introduce new features -->
- [ ] fix    <!-- 修复 Bug | Fix a bug -->
- [ ] refactor    <!-- 重构代码（既不修复 Bug 也不添加新功能） | Refactor code that neither fixes a bug nor adds a feature -->
- [ ] perf    <!-- 提升性能的代码变更 | A code change that improves performance -->
- [ ] style    <!-- 添加或更新不影响代码含义的样式文件 | Add or update style files that do not affect the meaning of the code -->
- [ ] test    <!-- 添加缺失的测试或纠正现有的测试 | Adding missing tests or correcting existing tests -->
- [ ] docs    <!-- 仅文档更新 | Documentation only changes -->
- [ ] ci    <!-- 修改持续集成配置文件和脚本 | Changes to our CI configuration files and scripts -->
- [ ] chore    <!-- 其他不修改 src 或 test 文件的变更 | Other changes that don’t modify src or test files -->
- [ ] build    <!-- 进行架构变更 | Make architectural changes -->

#### 🔀 变更说明 | Description of Change

**支持同时打开多个应用窗口，每个窗口拥有独立的服务器配置。**

此前桌面端只能运行单一窗口。需要在多个后端之间切换的用户（例如本地 vLLM 服务器和线上 OneAPI 服务）不得不反复手动修改服务器设置。本 PR 新增「新窗口」功能，使每个窗口都可以独立维护自己的接口地址、API Key、服务提供商和模型设置。由于所有窗口运行在同一个应用进程内，还规避了在 Windows Sandbox / Sandboxie 中启动多个 NextChat 实例时的崩溃问题（多个进程争抢同一个 WebView2 用户数据目录所致）。

具体改动：

- **新窗口创建（仅桌面端）：** 在侧边栏底部新增「新窗口」按钮（仅在 Tauri 应用内渲染，Web/PWA 版本不受影响），并支持 **Ctrl+Shift+N** 快捷键。窗口通过 Tauri 的 `WebviewWindow` API 创建，使用稳定且可复用的标签（`window-2`、`window-3`……）——每次选取最小空闲序号，因此重启应用后再次打开「第二个窗口」时，会恢复它之前的配置。同时在 `src-tauri/Cargo.toml` 中启用 `window-create` 特性，并在 `tauri.conf.json` 的 allowlist 中开启 `window.create: true`。
- **按窗口隔离状态：** 新增 `app/utils/window.ts`，在模块初始化时通过 `window_label` 查询参数同步读取窗口标签（store 在 import 时即开始 hydrate），主窗口和 Web 版本回退为 `main`。`app/utils/indexedDB-storage.ts` 为每个持久化的 zustand store 键添加窗口命名空间前缀，统一隔离各窗口的服务器配置、应用配置、聊天、面具、提示词、同步和 MCP 等 store，同时避免多个窗口并发写入同一个 IndexedDB 键产生竞争。
- **向后兼容：** 主窗口继续使用历史上不带前缀的存储键，所有既有用户数据（设置、聊天记录）完全保留、不受影响。在子窗口中执行 `clear()` 只会清理该窗口自己的命名空间；主窗口的清理仍然是全局重置。
- **类型与国际化：** 在 `app/global.d.ts` 中扩展了 `__TAURI__` 全局类型，补充 `window` 模块（`getAll`、`WebviewWindow`）；在 `en` 和 `cn` 中添加 `Locale.UI.NewWindow` 文案（其他语言通过现有 merge 机制回退到英文）；新增 16×16 风格的 `new-window.svg` 图标。

#### 📝 补充信息 | Additional Information

**测试方法：**

1. 运行 `yarn app:dev`（或使用打包后的应用）。
2. 点击侧边栏底部的新窗口图标，或按下 **Ctrl+Shift+N** —— 会打开标题为 `NextChat #2` 的第二个窗口。
3. 在窗口 #2 中：设置 → 填写自定义接口地址（例如本地 vLLM 的 `http://localhost:8000/v1`）和 API Key；确认主窗口仍使用自己的配置。
4. 在两个窗口中分别发送消息 —— 各自请求各自的后端；会话和聊天列表相互独立。
5. 关闭窗口 #2 后再次打开新窗口 —— 之前的副窗口配置会被恢复（标签复用 → 相同的存储命名空间）。
6. 重启应用 —— 主窗口完整加载所有既有数据；重新打开的第二个窗口恢复其之前的配置。
7. Web 版本（`yarn dev`）：不会渲染新窗口按钮，行为保持不变。

**注意事项 / 限制：**

- 真正的多*进程*实例（Sandboxie 等）不在本 PR 范围内；应用内多窗口已经覆盖了底层需求，且不会触发 WebView2 崩溃。
- 窗口位置和大小的持久化由现有的 `tauri-plugin-window-state` 处理，按窗口标签分别保存。
- `Ctrl+Shift+N` 在 webview 层注册，因此在所有桌面平台上均可用。

**验证情况：** 在本分支上运行 `tsc --noEmit`，仅存在因本地未安装依赖导致的既有报错；本 PR 涉及的文件均未引入新错误。建议合并前在 Windows 上用 `yarn app:dev` 做一次运行时冒烟测试（验证 `withGlobalTauri` 下的窗口创建以及按标签恢复的窗口状态）。
