# GitHub 项目状态看板

> **本仓是单仓看板模板。** 上游 `ScienceDiscovery/github-status-board` 的
> OIDC/Bot 采集流程已移到 `docs/upstream-workflows/`，模板使用
> `setup.yml` + `collect-selfhosted.yml`，只需要源仓链接和一份 JSON 配置。
> 当前 Pages 上是 `haowenz001/ci-gates-smoke-20261022` 的 smoke 示例；第一次
> **Setup dashboard** 会把它替换成你的源仓。

## 模板快速开始

1. 用本仓创建一个新的看板仓（建议 public，Pages 需要）。
2. 在 **Settings → Pages → Build and deployment** 选择 **GitHub Actions**。
3. 在 **Actions → Setup dashboard → Run workflow** 填写：
   - `source_repository`：要展示的代码仓，例如 `owner/service`；
   - `config_json`（可选）：
     ```json
     {
       "label": "service",
       "workflows": { "gate": ["^CI$", "CI Gates"], "daily": ["nightly"], "release": ["release"] },
       "branch_lines": [{ "key": "main", "ref": "main" }],
       "review_sla_days": 3,
       "stale_days": 30
     }
     ```
   - `reset`：切换源仓时勾选，清掉旧采集数据。
4. 源仓按 [`source-template/`](source-template/README.md) 复制门禁工作流、`.github/ci-gates.json`、`scripts/gate/gate.mjs` 和用例目录，改成自己的 build/lint/test 命令。
5. Setup 会写入 `board-config.json` 并触发 `Collect dashboard data (PAT)`；采集完成后 `Deploy dashboard Pages` 自动发布。
6. 源仓的 `CI` 运行上传 `ci-gates-reports` 后，看板 CI 页出现「质量门禁」卡片，显示每个 lane 的结论、命令通过数和用例 catalog 的 ready/todo/skipped 数量。

模板默认面向**公开源仓 + 公开 Pages**：采集使用本仓 `GITHUB_TOKEN` 读取源仓并提交 `site/`。私有源仓不要用 Actions 发布到公开 Pages；请在本机运行：

```bash
GSB_ALLOW_PRIVATE=1 GITHUB_TOKEN=... python3 publish.py --repo owner/private-repo
python3 server.py
```

`GSB_ALLOW_PRIVATE=1` 只允许生成本机 `dist/`，`publish.py --publish-repo` 仍会拒绝私有源仓，避免误发到公开 Pages。私有源仓要对外发布，需要私有 Pages、内网 runner 或额外的访问控制，这不是本模板默认提供的。

`board-config.json` 仍然兼容上游配置格式；`docs/` 里的上游文档保留作参考。

## 上游说明


静态 GitHub Pages 看板，展示 Issue、PR、门禁、每日构建、版本验证、UT / ST / E2E、标签化测试及 Node.js / Python 覆盖率（可展开到文件）。总览优先列出需要处理的流水线问题；Issue 与 PR 可在表格和本地看板之间切换；CI、E2E 与 Coverage 可在 main 和长期开发分支之间切换查看。采集由看板仓 GitHub Actions 执行；Bot 负责接收 Webhook 和触发刷新。

| 用途 | 数据源 | 站点 |
| --- | --- | --- |
| 正式 | openJiuwen-ai/sciencediscovery | https://sciencediscovery.github.io/github-status-board/ |
| 测试 | ScienceDiscovery/sciencediscovery | https://sciencediscovery.github.io/github-status-board-test/ |

## 使用

配置完成后，在 **Actions → Collect dashboard data (PAT) → Run workflow** 手动跑一次；之后每天 UTC 02:17 自动运行。采集提交 `site/` 后，`Deploy dashboard Pages` 通过 `workflow_run` 自动发布。上游的 `collect.yml` / `collect-on-push.yml` 已移到 `docs/upstream-workflows/`，模板不需要 OIDC Bot。

本地快速预览（Python 3.10+，凭据来自环境或已登录的 gh）：

```bash
python3 publish.py --repo owner/service
python3 server.py
```

此兼容命令生成有限近期快照；完整历史由 `--incremental` 模式基于目标仓自己的 `.sync/` 续跑。两种模式不要同时写同一站点。

Coverage 页从源仓 GitHub Actions 产物中读取 `node-coverage-summary-*` 和 `python-coverage-summary-*` JSON 摘要，分语言展示整仓汇总、趋势、目录树、路径指标与最近 PR 结果；看板本身不运行测试。页面说明见[页面与交互](docs/dashboard-pages.md)。

## 文档

[文档目录](docs/README.md) · [增量同步与历史](docs/incremental-history.md) · [Actions 配置](docs/actions-collection.md) · [测试报告契约](docs/test-reports.md) · [GitCode 同步记录](docs/gitcode-sync.md) · [页面与交互](docs/dashboard-pages.md) · [CI 分层历史](docs/ci-history-lanes.md) · [标签化测试](docs/tagged-tests.md) · [分支线](docs/branch-lines.md)

看板的状态、优先级、迭代、备注与列设置保存在本浏览器，按源仓隔离；可导入／导出字段。浏览器不持有 GitHub 凭据，不连接 Bot，也不写回 GitHub。

## 验证

```bash
PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s tests -v
PYTHONDONTWRITEBYTECODE=1 python3 test/journey-rerun-collection.py
node test/sync-e2e.mjs
node .e2e/node_modules/playwright/cli.js test --config test/playwright.config.cjs
```

修改功能时同步更新对应文档。测试使用独立临时数据，禁止将 fixture 发布为真实看板数据。
