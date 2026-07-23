---
topic: github-copilot-dotnet-sdlc
title: "GitHub Copilot for .NET: Applying AI Across the SDLC"
status: publish
updated: 2026-07-23
---

## Goal
Show .NET developers how to apply GitHub Copilot as an AI assistant across the full software development lifecycle (SDLC) — planning, coding, testing, code review, documentation, and maintenance — on a real .NET project, with concrete, phase-by-phase examples. Target audience: Intermediate .NET developers adopting Copilot on their team.

## Progress
- [x] Research — references.md created (51 confirmed, 0 TODO placeholders; +8 refs [REF-44..REF-51] added for the customization/extensibility H2)
- [x] Draft — article.md v1 written
- [x] Review — Reviewer Notes applied, references resolved
- [x] Revise — added new H2 "Customizing & extending Copilot for your .NET repo" (path-specific custom instructions, prompt files, custom agents, MCP servers); cited [REF-44]–[REF-51]
- [x] Re-review — new customization/extensibility H2 edited for style/grammar; technical framing (custom-agents rename, MCP caveats, config paths) verified; all 46 cited [REF-N] map to references.md and mirror the Sources Consulted block; zero TODO placeholders. No blockers.
- [x] Full end-to-end review — read whole article top to bottom against CLAUDE.md house style: frontmatter (title mirrors H1, description ~137 chars w/ focus keyphrase, keywords[0]==keyphrase, date/slug OK), hook + keyphrase in first 100 words, all H2s flow and are `---`-separated, ends with Next Steps CTA then Sources Consulted. Terminology consistent (IDE agent mode vs GitHub.com cloud agent; "custom agents" not chat modes; VS vs VS Code). Earlier cautions intact (@Test paid-only VS 2026 18.3+; EF Core illustrative; code review comment-only). Reference set (46) matches Sources Consulted exactly. Applied 3 minor copy-edits total. No blockers.
- [ ] Publish

## Next step
Run the **publish** step: set `author` in the frontmatter and publish `article.md`. No open blockers.

## Notes / open questions
- Angle confirmed: **phase-by-phase SDLC walkthrough** (plan → code → test → review → docs → maintain), each phase mapped to the right Copilot feature with .NET examples.
- Audience confirmed: **Intermediate** .NET developers.
- Recommended outline is in references.md (Synthesis → "Recommended outline"); now includes the customization H2.
- All three prior research gaps resolved with verified official URLs:
  - [REF-37] Matching-public-code / IP filter — "code referencing" (+ policy Allow/Block [REF-40], matched-code log [REF-41]). Confirmed.
  - [REF-38] EF Core + Copilot — no dedicated official walkthrough; closest verified source is the MSSQL-extension "Code Generation with GitHub Copilot for SQL" quickstart. Present EF Core scaffolding as **illustrative** ([REF-38]; [REF-42] for classic EF tooling).
  - [REF-39] VS Code testing with Copilot — "Test with GitHub Copilot" (+ smart actions [REF-43]). Confirmed.
- Customization/extensibility research (new H2) — confirmed sources + caveats the writer MUST honor:
  - Prompt files ([REF-44]): `.prompt.md` in `.github/prompts`, invoke with `/name`, support variables (`${selection}`, `${input:...}`) and file references. VS Code docs.
  - **Custom agents ([REF-45]): TERMINOLOGY CHANGED — "custom chat modes" renamed to "custom agents"; file format `.chatmode.md` → `.agent.md` (`.github/chatmodes/` → `.github/agents/`).** Legacy `.chatmode.md` still works. Use current terms; mention legacy once. Visual Studio custom-agents page: [REF-51].
  - Custom instructions ([REF-46] VS Code): path-specific `*.instructions.md` with `applyTo` globs in `.github/instructions/`; complements existing [REF-11]/[REF-23]. Added because it's a distinct, citeable page for path-specific instructions.
  - MCP servers ([REF-47][REF-48][REF-49][REF-50]): extend **agent mode** with external tools/context. **Availability caveat: MCP is GA and cross-IDE, but in Visual Studio it requires VS 2022 17.14+ or VS 2026; agent mode only; tools disabled by default (per-tool approval; server-trust dialog in VS 2026 18.7); Business/Enterprise must enable the "MCP servers in Copilot" policy; admins can set an allowlist.** Config: `.mcp.json`/`mcp.json` (VS) or `.vscode/mcp.json` (VS Code). .NET angle: Azure MCP, MSSQL/SQL Server MCP (live DB schema), Azure DevOps MCP, GitHub MCP.
  - VS Code vs Visual Studio: prompt files and custom agents/chat modes are documented as VS Code features (portable via the repo); custom *instructions* are the most broadly supported customization. Frame availability accordingly.
- Terminology caution (agents): distinguish IDE **agent mode** (local, VS/VS Code) from GitHub.com **cloud/coding agent** (issue-driven, GitHub Actions-powered). Older docs say "coding agent," newer say "cloud agent."
- Scope caution: "GitHub Copilot testing for .NET" is **Visual Studio 2026 (18.3+) and paid-plan only** — do not present as universally available.
