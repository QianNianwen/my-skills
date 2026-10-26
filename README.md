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

未附带 LICENSE，默认保留所有权利。若要允许他人复用，补一个 LICENSE 文件（如 MIT）。
