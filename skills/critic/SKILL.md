---
name: critic
description: Have a second agent critique a proposal and a third agent judge it. Ask the user only when a problem is found. Use as "/critic <task>".
---

# critic

Start with `/critic <task>`. The main session calls three agents in order: Proposer (proposes) → Critic (criticizes) → Coordinator (judges).

## Communication rules (exception to the global CLAUDE.md)

This skill does not use comm.md / agent-log. Hand-offs are a plain relay only: **sub → main → sub → main**.
- Main pastes each sub's reply **verbatim** into the next sub's prompt (no summarizing or rewriting).
- Subs cannot use `AskUserQuestion`. Only main asks the user.

## Language rule (English only)

- The user types in English. Main's replies, the `AskUserQuestion` question and options, and the final report are **all in English**.
- When launching a sub, always end the prompt with `Respond in English only.` (format keys such as `ACTION:` stay as they are).

## Steps

1. Launch `critic_proposer` with the Agent tool and pass the task text.
   Reply format: `ACTION:` and `REASON:`
2. Launch `critic_critic`. Pass the task text and the Proposer's reply verbatim.
   Reply format: `WEAK_POINTS:` (bullets) and `VERDICT: Strong | Weak | Broken`
3. Launch `critic_coordinator`. Pass the task text, the Proposer's reply, and the Critic's reply verbatim.
   Reply format: `COORDINATOR_VERDICT:` / `CRITIC_VERDICT:` / `FINAL_VERDICT:` / `SUMMARY:`
   (`FINAL_VERDICT` is the stricter one: Broken > Weak > Strong)
4. Branch.
   - `FINAL_VERDICT` is Strong and the action is not in "Operations that always need confirmation" below → execute the Action.
   - Weak / Broken, or the action is in "Operations that always need confirmation" → show the weak points and verdict, then `AskUserQuestion` (execute / revise and re-propose / abort).
   - "Revise and re-propose" → go back to step 1 with the user's remarks attached (at most 2 times).
5. Report the verdict, weak points, and execution result briefly.

If a reply does not follow the format (e.g. no `FINAL_VERDICT`), treat it as Broken.

## Operations that always need confirmation (AskUserQuestion even when Strong)

- Deleting files or data
- git push
- Sending anything externally (email, posts, API writes, etc.)
- Operations that cost money
