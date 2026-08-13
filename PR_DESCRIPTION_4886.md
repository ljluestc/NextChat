## feat: support multiple app windows with independent server configurations

Closes #4886

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

**Support multiple app windows, each with an independent server configuration.**

The desktop app previously ran as a single window, forcing users who switch between backends (e.g. a local vLLM server and an online OneAPI service) to repeatedly re-enter server settings. This PR adds a **New Window** action so each window maintains its own endpoint, API key, provider, and model settings. Because all windows live in one app process, this also avoids the crashes reported when running separate NextChat instances under Windows Sandbox / Sandboxie (WebView2 user-data-dir contention).

Changes:

- **New window creation (desktop only):** a "New Window" button in the sidebar footer (rendered only inside Tauri; web/PWA builds unchanged) plus a **Ctrl+Shift+N** hotkey. Windows are created via the Tauri `WebviewWindow` API with stable, recycled labels (`window-2`, `window-3`, …) — the smallest free index is chosen, so reopening a second window after a restart restores the exact configuration it had before. Enabled `window-create` in `src-tauri/Cargo.toml` and `window.create: true` in the `tauri.conf.json` allowlist.
- **Per-window state isolation:** new `app/utils/window.ts` reads the window label synchronously from the `window_label` query param at module init (stores hydrate at import time), falling back to `main` for the first window and web builds. `app/utils/indexedDB-storage.ts` prefixes every persisted zustand store key with the window's namespace, uniformly isolating access/server config, app config, chat, masks, prompts, sync, and MCP stores per window — and preventing concurrent windows from racing on the same IndexedDB keys.
- **Backward compatibility:** the main window keeps the historical unprefixed storage keys, so all existing user data (settings, chat history) is preserved untouched. `clear()` from a secondary window only clears its own namespace; clearing from the main window remains a global reset.
- **Typing & i18n:** extended the `__TAURI__` global typing in `app/global.d.ts` with the `window` module (`getAll`, `WebviewWindow`); added `Locale.UI.NewWindow` in `en` and `cn` (other locales fall back to English); new `new-window.svg` icon in the existing 16×16 style.

#### 📝 补充信息 | Additional Information

**How to test:**

1. `yarn app:dev` (or a packaged build).
2. Click the new-window icon in the sidebar footer, or press **Ctrl+Shift+N** — a second window titled `NextChat #2` opens.
3. In window #2: Settings → set a custom endpoint (e.g. local vLLM `http://localhost:8000/v1`) and API key; confirm the main window still uses its own configuration.
4. Send messages in both windows — each talks to its own backend; sessions/chat lists are independent.
5. Close window #2 and open a new window again — the previous secondary configuration is restored (label recycling → same storage namespace).
6. Restart the app — the main window loads all pre-existing data unchanged; a reopened second window restores its prior config.
7. Web build (`yarn dev`): no new-window button is rendered and behavior is unchanged.

**Notes / limitations:**

- True multi-*process* instances (Sandboxie, etc.) are out of scope; in-app windows cover the underlying need without the WebView2 crashes.
- Window position/size persistence is handled by the existing `tauri-plugin-window-state`, keyed per window label.
- `Ctrl+Shift+N` is registered at the webview level, so it works on all desktop platforms.

**Validation:** `tsc --noEmit` against this branch shows only pre-existing errors from missing dependencies in the checkout; no new errors in any file touched by this PR. Runtime smoke-testing on Windows with `yarn app:dev` (window creation via `withGlobalTauri`, per-label window-state restore) is recommended before merging.
