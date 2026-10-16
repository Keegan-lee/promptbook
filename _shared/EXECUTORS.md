# Running a kit with any executor

The kits are plain Markdown. Any agent that can read files, talk to a person, and call that person's tools can run them: Claude (Code, desktop, or web), Cursor, ChatGPT, or another agent. This file says what the executor needs, how to start it, and what to do when it lacks something.

## What the executor needs

| Need                        | Why                                                                 | If it is missing                                                                                                                         |
| --------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Read the kit and `_shared/` | The kit is the instructions.                                        | Upload or attach the folder. Every file the kit names must be readable, or the executor stops and says which one is missing.             |
| Talk to the person          | Every kit starts with an interview.                                 | None. A kit is never run unattended the first time.                                                                                      |
| Connectors to their tools   | The build writes to Airtable and reads mail, drive, CMS, and so on. | Use the fallback in the kit's capability table. Never fake a connector by asking for a password or key in chat.                          |
| A place to schedule jobs    | Routines run on a timer after the build.                            | Create each routine in another runtime from `ROUTINES.md` → "Where a job runs". The executor still does the build and writes the prompt. |
| Somewhere durable to write  | The briefs, the ID map, and the operator's manual.                  | Ask the person where (`INTERVIEW.md` rule 8). Never leave a brief only in chat.                                                          |

Connectors may be MCP servers, built-in apps, or plain API access through a workflow engine. The kits name **capabilities** ("the CRM base", "chat", "the keyword tool"), never one vendor's tool names. The build agent writes the real tool names into each routine when it fills the template.

## Start prompt

Paste this into the executor, with the kit folder and `_shared/` attached or checked out:

```
You are setting up part of our company runbook.
Read prompts/README.md, then prompts/_shared/EXECUTORS.md, INTERVIEW.md, AIRTABLE.md and ROUTINES.md.
Then read prompts/<kit>/README.md and run prompts/<kit>/KICKOFF.md.
List the connectors you can reach before you ask me anything.
Interview me one block at a time. Do not build until I say "go" to the brief.
```

To operate a kit that is already built, point the executor at that kit's filled `OPERATORS.md` and the brief instead of the kickoff.

## Before the first question

1. **List what you can reach.** Name every connector, server, or app available in this session, and which of the kit's capability rows each one covers. Some executors load tools lazily: search for them by capability ("airtable", "slack", "google drive") before you conclude a tool is absent.
2. **Look for what already exists** (`INTERVIEW.md` rule 3): an earlier Company brief, a base with a matching name, a brand or sales playbook, existing routines with the kit's prefix.
3. **Say what is missing**, with the fallback you will use, in the first message.

## Routines across runtimes

A routine template is one self-contained prompt. To move it between runtimes:

- Paste the filled prompt as the job's instructions. Do not split it across several messages or "memory" features: the routine must read the whole thing on every run.
- Give the job only the connectors the routine's hard rules name (`ROUTINES.md` → "Least privilege"). If the runtime cannot restrict tools per job, say so in the brief and keep the routine at `Dry Run` or `Drafts` until a person has read several runs.
- Check how the runtime reads the schedule's time zone. Write the cron in the home time zone where the runtime allows it; otherwise convert to UTC and note the daylight-saving shift in the brief.
- Some runtimes attach every connected account when a job's connector list is left empty. Always pass an explicit list, then read the job back and confirm it.
- Read the job back after every create or update. Compare the stored prompt with the filled template, character for character where the runtime allows it.

## Choosing a model

Use the most capable model the person has for routines that judge, write in someone's voice, reconcile two systems, or decide a status. A smaller model is fine for deterministic summaries over data a routine has already fetched. Record the model per routine in the brief, so a later change can be compared against a known baseline.

## Handing over

Every kit ends the same way, whatever the executor:

1. The brief, with its ID map filled.
2. Each routine created **disabled**, with its filled prompt stored where the person can read it.
3. `MANUAL.md` steps the person still has to click, with live links.
4. A filled `OPERATORS.md` for the team.
5. One line on what to run first, and what "working" looks like after a week.
