# Quickstart

Clone shipkit, bootstrap a ship directory, and run your first watch. This should take about 15-20 minutes the first time.

New to the vocabulary below (watch, drop, ticket, etc.)? See [GLOSSARY.md](GLOSSARY.md) first. Want to see what the end state looks like before you build it? See [`examples/`](examples/).

## Prerequisites

- [Claude Code](https://code.claude.com/docs) installed and working.
- Ship uses **custom subagents** and **PreToolUse hooks**, both Claude Code features. If you haven't used either before, skim these before bootstrapping so the setup steps make sense:
  - [Subagents](https://code.claude.com/docs/en/sub-agents) — what `agents/ship-crew.md` and `agents/ship-lookout.md` become once installed.
  - [Hooks](https://code.claude.com/docs/en/hooks) — what enforces the git-safety restrictions on Crew (`scripts/validate-crew-bash.sh`).
  - This quickstart doesn't re-explain either — the links above cover the prerequisites; the rest of this doc covers Ship-specific setup on top of them.

## 1. Clone shipkit

```
git clone <this-repo-url> ~/code/shipkit
```

(Or wherever you keep code — shipkit itself doesn't need to live anywhere special.)

## 2. Bootstrap a ship directory

Start a Claude Code session anywhere and say:

> Read the shipkit docs at `~/code/shipkit/` and bootstrap a new Ship for me.

Claude Code will walk the [bootstrap instructions in README.md](README.md#bootstrap-instructions-for-claude-code): pick a ship directory (separate from any one project repo — Ship coordinates across repos), create the directory structure, copy the hook scripts and subagent definitions into place, and help you fill in `captain.md`. It'll ask you a couple of questions along the way (where to put things, what you're working on) — answer those and let it finish.

**Verify it worked:** the agent should report back that `~/.claude/agents/ship-crew.md` and `ship-lookout.md` exist (pointing at your new ship directory's hook scripts), and that your ship directory has `queue.md`, `captain.md`, `projects/`, and `logs/` in place.

## 3. Fill in `captain.md`

Open `{ship-dir}/captain.md` and write a few real sentences under Situation and Priorities — what you're working on, what matters most this week. The Mate reads this to decide what to dispatch. A blank `captain.md` means the Mate has nothing to prioritize against.

## 4. Start your First Mate session

Start (or continue) a Claude Code session with your ship directory as the working directory, and say:

> You're First Mate on this ship. Read `ship/mate.md` for your standing orders.

The Mate reads `queue.md`, `captain.md`, and the inbox, then reports status. If everything's empty, that's expected on a fresh ship — it'll say so and ask what to work on.

## 5. Create your first ticket and dispatch

Tell the Mate what you want done, in plain language. It'll create a ticket under `projects/{area}/tickets/`, add it to the queue, and dispatch a Crew subagent with watch orders. Crew runs in the background — you don't need to babysit it.

## 6. Review the first watch

When the Crew subagent finishes, it'll have written a log under `logs/{area}/{ticket-id}/` and updated the ticket's "Watch history." Read the log — that's the handoff. The Mate will summarize it and tell you what's next (commit and push, or another watch, or a question for you).

That's the loop: Captain sets priority -> Mate dispatches -> Crew watches and logs -> Captain reviews -> repeat.

## Troubleshooting

**Bootstrap fails partway through / agent gets confused about paths.**
Bootstrap is a one-shot setup performed by an LLM reading instructions, not a script — it can misplace a path or skip a step, especially if you interrupt it with unrelated questions mid-flow. Fix: ask the agent to re-read the "Bootstrap Instructions" section of `README.md` and verify each of the 7 steps was actually completed (directory structure, hook scripts copied and executable, subagent definitions installed to `~/.claude/agents/` with `{SHIP_DIR}` substituted for the real path, `captain.md` created). It's safe to re-run bootstrap — steps are idempotent (copying a file that's already there just overwrites it).

**A Crew watch seems to hang or go silent.**
Crew runs in the background, so "no output yet" is often normal for anything more than a trivial task. If it's been unreasonably long: check whether the subagent is actually blocked on something it can't self-resolve (a destructive git operation it's not allowed to run, a missing permission, an ambiguous scope) — Crew's standing orders say to stop and write a log rather than spin, so a properly-behaving watch that's stuck should have already ended itself with a log explaining why. If there's no log and no output, treat it as a genuinely hung session: end it and re-dispatch with tighter watch orders, and consider reporting it if it reproduces — bounded sessions hanging silently is exactly the failure mode Ship is designed to avoid.

**"Role not assumed" — the agent isn't behaving like Mate/Crew.**
This usually means the session was never told which standing orders to read. Ship roles aren't automatic modes; a session becomes the Mate only when explicitly told to read `mate.md`, and Crew's standing orders are either baked into the `ship-crew`/`ship-lookout` subagent definitions (if you're dispatching via the Task tool) or need `crew.md` included directly in the prompt otherwise. If a session seems to be improvising instead of following the loop described in `mate.md` or `crew.md`, restate the role assignment explicitly and point it at the file.

**Hook doesn't seem to be blocking anything (Crew commits or pushes when it shouldn't).**
Check that the subagent definition in `~/.claude/agents/ship-crew.md` has `{SHIP_DIR}` correctly substituted with your actual ship directory's absolute path in the `hooks:` block, and that `scripts/validate-crew-bash.sh` is executable (`chmod +x`). See the [hooks documentation](https://code.claude.com/docs/en/hooks) for how Claude Code resolves and runs PreToolUse hooks if the path substitution looks right but it's still not firing.
