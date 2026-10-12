## 2026-10-05 - Task: 适配 DSH 0.2.0-rc.2 并安装到 desktop

### What was done

- 官方 npm `dsh-gpt-compat@0.2.3` 被桌面端兼容检查拒绝后，用户选择先适配再安装；未授予版本豁免。
- 构建本地版 `0.2.4-desktop.0`，对齐 Cordis、schema 库和 DSH peer/关联测试依赖；保留原权限判断代码。
- 通过插件管理器安装本地 tarball 到 desktop profile，运行时立即激活。

### Testing

- `pnpm build`、`pnpm typecheck` 通过。
- `pnpm test:coverage`：19/19 通过，包含 DSH 0.2.0-rc.2 真实 ToolRuntime 测试；行/语句 98.64%、分支 91.54%、函数 100%。
- `pnpm lint`、`pnpm exec publint`、`pnpm exec attw --pack --profile esm-only .` 通过。
- `pnpm format:check` 因全仓既有 CRLF 换行差异失败；改动配置与 README 的 `--end-of-line auto` 检查通过，未全仓格式化。
- 官方 registry 下载一度因 ECONNRESET 失败，切换 npmmirror 后完成依赖安装。
- 插件管理器安装返回 `application: applied`、exitCode 0；bundle 版本为 0.2.4-desktop.0、installed/enabled 均为 true；`include:gpt-compat` 的 fiberPhase 为 active。
- 安装过程中有 pnpm peer warning；没有 DSH incompatibility 警告。未执行 package:smoke；已验证真实 tarball 安装和运行时激活。

### Notes

- 修改：package.json、pnpm-lock.yaml、README.md、README.en.md；新增 docs/desktop-compat.md、progress.md；生成 dist/、coverage/、dsh-gpt-compat-0.2.4-desktop.0.tgz（构建产物）。
- 桌面 profile 的 package.json、pnpm-lock.yaml、node_modules 由插件管理器更新，加入 dsh-gpt-compat bundle。
- 回滚安装：在桌面插件管理器禁用或移除 dsh-gpt-compat。
- 回滚源码：`git restore -- package.json pnpm-lock.yaml README.md README.en.md`；本轮新增文档可单独移除。恢复源码不会自动卸载已安装 tarball 副本。

## 2026-10-12 - Task: 发布 GitHub 预构建安装包

### What was done

- 修正中英文 README：桌面端安装改为 GitHub Release 的预构建 `.tgz`，并明确说明仓库源码 URL 会触发 pnpm 构建脚本白名单错误。
- 发布 `v0.2.4-desktop.0` GitHub prerelease，附带已验证的 `dsh-gpt-compat-0.2.4-desktop.0.tgz`。
- 将 npm 发布工作流限制为正式 release，避免桌面适配 prerelease 误触发 npm 发布。

### Testing

- 重新执行 `pnpm pack` 生成包含最新 README 的预构建包。
- 使用 `gh release view` 和 Release asset URL 校验发布结果。

### Notes

- 修改文件：`.github/workflows/publish.yml`、`README.md`、`README.en.md`、`docs/desktop-compat.md`、`progress.md`。
- 安装地址：`https://github.com/UranusNo7/dsh-gpt-compat/releases/download/v0.2.4-desktop.0/dsh-gpt-compat-0.2.4-desktop.0.tgz`。
- 回滚方式：删除该 GitHub prerelease；恢复本次提交即可移除 README 和工作流变更。
- 发布验证：Release 为 prerelease，附件大小 161760 字节；通过公开 URL 下载后 SHA256 与本地包一致：`C482918C569E0691EFECADAD314B9E8871399DDB66B3C33189C6CD635E3FF24A`。
- `git diff --check` 和本轮文档/工作流 `prettier --check --end-of-line auto` 通过。GitHub release event 的 workflow run 查询为空，未观察到 npm 发布任务；朋友 macOS 的实际安装尚未验证。
