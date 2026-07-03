---
name: branch
description: Move the current branch's extra commits and local changes onto a target branch.
disable-model-invocation: true
argument-hint: "Which branch should receive the current work?"
---

Treat this as a **transplant**: move everything the current checkout has beyond the target branch onto that target branch, without dropping source work.

## Steps

1. Read the target branch name from the arguments. If it is missing, ask the user for it before doing any git writes.
2. Audit the transplant set before changing anything:
   - identify the current branch,
   - confirm the target branch exists locally,
   - list every commit in `target_branch..HEAD` in oldest-first order,
   - list tracked and untracked local changes,
   - check whether the target branch is already checked out by another worktree,
   - stop and surface the situation if the current branch already is the target branch.
3. If there are uncommitted changes, create one temporary local commit on the current branch to carry them through the transplant. Make it clear in the final report that this transport commit still exists on the source branch unless the user explicitly asks to rewrite history.
4. If the target branch is free in the current worktree, switch to it here and cherry-pick the full transplant set in order.
5. If the target branch is occupied by another worktree, do not treat that as failure. Use cherry-pick to finish the transplant in the worktree that already has the target branch checked out, or onto a fresh branch created from that target when the user asked for that variant.
6. If cherry-pick conflicts happen, resolve what you safely can. If the resolution is ambiguous or risky, stop and ask the user instead of guessing.
7. Report the result with:
   - source branch,
   - target branch,
   - whether the target branch was occupied by another worktree,
   - commits transplanted,
   - whether a temporary transport commit was created,
   - whether any conflicts were resolved or remain.

## Completion Criterion

Done means the target branch, or the agreed fresh branch forked from it, now contains every commit that was in `target_branch..HEAD` at audit time, plus every local tracked or untracked change that existed at audit time, with no source work discarded silently.
