# my-skills

个人 skill 集合，供 agent（dsh、Claude Code 等）按需加载。每个 skill 是自包含的一个目录，包含说明文档与可选的脚本/资源。

## 目录结构

```
my-skills/
├── README.md
└── skills/
    ├── _template/          # 新建 skill 时复制这个目录
    │   └── SKILL.md
    └── <skill-name>/       # 一个 skill 一个目录
        ├── SKILL.md        # 必需：入口说明
        └── ...             # 可选：脚本、模板、参考资料
```

## skill 约定

- 目录名用小写加连字符：`code-review`、`writing-for-agents`。
- `SKILL.md` 必须以 YAML frontmatter 开头，两个字段必填：

  ```yaml
  ---
  name: code-review
  description: 何时使用该 skill，一句话写清触发条件。
  ---
  ```

- `description` 决定 agent 会不会自动加载该 skill，写"什么时候用"而不是"这是什么"。
- 正文用祈使句写给 agent 看的操作指令，避免空泛描述。
- 大文件（图片、数据）放 `assets/`，引用时用相对路径。

## 新增一个 skill

1. 复制模板：`cp -r skills/_template skills/my-new-skill`
2. 改 `name` 与 `description`，补正文。
3. 本地验证：让 agent 读一遍 `SKILL.md`，确认它能照做。
4. 提交：`git add skills/my-new-skill && git commit -m "add: my-new-skill"`

## 在本机挂载

dsh 从 `$DSH_HOME/skills/` 读取 skill。仓库克隆到本地后，用目录联接或符号链接挂进去：

```powershell
# 单个 skill
New-Item -ItemType SymbolicLink -Path "$env:DSH_HOME\skills\my-new-skill" -Target "F:\PROJECT\my-skills\skills\my-new-skill"
```

Windows 建符号链接需要开发者模式或管理员权限；没有权限时用 `mklink /J` 做目录联接。

## 许可

本项目采用 **GNU AGPL-3.0**（GNU Affero General Public License v3.0），法律全文见 [LICENSE](LICENSE)，官方说明见 <https://www.gnu.org/licenses/agpl-3.0.html>（以英文原文为准，中文译文仅供参考）。

Copyright (C) 2026 QianNianwen

三条核心义务：

- **必须开源**：分发本项目或其修改版时，必须提供完整的对应源码，并以 AGPL-3.0 授权。
- **网络使用也算分发**：把修改版部署成网络服务供他人使用，也必须向使用者提供源码。这是 AGPL 相对 GPL 的关键差别，也是"要用就必须开源"最硬的约束。
- **保留声明**：不得移除版权声明与许可证文本，修改版须显著标注改动内容与日期。

**允许商业使用。** 商业方若不愿承担上述开源义务，只能另行取得商业授权 —— 换句话说，想商用又不想开源，是不被允许的。

SPDX 标识：`AGPL-3.0-only`（若允许他人按未来版本再授权，写 `AGPL-3.0-or-later`）。

### 与其他许可证的关系

- 相比之前的 CC BY-NC-SA：AGPL 管住了网络服务场景，且**允许商用**，因此是 OSI 认证的真正开源许可证，GitHub 也能正确识别。
- AGPL-3.0 与 GPL-3.0 兼容，两者代码可组合，组合后的整体按 AGPL-3.0 分发（FSF 已明确）。
