# Loop engineering: Getting started with loops

> **Source**: [claude.com/blog/getting-started-with-loops](https://claude.com/blog/getting-started-with-loops)
> **Author**: Delba de Oliveira and Michael Segner (Claude Code team, Anthropic)
> **Published**: 2026-06-30
> **Downloaded**: 2026-07-15

> Learn how the Claude Code team defines agentic loops, with practical guidance on progressing from turn-based to goal-based, time-based, and proactive loops—and when to use each.

---

## Getting started with loops

There's a lot of talk right now about loop engineering or "designing loops" instead of prompting your coding agent. If you spend some time on X trying to pin down what a loop actually is, you'll come across multiple different answers.

On the Claude Code team, we define **loops as agents repeating cycles of work until a stop condition is met**. We categorize a few different types of loops based on:

- How they are triggered
- How they are stopped
- What Claude Code primitive is used
- What type of task is most appropriate for each.

We'll cover the main loop types, when to use each, and how to maintain code quality while managing token usage. Not all tasks require complex loops; start with the simplest solution and use these patterns selectively.

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Or read the [documentation](https://code.claude.com/docs/en/overview).

---

## Turn-based loops

![Turn-based loop diagram](images/01-turn-based-loops.png)

Every prompt you send starts a manual loop with you directing each turn. Claude gathers context, takes action, checks its work, repeats if needed, and responds. We call this the **agentic loop**.

For example, ask Claude to create a like button. It reads your code, makes the edit, runs the tests, and hands back something it *believes* works. You then manually check the work, and write the next prompt.

You can improve the verification step by encoding your manual steps as a `SKILL.md` so Claude can check more of its own work, end-to-end. (For choosing between skills, hooks, and subagents for this kind of automation, see our guide to steering Claude Code.)

This should include tools or connectors to allow Claude to *see*, *measure*, or *interact* with the result. The more quantitative the checks are, the easier it is for Claude to self-verify.

For example, in your `SKILL.md` file you may specify:

```yaml
---
name: verify-frontend-change
description: Verify any UI change end-to-end before declaring it done.
---

# Verifying frontend changes

Never report a UI change as complete based on a successful edit alone.
Verify it the way a human reviewer would:

1. Start the dev server and open the edited page in the browser.
2. Interact with the change directly. For a new control (button, input,
   toggle): click it, confirm the expected state change, and screenshot
   before/after.
3. Check the browser console: zero new errors or warnings.
4. Use the Chrome Devtools MCP, run a performance trace and audit
   Core Web Vitals.

If any step fails, fix the issue and rerun from step 1 — do not hand
back partially verified work.
```

---

## Goal-based loop (/goal)

![Goal-based loop diagram](images/02-goal-based-loop.png)

Sometimes, a single turn is not enough, especially for more complex tasks. Agents do better when they can iterate. You can extend how long Claude keeps iterating by defining what done looks like with `/goal`.

When you define the success criteria, Claude doesn't have to make a determination on what is "good enough" and end the loop early. Each time Claude tries to stop, an evaluator model checks your condition and sends it back to work until the goal is met or a number of turns you define is reached.

This is why deterministic criteria, such as number of tests passed or clearing a certain score threshold, are so effective.

For example:

```
/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries.
```

---

## Time-based loop (/loop and /schedule)

Some agentic work is recurring: the task stays the same and only the inputs change. For example, summarizing Slack messages every morning. Other work depends on external systems, and a simple way to interface with one is to check it on an interval and react to what changed. For example, a PR which may receive code reviews or fail CI.

For these, you can trigger when Claude runs with `/loop` which re-runs a prompt on an interval. For example:

```
/loop 5m check my PR, address review comments, and fix failing CI
```

`/loop` runs on your computer, so if you turn it off, it stops. You can move the loop to the cloud by creating a routine with `/schedule`.

---

## Proactive loops

![Proactive loop diagram](images/03-proactive-loops.png)

The primitives above, along with other Claude Code features like **auto mode** and **dynamic workflows** (research preview) can be composed into a loop for long-running work.

For example, to handle incoming feedback, you can use:

- `/schedule` to check for new messages on an interval
- `/goal` to define what triage means
- A workflow to explore multiple solutions in parallel

Putting it together, a prompt could look like this:

```
/schedule every hour: check #project-feedback for bug reports.
/goal: don't stop until every report found this run is triaged,
      actioned, and responded to. When fixing a bug, use a workflow
      to explore three solutions in parallel worktrees and have a
      judge adversarially review them.
```

---

## Maintaining code quality

The quality of a loop's output depends on the system around it. When designing the system:

- When an individual result doesn't meet the standard, don't stop at fixing the individual issue, try to encode it to improve the system for all future iterations.
- Build verification into the loop (SKILL.md examples above), don't rely on human review at the end.
- Keep state external (database, file, PR) so work can be resumed or audited.

---

## Managing token usage

To manage token usage, loops should have clear boundaries:

- Set explicit stop conditions (turn count, time, deterministic goal)
- Use `/goal` with measurable criteria, not vibes
- Pick the right model/effort for the task — your [model and effort level](https://code.claude.com/docs/en/model-config) choices are among the biggest levers on what a loop costs.
- Resume sessions instead of starting fresh when state is preserved

---

## Getting started

To summarize:

- Start with the simplest loop (turn-based) and only add primitives (`/goal`, `/loop`, `/schedule`, workflows) when the task demands it.
- Encode verification as skills or workflows, don't rely on the model's good intentions.
- Define success criteria deterministically.
- Monitor and iterate on the loop itself, not just the individual outputs.

> To get started with loops, look at the work you already do. Pick one task where you're the bottleneck and ask which piece you could hand off: can you write the verification check? Is the goal clear enough? Does the work arrive on a schedule?

Once you have an idea, run the loop, observe the results like where it stalls or over-reaches, and don't be afraid to iterate on it.

For more information, read the Claude Code docs on [running agents in parallel](https://code.claude.com/docs/en/parallel-agents), as well as the [loop](https://code.claude.com/docs/en/loop), [schedule](https://code.claude.com/docs/en/schedule), [goal](https://code.claude.com/docs/en/goal), and [dynamic workflows](https://code.claude.com/docs/en/dynamic-workflows) pages.

*This article was written by Delba de Oliveira and Michael Segner.*

---

## Saved assets

| File | Description |
|------|-------------|
| `loop-engineering-getting-started-with-loops.md` | This file — full article text |
| `images/01-turn-based-loops.png` | Turn-based loop diagram (2024×1012) |
| `images/02-goal-based-loop.png` | Goal-based (/goal) loop diagram (2024×1050) |
| `images/03-proactive-loops.png` | Proactive loop diagram (2024×1072) |
