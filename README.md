# Canine Skills

Skills for your own AI tool from Canine Development: it writes about your codebase (an assessment, an onboarding note, a changelog,
release notes, a sprint plan, a review before you ship) with the facts our scan measured beside its own reading of the code. Your AI
writes, at your cost and in your words; the scan gives it what reading files cannot: the whole repository at once, its git history,
today's vulnerability databases and whether earlier fixes held. Open source (MIT).

## Get them

**The easy way: ask your AI.** Paste this into your AI tool:

> Get the Canine skills for me from https://github.com/CanineCC/canine-skills . In Claude Code: run `claude plugin marketplace add
> CanineCC/canine-skills` and then `claude plugin install cca@canine`. In any other tool: copy the folders under `cca/skills` (each
> holds a SKILL.md) into the folder where you read skills; if you read no skills folder, add a line to AGENTS.md in my project saying
> to read the matching skill there before writing about the codebase. Then tell me in one sentence what you did.

**Claude Code by hand:**

```
claude plugin marketplace add CanineCC/canine-skills
claude plugin install cca@canine
```

(Inside a session: `/plugin marketplace add CanineCC/canine-skills`, then `/plugin install cca@canine`.) The plugin brings the
skills, the `cca` MCP server they read from and the `/cca-fix` command. Once installed, your AI picks a skill by itself when you ask
for that kind of work, like any other skill: "write an onboarding note for Lasse", "is this ready to ship?", "release notes for our
customers as a PDF".

The skills are plain Markdown (`SKILL.md`), generated from one source in Canine's own repository. They read their facts through the
`cca` MCP server, which serves data only.

## CCA (for Assure, Assert and CCA)

One MCP server for every product: `/mcp/cca`.

- **MCP server `cca`** at `https://app.assure.canine.dev/mcp/cca` unless `CCA_MCP_URL` says otherwise. **Sign in, no key:** type
  `/mcp` in Claude Code, pick `cca` and sign in. Your browser opens the sign-in page; pick the project and click Allow. That makes
  a key for that one repository; switch it off again on the product's "Set up your AI tool" page. A repository key from a
  product's AI tool page works too (`Authorization: Bearer`), as does an organisation's API key (`cca_…`), which gets the `/v1`
  API as tools instead of the fix loop. Allow `mcp__cca__*`.
- **`/cca-fix`** works the repository's findings: `next_task` (security first), fix every instance, `resolve_finding` each,
  dispute or accept what should not be fixed, until `next_task` returns nothing. The next scan checks each fix.
- **`/cca-scan`** starts a scan of the repository your working tree belongs to (matched by its `origin` remote) through the API,
  with your organisation's API key in `CCA_API_KEY` (scopes `account:read`, `scans:write`, `runs:read`, `results:read`; the
  address is `https://api.canine.dev` unless `CCA_API_URL` says otherwise), waits for the run and reads the scores and open
  findings into the session. It measures the default branch's latest pushed commit, never local changes. It pre-approves only its
  own script.
- **Skills** (`skills/`): `assess` (a written assessment for a developer, management or a buyer), `onboard` (a note for someone new to
  the codebase), `progress` (how quality has moved, and whether reported fixes held), `compare` (review a few files yourself, then
  compare with what the scan found), `before-ship` (ready or not yet, and what must be fixed first), `safe-change` (before you change
  a file: what changes with it, what depends on it, what is already wrong in it), `sprint-planning` (what to take into the next
  sprint, fitted to the capacity), `retrospective` (how the code did over a sprint, and what to try next) and `changelog` (from the
  git history, with security from the scan). In each, the scan is your AI's map: it has read the whole repository, its git history and the
  vulnerability databases, so your AI knows where to look, then opens those files itself, checks each finding in context and adds
  what only reading shows, keeping what the scan measured apart from what it saw. Your AI uses them when you ask for that kind of
  work.
- **Reminder:** when a change touches C#, Java, Kotlin or Scala, a Stop hook reminds the agent to run `check_dependencies` before
  reporting the work done. It never blocks. Turn it off with `CCA_NUDGE=off`.

## Assay

- **Key:** an organisation API key with the scope `results:read`, from Assay's "Your AI assistant" page, in `ASSAY_API_KEY`.
- **MCP server `assay`** (`/mcp/assay`): READ-ONLY. list_codebases, get_codebase, get_report (the decision report's text and
  verdict) and explain_dimension (what a dimension means for a deal and who should care). It changes no code and orders nothing.
  Allow `mcp__plugin_assay_assay__*`.
- **Skill** `due-diligence`: a memo on one codebase for the people deciding: what you would inherit, the risks that carry a price,
  what to ask the seller and what the seller should fix or disclose before closing. Where you have the code, your AI opens what the
  report points to. With no Assay key but the `cca` server connected, it writes the memo from that project's latest scan and says that
  nothing in it is verified.
