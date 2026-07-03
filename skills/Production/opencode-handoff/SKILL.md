---
name: opencode-handoff
description: Hand the current conversation off to a fresh OpenCode session that picks up the work in the background.
argument-hint: "What will the next OpenCode session be used for?"
disable-model-invocation: true
---

Write a handoff summary of the current conversation so a fresh OpenCode session can continue the work.

Before launching, check for `.opencode-handoff` in the current working directory. If it exists, read it as the model roster. The roster is a Markdown table with columns `model`, `variant`, `cost`, `intelligence`, and `taste`. The column is singular `variant`, not `variants`.

Generate or update `.opencode-handoff` by running `opencode-handoff-init` from within the repo root.

If `.opencode-handoff` does not exist, use this built-in support table:

| model | variant | cost | intelligence | taste |
| --- | --- | ---: | ---: | ---: |
| `opencode/deepseek-v4-flash-free` | `medium` | 10 | 5 | 3 |
| `opencode/deepseek-v4-flash-free` | `max` | 10 | 6 | 4 |
| `opencode/nemotron-3-ultra-free` | `high` | 10 | 3 | 3 |

Treat scores as rankings where higher is better. Cost reflects what the user actually pays, not list price. Intelligence is how hard a problem the model can handle unsupervised. Taste covers UI/UX, code quality, API design, and copy.

Apply the scores this way:

- These are defaults, not limits. If a cheaper or lower-ranked model's output does not meet the bar, rerun or redo the work with a smarter model without asking.
- Judge the output, not the price tag. Escalating costs less than shipping mediocre work.
- Use cost only as a tie-breaker. For work that ships, rank by `intelligence`, then `taste`, then `cost`.
- Anything user-facing, including UI, copy, API design, and public docs, needs `taste >= 7` when such a model is available.
- Reviews of plans or implementations primarily need intelligence.

Choose a supported `variant` for the selected model. Do not invent a `variant`; it is provider-specific. Use `medium` or `low` for smoke tests and simple follow-ups; use `high` for normal handoffs; use `max` only when the selected model supports it and the handoff clearly needs deeper reasoning. If no supported model is available, omit `--model` and `--variant`. If a model has no selected variant, omit `--variant`.

Because OpenCode has no native background flag, save the summary to a temporary file outside the workspace and launch it as a background process:

```sh
nohup opencode run \
  --title "<descriptive title>" \
  --dir "<current working directory>" \
  --model "<provider/model>" \
  --variant "<provider-specific variant>" \
  "Continue from the attached handoff summary. Start working immediately." \
  --file "<handoff file>" \
  > "<log file>" 2>&1 &
```

Remove the `--model` or `--variant` lines when they are not selected.

Always pass `--title` with a descriptive title. It sets the session title shown by `opencode session list`.

Include a "suggested skills" section in the summary, which suggests skills that the session should invoke.

Do not duplicate content already captured in other artifacts (PRDs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

Redact any sensitive information, such as API keys, passwords, or personally identifiable information -- the summary becomes the session prompt.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the summary accordingly.

After launching, report the title, process id, selected model and variant if any, temporary handoff file, and log file. The user can inspect sessions with `opencode session list`; do not wait for the session to finish.
