# iris-gotcha — Developer Context

A Claude Code plugin that ships one Skill (`iris-gotcha`). Pure Markdown — no build step. Future Claude sessions working on this repo: see below.

## Anatomy

```
.claude-plugin/
  plugin.json          # plugin manifest (CC reads on /plugin install)
  marketplace.json     # marketplace registry (CC reads on /plugin marketplace add)
skills/iris-gotcha/
  SKILL.md             # main skill: triggers + capture/recall/audit/push flows
  definitions.md       # CANONICAL 7-category taxonomy — edits here ripple to every future classification
```

## Schema gotchas (don't repeat these)

- `plugin.json` → `repository` MUST be a **string URL**, not `{type, url}`. CC validator rejects the object form.
- `marketplace.json` → for a plugin co-located with the marketplace, use `"source": "./"` (string). The `{source: "github", repo: ...}` form makes CC clone from a remote even when the marketplace was added from a local path.
- Full incident notes live in the author's local notebook: `~/.claude/iris-gotcha/lesson/2026-05-15-cc-*.md`

## Dev loop

Editing `SKILL.md` or `definitions.md` (content changes):

```bash
# Just edit. Changes pick up on next session start. /reload-plugins to refresh in current session.
```

Editing `plugin.json` / `marketplace.json` (manifest changes):

```bash
/plugin marketplace remove iris-gotcha
/plugin marketplace add danielwii/iris-gotcha   # or local path during dev
/plugin install iris-gotcha
/reload-plugins
```

## Version bump checklist

Both files must move together — they're independently validated and a mismatch can cause silent staleness:

- [ ] `.claude-plugin/plugin.json` → `version`
- [ ] `.claude-plugin/marketplace.json` → `metadata.version` AND `plugins[0].version`
- [ ] Update README's "Versioning" section with changes
- [ ] `git tag v<new>` + push
- [ ] (optional) GitHub release

## Skill design invariants

- **7 categories are canonical**: `experience` / `lesson` / `rule` / `architecture` / `topology` / `habit` / `best-practice`. Adding/removing requires updates in BOTH `definitions.md` AND the index template inside `SKILL.md`.
- **Category identifiers are English; Chinese kept as glosses** (since v0.2.0). The `type:` frontmatter field always uses the English identifier. Directory names also use the English identifier.
- **`disambiguation` frontmatter field is mandatory** on every entry. Removing this field is a regression — the anti-collapse mechanism depends on it.
- **Severity ladder lives only in `SKILL.md`** (5 levels: `low` / `medium` / `high` / `critical` / `zero-tolerance`). Don't fork this elsewhere.
- **No hooks.** All triggers are LLM-driven via the skill description. If you're tempted to add a PreToolUse/PostToolUse/etc. hook, reconsider — that path was deliberately rejected during design (see `/Users/daniel/.claude/plans/rule-hazy-hickey.md`).

## What this plugin is NOT

- Not a memory store like `claude-mem` (which records *what happened*). This records *what we learned should happen next time*.
- Not a CLAUDE.md auto-updater like `claude-diary` (which distills patterns into rules). This captures explicit, classified entries on demand.
- Not a knowledge graph like GaZmagik `claude-memory-plugin` (which links entries semantically with embeddings). This uses keyword + LLM judgment only.

## Memory data lives elsewhere

Plugin code: this repo.
User entries: `~/.claude/iris-gotcha/` on each machine (not committed here; sync mechanism deliberately deferred — see Open questions in the design doc).
