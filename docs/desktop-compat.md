# 桌面端本地适配

## 目标与结果

- 插件版本：`0.2.4-desktop.0`，本地构建版，未发布到 npm。
- 目标运行时：DSH `0.2.0-rc.2`，profile 为 `desktop`。
- Cordis：`~4.0.4`；schemastery：`~3.18.4`。
- DSH peer 和测试依赖对齐到 `0.2.0-rc.2`，不申请版本豁免。
- 现有 `tools/pre-execute`、`tools.get`、`sandboxPolicy.resolve` 接口兼容；未修改权限判断逻辑。
- 本地 tarball 已通过插件管理器安装，运行时 `include:gpt-compat` 为 `enabled: true`、`fiberPhase: active`。已即时生效，无需重启。
- 预构建安装包发布在 GitHub Release `v0.2.4-desktop.0`；应使用 Release asset 的 `.tgz` URL，不要使用仓库源码 URL或 GitHub 自动生成的 source archive。

## 重建与安装

```bash
pnpm install --frozen-lockfile
pnpm build
pnpm typecheck
pnpm test:coverage
pnpm lint
pnpm exec publint
pnpm exec attw --pack --profile esm-only .
pnpm pack
dsh plugin --profile desktop add ./dsh-gpt-compat-0.2.4-desktop.0.tgz
```

也可用桌面端插件管理器安装该 tarball。不要用 npm 的 `0.2.3` 替代：其 DSH peer 范围不包含目标运行时。

## 验证与限制

19 项测试通过，包含新版本真实 ToolRuntime 的参数改写测试；总覆盖率为行/语句 98.64%、分支 91.54%、函数 100%。lint、publint 与 ESM 声明解析检查通过。

默认配置为 `providers: ["*"]` 和 `tools: ["bash", "pwsh", "write", "edit"]`。只删除当前策略下不扩权的已知请求；真正扩权、未知契约仍保留给 DSH 原生校验。

全仓 `pnpm format:check` 因既有 CRLF 换行差异失败，未全仓格式化。改动的配置和 README 使用 `prettier --check ... --end-of-line auto` 检查通过。未执行 `package:smoke` 脚本；真实 tarball 安装和运行时激活已验证。

安装时 pnpm 输出 peer dependency warning，但安装器没有报告 DSH 版本不兼容，运行时已激活。

## 回滚

在桌面端插件管理器中先禁用 `dsh-gpt-compat`，必要时移除该 bundle。这不需要修改 DSH 程序文件，也不需要更改当前权限预设。

源码恢复可执行：

```bash
git restore -- package.json pnpm-lock.yaml README.md README.en.md
```

执行源码恢复后不要继续使用旧的本地构建产物重新安装；需重新构建。已安装的插件是 tarball 的副本，恢复源码不会自动卸载它。
