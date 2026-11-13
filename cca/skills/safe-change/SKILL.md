---
name: safe-change
description: "Prepares a change to one file so it does not break what depends on it: the files that change together with it, the modules that depend on it, the findings already in it, the tests that cover it and the order to work in, written before any code. Use whenever someone is about to change, refactor, move or delete a file or module and asks what to watch out for, what could break, or what touches it. Uses the cca MCP server's tools."
---

# Before changing a file

Default reader: the developer about to make the change; file paths, function names and commit ids are expected. Period: the whole
git history the scan read. Form: a change brief in the chat. If the reader does not write code (a product owner, a manager),
follow "For a reader who does not write code" below. The file is the one the person names; if none, the one file with
uncommitted changes (`git status`); if there is not exactly one, ask which file.

## Words
- **server**: the cca MCP server; its tools are named `cca:<tool>`. A **scan** is one run of it, named by its date and commit.
- **finding**: one problem the scan found at a place in the code. A **refused fix** is a finding with a `refusedResolution`:
  reported fixed and found again by a later scan.
- **change brief**: what this skill writes. **Co-change partners**: files that changed together with this one in the git
  history. **Dependent modules**: every module that depends on the file's module, directly or not. **Callers**: the places in
  the code that call it, found by reading.

## Rules for this skill
- If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in one line: in
  Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write without the scan
  only if they ask, and then say at the top that the scan was not used.
- The person's own words win over these defaults: the reader, the focus and the form (a PDF or a document: make it with what
  your tool has, or say it cannot). When something else is missing, take the default and say in one line which you used. Never
  hand the person a list of technical options to choose from.
- Every number comes from a server answer or a count you made with git or the files, and names its scan. A claim about what a
  test or a caller does comes from reading it; if you did not read it, say so.
- Keep what the scan measured (name the scan) apart from what you read (name the files).
- A finding is measured the same way on every repository: never call a correct one a matter of taste.
  Never explain how the score is counted; say what was found and what it means.

## Workflow

Copy this checklist and tick it off:

```
Safe-change progress:
- [ ] 0. Ask the server about the file
- [ ] 1. Co-change partners and dependent modules
- [ ] 2. What is already wrong there
- [ ] 3. Read the file, its partners, tests and callers
- [ ] 4. Write the change brief, before any code
- [ ] 5. Make the change, if asked, and check the diff
- [ ] 6. Check the change brief and add the closing paragraph
```

### 0. Ask the server about the file
Call `cca:get_architecture` with `path` set to the file. If the tool is missing or says you must sign in, follow the sign-in rule.

### 1. Co-change partners and dependent modules
From that answer keep the co-change partners, with how often each changed with the file, and `baseline` (the scan and its
commit). If no partner is named, quote its `note`. The file's module is the namespace or package declared in its first lines, or
else its directory. Call `cca:get_architecture` with `module` set to it (if the answer offers closest names, take the one that
matches the file's directory) and keep the dependent modules.
If no name matches: say in one line that the scan has no module for this file, and leave dependent modules out.

### 2. What is already wrong there
`cca:list_findings`, walking every page with `offset` until `truncated` is false. Keep the findings in the file and in its
co-change partners. For each refused fix, read its `refusedResolution` note before trying the same fix.
If no finding matches: the brief says the scan found nothing open there.

### 3. Read the file, its partners, tests and callers
Open the file, its co-change partners, its tests and its callers. A test you did not open is named as not read.

### 4. Write the change brief, before any code
Use this shape:

```markdown
# Changing <file>

<Two sentences: what the change touches beyond this file, and the main risk. Scan of <date>, commit <baseline commit>.>

## What it touches
<Each co-change partner (changed together in <n> of <m> commits), caller and dependent module, and why it matters here.>

## What is already wrong here
<Findings to fix with this change, those it must not make worse, and each refused fix with what already failed.>

## Tests
<Which tests cover it and what they check; which are missing.>

## Order
<The steps, in the order to make them.>
```

### 5. Make the change, if asked, and check the diff
Only if the person asked for the change; otherwise end the brief with one line saying no code was changed. After the change,
pass `git diff -U100000 <baseline commit>` as `diff` to `cca:check_dependencies` (the wide context puts each changed file in
whole, so its imports are seen). For each answer with `"drift": true`, fix the dependency or say plainly why it is intended, and
pass the new diff again, until every remaining drift is one you explain. End the brief with one line on how the check ended.
If the tool fails: say the dependency check was not run.

### 6. Check the change brief and add the closing paragraph
Check, one by one:
- every file you name exists;
- every co-change partner and dependent module came from step 1, with its count;
- every claim about a test comes from reading it, and an unread test is named as not read;
- every refused fix from step 2 is under "What is already wrong here" with what already failed;
- the scan is named (its date, and its commit unless the reader does not write code);
- the brief says whether code was changed, and how the dependency check ended;
- for a reader who does not write code, the brief passes the section below.

Fix each item that fails, then check the whole list again. When all pass, add the closing paragraph: a short paragraph on what
the server gave this answer, only from the `contribution` items that changed it, each as what the brief would otherwise have
got wrong or lacked. If none changed it, leave the paragraph out.

## For a reader who does not write code
The risk in plain words first, on one screen, with the same numbers as the body. No tool names, commit hashes or check ids; the
scan by its date only. Say what was not read or checked as plainly as what was.

## Examples

For the developer making the change, lines under "What it touches", "What is already wrong here" and "Order", and the last line:

> `src/orders/checkout.py` changed together with this file in 18 of its 22 commits (scan of 12 November, commit `a1b2c3d`), and
> `CheckoutView.submit` calls `apply_discount` expecting a total that is never negative; keep that true.
>
> The scan of 12 November finds `apply_discount` too complex (one finding); splitting it is part of this change. A refused fix:
> reported fixed in `d4e5f6a` and found again, because the split left the rounding branch in `apply_discount`; do not repeat it.
>
> 1. Add a test for a discount larger than the total. 2. Split `apply_discount`, rounding included. 3. Change `checkout.py` in
> the same commit.
>
> No code was changed; the dependency check was not run.

For a product owner asking what could break, the opening:

> Changing how prices are worked out also touches checkout, which changed together with it in 18 of 22 changes (scan of
> 12 November), and the order and invoice parts that rely on it. The main risk is a negative total reaching an invoice, and no
> test checks for that today.
