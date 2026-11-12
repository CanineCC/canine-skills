# Canine Skills

Skills and plugins for your AI coding tool from Canine Development, built on what our scans measure. One marketplace (`canine`),
published as `CanineCC/canine-skills` and staged in Canine's own repository, where every skill is generated from the same source the
service serves, so the two never differ. Open source (MIT).

```
claude plugin marketplace add CanineCC/canine-skills
claude plugin install cca@canine
```

(Inside a session: `/plugin marketplace add CanineCC/canine-skills`, then `/plugin install cca@canine`.)

The skills are plain Markdown (`SKILL.md`), so any AI tool that reads skills can use them; they work through the `cca` MCP server.

## CCA (for Assure, Assert and CCA)

One MCP server for every product: `/mcp/cca`.

- **MCP server `cca`** at `https://app.assure.canine.dev/mcp/cca` unless `CCA_MCP_URL` says otherwise. **Sign in, no key:** type
  `/mcp` in Claude Code, pick `cca` and sign in. Your browser opens the sign-in page; pick the project and click Allow. That makes
  a key for that one repository; switch it off again on the product's "Set up your AI tool" page. A repository key from a
  product's AI tool page works too (`Authorization: Bearer`), as does an organisation's API key (`cca_…`), which gets the `/v1`
  API as tools instead of the fix loop. Allow `mcp__plugin_cca_cca__*`.
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
  a file: what changes with it, what depends on it, what is already wrong in it) and `review-branch` (a branch's scan against the
  default branch, and your AI's own reading of the diff). In each, the scan is your AI's map: it has read the whole repository, its git history and the
  vulnerability databases, so your AI knows where to look, then opens those files itself, checks each finding in context and adds
  what only reading shows, keeping what the scan measured apart from what it saw. Your AI uses them when you ask for that kind of
  work; each is the same text the server offers as an MCP prompt of the same name, generated from one source.
- **Reminder:** when a change touches C#, Java, Kotlin or Scala, a Stop hook reminds the agent to run `check_dependencies` before
  reporting the work done. It never blocks. Turn it off with `CCA_NUDGE=off`.

## Assay

- **Key:** an organisation API key with the scope `results:read`, from Assay's "Your AI assistant" page, in `ASSAY_API_KEY`.
- **MCP server `assay`** (`/mcp/assay`): READ-ONLY. list_codebases, get_codebase, get_report (the decision report's text and
  verdict) and explain_dimension (what a dimension means for a deal and who should care). It changes no code and orders nothing.
  Allow `mcp__plugin_assay_assay__*`.
- **Skill** `due-diligence`: a memo on one codebase for the people deciding: what you would inherit, the risks that carry a price,
  what to ask the seller and what the seller should fix or disclose before closing. Where you have the code, your AI opens what the
  report points to. The same text the server offers as an MCP prompt.
