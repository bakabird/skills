---
name: to-branch
description: Move the current branch's extra commits and local changes onto a target branch.
disable-model-invocation: true
argument-hint: "target_branch [--squash|--no-squash]"
---

Treat this as a **transplant**: move everything the current checkout has beyond the target branch onto that target branch, without dropping source work.

Keep the run **tight**: gather the facts once, land once, verify once. Do not keep re-checking the same branch/status/log facts unless a git write happened or a conflict changed the state.

Arguments control the landing shape:

```text
main --no-squash   # replay commits onto main, preserving commit boundaries
main --squash      # land the audited work as one target-branch commit
```

If the squash flag is missing, use `--no-squash`.

## Steps

1. Read the target branch name and squash flag from the arguments. If the target is missing, ask the user for it before doing any git writes. Treat `--squash` as squash mode; treat `--no-squash` or no flag as replay mode. If both flags appear, stop and ask the user to choose one.
2. Do one compact audit before changing anything. Collect all of these facts in as few tool calls as practical:
   - identify the current branch,
   - confirm the target branch exists locally,
   - list every commit in `target_branch..HEAD` in oldest-first order, unless HEAD is detached and equal to the target tip,
   - list tracked and untracked local changes,
   - check whether the target branch is already checked out by another worktree,
   - stop and surface the situation if the current branch already is the target branch.
3. Decide the whole route from that audit, then execute it. Avoid exploratory git reads after this point until a write has occurred.
4. If there are uncommitted changes in the source checkout, create one temporary local commit on the source checkout to carry them through the transplant. Make it clear in the final report that this transport commit still exists on the source unless the user explicitly asks to rewrite history.
5. Choose the target worktree:
   - If the target branch is free in the current worktree, switch to it there.
   - If the target branch is occupied by another worktree, do not treat that as failure. Finish the transplant in the worktree that already has the target checked out, or onto a fresh branch created from that target when the user asked for that variant.
   - Before writing in an occupied target worktree, inspect its status. If it has unrelated local changes, preserve them and only proceed when the transplant can be applied without touching those files.
6. Land the audited work:
   - In replay mode, cherry-pick every audited commit in order onto the target worktree.
   - In squash mode, apply the audited work without committing first, verify the resulting target diff, then create one target commit. Prefer `cherry-pick -n` of the audited commits; when the source is based on an older target, this avoids `merge --squash` accidentally dragging old-base differences into the target.
   - If there are no audited commits but there is a temporary transport commit, land that commit using the same replay or squash rule.
7. If conflicts happen, resolve what you safely can. If the resolution is ambiguous or risky, stop and ask the user instead of guessing.
8. Do one final verification pass:
   - target branch now points at the landed commit or commits,
   - target status is clean except for unrelated pre-existing local changes in an occupied worktree,
   - source checkout still contains no uncommitted transplant work,
   - run any already-required validation from the work session when practical, especially `pnpm build` before a code merge in this repo.
9. Report the result with:
   - `Merge: ✅` if the target now contains all audited work, or `Merge: ❌` if it does not,
   - source branch,
   - target branch,
   - squash mode,
   - whether the target branch was occupied by another worktree,
   - commits transplanted,
   - whether a temporary transport commit was created,
   - whether any conflicts were resolved or remain,
   - target commit or commits,
   - validation run or skipped.

## Tool Budget

Aim for this shape:

```text
audit facts  ->  git write(s)  ->  final verify  ->  report
```

Prefer one compact shell read for the audit over many tiny reads. Prefer one compact final verify after all writes over repeated `git status` checks. Extra reads are justified only for conflicts, unexpected dirty files, sandbox failures, or choosing between occupied-worktree routes.

## Completion Criterion

Done means the target branch, or the agreed fresh branch forked from it, now contains every commit that was in `target_branch..HEAD` at audit time, plus every local tracked or untracked change that existed at audit time, with no source work discarded silently. In squash mode, "contains" means the combined patch is present as one target commit rather than preserving the source commit boundaries.
