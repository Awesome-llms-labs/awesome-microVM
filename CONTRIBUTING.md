# Contributing

Thanks for helping keep this list the most current directory of the microVM ecosystem!

## Adding an entry

1. **Check it fits:** a microVM / lightweight VMM, an orchestration or integration tool for them, a building-block crate ecosystem, or a managed platform / production user built on one. (We deliberately excluded unvetted tiny entrants — see README scope notes — so a PR must point at a primary source: the project's repo or official docs.)
2. **Add to the right section** of `README.md` (alphabetical-ish order within a section is fine, keep categories pure):
   - MicroVMs & lightweight VMMs → runnable monitors, runtimes, and embeddable libraries
   - Building blocks → shared crate/component ecosystems, not runnable VMMs
   - Orchestration & integration → lifecycle tooling on top of VMMs
   - Managed platforms & production users → the table (platform → underlying tech, with an evidence column)
   - Archived / deprecated → retired projects with the archived date and named successor
3. **One entry = one bullet** (or one table row for platforms). Format:
   `- [Name](https://official-site-or-repo) — ` one-line description + 2–4 key features inline. (Shown as an unlinked code example above — use the real official URL.)
   Tag license confidence honestly: ⚠️ "reported, not re-verified from primary files" when you haven't read the project's license file yourself.
4. **Add the matching record** to `data/microvms.json` with these exact fields:

| field | type | values |
|---|---|---|
| `name` | string | project name |
| `url` | string | official https:// URL |
| `description` | string | one sentence |
| `language` | string | primary language, or `"n/a"` |
| `license` | string | SPDX id, or `"unknown (not verified)"` / `"n/a (managed service)"` |
| `license_verified` | bool | `true` only if you read the license in a primary file |
| `status` | string | `active` / `maintenance` / `archived` / `commercial` |
| `category` | string | `vmm` / `orchestration` / `platform` / `archived` |
| `features` | string[] | 3–6 key capabilities |

5. **Status changes:** if a project is archived, deprecated, acquired, or changes license, update its README entry *and* add a row (newest-first) to `docs/status-changes.md`.

## Style rules

- Link the **official site** (repo or docs homepage), never a blog post or reseller.
- Facts that can change (release versions, pricing, star counts) get "as of" context in prose or are omitted — link the source instead of hard-coding numbers that rot.
- Vendor claims stay labeled as vendor claims ("vendor claims", "reported by community sources").
- Keep README descriptions to one entry per bullet; put depth in `docs/`.

## CI

Every PR runs:
- **Link check** (lychee) over all markdown files — no dead links.
- **JSON validation** — `data/microvms.json` must parse, every record must have the required fields, and `status`/`category` must be from the allowed sets above.

Run locally before pushing:

```bash
python3 -c "import json; d=json.load(open('data/microvms.json')); print(f'ok: {len(d)} entries')"
```
