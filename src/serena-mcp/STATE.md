---
topic: serena-mcp
title: "Serena MCP: Semantic Code Tools for Smarter AI Coding Agents"
status: publish
updated: 2026-07-22
---

## Goal
Explain what the Serena MCP server is, why it makes AI coding agents more accurate and token-efficient, how it works (LSP-backed semantic tools), and its pros and cons — for **Intermediate** developers already using AI coding agents (Claude Code, Cursor, etc.).

## Progress
- [x] Research — references.md created (8 confirmed, 1 optional TODO placeholder: REF-7)
- [x] Draft — article.md v1 written
- [x] Review — Reviewer Notes applied; house style, SEO, and reference completeness verified
- [ ] Publish

## Next step
Publish. Article is clean — no `[REF-N] TODO` in the body, house style met, all inline claims map to confirmed references (REF-1, 2, 3, 4, 5, 8, 9).

## Notes / open questions
- Confirm target audience (assumed Intermediate) and BLUF at Phase 0 checkpoint.
- Installation method conflict: official README uses Python/`uv` (`uv tool install serena-agent` / `uvx`); one third-party post claims `npm install @oraios/serena` — the README is authoritative, npm claim looks wrong. Writer should not repeat the npm command.
- Token savings: the widely repeated "~70%" is ANECDOTAL only (REF-5, DEV.to hearsay) — cite as anecdote, not benchmark. The only hard, methodology-backed number found is Serena's tool-schema context OVERHEAD: 29 tools ≈ 7,348 schema tokens / ≈ 23,878 tokens in Claude Code (REF-8, serena-slim) — that is a cost, not a saving. Keep headline savings claims qualitative.
- RESOLVED: ManoMano "Project AEGIS" benchmark (REF-9) now VERIFIED. On a large Java refactor (381 classes / 36,407 LOC / 1,017 tests): Claude+Serena = 45 min, $27.30, 4 subagents, ALL tests pass; Vanilla Claude = 60 min, $23.54, 12 subagents, build FAILED; Claude+built-in-LSP = 60 min, $28.63, build FAILED (LSP hallucinated). Honest con: on trivial read-only queries Serena cost ~4x and was ~60% slower than vanilla. Writer should lead the "why" with this reliability story, not a flat token %.
- RESOLVED (REF-6, verified 2026-07-22 against live source/docs): overview tool is `get_symbols_overview` (not `symbol_overview`); delete tool is `safe_delete_symbol`. Ad-hoc launch is `uvx -p 3.13 --from git+https://github.com/oraios/serena serena start-mcp-server` (NOT `--from serena-agent`); install is `uv tool install -p 3.13 serena-agent`. Article corrected accordingly.
- Optional: cite Anthropic's official MCP announcement (REF-7 TODO) for MCP background.
- Draft cites only CONFIRMED refs (REF-1, 2, 3, 4, 5, 8, 9). No `[REF-N] TODO` placeholders (REF-6, REF-7) are relied on in the article body, so nothing blocks publish on that front. Reviewer may still verify exact tool names against the live README (REF-6) before publish.

## Reviewer pass (2026-07-22) — status: publish
- Edit applied: rewrote the H3 "Lead with reliability, not a token percentage" to "Reliability, not a token-savings number" (removed author-instruction voice; reader-facing heading).
- Verified frontmatter: title mirrors H1; description ~142 chars and contains "Serena MCP"; keywords[0] == focus_keyphrase; slug/date/author correct.
- Verified house style: keyphrase in first ~70 words; short paras; H2/H3; bold terms; tables/lists; `bash`-tagged code block; `---` between H2s; ends with Next Steps CTA then `*Sources Consulted:*`.
- Verified references: body uses only confirmed refs; no TODO markers; no orphans; Sources Consulted mirrors the confirmed set. ManoMano/REF-9 figures match references.md exactly; "~70%" framed as anecdotal (REF-5); install shown as `uv`/`uvx`, no npm.
- SME / pre-publish verification items:
  1. ✅ DONE — Tool names verified against live source: `get_symbols_overview` (was `symbol_overview`) and `safe_delete_symbol` corrected in the article (REF-6).
  2. ✅ DONE — `uvx` command verified: article now uses `uvx -p 3.13 --from git+https://github.com/oraios/serena serena start-mcp-server` and `uv tool install -p 3.13 serena-agent` (REF-6).
  3. ⬜ Fill `author:` at publish time (intentionally empty for now).
  4. ⬜ Optional: add REF-7 (Anthropic MCP announcement URL) if a primary Anthropic citation is wanted — not required.
