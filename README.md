# Quota Reset Tasks（QRT）

**让 Codex 在当前五小时额度重置一分钟后，执行一次已经授权的任务。**

QRT 是一个面向 Codex 桌面应用的 skill。它读取当前账户的五小时额度重置时间，创建单次原生定时任务；任务全部完成且必要验证通过后，执行者按准确 ID 删除对应调度。

分享与源码地址：**[github.com/C1ro666/Quota-Reset-Tasks](https://github.com/C1ro666/Quota-Reset-Tasks)**

## 安装

下载或克隆本仓库，把仓库根目录命名为 `qrt`，放入 Codex 的用户级 skills 目录。最终结构应为：

```text
skills/
└── qrt/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

| 环境 | 默认安装位置 |
| --- | --- |
| Windows | `%USERPROFILE%\.codex\skills\qrt` |
| macOS / Linux | `~/.codex/skills/qrt` |
| 已设置 `CODEX_HOME` | `$CODEX_HOME/skills/qrt` |

已安装其他 qrt 版本时，先备份，再用本版本替换。安装后让 Codex 刷新 skill 列表；尚未发现时，可直接提供本地 `SKILL.md` 路径让 Codex 读取。

macOS / Linux 示例（未设置 `CODEX_HOME` 时）：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/C1ro666/Quota-Reset-Tasks.git ~/.codex/skills/qrt
```

Windows PowerShell 示例（未设置 `CODEX_HOME` 时）：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.codex\skills" | Out-Null
git clone https://github.com/C1ro666/Quota-Reset-Tasks.git "$env:USERPROFILE\.codex\skills\qrt"
```

安装不会自动创建定时任务。仅安装本 skill 时无需 Python 或其他第三方依赖；使用克隆示例需要 Git。

## 使用

在 Codex 当前聊天中输入：

```text
$qrt 额度重置后整理本项目的文档，检查内容与链接，保存修改并汇报结果。
```

也可使用 `qrt 任务内容`，或自然语言明确调用 `$qrt`。

指定执行模型和思考强度：

```text
$qrt 模型=gpt-6-astra 强度=high 任务=整理本项目的文档，检查并保存修改。
```

`模型=` 与 `强度=` 可以交换顺序。只有 `任务=` 之前的参数会被解析，后面的任务正文原样保留。示例模型不保证每个账户可用，实际以当前客户端工具支持的模型与组合为准。

| 调用方式 | 执行位置与设置 |
| --- | --- |
| 不指定模型或思考强度 | 当前聊天的 heartbeat，沿用执行时该聊天的设置 |
| 指定任一参数 | 独立本地 cron 任务，使用原生 `model` 和 `reasoningEffort` 设置 |

指定参数即选择独立定时任务。只指定一项时，能可靠读取当前聊天设置便沿用另一项；否则会询问缺项。必要的聊天背景与文件位置会写入该任务。把模型名称写在任务正文中，不能直接切换实际模型。

## 调度与完成规则

- 目标时间为当前五小时窗口的 `resetsAt + 60 秒`，按用户当前时区设置。
- 读取不到有效额度信息时，不猜测时间、不创建任务，回复 `无法读取，设置失败`。
- 两种调度均保留 `COUNT=1`，只安排一次触发。
- 创建后先绑定真实调度 ID 和完成后清理指令，再启用；保存与设置确认成功后回复 `定时任务设置成功`。
- 只有任务全部完成且必要验证通过后，才删除该 ID 对应的定时任务。触发、开始、失败、中断或等待用户输入都不算完成。
- 删除失败或结果不明时如实报告；只删除对应调度，保留聊天和任务成果。

`COUNT=1` 只限制运行次数，不等于删除任务，也不提供失败后的自动重试或中断后的再次唤醒。本版本按已读取的重置时间安排工作，不承诺执行时额度仍未被其他使用消耗。

## 运行条件

Codex 桌面应用需要提供额度查询和原生定时任务工具，包括创建、更新、查看与删除能力。计划在本地运行；执行时电脑、应用和任务所需文件必须可用。

本 skill 是给 Codex 读取的指令，不是独立后台服务。客户端的定时触发、模型可用性和权限仍以当前产品能力为准；工具不可用或不支持的设置不会被当作成功。它不会付费重置额度，也不会使用赠送重置。

## 文件

| 文件 | 用途 |
| --- | --- |
| [SKILL.md](SKILL.md) | 调度流程、参数解析和完成后清理指令 |
| [agents/openai.yaml](agents/openai.yaml) | Codex 界面元数据与调用提示 |
| [LICENSE](LICENSE) | MIT 开源许可证 |

## English overview

QRT is a Codex desktop skill that schedules an authorized task once, one minute after the current five-hour quota window resets. It supports explicit model and reasoning-effort settings. The executor deletes the exact automation only after completing the task and passing the necessary verification.

Install this repository as the `qrt` directory under your Codex user skills directory, then invoke `$qrt <task>`. With no model options, QRT uses a heartbeat in the current chat. Specifying a model or reasoning effort selects a separate local cron task. This version does not provide automatic retries.

## 许可证

本项目采用 [MIT License](LICENSE)。
