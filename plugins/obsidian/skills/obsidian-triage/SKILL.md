---
name: obsidian-triage
description: >
  Manages an Obsidian inbox triage session — sorting clippings, fleeting notes, and other inbox items into their correct vault locations (Areas, Reference, Archive, or deletion). Use this skill whenever the user is going through inbox files and deciding where each one belongs, discussing what to do with a clipping or note, or explicitly calls /obsidian-triage. Proactively invoke at the start of any inbox-sorting workflow, even if the user just says "let's go through my inbox" or "let's sort these notes". The skill maintains a live Triage Session note in the vault that tracks every decision and outstanding to-do as the conversation progresses.
---

# Obsidian Inbox Triage

This skill manages a structured triage session for sorting Obsidian inbox items. It keeps a live session note in the vault updated after each decision, so nothing is lost and the user has a permanent record.

## Starting a Session

### Offer a workflow mode first

Before diving in, check how many files are in scope. If there are more than ~10 files, offer the user a choice:

> "You have X clippings. Want to go one by one, or should I scan them all first and give you a grouped overview by topic so you can triage in batches?"

- **One-by-one**: present each file, discuss, decide, move on
- **Batch overview**: scan all files, group by topic (Woodwork, Programming, Personal Development, etc.), present the clusters, let the user make broad decisions (e.g. "move all woodworking ones to Reference/Woodwork"), then handle exceptions individually

Either way, initialise the session note first.

## Session Note

### Initialisation

At the start of a triage session, create the session note if it doesn't already exist:

```bash
obsidian create name="Triage Session YYYY-MM-DD" content="<initial content — see template below>" silent
```

Use today's actual date. If a session note for today already exists, continue appending to it rather than creating a new one.

### Template for new session note

```markdown
---
created: <ISO timestamp>
tags: [triage]
status: in-progress
scope: <folder being triaged, e.g. 0.1 Inbox/Clippings>
mode: <one-by-one or batch>
total: <file count, set after scanning>
processed: 0
---

# Triage Session <YYYY-MM-DD>

## Decisions Log
| File | Origin | Destination | Rationale |
|------|--------|-------------|-----------|

## Action Items

## Deferred
```

## Updating the Note After Each Decision

After every file is decided, update the session note. Use `obsidian append` to add to specific sections.

### After a move/delete decision

Append a row to the Decisions Log and increment the processed count:

```bash
obsidian append file="Triage Session YYYY-MM-DD" content="\n| [[<filename>]] | <origin path> | <destination or DELETED> | <one-line rationale> |"
obsidian property:set name="processed" value=<new count> file="Triage Session YYYY-MM-DD"
```

Track the count yourself — increment by 1 after each decision.

### After a to-do is created

Action items always go in the session note's Action Items section — not appended to the clipping file itself. The session note is the single place the user checks after triage.

```bash
obsidian append file="Triage Session YYYY-MM-DD" content="\n- [ ] <action item, e.g. 'Watch and take notes from [[How to Get into FLOW Every Time You Work]]'>"
```

To target a section directly:

```bash
obsidian append file="Triage Session YYYY-MM-DD" section="Action Items" content="- [ ] <item>"
```

(If the CLI supports `section=`, prefer it. Otherwise append at the end — the heading structure will still group them visually.)

### After deferring a file

```bash
obsidian append file="Triage Session YYYY-MM-DD" content="\n- [[<filename>]] — <reason deferred>"
```

## File Operations

Use the obsidian CLI for all moves so Obsidian's link index stays consistent:

```bash
# Move a file
obsidian move path="0.1 Inbox/Clippings/<filename>.md" to="03 Reference/<filename>.md"

# Move to a subfolder
obsidian move path="0.1 Inbox/Clippings/<filename>.md" to="02 Areas/Woodwork/<filename>.md"

# Delete (no CLI delete — tell the user to delete manually, or use Bash rm as a last resort)
```

For deletion, prefer telling the user to delete from within Obsidian (right-click → Delete) to keep the link index clean. Only use filesystem deletion if the user explicitly requests it.

## Triage Decision Framework

When discussing each file with the user, the key question is: **what is this and will I ever return to it?**

| Type | Destination |
|------|-------------|
| Evergreen how-to, reference material, build plans | `03 Reference/` — organised by topic (Woodwork, Programming, PKM, etc.) |
| Ongoing responsibility or practice (health, PKM, flying) | `02 Areas/` — under the relevant area |
| Completed or one-off project resource | `04 Archive/` |
| No notes captured, no clear future use | Delete or defer |
| Needs further engagement before filing | Add to-do, defer in session note |

For clippings with **no notes captured** (just the video description or article summary), surface this explicitly: "There are no notes here — just the auto-captured description. Do you want to watch/read it first, or file it as a bookmark-level reference?"

## Wikilink formatting

Always format file names as Obsidian wikilinks in the session note: `[[Note Name]]` (no path, no extension). Obsidian resolves these automatically.

## Ending a Session

When the user is done (or explicitly wraps up), mark the session complete:

```bash
obsidian property:set name="status" value="complete" file="Triage Session YYYY-MM-DD"
```

Then give a brief summary: how many processed, any outstanding action items, any deferred files.

## Session Continuity

If the user returns to triage later in the same conversation, check whether the session note already exists before creating a new one. If it exists, read it to restore context (what's been processed, what's pending) before continuing.

## What to Track Without Being Asked

- Every file decision (move, delete, defer) → Decisions Log row
- Every to-do mentioned → Action Items
- Every file the user wants to revisit → Deferred section
- Count of processed files (maintain mentally, update Stats conversationally)
