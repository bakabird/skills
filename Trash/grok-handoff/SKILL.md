---
name: grok-handoff
description: 将当前对话交接给一个新的 Grok Build 后台会话继续执行。
argument-hint: "下一个 Grok Build 会话要做什么？"
disable-model-invocation: true
---

## 编写交接摘要

编写任务的交接摘要，让新的 Grok Build 会话可以继续工作。

摘要中包含一个名为 `suggested skills` 的小节，用来建议新会话应该调用哪些技能。

不要重复已经写入其他产物的内容，例如 PRD、计划、ADR、issue、commit、diff。改用路径或 URL 引用它们。

删去所有敏感信息，例如 API key、密码、个人身份信息。交接摘要会成为新会话的提示词。

如果用户传入了参数，将其视为下一个会话的重点说明，并据此调整摘要。

在交接文件本身写入“立即继续”的指令。文件开头使用类似这样的短指令：

```text
根据这份交接摘要继续。立即开始工作。
```

## 启动 Grok Build 后台任务

Grok Build 交接使用 `--effort`。支持的值为 `low`、`medium`、`high`、`xhigh`、`max`。

使用下面的内置支持表：

| model | effort | cost | intelligence | taste |
| --- | --- | ---: | ---: | ---: |
| `grok-composer-2.5-fast` | `low` | 10 | 4 | 5 |
| `grok-composer-2.5-fast` | `medium` | 10 | 5 | 5 |
| `grok-composer-2.5-fast` | `high` | 10 | 6 | 5 |
| `grok-build` | `low` | 10 | 5 | 4 |
| `grok-build` | `medium` | 10 | 6 | 5 |
| `grok-build` | `high` | 10 | 7 | 5 |
| `grok-build` | `xhigh` | 10 | 8 | 5 |
| `grok-build` | `max` | 10 | 9 | 5 |

评分按排名理解，越高越好。`cost` 表示用户实际支付成本，不是标价。`intelligence` 表示模型在无人监督下能处理多难的问题。`taste` 覆盖 UI/UX、代码质量、API 设计和文案。

按以下方式使用评分：

- 这些是默认值，不是限制。如果更便宜或排名更低的选项输出不达标，无需询问，直接用更强的 effort 重跑或重做。
- 判断输出质量，不判断价格标签。提升 effort 的成本低于交付平庸结果。
- `cost` 只作为平局时的比较项。要交付的工作按 `intelligence`、`taste`、`cost` 依次排序。
- 任何面向用户的内容，包括 UI、文案、API 设计和公开文档，都需要可用范围内最好的 `taste`。
- 评审计划或实现时，主要看 `intelligence`。

为选中的模型选择受支持的 `effort`，不要编造不存在的 `effort`。

冒烟测试和简单跟进使用 `medium` 或 `low`；普通交接使用 `high`；复杂实现或评审使用 `xhigh`；只有交接明确需要最深推理时才使用 `max`。

如果没有可用的受支持模型，省略 `--model`。如果未选择 effort，省略 `--effort`。

## screen 后台启动器

使用 `screen` 作为后台启动器。它会启动一个具名的分离会话，并立即返回。

所有准备完成后、启动前，要求用户精确回复以下内容来批准：

```text
同意执行 screen
```

只有收到这条精确回复后才能运行 `screen`。否则停止，并报告已准备的文件和已选择的选项。

如果沙箱阻止终端复用器执行，则在沙箱外运行 `screen` 启动命令。

```sh
SESSION_NAME="<简短安全的会话名>"
WORKDIR="<当前工作目录>"
MODEL="<model>"
EFFORT="<effort>"
HANDOFF_FILE="<交接文件>"
DONE_FILE="<完成状态文件>"
LOG_FILE="<日志文件>"

screen -D -m -S "$SESSION_NAME" env \
  WORKDIR="$WORKDIR" \
  MODEL="$MODEL" \
  EFFORT="$EFFORT" \
  HANDOFF_FILE="$HANDOFF_FILE" \
  DONE_FILE="$DONE_FILE" \
  LOG_FILE="$LOG_FILE" \
  sh -c '
  exec > "$LOG_FILE" 2>&1
  grok \
    --cwd "$WORKDIR" \
    --model "$MODEL" \
    --effort "$EFFORT" \
    --output-format plain \
    --prompt-file "$HANDOFF_FILE"
  code=$?
  echo "完成于 $(date), exit code=$code" > "$DONE_FILE"
  exit "$code"
'

echo "已启动 Grok 后台任务"
echo "会话: $SESSION_NAME"
echo "完成状态文件: $DONE_FILE"
echo "日志文件: $LOG_FILE"
```

未选择对应值时，删除 `--model` 行或 `--effort` 行。

为报告和文件名选择一个描述性标题，但不要向 `grok` 传递 `--title`。

启动后，报告描述性标题、screen 会话名、选中的模型和 effort（如有）、临时交接文件、完成状态文件和日志文件。不要等待会话结束。

任务仍在运行时，用 `screen -r "<会话名>"` 接入会话。用 `screen -ls` 列出活跃会话。用 `cat "<完成状态文件>"` 检查任务是完成还是失败。
