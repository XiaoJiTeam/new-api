# Issue 与 PR 分诊

组织领域：**人工智能**。Repository 使用 Project 原生字段。
配置入口是 `.github/triage.json`，公共实现版本为 `23e49fc34a7820d3ef36e4bc03af430adc74f6a2`。中央配置维护后需重新生成本仓库文件，不能只改副本。

## 使用方式

Issue 使用表单选择类型和模块；PR 根据修改文件识别模块。PR 描述中的 `### 事项类型` 下可写一个 `kind/bug`、`kind/feature`、`kind/task`、`kind/refactor`、`kind/docs`、`kind/dependency` 或 `kind/question`。
不分析标题关键词，不根据 `[P0]` 自动设置优先级。类型和优先级各最多一个，模块可以多个。

有仓库 write、maintain 或 admin 权限的负责人可以在评论中执行：

```text
/kind bug
/area set automation
/priority p2
/triage accept
```

每行一个命令，也可以分开评论。`/area set` 后续更新保留人工模块，`/area auto` 恢复自动识别。
`/triage needs-info` 标记缺少信息；信息补齐后，用 `/triage accept` 确认。
确认要求一个类型、一个优先级和至少一个模块。P0=立即处理，P1=高优先级，P2=正常排期，P3=可延后；未定级保持为空。

人工在 GitHub 界面修改模块或类型也受保护。保护依据是 GitHub 时间线中的操作者和维护者命令，不另发机器人状态评论。
命令执行结果和拒绝原因在 Actions 的分诊运行中查看。对失效/不完整表单不猜模块、不清空已有结果。
GitHub 每个 concurrency group 可能只保留一个待执行运行；脚本根据当前条目和完整时间线重新计算，旧维护者命令不会覆盖后续命令。

## 本仓库模块

| 标签 | 范围 | 自动识别路径 |
|---|---|---|
| `area/relay` | 上游模型转发 | `relay/**` |
| `area/protocols` | 模型协议转换 | `relaykit/**` |
| `area/token-counting` | Token 统计 | `tokenkit/**` |
| `area/web` | 管理界面 | `web/**` |
| `area/desktop` | 桌面封装 | `electron/**` |
| `area/plugins` | 任务插件 | `plugins/**`、`pkg/jsplugin/**` |
| `area/billing` | 计费与配额 | `pkg/billingexpr/**`、`service/quota*`、`service/tiered_settle*`、`service/*billing*`、`setting/billing_setting/**` |
| `area/authentication` | 会话授权与 OAuth | `oauth/**`、`service/authz/**` |
| `area/i18n` | 国际化 | `i18n/**`、`web/src/i18n/**` |
| `area/database` | 数据库结构与持久化 | `model/**` |
| `area/deployment` | 构建部署与运行配置 | `Dockerfile*`、`docker-compose*.yml` |
| `area/documentation` | 项目文档与手册 | `docs/**`、`README*` |
| `area/automation` | 仓库工作流与分诊配置 | `.github/**` |

## Project 与运行条件

标签自动化使用当前仓库的 GITHUB_TOKEN。私有仓库复用中央工作流，默认运行于 xiaoji-builder 组；仓库变量 `TRIAGE_RUNNER` 可覆盖为 JSON 字符串 `"ubuntu-latest"` 或经批准的 runner 配置。
配置组织变量 `TRIAGE_APP_ID` 和组织 Secret `TRIAGE_APP_PRIVATE_KEY` 后，可在分诊运行结束时同步 Project #1。
App 必须安装到接入仓库，具备 Contents 读、Issues/Pull requests 写、组织 Projects 写权限。不要复用个人登录令牌或业务 Secret。
中央每小时补偿同步；第一步汇总流程可调用中央 `triage-project.yml` 的 workflow_dispatch，传 `repository` 和 `number`。
Domain 只填空以保留人工覆盖；Priority/Triage 从标签单向更新。条目尚未加入 Project 时不创建条目，由后续补偿处理。
公共 fork 使用内嵌的同版本分诊程序与 GitHub 托管 runner；Project 由中央补偿，不向公共仓库分发 App 私钥。

## 历史数据与验证

历史条目先生成报告，负责人逐项将 `approved` 改为 true 后应用。回填只补标签，不重开条目，不推断优先级；条目在预览后变化则跳过。
合并工作流后，新建一条测试 Issue、一个跨模块测试 PR，验证手工覆盖、越权命令、重新运行和晚加入 Project。文件存在或本地测试通过不代表线上已启用。
