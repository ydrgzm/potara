---
name: potara
description: >-
  Fuse two or more skills into one unified skill that combines all their
  instructions, scripts, references, and triggers. Use when the user wants to
  merge skills, compose multiple skills into one, run several skills together as
  a single capability, or "make a fusion skill". The fused result is stronger and
  broader than any input skill on its own.
---

# Potara: skill fusion

Potara merges separate skills into one. The fused skill inherits every fusee's
procedures, helper scripts, reference docs, output conventions, and trigger
phrases, and runs them as a single capability.

The name comes from the Potara earrings in Dragon Ball. That fusion does not
need the two people to be matched in power, and it holds together better than a
Fusion Dance.

## When to use

- "Fuse skill A and skill B"
- "I want these three skills to work as one"
- "Combine the dataviz and report skills"
- "Make a fusion skill from X and Y"
- User points at 2+ skill folders and wants one merged folder

Do not use this for installing a skill, for writing a brand new single skill
(use `skill-creator`), or for running skills one after another without merging
them (just invoke each skill).

## Locating fusees

Given a fusee name, find its folder by checking these paths in order and taking
the first hit:

1. `./skills/<name>/SKILL.md` (project, this repo layout)
2. `./.claude/skills/<name>/SKILL.md` (project-local)
3. `~/.claude/skills/<name>/SKILL.md` (user global)
4. Plugin skill directories: any `*/skills/<name>/SKILL.md` under installed plugins

If a name matches nothing, stop and tell the user. If it matches more than one
location, list the candidates and ask which one. New folders written by `dance`
go to `./skills/<portmanteau>/`. `namek` edits the dominant fusee in whatever
folder it was found in.

## Fusion modes

Pick a mode from what the user asked for. Ask only if it is genuinely unclear.

### `potara` (default): runtime fusion

Merge the fusees' instructions in context for the current session. Nothing is
written to disk. The merge ends when the session ends.

1. Read each fusee's `SKILL.md`, plus any files it points to that you need now.
2. Build one merged procedure. Take the union of the steps, drop duplicates, and
   order them so every step's inputs come from an earlier step.
3. Merge the trigger intent, output format, and rules using the
   **Conflict resolution** section below.
4. State the merged plan back to the user in 3 to 6 lines, then run it.

### `dance`: symmetric permanent fusion

Two fusees of similar scope produce a new skill folder with its own name, formed
as a portmanteau (for example `dataviz` + `report` becomes `datareport`).

This mode needs the fusees to have compatible output types and rules that do not
contradict each other. If the two skills do very different jobs, warn the user
first. A forced Dance produces a weak, defective skill.

1. Read both `SKILL.md` files in full, plus every bundled asset.
2. Write `skills/<portmanteau>/SKILL.md`:
   - A new `name` and `description` that cover both trigger domains.
   - A merged body: shared intro, then the unified procedure, then one combined
     rules and output section.
3. Copy both fusees' scripts and references into the new folder. On a name
   collision, rename to `<fusee>-<file>`. Fix any in-skill path references.
4. Report the new folder tree and the merged description.

### `namek`: asymmetric permanent fusion

One dominant fusee absorbs one or more helper fusees. The dominant skill keeps
its name, voice, and output format. The absorbed skills hand over their
procedures, scripts, and knowledge. This rewrites the dominant skill's folder in
place and cannot be undone.

1. Confirm which fusee is dominant.
2. Tell the user plainly that this permanently rewrites the dominant skill's
   `SKILL.md`, and get explicit confirmation before writing anything.
3. Read all fusees.
4. Copy the dominant `SKILL.md` to `SKILL.original.md` in the same folder. Do
   not overwrite an existing backup; add a `-2`, `-3` suffix instead.
5. Fold each helper's procedure into the dominant `SKILL.md` as new sections or
   steps, rewritten in the dominant skill's voice.
6. Copy the helper scripts and references into the dominant folder.
7. Report what the dominant skill absorbed and the backup path.

## Conflict resolution

Use this order when two fusees give instructions that clash on format, tone,
step order, or a hard rule:

1. **Specificity wins.** The more specific instruction beats the more general
   one.
2. **Dominant wins.** In `namek` mode, the dominant fusee breaks any remaining
   tie. In `dance` mode, take the stricter or safer instruction.
3. **Keep every safety and security rule.** If any fusee has one, the fused
   skill keeps it.
4. **Ask about the rest.** If two instructions cannot both hold, tell the user
   and ask which to keep. Do not guess.

## Output

End every fusion with:

- **Fusees:** names of the merged skills
- **Mode:** `potara`, `dance`, or `namek`
- **Result:** the new skill name for `dance`, the affected folder for `namek`,
  or "runtime only" for `potara`
- **Conflicts:** how each clash was resolved, or "none"
- **Backup:** for `namek`, the path to the saved `SKILL.original.md`

## Failure modes

- **Fusees do unrelated jobs.** The merged description turns vague and stops
  triggering well. Warn the user and offer `potara` runtime mode instead of a
  permanent `dance`.
- **No valid step order.** The merged steps have a cycle and cannot be sorted.
  Report which steps depend on each other.
- **A fusee is itself a Potara output.** This is allowed, but say so. Nested
  fusions drift. Re-fusing the original inputs is usually better.
- **Two assets share a filename.** Never overwrite a fusee's script. Always
  rename.

## Examples

**Runtime fuse for one task**
> User: "Fuse humanizer and no-ai-slop and clean up this draft."
> Mode `potara`. Plan: run the no-ai-slop structural edits first, then the
> humanizer pattern pass. Both skills define a list of AI tells, so union the
> two lists. Then run it.

**Build a permanent combined skill**
> User: "Make one skill out of dataviz and my report-writer."
> Mode `dance`. Write `skills/datareport/SKILL.md`, copy the palette references
> and report templates, use the portmanteau name, and write a description that
> covers both charts and prose.

**Teach an existing skill new tricks**
> User: "Have my deploy skill absorb the rollback skill."
> Mode `namek`. Dominant is deploy. Fold the rollback procedure in as a new
> section, copy its scripts, and keep deploy's voice and output format.
