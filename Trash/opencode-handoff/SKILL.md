---
name: opencode-handoff
description: 将当前对话交接给一个新的 OpenCode 后台会话继续执行。
argument-hint: "下一个 OpenCode 会话要做什么？"
disable-model-invocation: true
---

## 编写交接摘要

编写当前对话的交接摘要，让新的 OpenCode 会话可以继续工作。

摘要中包含一个名为 `suggested skills` 的小节，用来建议新会话应该调用哪些技能。

不要重复已经写入其他产物的内容，例如 PRD、计划、ADR、issue、commit、diff。改用路径或 URL 引用它们。

删去所有敏感信息，例如 API key、密码、个人身份信息。交接摘要会成为新会话的提示词。

如果用户传入了参数，将其视为下一个会话的重点说明，并据此调整摘要。

## 启动 OpenCode 后台任务

启动前，检查当前工作目录中是否存在 `.opencode-handoff`。如果存在，将它作为模型名单读取。名单是一个 Markdown 表格，列名为 `model`、`variant`、`cost`、`intelligence`、`taste`。列名是单数 `variant`，不是 `variants`。

如果 `.opencode-handoff` 不存在，使用下面的内置支持表：

| model | variant | cost | intelligence | taste |
| --- | --- | ---: | ---: | ---: |
| `opencode/deepseek-v4-flash-free` | `medium` | 10 | 5 | 3 |
| `opencode/deepseek-v4-flash-free` | `max` | 10 | 6 | 4 |
| `opencode/nemotron-3-ultra-free` | `high` | 10 | 3 | 3 |

评分按排名理解，越高越好。`cost` 表示用户成本参考值，不是具体价格。`intelligence` 表示模型在无人监督下能处理多难的问题。`taste` 覆盖 UI/UX、代码质量、API 设计和文案。

按以下方式使用评分：

- `cost` 只作为平局时的比较项。要交付的工作按 `intelligence`、`taste`、`cost` 依次排序。
- 如果评分更低的模型输出不达标，无需询问，直接用更聪明的模型重跑或重做。
- 评审计划或实现时，主要看 `intelligence`。

为选中的模型选择受支持的 `variant`。不要编造不存在的 `variant`；variant 由提供方定义。

如果没有可用的受支持模型，省略 `--model` 和 `--variant`。如果该模型未选择 variant，省略 `--variant`。

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
TITLE="<描述性标题>"
WORKDIR="<当前工作目录>"
MODEL="<provider/model>"
VARIANT="<provider-specific variant>"
HANDOFF_FILE="<交接文件>"
DONE_FILE="<完成状态文件>"
LOG_FILE="<日志文件>"

screen -D -m -S "$SESSION_NAME" env \
  TITLE="$TITLE" \
  WORKDIR="$WORKDIR" \
  MODEL="$MODEL" \
  VARIANT="$VARIANT" \
  HANDOFF_FILE="$HANDOFF_FILE" \
  DONE_FILE="$DONE_FILE" \
  LOG_FILE="$LOG_FILE" \
  sh -c '
  exec > "$LOG_FILE" 2>&1
  opencode run \
    --title "$TITLE" \
    --dir "$WORKDIR" \
    --model "$MODEL" \
    --variant "$VARIANT" \
    "根据附加的交接摘要继续。立即开始工作。" \
    --file "$HANDOFF_FILE"
  code=$?
  echo "完成于 $(date), exit code=$code" > "$DONE_FILE"
  exit "$code"
'

echo "已启动 OpenCode 后台任务"
echo "标题: $TITLE"
echo "会话: $SESSION_NAME"
echo "完成状态文件: $DONE_FILE"
echo "日志文件: $LOG_FILE"
```

未选择对应值时，删除 `--model` 行或 `--variant` 行。

始终传入带描述性标题的 `--title`。

启动后，报告标题、screen 会话名、选中的模型和 variant（如有）、临时交接文件、完成状态文件和日志文件。不要等待会话结束。

任务仍在运行时，用 `screen -r "<会话名>"` 接入会话。用 `screen -ls` 列出活跃会话。用 `cat "<完成状态文件>"` 检查任务是完成还是失败。
