# 安装指南（给 AI 阅读）

> 目标：把本仓库中的 DeepSeek Harness Agent preset **「标准模式+gitbash」（id: `cordis-gitbash`）** 安装到目标机器上。
> 阅读者是一个 AI 助手。请严格按照以下步骤执行：**每完成一步，先核对「预期结果」是否符合，再进入下一步**。任何一步结果异常，先看「常见问题排查」，不要跳过。

---

## 0. 任务总览

你要做四件事：

1. 确认目标机器满足前置条件。
2. 把本仓库的文件复制到 DSH 用户 preset 根目录下的 `cordis-gitbash/` 文件夹。
3. 验证 preset 能被 DSH 识别。
4. 新建一个会话，验证 `bash`（Git Bash）工具可用。

安装完成后，Agent 会默认**优先使用 Git Bash 而非 PowerShell** 执行 shell 命令（Windows 下）。

---

## 1. 前置条件检查

逐项确认，任一不满足先停下来解决：

| 项目 | 要求 | 检查方法 |
| --- | --- | --- |
| DeepSeek Harness (dsh) | 已安装且能运行 | 终端执行 `dsh --version`，能看到版本号 |
| Node.js | 已安装（dsh 运行依赖） | `node --version` |
| Git Bash（Windows） | 已安装 | `where bash` 能找到 `bash.exe`（常见路径 `C:\Program Files\Git\bin\bash.exe`） |
| 用户 preset 目录 | 存在或可创建 | `$DSH_HOME/.agent-presets`（默认 `~/.dsh/.agent-presets`） |

> 说明：`$DSH_HOME` 默认是 `~/.dsh`（Windows 下即 `C:\Users\<用户名>\.dsh`）。如果环境变量 `DSH_HOME` 已设置，请以它为准。

**预期结果**：以上各项都通过；`~/.dsh/.agent-presets` 目录存在（或你能创建它）。

---

## 2. 确定目标路径

- 源目录：本仓库的根目录（含 `preset.yml`、`agent.cordis.yml`、`tool-gitbash-v2.mjs`、`skills/`、`images/`）。
- 目标目录：`<DSH_HOME>/.agent-presets/cordis-gitbash/`。

> ⚠️ 绝对不要复制到部署自带的 `agent-presets` 目录（位于 dsh 安装目录下）——那里随版本升级会被覆盖。**只放到 `~/.dsh/.agent-presets/` 用户根目录。**

---

## 3. 复制文件

把仓库里的 preset 内容复制为 `cordis-gitbash` 目录。按目标机器平台选择命令。

### Windows（PowerShell）

```powershell
# 假设仓库已经 clone/解压到当前目录
New-Item -ItemType Directory -Force "$env:USERPROFILE\.dsh\.agent-presets"
Copy-Item -Recurse -Force .\preset.yml .\agent.cordis.yml .\tool-gitbash-v2.mjs "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash\"
Copy-Item -Recurse -Force .\skills "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash\skills"
Copy-Item -Recurse -Force .\images "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash\images"
```

> 先建目录再复制，避免 `Copy-Item` 自动创建出来的目录层级不对。`skills/` 和 `images/` 必须整体复制成目标下的同名子目录。

### Linux / macOS（bash）

```bash
mkdir -p ~/.dsh/.agent-presets
cp -r . ~/.dsh/.agent-presets/cordis-gitbash
```

> 若仓库目录下还带有 `.git/`，可用 `cp -r` 后再删除 `~/.dsh/.agent-presets/cordis-gitbash/.git`，preset 目录不需要 git 元数据。

**预期结果**：`~/.dsh/.agent-presets/cordis-gitbash/` 下出现如下文件：

```
cordis-gitbash/
├── preset.yml
├── agent.cordis.yml
├── tool-gitbash-v2.mjs
├── skills/
│   ├── cordis-plugin-development/SKILL.md
│   └── editing-cordis-compositions/SKILL.md
└── images/
    ├── model-select.png
    └── chat-example.png
```

---

## 4. 校验文件就位

执行校验命令，逐项核对：

```bash
# Windows PowerShell
Test-Path "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash\preset.yml"
Test-Path "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash\agent.cordis.yml"
Test-Path "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash\tool-gitbash-v2.mjs"
Test-Path "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash\skills\cordis-plugin-development\SKILL.md"

# Linux / macOS
ls ~/.dsh/.agent-presets/cordis-gitbash/
```

**预期结果**：所有路径都存在，`preset.yml` 首行为 `name: 标准模式+gitbash`。

---

## 5. 确认 preset 能被识别

DSH 的模式选择器会扫描 `~/.dsh/.agent-presets/` 下的每个目录。

1. 打开 DeepSeek Harness Web GUI（默认 `http://127.0.0.1:3080`）。
2. 新建会话（或打开会话顶部的模式下拉框）。
3. 在模式列表中应能看到 **「标准模式+gitbash」**，描述为：*基于 cordis 预设，加入「优先使用 Git Bash 而非 PowerShell」的提示词偏好，并注册一个真正运行 Git Bash 的 bash 工具（Windows 下生效，非受限运行）。*

> 参考效果见仓库内 `images/model-select.png`（列表最后一项、带对勾的即为该模式）。

**预期结果**：下拉框里能看到该模式并选中。

---

## 6. 端到端验证 `bash` 工具

这是最关键的验收步骤——确认 `bash` 工具真的能用，而不是 PowerShell。

1. 用 **标准模式+gitbash** 新建一个会话。
2. 让 Agent 执行一条简单的 shell 命令，例如：
   > 「执行 `pwd && ls -la`，看看当前目录有什么」

3. **预期结果**：
   - 对话中出现 `bash` 工具调用（工具名应为 **Bash**，而不是 **PowerShell** / **Pwsh**）。
   - 命令正常返回 `pwd` 的路径和目录列表。
   - 会话不报 `Error: cannot get property "timer" without inject`。

> 参考效果见仓库内 `images/chat-example.png`（红框处即 `bash` 工具调用）。

如果 Agent 仍然倾向用 PowerShell，提示词段落会要求它改用 `bash`；若问题依旧，检查第 8 节的排查项。

---

## 7. 完成清单

- [ ] 前置条件全部满足（dsh、Node、Git Bash、preset 目录）
- [ ] 文件已复制到 `~/.dsh/.agent-presets/cordis-gitbash/`（不是部署自带目录）
- [ ] `Test-Path` / `ls` 校验通过
- [ ] GUI 模式选择器中能看到并选中「标准模式+gitbash」
- [ ] 新会话里 `bash` 工具调用成功，无 `timer` 报错

全部打勾即安装成功。✅

---

## 8. 常见问题排查

| 现象 | 原因与处理 |
| --- | --- |
| 报 `Error: cannot get property "timer" without inject` | 使用了旧版 `tool-gitbash.mjs`（v1）。确认安装的是 `tool-gitbash-v2.mjs`，且 `agent.cordis.yml` 中该行写的是 `name: ./tool-gitbash-v2.mjs`。若服务器进程运行了很久，重命名插件文件可绕开 ESM 模块缓存；或重启 `dsh web` 服务器后重试。 |
| 找不到 bash 工具 / 工具不存在 | 该 preset 的 `bash` 工具仅在 **Windows** 上注册（`agent.cordis.yml` 中 `disabled: !!js process.platform !== 'win32'`）。Linux/macOS 请用系统自带的 `bash` 工具。 |
| 报 Git Bash 未找到 | 未安装 Git for Windows，或 `bash.exe` 不在常见路径。安装 Git for Windows，或确认 `E:\Program Files\Git\bin\bash.exe`、`C:\Program Files\Git\bin\bash.exe` 存在。 |
| 模式列表中看不到「标准模式+gitbash」 | preset 目录名或位置不对（必须叫 `cordis-gitbash` 且在用户根目录），或 GUI 需要刷新/重开会话。 |
| bash 命令被沙箱拒绝 | 该工具设计为**非受限运行**（Git Bash 无法在 harness 文件沙箱中启动）。若遇到沙箱相关报错，确认安装的 `tool-gitbash-v2.mjs` 是完整的最新版。 |
| 复制时目录层级不对 | Windows 下先 `New-Item` 建好目标目录再 `Copy-Item`；`skills/`、`images/` 用 `-Recurse` 整体复制。 |

---

## 9. 其他信息

- 想了解这个 preset 为什么存在（PowerShell 出错 → 改用 Git Bash）：见仓库根目录 `README.md` 的「为什么做这个插件」。
- 想修改这个 preset：直接编辑 `~/.dsh/.agent-presets/cordis-gitbash/agent.cordis.yml`，改动对新会话生效。
