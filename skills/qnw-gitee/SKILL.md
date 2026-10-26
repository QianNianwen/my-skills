---
name: qnw-gitee
description: 操作 Gitee / gitee-cli 的命令手册，只收录逐条实测成功的命令；含 CLI 帮助查不到的企业专属规则（program、issue 类型、优先级、父子任务、企业标签、工作项状态 issue_state_id）与铁律「企业级 PATCH 会清空 issue 全部标签」。Use when the user asks to operate Gitee / gitee-cli or manage issues/PRs on enterprise repos; unlisted commands are deliberately absent — discover them by the experience flow instead of guessing.
disable-model-invocation: true
---

# QNW Gitee 操作手册

gitee CLI。所有命令自动复用 `~/.config/gitee/credentials.yml` 中持久化的凭据——**新会话无需重新登录**；只有 logout、删凭据文件或 token 被撤销才需要 `auth login`。

## 占位符约定

- `<enterprise>` — 企业空间地址（path）
- `<repo>` — 仓库路径（path）
- `<owner>/<repo>` — 仓库全名
- `<project-id>` — 企业项目(program)的数字 ID
- `<issue-id>` — issue 的数字 ID（作 parent_id 等用）
- `<编号>` — issue 编号（大小写敏感、不含 #）
- `<PR号>` — PR 数字编号

实际值不硬编码：企业/项目用 API 动态获取（见[企业专属规则](#企业专属规则)）；唯一例外是**工作项状态 id**——无公开枚举入口，本手册为 row_rabbit 企业实测写死（见「工作项状态 issue_state_id」节；其他企业/配置变更时按该节「获取法」现取）。

## 收录边界：只写实测，其余走经验流程

**本手册只收录逐条对真实仓库执行成功（✅）的命令。** 未收录的命令（含 CLI 其他子命令、新版本功能）一律**不臆造用法**。

遇到未收录/新命令时，走**经验流程**让用户自己发现，不要替用户编造：

1. 告诉用户该命令不在手册中，需要现场验证。
2. 先查官方用法：`gitee <命令> --help`。
3. 在**无害对象**上试执行（测试 issue、draft PR、临时分支等），确认真实行为与参数。
4. 实测成功后，**提醒用户更新本 skill**：把该命令及确切用法补录为 ✅；若行为与预期不符也提醒修正。

## 铁律：企业级 PATCH 会清空该 issue 全部标签（✅ 实测，必现）

- 只要对 `/enterprises/<enterprise>/issues/<编号>` 发 **PATCH**——不论 body 传什么字段（state/parent_id/program…）、什么值、是否真的变更了状态——该 issue 的**所有标签即被移除**（操作日志各记一条 `remove_label`）。已实测 `state=progressing / open / closed` 及不带 state 的空字段 PATCH 均复现。
- **不清标签**的操作：`gitee issue assign`、`gitee issue close/reopen`、仓库级 PATCH `/repos/<enterprise>/issues/<编号>`（实测 state / program / priority / parent_id / **body** 均可走仓库级；body 整段替换中文多行正文后标签保留 ✅）。
- **结论：凡对已有标签的 issue 做字段修改，一律用仓库级 PATCH**。企业级 PATCH 只在该字段确无仓库级入口时才用，且 PATCH 后**立即**用 labels POST 补回标签。
- 唯一例外：新创建、尚未来得及打标签的 issue，此时企业级 PATCH 无标签可清。

## 认证 auth（✅）

```bash
printf '%s' "$GITEE_TOKEN" | gitee auth login --with-token   # 登录（token 从 stdin）
```
- 登录用 **Personal Access Token**（Gitee 个人设置页生成）；非交互环境必须 `--with-token` 从 stdin 读。
- 查状态 / 登出：`gitee auth status` / `gitee auth logout`；多主机用 `--hostname` 区分。

## issue 命令（✅ 全部实测）

```bash
gitee issue create -t "标题" -b "描述" --labels bug     # labels 必须是已存在的企业标签
gitee issue view <编号>                                 # 查看详情
gitee issue edit <编号> -t "新标题" -b "新描述"           # 至少一个编辑参数，否则进交互
gitee issue assign <编号> <用户名>                      # 用户名是 login，非显示名；已关闭的 issue 也可指派
gitee issue comment <编号> -b "评论"
gitee issue close <编号>    # state → closed
gitee issue reopen <编号>   # state → open
```
> 状态(state)、项目(program)、优先级(priority)、类型(issue_type)、父子(parent_id) 这些字段 **CLI 的 issue edit 不暴露**，必须走 `gitee api`，见[企业专属规则](#企业专属规则)。

> 新建 issue 的**必做清单**（program、issue_type、标签、指派、创建后自查）见同目录 [`ISSUE-TRACKER-TEMPLATE.md`](ISSUE-TRACKER-TEMPLATE.md)「新建 issue 必做清单」一节，以它为准。

## PR 命令（仅 create ✅ 实测）

```bash
gitee pr create --base master --head test -t "标题" -b "描述"   # 默认 head=当前分支
```
> PR 的 list/view/diff/merge/close/reopen/comment/review/checkout/fetch/edit 等**未实测，不在手册**——需要时走[经验流程](#收录边界只写实测其余走经验流程)。

## gitee api 直连（✅，企业专属字段的入口）

```bash
gitee api <endpoint> [-X METHOD] [-f key=value]... [--body 'JSON'] [-p] [--search 关键词]
gitee api --search "关键字"       # 搜 OpenAPI 端点
gitee api /user                   # 当前用户
```
- `-f key=value` 拼 form 字段（POST/PATCH 用），`--body` 传原始 JSON，`-p` 美化输出。
- **GET 的过滤参数必须写进 URL 查询串**（如 `/repos/<owner>/<repo>/issues?state=all&per_page=100`）；用 `-f` 传，GET 会**静默忽略**（实测传 `state=all` 仍只返回 open 的）。
- endpoint 相对 `/api/v5`；企业命名空间 `/enterprises/{enterprise}/...`，仓库 `/repos/{owner}/{repo}/...`。
- Swagger 可下载本地全文检索：`gitee api /swagger_doc.json -p > gitee_swagger.json`。
- **字段自查（建票后核验，比 `gitee issue view` 全）**：`gitee issue view` 是人读摘要，看不到 parent_id / depth / program 等；用原始 JSON 响应可编程核验必做清单四项：
  ```powershell
  $i = gitee api "/repos/row_rabbit/xyt-pv-ess-hub/issues/<编号>" | ConvertFrom-Json
  $i.issue_type; $i.program.name; ($i.labels.name -join ','); $i.parent_id; $i.depth   # 类型/program/标签/父子一次拉全
  ```
  响应里 `issue_type` 是中文串（任务/需求/缺陷）、`depth=1` 表示已挂为子票（✅ 实测，批量建票后逐张复核可用）。

## 企业专属规则（✅）

`--help` 查不到，全部为真实操作验证。**企业仓库的 issue 挂在企业命名空间下**（url 为 `/enterprises/<enterprise>/...`）。改字段一律走仓库级 PATCH（见**铁律**一节），下文各字段只写取值与命令。

### 项目 program：企业 issue 必须归属项目
- 企业仓库的 issue **必须设置项目（program）**，否则在网页 issue 列表/项目视图中**不可见**（实测有此现象）。
- 获取项目 ID：`gitee api /repos/{owner}/{repo}` 响应的 `programs` 数组。
- 创建时带上：`gitee api /repos/<enterprise>/issues -X POST -f repo=<repo> -f "title=..." -f program=<project-id>`
- 补设/修改：`gitee api /repos/<enterprise>/issues/<编号> -X PATCH -f repo=<repo> -f program=<project-id>`

### issue 类型（需求/任务/缺陷）— 只在创建时生效
- 企业三种类型：**任务**、**需求**、**缺陷**。
- **创建**时指定（传中文标题，ident/ID 无效——实测 ident/数字 ID 都回落到默认类型）：
  ```bash
  gitee api /repos/<enterprise>/issues -X POST -f repo=<repo> -f "title=..." -f issue_type="需求"
  ```
- **修改**已有 issue 类型：API 静默忽略（企业级/仓库级 PATCH、ident/ID/中文都试过无效）→ 只能网页端改。

### 优先级 priority
- 取值：**0 无优先级、1 不重要、2 次要、3 主要、4 严重**（企业级 PATCH 传 priority 静默无效，必须仓库级）。
  ```bash
  gitee api /repos/<enterprise>/issues/<编号> -X PATCH -f repo=<repo> -f priority=4
  ```

### 状态 state（仓库级）
```bash
gitee api /repos/<enterprise>/issues/<编号> -X PATCH -f repo=<repo> -f state=progressing
# state: open | progressing | closed
```
- **`state=progressing` 只对「任务」类型生效**——progressing 底层映射任务工作流的「进行中」状态；对「缺陷/需求」类型**静默忽略**（HTTP 200 无报错、`updated_at` 不变、操作日志无记录，✅ 实测 IKE069 缺陷、IKE0Y3 需求）。
- 缺陷/需求置中间态须走 `issue_state_id`（见下节）；`gitee issue close/reopen` 对所有类型可用（缺陷 close → 已完成）。

### 工作项状态 issue_state_id（缺陷/需求置中间态，✅ 实测）
- 机制：企业 issue 的真实状态是**工作项状态**（响应里的 `issue_state_detail`），**按 issue_type 分状态集**；`state`(open/progressing/closed) 是不同步的兼容字段（实测：置「开发中/修复中」后顶层 `state` 仍为 open，仅「已完成」→ closed）。
- 官方文档无此参数：OpenAPI swagger_doc.json 5.4.92 与官方 mcp-gitee 均只暴露 `state` 枚举，`IssueState` 仅是响应只读对象（✅ 核对）。
- 命令（企业级 PATCH → **会清空标签，成功后立即用 labels POST 补回**）：
  ```bash
  gitee api /enterprises/<enterprise>/issues/<编号> -X PATCH -f issue_state_id=<状态id>
  ```
- **跨类型设置报 403**：`状态不属于该任务的工作项类型`（✅ 实测：需求设 441563/441570、缺陷设 441563 均 403）。
- **本企业（row_rabbit）状态 id 写死**（状态 id 无公开枚举入口，只能实测；serial 顺序：待办 0 → … → 修复中 8）：
  - 任务：待办 441562 → **进行中 441563**（即 `state=progressing`）→ 已完成 441564
  - 缺陷：待办 441562 → **修复中 441570**（领取置位用）→ 已完成 441564
  - 需求：待办 441562 → **开发中 441568**（领取置位用）→ 已完成 441564
  - 其余实测存在：已拒绝 441565、已确认 441566、设计中 441567、已验收 441569、已取消 441571
  - 遇 403 或状态名对不上 = 企业状态配置已变/非本企业 → 按「获取法」现取，勿沿用写死值。
- **状态 id 获取法**（无枚举 API 时的四条路）：
  1. 找**同类型**、已处于目标状态的 issue，读详情响应 `issue_state_detail.id`；
  2. 网页手动改一次后，查操作日志 `/repos/<enterprise>/issues/<编号>/operate_logs` 的 `after_change_id`（`change_issue_state` 记录）；
  3. 企业级列表 `state=all` 每项含 `issue_state_detail`，按类型聚合即「该类型用过的状态 id」；
  4. 测试 issue 上逐 id 试探：200 = 属于该类型，403 = 不属于。

### 父子任务 parent_id
```bash
gitee api /repos/<enterprise>/issues/<子编号> -X PATCH -f repo=<repo> -f parent_id=<父的issue-id>
# 设置与解除（parent_id=0）仓库级 PATCH 均可用，标签保留
```
- `parent_id` 是**父任务的数字 id**（列表响应的 `id` 字段，不是字母编号）；设置后子任务 `depth=1`、出现 `parent_url`。
- **父与子的类型必须一致**，否则 403：`子工作项的任务属性必须与父工作项一致`。
- **环被拒绝** 403：`父任务和当前任务已存在子孙关系`。
- **解除**：`-f parent_id=0`（depth 归 0）。解除中间节点**不会级联**其子孙。
- 多层嵌套（depth=2 孙任务）实测可行。
- **反查：拿到数字 id 想找字母编号**——`gitee issue view <数字id>` 会 404（数字 id 不是编号）；用**列表**端点 `gitee api "/repos/<enterprise>/<repo>/issues?state=all&per_page=100"` 扫 `id`，字母编号在响应的 **`number`** 字段（详情/列表响应均无 `ident` 字段；列表项的 `html_url` 尾段也是编号，可作交叉验证）。父任务反查（如从子票 `parent_id` 找父编号）只能靠这个列表扫。

### 标签体系：企业仓库的 issue 只能用企业标签
- **企业标签**（`/enterprises/{enterprise}/labels`）：企业共享、所有仓库通用，**issue 只能挂这种**；该端点**只读**——POST 405，企业级没有创建入口。
- **wayfinder 流程标签已存在**（✅ 实测，直接挂无需新建）：`wayfinder-map` 与 `wayfinder-<type>`（type ∈ `research`/`prototype`/`grilling`/`task`；Gitee 标签名不支持 `:` → 用连字符 `-`）。
- **创建的真实入口是仓库级 labels 端点**：GET `/repos/{enterprise}/{repo}/labels` 返回的即企业标签（url 指向企业命名空间）；Swagger 把该端点写成"创建仓库任务标签"：
  ```bash
  gitee api /repos/<enterprise>/<repo>/labels -X POST -f "name=<新标签>" -f color=1083d6   # color 为 6 位 hex
  ```
  - 需**企业管理员**：普通成员实测 401 `新添企业仓库任务标签需要企业管理员权限`；仅验证到权限门禁，管理员跑通前不标 ✅。
  - 无管理员账号时，建标签仍走网页端企业设置。
- **API 返回的 `label.color` 不带 `#` 前缀**（形如 `"1083d6"`）：只是数据值；但直接写进网页 CSS 会被浏览器当无效值丢弃（✅ 实测，表现为样式不生效）。渲染前补 `#`。
- **仓库级 project_labels**（`/repos/{o}/{r}/project_labels`）：仓库专属，**无法挂到 issue**（实测报权限错）。
- issue 打/换/删企业标签（**标签名必须是已存在的企业标签**）：
  ```bash
  gitee api /repos/<enterprise>/<repo>/issues/<编号>/labels -X POST --body '["bug","测试"]'   # 追加
  gitee api /repos/<enterprise>/<repo>/issues/<编号>/labels -X PUT  --body '["bug"]'          # 全量替换
  gitee api /repos/<enterprise>/<repo>/issues/<编号>/labels/<名称> -X DELETE                    # 删单个
  gitee api /repos/<enterprise>/<repo>/issues/<编号>/labels -X DELETE                           # 删全部
  ```
- 仓库级 project_labels 的增删（中文名稳定；**单字母如 A/B/C 会被系统规整**并可能撞内置语言标签，避免）：
  ```bash
  gitee api /repos/<enterprise>/<repo>/project_labels -X POST --body '["示例标签1","示例标签2"]'
  gitee api /repos/<enterprise>/<repo>/project_labels -X DELETE --body '["示例标签1"]'
  ```

## 环境与坑（Windows 实测）

- **Git Bash 路径改写**：MSYS 会把以 `/` 开头的 endpoint 参数改写成 Windows 路径（`/user` → `C:/Program Files/Git/user`），所有 `gitee api` 请求 404。命令前加 `MSYS_NO_PATHCONV=1`（✅ 实测）。
- **PowerShell 引号**：`-j=字段` 这类参数需整体加引号 `"-j=number,title"`，否则报 ParserError；`--body '["a","b"]'` 单引号包裹即字面量；子命令不要拼成单个字符串参数传；`&` 拼接与转义易错，长命令尽量整条字符串直跑。
- **长中文多行正文（body）传参**：用 PowerShell here-string（`@'...'@`）存中文多行文本，再**明文直传** `-f "body=$var"` 即正确存储（含中文、`#`、换行，✅ 实测）。**禁止**预先 `[uri]::EscapeDataString()` 或任何 URL 编码——实测 Gitee 会把 `%23%23%20...` 原样存为正文（6 票踩坑实证），只能事后逐张 PATCH 修复。
- issue 编号是大小写敏感的字母数字串、不带 #；PR 编号是纯数字。

## 编程调用：Node 子进程（✅ 实测，dsh-gitee-board 看板全程使用）

```js
spawn('gitee', ['api', endpoint], {
  windowsHide: true,
  env: { ...process.env, MSYS_NO_PATHCONV: '1' },  // 实测无害，防其他入口再走 Git Bash 转换
  stdio: ['ignore', 'pipe', 'pipe'],
})
```

- `gitee api` 的 stdout 是**原始紧凑 JSON，可直接 JSON.parse**（✅ 大量调用实测）；`-p` 仅人读美化。
- Node 直接 spawn 不经过 MSYS，**无路径改写问题**；stdout 无 GBK 乱码——乱码只发生在 `gitee issue list --json` 输出经 Git Bash 管道/控制台时（UTF-8 被按 GBK 转码）。
- 参数数组传递、无 shell，endpoint 拼进 URL 查询串时注意自行 URL 编码值。
- 非零退出 = 失败；错误文本中 `401`/`unauthorized` 即凭证失效（启发式匹配，未做系统性验证）。
- 列当前账号全部仓库（个人/组织/企业三命名空间一把抓，看板仓库列表用它）：`gitee api '/user/repos?type=all&sort=updated&per_page=100&page=N'` 翻页（per_page 最大 100）；企业空间从响应里 `namespace.type=="enterprise"` 的 `namespace.path` 归并，`/user/enterprises` 对部分账号返回 `[]` **不可依赖**（✅ 实测）。
