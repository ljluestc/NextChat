close #4799

#### 💻 变更类型 | Change Type

- [x] feat    <!-- 引入新功能 | Introduce new features -->
- [ ] fix    <!-- 修复 Bug | Fix a bug -->
- [ ] refactor    <!-- 重构代码（既不修复 Bug 也不添加新功能） | Refactor code that neither fixes a bug nor adds a feature -->
- [ ] perf    <!-- 提升性能的代码变更 | A code change that improves performance -->
- [ ] style    <!-- 添加或更新不影响代码含义的样式文件 | Add or update style files that do not affect the meaning of the code -->
- [ ] test    <!-- 添加缺失的测试或纠正现有的测试 | Adding missing tests or correcting existing tests -->
- [x] docs    <!-- 仅文档更新 | Documentation only changes -->
- [ ] ci    <!-- 修改持续集成配置文件和脚本 | Changes to our CI configuration files and scripts -->
- [ ] chore    <!-- 其他不修改 src 或 test 文件的变更 | Other changes that don’t modify src or test files -->
- [ ] build    <!-- 进行架构变更 | Make architectural changes -->

#### 🔀 变更说明 | Description of Change

本 PR 实现了 #4799：支持通过环境变量 `BRAND_NAME` 自定义网站标题与品牌展示。

主要变更如下：

1. 新增品牌配置能力
   - 新增 `BRAND_NAME` 环境变量读取与透传逻辑。
   - 当 `BRAND_NAME` 为空时，保持默认品牌 `NextChat`。
   - 当 `BRAND_NAME` 非空时，页面标题与侧边栏标题显示为自定义品牌名。
2. 侧边栏副标题逻辑
   - 默认品牌（`NextChat`）下，副标题保持：
     - `Build your own AI assistant.`
   - 自定义品牌（`BRAND_NAME` 非空）下，副标题改为：
     - `Powered by NextChat`
3. 页面元信息标题支持自定义
   - `metadata.title` 与 `appleWebApp.title` 支持使用 `BRAND_NAME`。
4. 文档与示例更新
   - 在 `.env.template` 中新增 `BRAND_NAME` 配置示例。
   - 在 `README.md` 中补充 `BRAND_NAME` 说明与行为说明。

变更文件：

- `.env.template`
- `README.md`
- `app/components/sidebar.tsx`
- `app/config/build.ts`
- `app/config/server.ts`
- `app/layout.tsx`

#### 📝 补充信息 | Additional Information

- 关联 Issue：#4799
- 兼容性说明：
  - 未设置 `BRAND_NAME` 时，行为与现有版本保持一致（向后兼容）。
- 本地校验：
  - 已对相关前端文件执行 lint 校验并通过。

Co-Authored-By: Oz <oz-agent@warp.dev>
