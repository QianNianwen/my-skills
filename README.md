# my-skills

个人 skill 集合，供 dsh、Claude Code 等 agent 按需加载。每个 skill 是一个自包含目录：`SKILL.md` 是入口，脚本、模板等资源放在同目录下。

## 技能

| 技能 | 调用 | 用途 |
| --- | --- | --- |
| [`anti-drift`](skills/anti-drift/SKILL.md) | `/anti-drift` | 以交付物为边界，约束取证范围并及时收敛任务。 |
| [`qnw-gitee`](skills/qnw-gitee/SKILL.md) | `/qnw-gitee` | Gitee / gitee-cli 操作手册，含企业仓库 issue/PR 的实测命令与专属规则。 |
| [`_template`](skills/_template/SKILL.md) | — | 新建 skill 的模板，不挂载、不直接调用。 |

两个正式技能都设置了 `disable-model-invocation: true`，只能由人显式调用（dsh 中输入上表的 `/...`），不会自动加载。`qnw-gitee` 中的工作项状态 id 等企业专属值是 row_rabbit 的实测结果，换企业时按该技能「状态 id 获取法」现取，不要照抄。

## 目录结构

```
my-skills/
├── README.md
├── LICENSE
└── skills/
    ├── _template/
    │   └── SKILL.md
    ├── anti-drift/
    │   └── SKILL.md
    └── qnw-gitee/
        ├── SKILL.md
        ├── ISSUE-TRACKER-TEMPLATE.md
        └── agents/
            └── openai.yaml
```

## 挂载到本机

dsh 从 `$DSH_HOME/skills/` 读取 skill。克隆仓库后，按需为单个技能建立链接；`_template` 无需挂载。

```powershell
git clone https://github.com/QianNianwen/my-skills.git F:\PROJECT\my-skills

$skill = "anti-drift"
New-Item -ItemType SymbolicLink `
  -Path "$env:DSH_HOME\skills\$skill" `
  -Target "F:\PROJECT\my-skills\skills\$skill"
```

Windows 建符号链接需要开发者模式或管理员权限；没有权限时改用目录联接：

```powershell
cmd /c mklink /J "%DSH_HOME%\skills\anti-drift" "F:\PROJECT\my-skills\skills\anti-drift"
```

macOS / Linux：

```bash
ln -s /path/to/my-skills/skills/anti-drift "$DSH_HOME/skills/anti-drift"
```

Claude Code 等其他 agent 按其 skill 目录约定放置或链接，技能内容本身不变。

## skill 约定

- 目录名用小写加连字符：`code-review`、`writing-for-agents`；`_template` 是唯一例外。
- `SKILL.md` 以 YAML frontmatter 开头，`name`、`description` 必填；`name` 是调用名，建议与目录名一致：

  ```yaml
  ---
  name: code-review
  description: 何时使用该 skill，一句话写清触发条件。
  ---
  ```

- 默认是 model-invoked：`description` 常驻上下文，决定 agent 是否自动加载，写“什么时候用”而不是“这是什么”。
- 只由人工显式调用时，加 `disable-model-invocation: true`；此时 `description` 转为给人看的一句话摘要，不再参与自动发现。
- 正文用祈使句写给 agent，步骤写清完成判据；大文件（图片、数据）放 `assets/`，引用时用相对路径。
- 平台附加的展示元数据可放 `agents/`（如 `agents/openai.yaml`），不影响 dsh 加载。

## 新增一个 skill

1. 复制模板：`cp -r skills/_template skills/my-new-skill`
2. 改 `name` 与 `description`，补正文；手动调用的技能加上 `disable-model-invocation: true`。
3. 本地挂载后让 agent 读一遍 `SKILL.md`，确认它能照做。
4. 提交：`git add skills/my-new-skill && git commit -m "add: my-new-skill"`

## 许可

本项目采用 **GNU AGPL-3.0**（SPDX：`AGPL-3.0-only`），法律全文见 [LICENSE](LICENSE)，官方说明见 <https://www.gnu.org/licenses/agpl-3.0.html>。

- 分发本项目或修改版，或将其作为网络服务提供，须以 AGPL-3.0 提供完整对应源码，并保留版权与许可声明。
- 允许商业使用；不愿承担上述开源义务的，需另行取得商业授权。
- 修改版须显著标注改动内容与日期。

Copyright (C) 2026 QianNianwen
