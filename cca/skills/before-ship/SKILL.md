---
name: before-ship
description: "Decides whether the code is ready to ship, release or deploy: checks secrets, vulnerable dependencies, reachable security bugs, refused fixes and debug settings that would ship, lists what must be fixed before shipping and what can wait, and offers to fix what must be fixed before shipping. Use whenever someone asks if it is safe to deploy, ready for production, ready to release or launch, whether any secrets or vulnerable packages would go out, or what blocks going live. Uses the cca MCP server's tools."
---

# Ready to ship?

**Defaults.** Reader: the person who asked, who writes code and decides whether to ship now. Period: the latest scan plus any
uncommitted changes. Form: Markdown in the chat. The person's own words win over these defaults: the reader, the focus and the
form (a PDF or a document: make the file with what your tool has, or say it cannot). When something is missing, take the default,
do the work and say in one line which default you used; never hand the person a list of technical options to choose from.

**Words.** The **server** is the cca MCP server (tools `cca:<tool>`). A **scan** is one analysis of the repository by the server, named by commit and
date. A
**finding** is one problem at one place; a **cause** groups findings with one fix. The **plan** is the ranked list of causes in
`cca:audit`, each with its point gain and effort. **Security findings** are those of `D13` (secrets in the code), `D28` (secrets
in the git history), `D29` (security bugs), `D30` (vulnerable dependencies), `D31` (container and infrastructure configuration)
and `D32` (personal data). **Drift** is a `closes-cycle` or `points-outward` verdict from `cca:check_dependencies`; a **new dependency**
is a `new-dependency` verdict from it.

## Rules for this skill

- **Scan and reading apart.** Name the scan for what it found and the files for what you read; never merge the two.
- **Numbers.** Every number comes from the server's answer or a count you made; every point or effort figure from the plan.
- **Findings.** A finding is measured the same
  way on every repository: never call a correct one a matter of taste. Never explain how the score is counted; say what was
  found and what it means.

## What goes under Fix before shipping
Only what meets one of these rules:
- an open finding with a `refusedResolution` (an earlier fix that the scan found again);
- a secret (`D13` in the code, `D28` in the history), or a `D29`, `D31` or `D32` finding a user or outside caller can reach;
- a `D30` dependency with a known vulnerability in code that ships;
- debug settings or test credentials in the configuration that ships;
- drift, or a `new-dependency` verdict you cannot give a plain reason for.

**Can wait** names at most the plan's first three causes not under Fix before shipping, one line each.

## Workflow

Copy this checklist and tick it off:

```
Before-ship progress:
- [ ] 0. Read the scan
- [ ] 1. Refused fixes
- [ ] 2. Security findings
- [ ] 3. Read the code
- [ ] 4. Uncommitted changes
- [ ] 5. Configuration and start-up
- [ ] 6. Write the verdict
- [ ] 7. Check the verdict
```

**0. Read the scan.** `cca:audit`: keep `scan` (commit and date), the plan and `queue.resolutionsRefused`. If the server's tools
are missing or say you must sign in, stop before you write anything and tell the person in one line: in Claude Code,
type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write without the scan only if they ask,
and then say at the top that it was not used.

**1. Refused fixes.** If `queue.resolutionsRefused` is 0, go to step 2. Otherwise `cca:list_findings` with `state` `open`, every
page by `offset` up to `totalMatching`; keep each row whose `refusedResolution` is not null, with its note.

**2. Security findings.** `cca:list_findings` with `dimension` set to each of `D13`, `D28`, `D29`, `D30`, `D31`
and `D32` in turn and `state` `open`, every page by `offset` up to
`totalMatching`. `cca:get_finding` with the fingerprint for each whose message does not say what and where.

**3. Read the code.** Open every file the findings of steps 1 and 2 name: can a user or an outside caller reach it, and what does
the fix take? One that does not hold: `cca:dispute_finding` with your evidence; it then leaves Fix before shipping.

**4. Uncommitted changes.** If `git status --porcelain` prints nothing, go to step 5. Otherwise call `cca:check_dependencies` with
the output of `git diff -U100000 HEAD` (whole files as context, so the server sees each import in
place) and keep each dependency's verdict. Files marked `??` are not in that diff: name them under
Not checked. If the call fails: put the uncommitted changes under Not checked.

**5. Configuration and start-up.** Read how the application is configured and started (environment, build and deployment files)
for debug settings or test credentials that would ship.

**6. Write the verdict.** For a reader who writes code:

```markdown
**<Ready | Not yet>**: <one line why, with the scan's commit and date>
## Fix before shipping
1. <Where> - <why it matters> - <what to do>
## Can wait
<At most the plan's first three causes not under Fix before shipping, one line each, with points and effort; "Nothing" if the plan is empty.>
## Not checked
<What neither the scan nor your reading covered.>
I can fix the Fix before shipping list now, one item at a time.
```

When Fix before shipping is empty, write "Nothing." under it and leave out the offer line.

For a reader who does not write code: no tool names, fingerprints, commit hashes or dimension ids; say what was not checked as
plainly as what was.

```markdown
**<Ready | Not yet>**: <one line why, with the scan's date>
1. <What is wrong, in plain words>. <What the team does about it.>
Can wait: <the same causes, in plain words, one line>. Not checked: <one line>.
```

When nothing must be fixed first, write "Nothing must be fixed first." in place of the numbered list.

**7. Check the verdict.** Check: "Ready" only if Fix before shipping is empty; every item there meets a rule under What goes under
Fix before shipping; in the first template every item says where, why and what, in that order, and in the second what is wrong
and what the team does; every file named exists; every effort or point figure comes from the plan; scan facts and your reading
are kept apart. Fix what fails and check again.

**Closing paragraph.** End with the closing paragraph the server's instructions describe, from the `contribution` items that changed this answer.

## When the person says fix

Fix one item at a time; `cca:resolve_finding` with its fingerprint and a note on what changed. A secret in the git history cannot be
resolved by an edit: rotate it and say so. After the last item, one `cca:request_rescan`, then `cca:audit`. With a new `scan`:
check that `queue.resolutionsRefused` did not rise and `cca:get_finding` returns null for each fixed fingerprint; reopen what
failed and write the verdict again. With the old `scan`: say the fixes are made and the next scan checks them.

## Examples

For the developer who asked:

> **Not yet**: one secret is in the code and one dependency has a known remote exploit (scan `8d2e41b`, 12 Nov).
>
> ## Fix before shipping
> 1. `config/settings.py` line 14 - a live payment API key is committed and ships in the image - move it to the environment and
>    rotate the key.
> 2. `package.json` - `express-fileupload` 1.1.7 has a known remote code execution flaw, reachable from the upload form - raise it
>    to 1.4.0.
> ## Can wait
> The three largest classes in `src/orders/` (plan: 3.1 points, medium effort).
> ## Not checked
> Whether the production environment sets its own secrets; the scan does not see the server.
>
> I can fix the Fix before shipping list now, one item at a time.
>
> Without the scan, this verdict would have missed the payment key, which sits in a settings file no recent commit touched.

For a client who does not write code:

> **Not yet**: two problems must be fixed first (scan of 12 November).
> 1. A payment password is stored inside the program, so anyone with a copy can use it. The team moves it out and changes it.
> 2. A component used for uploads has a known flaw that lets an outsider take over the server. The team updates it.
> Can wait: tidying three oversized parts of the order handling. Not checked: how the live server is set up, which the scan
> cannot see.
>
> Without the scan, this answer would have missed the payment password, which no one had looked at this year.
