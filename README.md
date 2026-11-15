# Teach to Learn

**English** | [简体中文](README_CN.md)

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Agent Skill](https://img.shields.io/badge/Agent_Skill-Codex_CLI-black.svg)
![Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen.svg)

> **Stop mistaking "I understood the AI's explanation" for actual mastery.**
> A learner-focused Agent Skill that defines observable completion criteria first, then guides you to independent execution through diagnostic checks, 5–15 minute micro-lessons, open-ended active recall, spaced repetition, and unseen transfer tasks.

Ideal for multi-session learning in programming, languages, academic concepts, or workplace skills. For quick one-off questions, ask your assistant directly without setting up a learning directory.

```text
Observable Goal ─▶ Short Diagnostics (≤3) ─▶ 5–15 min Micro-Lesson ─▶ Open-Ended Attempt ─▶ Evidence & 1/3/7/14d Review Queue ─▶ Unseen Transfer Task
```

*`SKILL.md` and `references/` are authored in Chinese and request the learner's language. Fixed state IDs are shared across languages; behavioral compatibility must be checked with the [manual acceptance scenarios](evals/scenarios.md).*

## What Changed Compared to the Original `teach`

Redesigned based on [Matt Pocock's `teach`](https://github.com/mattpocock/skills/tree/5863806dd4e15c63174bee4e7515a4256792ecfa/skills/productivity/teach), preserving goal-driven learning, trusted sources, micro-lessons, and cross-session tracking while addressing key failure modes:

| Limitation or Risk in Upstream | Specific Adjustment in `teach-to-learn` |
| --- | --- |
| Infers baseline level from goals and self-reports on turn one | Runs up to 3 short diagnostic tasks (one at a time), separating self-reports from observed evidence; skipping is always allowed |
| Mentions spaced repetition only as a general principle | Implements an adjustable **1, 3, 7, and 14-day** review queue with explicit rules for failed, skipped, and overdue items |
| Lacks clear completion or exit criteria | Agrees on observable completion standards upfront; transitions to real-world practice or concludes after passing an unseen transfer task |
| Multiple-choice options can leak answer cues | Defaults to open recall, prediction, debugging, or hands-on demos; passing multiple-choice questions never counts as independent mastery |
| Reading material is easily conflated with learning | Logs actual user attempts and assistance levels; distinguishes provisional independence from delayed consolidation |
| Interactive HTML results may not sync back to chat | Requires users to paste results when read-back capability is unverified; keeps status as "pending verification" until received |
| Tooling and source access depend on the host | Supports pure-chat continuation without file permissions via copyable summaries; marks offline explanations as unverified |

*Disclaimer: These are structural adjustments targeting a specific upstream commit—not a guarantee of consistent execution across all models, nor empirical proof of learning outcomes.*

## Installation (Codex CLI)

Requires Codex CLI with local skills support and Git. The skill itself has **zero runtime dependencies**.

Run the following commands on macOS, Linux, or WSL (ensure the target directory does not already exist):

~~~sh
mkdir -p ~/.agents/skills
git clone https://github.com/olele114/teach-to-learn.git ~/.agents/skills/teach-to-learn
~~~

Type `/skills` inside Codex to verify `teach-to-learn` appears (restart the session if needed).
Alternatively, place the entire repository inside `.agents/skills/teach-to-learn/` within a project to share it with your team.
**Do not copy `SKILL.md` alone:** `references/` contains required state and lesson quality specifications.

> **Important:** Keep your **installation directory** separate from your **learning directory**. Start Codex inside your own learning workspace rather than inside the skill installation folder.
> Other hosts compatible with the Agent Skills specification can also import this directory; check your host's documentation for exact paths and invocation syntax. This repository is a standalone skill—not a plugin, MCP server, or model configuration.
> Format and paths follow the [Codex skills documentation](https://developers.openai.com/codex/skills/).

## First Use

Send the following prompt inside the **Codex chat input** (not your OS terminal):

~~~text
$teach-to-learn
I want to learn Python asyncio with the goal of independently debugging an async HTTP client.
I know basic Python, but I don't understand the event loop. Keep sessions to 15 minutes in English.
Start in chat only—do not create any files yet.
~~~

The assistant will clarify your goal first, then present **one short diagnostic task at a time** and wait for your response.
Saying "I don't know," skipping a question, or shortening the lesson is always allowed; skipped items are recorded as `Unassessed` without forced re-testing.

To persist progress across sessions, explicitly specify a directory and grant write permission:

~~~text
Save this topic under ./learning/python-async/.
Tell me which files you plan to write first, and follow my edit approval rules.
~~~

The assistant will inspect existing files and ask for approval before writing; if permission is declined, learning continues seamlessly in chat.
Learning records may contain your responses and preferences—choose your storage location and sharing settings carefully. This skill never commits or uploads files automatically.

## How a Session Flows

~~~text
You: Today, just teach me how to distinguish awaiting from blocking.
Assistant: Explains the purpose, defines essential terms, walks through one worked example, and gives a new open-ended task—then stops and waits.
You: Submit your prediction and reasoning, or paste your code execution output.
Assistant: Points out exact strengths and misconceptions; offers a minimal hint first if needed.
You: Retry the task, or say "just show the answer" / "pause for today."
Assistant: Summarizes observed evidence, assistance level, unverified items, and next-step recommendations.
~~~

This is an illustrative workflow, not an actual log. Generating a lesson, reading an answer, or saying "got it" never counts as independent completion.
Code that cannot be executed in the current environment is marked `Unrun`; clicks or scores inside generated HTML pages do not automatically sync back to chat.

## Resuming & Common Prompts

Open a new session in the same learning directory and invoke the skill explicitly:

~~~text
$teach-to-learn Resume learning in this directory. Read the saved state first, and only review the two most relevant items today.
~~~

You can also steer the session at any time:

- *"No new concepts today—give me an unseen transfer task that doesn't repeat last session's answer."*
- *"This step feels too steep. Check which prerequisite concept I'm missing first."*
- *"I need a hint—record the assistance used and don't count this attempt as independent."*
- *"No HTML; keep everything in plain text and cap lessons at 5 minutes."*
- *"Confirm with me before changing the learning goal, and don't schedule extra drills this week."*

If you are learning without saved files, paste the **resume summary** generated at the end of your previous session. Without a summary, the assistant will start with a brief diagnostic check rather than assuming memory of past sessions.

## File Structure

Core files are `SKILL.md`, `README.md`, `README_CN.md`, `LICENSE`, `ATTRIBUTION.md`, and the specifications under `references/`.
The [constructed learning example](examples/python-await/README.md) shows a complete workspace; [manual acceptance scenarios](evals/scenarios.md) define checks without adding runtime dependencies.
Actual learning directories are created on demand when authorized—never automatically upon installation:

~~~text
learning/python-async/
├── MISSION.md                  # Observable goal, completion criteria, preferences
├── RESOURCES.md                # Sources, chapters, versions, and verification status
├── STATE.md                    # Capability evidence, assistance levels, and next steps
├── REVIEW.md                   # Spaced repetition queue read at the start of next session
├── learning-records/0001-*.md  # Summaries of real user attempts (no answer keys stored)
├── lessons/0001-*.md           # Micro-lessons (or HTML when requested)
└── reference/                  # Glossaries and quick-reference sheets
~~~

See [State and Review Specification](references/state-and-review.md) for field schemas and [Lesson Quality Conventions](references/lesson-quality.md) for pedagogical rules.

## Understanding Progress & Spaced Repetition

- **Stable IDs:** Store `unassessed`, `scaffolded`, `provisional`, or `consolidated`; translate display labels without duplicating concept records. Existing Chinese and English labels remain readable.
- **State changes:** Assessable errors or assistance set the current concept to `scaffolded` while preserving history. The next fresh independent success restores `provisional`; consolidation requires another session at least **24 elapsed hours** after the most recent independent success.
- **Review intervals:** Initial or recovery success schedules +24 hours with count 0. Only qualifying due reviews advance the count and schedule +72, +168, then +336 hours. Early practice and duplicate events do not advance it.
- **Scheduling:** Record timezone-aware timestamps; use relative next-session advice when time is unknown. Skips defer the item for the current session without marking failure. Users can change or decline the schedule.
- **Continuation:** Read legacy records without automatic migration, and confirm missing fields. There are no background notifications or automatic graders; the learner chooses practice, maintenance, or completion.

## Examples, Acceptance & Compatibility

Read the [constructed Python await example](examples/python-await/README.md) for a lesson, two session records, and matching state/review files. It contains answers and is not real learner data or evidence of efficacy.
Use the [eight manual scenarios](evals/scenarios.md) to check diagnostics, recovery, skipping, timing, permissions, replay, and cross-language continuation. Record host/model versions and observed behavior; a written scenario is not a passed test.
No real model-behavior or cross-host acceptance run is recorded in this repository yet. Other compatible hosts remain unverified until exercised.

## Limitations & Attribution

- This is an instructional specification for AI agents—it contains no automated grader or efficacy guarantee, and pre-release documentation checks do not replace empirical learner studies.
- Always verify critical sources and operational commands; unverified claims, self-reports, and direct observations are kept strictly separate.
- **Updating:** Check for local edits first; if clean, run `git pull --ff-only` inside your installation directory and re-check `/skills`.
- **Uninstalling:** Remove only the confirmed skill installation directory; keep your learning directories intact.
- Licensed under the [MIT License](LICENSE). Upstream commit details, copyright notices, and changelog notes are documented in [ATTRIBUTION.md](ATTRIBUTION.md).
