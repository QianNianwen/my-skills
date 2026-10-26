# Issue tracker: Gitee(企业仓库)

> 本文件是仓库级 `docs/agents/issue-tracker.md` 的**去敏完整模版**（占位符 `<owner>/<repo>/<enterprise>/<enterprise-id>/<project-id>/<编号>`，无真实信息）。`/setup-matt-pocock-skills` 在 Gitee 企业仓库生成 `docs/agents/issue-tracker.md` 时，应以本模版为参照实例化（填入真实仓库/企业/program 与 triage 标签），再补本仓库特例。常用命令的权威来源是 `qnw-gitee` 技能。

本仓库的 issue 与 spec 存放于 **Gitee 企业 Issue**。仓库 `<owner>/<repo>`(<仓库一句话简介>)隶属企业「<企业名>」(企业 path `<enterprise>`,enterprise_id `<enterprise-id>`),issue 挂在该企业命名空间下,而不是普通个人仓库。

## 新建 issue 必做清单(总是成立)

创建任何 issue 后,逐项核对,缺一不可(漏打标签是本仓库实测踩过的坑):

1. **必设 program**:创建时传 `program=<project-id>`(<项目名>),否则在网页 issue 列表/项目视图不可见。
2. **必选 issue_type**:创建时传中文 `issue_type="任务|需求|缺陷"`;事后无法修改,只能在网页端改。
3. **必打标签**(挂企业标签,名称必须已存在,见 qnw-gitee 技能「标签体系」):
   - **至少一个 triage 标签**:待分流 / 待补充信息 / 可直接实施 / 需人工处理 / 不处理(按 issue 角色选一个,不要留空);
   - 业务域标签(产品线/技术栈/版本等)适用时一并打。
4. **必指派负责人**(适用时):`gitee issue assign <编号> <login>`。
5. **必加标题子票编号前缀**(仅子票):新建 issue 若挂在某父任务(parent_id)下,标题须以 `[n] ` 开头(如 `[3] 实施:...`),n 为**同一父任务下所有子票间唯一**的递增序号;无父任务的独立 issue 不加。编号规则见「issue 编号与标题前缀」节。
6. **创建后自查**:用 `gitee issue view <编号>` 复核 program / issue_type / 标签 / 指派四项齐全,再宣称完成。

> 常用命令清单见 qnw-gitee 技能;本文件是企业专属规则与必做清单的权威说明。

## 领取 issue(Claim)(总是成立)

**领取 = 指派 + 置中间态(任务:进行中 / 缺陷:修复中 / 需求:开发中)**,是同一个动作的两半,做完指派必须紧接着改状态,缺一不可:

1. `gitee issue assign <编号> <login>`(login,非显示名);
2. 立即按 **issue 类型**置状态——置位方式与类型绑定,不可混用:
   - **任务**:**仓库级 PATCH** `state=progressing`(`gitee api /repos/<enterprise>/issues/<编号> -X PATCH -f repo=<repo> -f state=progressing`,不清标签);
   - **缺陷 / 需求**:仓库级 `state=progressing` 会被**静默忽略**(200 无变化),必须**企业级 PATCH** `issue_state_id=<该类型中间态id>`(缺陷=修复中、需求=开发中;具体 id 查 qnw-gitee 技能「工作项状态 issue_state_id」节)——企业级 PATCH 会清空标签,PATCH 后**立即**用 labels POST 补回原标签;
3. **向上同步父任务**:领取的是子任务时,对其父任务逐级向上执行联动——父任务按**其自身类型**用第 2 步同款方式置中间态;父任务尚无负责人时,把**第一个领取子任务的负责人**填入父任务(`gitee issue assign`)。父任务表示「其下有工作进行中」,不要求父任务本身有人动手。父任务的父任务同样处理,直到没有父任务为止。

领取后的状态是并发会话的占用信号:assigned 且处于中间态(进行中/修复中/开发中)才代表「有人在做了」,只指派不改状态等于没领取。收尾时:工作完成并提交后 `gitee issue close <编号>`;中途停止不关闭,状态保持中间态。父任务在其全部子任务关闭且自身无其他未完成工作时才 close。

## 权威命令来源

**本文件只写规则与步骤,不收录命令大全。** 需要任何命令时,查 `qnw-gitee` 技能——它是唯一权威命令参考,只收录对真实仓库**实测成功**的命令。所有操作使用 `gitee` CLI(凭据持久化于 `~/.config/gitee/credentials.yml`,新会话无需重登),CLI 未暴露的字段走 `gitee api`。手册未收录的命令**不得臆造用法**:先 `gitee <cmd> --help`,在无害对象上实测,成功后再提醒更新 `qnw-gitee` 技能。上两节内联的命令是步骤原语,属于流程本身,不视为重复。

## 企业专属规则(总是成立)

- issue **必须归属项目(program)**,否则在网页 issue 列表/项目视图中不可见。
- program id 从 `gitee api /repos/<owner>/<repo>` 响应的 `programs[]` 取。
- issue 只能挂**企业标签**(仓库级 `project_labels` 无法挂到 issue)。创建入口见 `qnw-gitee` 技能「标签体系」(仓库级 labels 端点、需企业管理员;企业级 labels 只读、POST 405)。
- issue 类型(任务/需求/缺陷)仅在**创建时**生效(传中文名,ident/ID 无效);之后改类型 API 静默忽略,只能网页端改。
- 优先级 `priority` 取值 0-4(0 无、1 不重要、2 次要、3 主要、4 严重),且**企业级 PATCH 会静默忽略**,必须走仓库级。
- 状态按 **issue_type 隔离**、置位方式与类型绑定(领取步骤见上节):任务的中间态是「进行中」(仓库级 `state=progressing` 即映射它);缺陷是「修复中」、需求是「开发中」——后两者仓库级 `state=progressing` 静默忽略,必须企业级 PATCH `issue_state_id=<该类型中间态id>`(官方 API 无此参数,只接受工作流状态 id;跨类型设 id 报 403 `状态不属于该任务的工作项类型`)。`issue_state_detail`(工作流状态)与顶层 `state`(open/progressing/closed)是两套不同步的字段。
- **企业级 PATCH 会清空该 issue 全部标签**(实测):只要对 `/enterprises/<enterprise>/issues/<编号>` 发 PATCH——不论传什么字段、什么值、是否真的变更了状态——该 issue 的所有标签即被移除(`gitee issue assign` / `close` / `reopen` 以及**仓库级** PATCH 不清)。凡对已有标签的 issue 做字段修改,一律用仓库级 PATCH(`gitee api /repos/<enterprise>/issues/<编号> -X PATCH -f repo=<repo> ...`;实测 state/program/priority/parent_id 仓库级均支持);确需企业级 PATCH 时,PATCH 后立即用 labels POST 补回标签。完整机制见 qnw-gitee 技能「企业专属规则」。
- 父子任务:父与子类型必须一致否则 403;解除用 `parent_id=0`;解除中间节点不会级联子孙。
- PowerShell 传参:`--body '["a","b"]'` 用单引号包裹为字面量;`-j=字段` 这类参数需整体加引号。

## issue 编号与标题前缀(总是成立)

**系统编号**:Gitee 为每个 issue 自动分配系统编号——全大写字母+数字混排(如 IKDD9A,历史上为 I 开头,不保证未来前缀)。识别规则:对话/正文/提交中出现的这种代号优先疑为 issue 编号,大小写敏感、不含 `#`、非 commit 哈希;直接 `gitee issue view <编号>` 验证——查得到即 issue,查不到再当普通代号。PR 编号是纯数字。系统编号**不可自选**。

**标题子票前缀**:新建 issue 若挂在某父任务(`parent_id`)下,标题须以 `[n] ` 开头(如 `[3] 实施:...`),n 为**同一父任务下所有子票间唯一**的递增序号——从 1 起,新子票取当前最大 n+1,已关闭的不复用。不同父任务各自独立从 1 起。排序即创建顺序,方便一眼看出某父任务下有哪些票、缺哪张。无父任务的独立 issue 标题**不加**前缀。前缀只用于标题、不用于编号;命令一律用系统编号,如 `gitee issue view IKDD9A`。

> 标题前缀与父子结构配合:父任务标题本身可含 `[n]`(当它也是某更上级的子票时);顶层地图/规格 issue 不加。查询某父任务已有子票以确定下一个 n:`gitee api /repos/<enterprise>/<repo>/issues?state=all&per_page=100`,过滤 `parent_id` 为该父的数字 id,取 `title` 中 `[n]` 最大值。

仓库信息从 `git remote -v` 推断,命令在克隆目录内运行。

## 当技能说 "publish to the issue tracker"

创建一条 Gitee 企业 issue(必须带 `program=<project-id>`;需要时在创建时指定 `issue_type`)。完整创建流程见 qnw-gitee 技能。

## 当技能说 "fetch the relevant ticket"

运行 `gitee issue view <编号>`。

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue on this repo's issue tracker, labelled `wayfinder-map`; its **tickets** are child issues of the map.

- **Map**: a single issue labelled `wayfinder-map`, holding the Destination / Notes / Decisions-so-far / Fog body.
- **Child ticket**: an issue whose `parent_id` is the map's **数字 id**(`gitee api /repos/<enterprise>/<repo>/issues/<编号>` 响应的 `id`,非字母编号;父子类型须一致)。Labels:`wayfinder-map` 与 `wayfinder-<type>`(连字符,非冒号——Gitee 标签名不支持 `:`,实测:type ∈ `research`/`prototype`/`grilling`/`task`)。Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: Gitee issue 无原生依赖关系,用正文顶部 `Blocked by:<编号1>、<编号2>` 行约定(该行随关闭清除)。A ticket is unblocked when every blocker is closed.
- **Frontier query**: `gitee api "/repos/<enterprise>/<repo>/issues?state=open&per_page=100"` 拉 open issues,按响应的 `parent_id` = 地图的数字 id 过滤出子 ticket(列表响应含 `parent_id`,✅ 实测;**企业级列表无法按 program 过滤**——GET 传 `-f` 被静默忽略、`?program=` 查询串被忽略,勿用),剔除仍有 open `Blocked by` 行或已有 assignee 者;first in map order wins.
- **Claim**: `gitee issue assign <编号> <login>`,then mark in-progress per the issue type（任务:仓库级 PATCH `state=progressing`;缺陷/需求:企业级 PATCH `issue_state_id=<该类型中间态id>`）——置位步骤与补标签细则见上文「领取 issue(Claim)」节;the session's first write. Claimed = assigned + 中间态(进行中/修复中/开发中): assignee is working it, concurrent sessions skip it.
- **Resolve**: post the answer as a resolution comment,then **close only on the human's explicit confirmation for HITL tickets**(grilling/prototype;原型=评审通过),while AFK tickets (research/task) close right after the answer is posted;then append a context pointer(gist + link)to the map's Decisions-so-far. The human's confirmation is part of HITL resolution — the agent never implies it by posting the answer alone.
- **术语补录**(Resolve 收尾,关闭前完成):盘点本轮问答/产出(design doc、comment、地图行)引入的领域术语,对照仓库根 `CONTEXT.md`:已在的不动;新增的按分组就地补录并提交;无新术语则跳过并在 resolution comment 末尾注明「术语:无新增」。完成标准 = 关闭时 CONTEXT.md 已含本轮全部新领域词(或已注明无)。设计/规格类票尤其适用;HITL 票的人工确认前做完,确认后不再改术语。
