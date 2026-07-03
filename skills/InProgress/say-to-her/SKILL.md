---
name: say-to-her
description: |
  Transform source material (articles, papers, news, tech content, or any topic) into short,
  dramatic, entertaining conversational snippets with delivery stage directions. Extracts ONE
  interesting nugget, dramatizes it with storytelling techniques, and annotates with pauses,
  voice modulation, gestures, and expressions so even the most introverted engineer can deliver
  it naturally and engagingly to his girlfriend or partner.

  Use when:
  - User wants to turn an article or topic into something to tell their girlfriend/partner
  - User asks to make technical content conversational or girlfriend-friendly
  - User wants a casual script for sharing something interesting
  - User mentions "say to her", "tell her about", "explain to my girlfriend"
  - User wants dinner conversation material or talking points from source material

  Triggers: "say to her", "tell my girlfriend", "girlfriend-friendly", "make it conversational",
  "how do I explain this to her", "turn this into something I can say", "dinner conversation"
---

# Say To Her

Transform any source material into a single dramatic, entertaining conversational snippet — complete with stage directions — that a programmer can confidently deliver to his girlfriend.

## Core Principles

- **ONE nugget only.** Scan the source, find the single most interesting or surprising point, discard everything else. Never summarize.
- **Dramatize ruthlessly.** Exaggerate, add flair, make it theatrical. Entertainment value over accuracy.
- **Stage directions are MANDATORY.** Every output must include inline delivery annotations — pauses, voice cues, gestures, expressions. No exceptions.
- **Storytelling, not lecturing.** Sound like an excited boyfriend sharing something wild, never like a teacher or presenter.
- **No jargon leakage.** Translate all technical terms into casual language or wrap them in playful explanation.
- **No moralizing.** Suppress all "this raises important questions about..." instincts. Pure entertainment.
- **Output length: 100-250 words**, speakable in 30-90 seconds.
- **Output language: match the user's prompt language.** If they write in Chinese, the script is in Chinese. If English, English.
- **Default audience: smart non-technical person** who is curious but has zero context on the topic.

## Transformation Workflow

Transforming source material into a script follows these steps:

1. **Accept source material** — pasted text, topic description, article excerpt, any format. Do NOT fetch URLs; work only with provided text.
2. **Scan for the ONE most interesting nugget** — look for what's surprising, counterintuitive, viscerally impressive, or just plain weird. Ignore everything else.
3. **Discard the rest** — ruthlessly. If the source has 10 interesting facts, pick 1. Commit to it.
4. **Dramatize with storytelling structure:**
   - **Hook** — grab attention: "So you know how...?" / "Okay, get this—" / "Wait, I read the craziest thing today"
   - **Tension** — build curiosity, set up the reveal, make her lean in
   - **Reveal** — deliver the punchline, the mind-blowing fact, the "wait WHAT?" moment
   - **Closer** — playful tag-out line, callback, or "right??" to land the ending
5. **Annotate with delivery stage directions** throughout the script — not just at the start. See `references/tone-guide.md` for the full annotation catalog. Aim for roughly 1 annotation per 2-3 sentences.
6. **Format as a ready-to-use script** using the output template below.

## Output Template

Always format output as follows:

```
## Your Script

**Topic**: [one-line summary of the nugget]
**Vibe**: [emotional flavor, e.g. "mind-blown revelation", "cute fun fact", "conspiracy-theory dramatic"]
**Best delivered**: [context suggestion, e.g. "over dinner", "random text", "while cooking together"]

---

[The script with inline (delivery annotations) woven throughout]

---

**Pro tip**: [one practical delivery suggestion, e.g. "Pause after the big number — let her do the math in her head" or "If she laughs at the hook, ride that energy into the reveal"]
```

## What NOT to Produce

Avoid these anti-patterns at all costs:

- **The TED Talk** — informative but boring, no personality, sounds like a presentation. Missing: hooks, stage directions, casual language.
- **The Wikipedia Entry** — accurate but dry, tries to cover everything instead of cherry-picking one nugget. Missing: storytelling arc, dramatization.
- **The Lecture** — talks down to her, jargon-heavy, explains too much background. Missing: respect for the audience, entertainment value.
- **The Robot** — no stage directions, flat delivery cues. The words might be good but there's no guidance on HOW to say them.

See `references/examples.md` for full side-by-side comparisons of bad vs. good transformations.

## Edge Cases

- **Source is already simple/casual** — still dramatize. Don't parrot; add storytelling structure and stage directions.
- **Source is very long** — fine. Just pick ONE nugget from it. Length of input has no bearing on output.
- **Source is in a different language than user's prompt** — translate and transform in one step. Output in the user's language.
- **Source has no obvious interesting nugget** — find the least boring bit and dramatize it hard. If truly nothing works, tell the user and suggest different material.
- **Same source provided again** — pick a DIFFERENT nugget this time. Never repeat.
- **Sensitive or controversial topic** — transform it, but prepend: `(read the room before telling this one)` at the top of the script.

## References

Load these reference files when generating output:

- **`references/examples.md`** — Complete source-to-script transformation examples (4 good, 2 anti-patterns) demonstrating the exact tone, structure, and annotation density expected. Load this on first use or when calibrating output quality.
- **`references/tone-guide.md`** — Full delivery annotation catalog organized by category (pauses, voice, gestures, expressions, pacing, engagement hooks) with usage guidance and combination rules. Load this when selecting and placing stage directions.
