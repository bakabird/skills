---
name: grok-handoff
description: Hand the current conversation off to a fresh Grok Build session that continues the work in the background.
argument-hint: "What will the next Grok Build session be used for?"
disable-model-invocation: true
---

## write handoff summary

Write a handoff summary of the current conversation so a fresh Grok Build session can continue the work.

Include a "suggested skills" section in the summary, which suggests skills that the session should invoke.

Do not duplicate content already captured in other artifacts (PRDs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

Redact any sensitive information, such as API keys, passwords, or personally identifiable information -- the summary becomes the session prompt.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the summary accordingly.

Include the instruction to continue immediately inside the handoff file itself. Start the file with a short directive such as:

```text
Continue from this handoff summary. Start working immediately.
```

## launch grok build background task

Use `--effort` for Grok Build handoffs. Supported values are `low`, `medium`, `high`, `xhigh`, and `max`.

Use this built-in support table:

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

Treat scores as rankings where higher is better. Cost reflects what the user actually pays, not list price. Intelligence is how hard a problem the model can handle unsupervised. Taste covers UI/UX, code quality, API design, and copy.

Apply the scores this way:

- These are defaults, not limits. If a cheaper or lower-ranked option's output does not meet the bar, rerun or redo the work with a stronger effort without asking.
- Judge the output, not the price tag. Escalating effort costs less than shipping mediocre work.
- Use cost only as a tie-breaker. For work that ships, rank by `intelligence`, then `taste`, then `cost`.
- Anything user-facing, including UI, copy, API design, and public docs, needs the best available `taste`.
- Reviews of plans or implementations primarily need intelligence.

Choose a supported `effort` for the selected model. Do not invent an `effort`.

Use `medium` or `low` for smoke tests and simple follow-ups, `high` for normal handoffs, `xhigh` for complex implementation or review work, and `max` only when the handoff clearly requires the deepest reasoning.

If no supported models are available, omit `--model`. If no effort is selected, omit `--effort`.

Run the `nohup` launch outside the sandbox when sandboxing blocks background execution.

```sh
nohup sh -c '
grok \
  --cwd "<current working directory>" \
  --model "<model>" \
  --effort "<effort>" \
  --output-format plain \
  --prompt-file "<handoff file>"
code=$?
echo "finished at $(date), exit code=$code" > "<done file>"
exit $code
' > "<log file>" 2>&1 &

echo $! > "<pid file>"
```

Omit `--model` and `--effort` when they are not selected.

Choose a descriptive title for the report and filenames, but do not pass `--title` to `grok`.

After launching, report the descriptive title, process ID, selected model and effort if any, temporary handoff file, done file, and log file. Do not wait for the session to finish.

Run `cat "<done file>"` to check whether the task completed or failed with an error.
