---
name: outsource-mode-claude
description: 进入外包 + Claude 咨询的分工模式：自己只做调度、决策、审阅、计划，执行交给 /handoff-outsource，难点交给 /consult-claude。
disable-model-invocation: true
argument-hint: "[--outsource \"aid opus xhigh\"] [--consult-effort medium]"
---

参数覆盖（缺省时用下面分工里写的默认值）：

- `--outsource "<binary> <model> <effort>"` — 覆盖外包参数，原样传给 `/handoff-outsource`。
- `--consult-effort <level>` — 覆盖咨询的思考程度，模型仍用分工里写的那个。

把下面这段当作本会话后续的**长期契约**，而不是一次性指令：每接到一个新任务，都先按分工判断该自己做、外包，还是先咨询。

---

请在会话的后续中采用如下的分工。

### 分工

- 你：调度、决策、审阅、计划
- /handoff-outsource：脏活累活、资料收集、具体执行。
    - 使用 aid opus xhigh
    - 启动后请不设超时时间地等待相关 shell/Terminal 命令执行完毕
    - 外包工作期间，请等待别瞎折腾。外包结束工作后会向你汇报。
- /consult-claude：针对重点问题、难点问题进行咨询。
    - 场景：涉及项目核心代码的改动；复杂计划执行前把关；一项复杂的任务整体执行完毕后。
    - 使用 opus medium
    - 启动后请不设超时时间地等待相关 shell/Terminal 命令执行完毕
