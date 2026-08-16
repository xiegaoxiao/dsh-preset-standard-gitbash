# DSH Agent Preset：标准模式+gitbash（cordis-gitbash）

基于 DeepSeek Harness 官方 `cordis` 预设的自定义 Agent preset，在标准编码代理能力之上增加 **优先使用 Git Bash 而非 PowerShell** 的提示词偏好，并注册一个真正运行 Git Bash 的 `bash` 工具（Windows 下生效，非受限运行）。

## 特性

- 保留 `cordis` 预设的全部能力（自我引用 Cordis 工具集、composition 编辑技能、delegation / workflow / goal / plan-mode 等）。
- 新增系统提示词段落：明确要求 Agent 优先使用 `bash`（Git Bash）而非 `pwsh`，并使用 POSIX 风格命令。
- 注册本地 `bash` 工具：通过 `ctx.subprocess` 直接启动 Git Bash（`bash -c`），支持 `workdir`、`timeoutMs`、输出截断与 spill 文件。
- 该工具仅在 Windows 下生效；Git Bash（MSYS2）无法在 harness 文件沙箱中启动，因此工具**非受限运行**（与普通终端一致）；`pwsh` 保持沙箱化。

## 文件结构

```
dsh-preset-standard-gitbash/
├── preset.yml               # 模式显示名称与描述（标准模式+gitbash）
├── agent.cordis.yml         # 该 preset 的 Cordis composition（插件行）
├── tool-gitbash-v2.mjs      # 本地 Git Bash 工具插件
└── skills/                  # 随 preset 分发的技能（SKILL.md）
    ├── cordis-plugin-development/
    └── editing-cordis-compositions/
```

## 安装

将 `cordis-gitbash` 目录放入 DSH 的用户 preset 根目录：

```bash
# Linux / macOS
mkdir -p ~/.dsh/.agent-presets
cp -r dsh-preset-standard-gitbash ~/.dsh/.agent-presets/cordis-gitbash

# Windows (PowerShell)
New-Item -ItemType Directory -Force "$env:USERPROFILE\.dsh\.agent-presets"
Copy-Item -Recurse -Force .\dsh-preset-standard-gitbash "$env:USERPROFILE\.dsh\.agent-presets\cordis-gitbash"
```

然后在 DeepSeek Harness Web GUI 的模式选择器中选用 **标准模式+gitbash** 即可。

> 注意：不要把它放进部署自带的 `agent-presets` 目录（随部署升级会被覆盖）；请放在 `~/.dsh/.agent-presets/` 用户根目录。

## 关于 `bash` 工具（v2 修复说明）

早期版本（`tool-gitbash.mjs`）在 `execute` 里调用 `ctx.timeout()`（Cordis 的 timer 服务 mixin），但加载的模块未声明 `timer` 注入，导致每次调用都报：

```
Error: cannot get property "timer" without inject
```

`tool-gitbash-v2.mjs` 的修复：

- 完全移除对 `timer` 服务的依赖（`inject` 仅声明 `systemPrompt` 与 `tools`）。
- 命令超时改用宿主进程自身的 `setTimeout` / `clearTimeout`（原生 loader 插件运行在普通 Node 进程中，该 API 可用），超时后调用 `proc.terminate()` 终止进程树。
- 文件名带 `-v2` 也用于绕过长驻 `dsh web` 进程的 ESM 模块缓存：新会话会以新 URL 导入修复后的模块，无需重启服务器。

## 说明

- 该 preset 面向个人环境配置，如需公开分享请注意其中不包含任何密钥或敏感信息。
- 模式 id 为 `cordis-gitbash`（id 仅允许小写字母/数字/连字符，因此显示名「标准模式+gitbash」与目录 id 不同）。
