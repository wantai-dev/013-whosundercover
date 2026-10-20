# wantai-dev/013-whosundercover 开发集成与分层 CI

开发成果统一汇总到 `dev`，不为功能小改动创建 PR；集成 PR 只走 `dev → main`。

日常本地按实际变更与影响路径选择检查。远端只在 `dev → main` 集成 PR 阶段手动触发 `.github/workflows/integration-ci.yml`；开 PR、push 和定时器不触发远端 CI。生产发布另行手动执行，并保留已有授权、输入与环境守卫。

`.development-ci.json` 是检查清单。`changed` 按 `checks[].paths` 选择；`full` 执行全部已声明 checks。命令按 argv 执行，不拼接用户输入到 shell。

每天 05:00（Asia/Taipei）完整检查由本机统一调度器串行执行，不在 GitHub Actions 新增回归 schedule。执行机器、隔离检出、基线 SHA、命令和日志以本机调度登记为准；本文不表示定时任务已经启用。

迁移来源：commit `daacb0fde6e71722ad09509c3d712cc02209e80c`，tree `ae1cbdcf7dcd010e2d8279e9059302d051eecb9e`。快照用于评审，不表示已应用远端或已通过产品回归。

## 安装

- `.`：`env COREPACK_ENABLE_AUTO_PIN=0 COREPACK_HOME=.local/development-ci/corepack corepack pnpm@10.20.0 install --frozen-lockfile --ignore-scripts`

## 检查命令

| ID | 工作目录 | 命令 |
| --- | --- | --- |
| eslint | `.` | `env COREPACK_ENABLE_AUTO_PIN=0 COREPACK_HOME=.local/development-ci/corepack corepack pnpm@10.20.0 run lint` |
| build | `.` | `env COREPACK_ENABLE_AUTO_PIN=0 COREPACK_HOME=.local/development-ci/corepack corepack pnpm@10.20.0 run build` |
| offline-game-rules | `.` | `env COREPACK_ENABLE_AUTO_PIN=0 COREPACK_HOME=.local/development-ci/corepack corepack pnpm@10.20.0 exec vitest run src/lib/gameRules.test.ts src/lib/offlineGame.test.ts` |

## 能力与边界

- 命令来源：package.json lint/build/test及src/lib/gameRules.test.ts、offlineGame.test.ts。
- 采用pnpm-lock.yaml（lockfileVersion 9.0），通过Corepack固定pnpm@10.20.0；COREPACK_ENABLE_AUTO_PIN=0禁止自动改写package.json，COREPACK_HOME=.local/development-ci/corepack将缓存限定到仓库受管目录。所有命令工作目录均为仓库根目录；不回退到无锁npm。
- 选定测试仅内存离线游戏规则，不调用Supabase，不代表联网多人房间、RLS或生产验收。

只运行列明且不依赖生产凭据的离线检查。失败应报告原始失败；真实 API、账号、数据写入、采集、部署和外部接受度需独立验收。

pnpm@10.20.0 已通过官方 npm registry 只读核实：`npm view --registry=https://registry.npmjs.org pnpm@10.20.0 version` 输出 `10.20.0`。本次未安装产品依赖或执行构建/测试。
