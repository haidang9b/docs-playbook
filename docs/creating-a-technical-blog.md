# How to Create a Technical Blog Post

A step-by-step guide for writers using this toolkit. It walks the full pipeline — **scaffold → research → draft → review → publish** — using the three agents and a per-topic workspace. For the terse SOP version, see [`../.claude/workflows/create-blog-workflow.md`](../.claude/workflows/create-blog-workflow.md); for project conventions, see [`../CLAUDE.md`](../CLAUDE.md).

> **The golden rule:** you stay the author. The agents accelerate research, drafting, and editing — you own accuracy, voice, and the final call at every **Human Checkpoint ✅**.

---

## Before you start

- **Claude Code** open in this repository. The agents in `.claude/agents/` and skills in `.claude/skills/` are auto-discovered.
- **A topic in mind**, boiled down to one sentence (the "BLUF" — Bottom Line Up Front) and one audience (Beginner / Intermediate / Advanced).

Throughout this guide the running example is a post on Redis caching. Its slug is `caching-strategies-redis`.

---

## Step 1 — Scaffold the topic

Every article gets its own folder so the draft, its sources, its assets, and its progress live together.

1. Copy the template to a new kebab-case slug:
   ```bash
   cp -r src/_template src/caching-strategies-redis
   ```
2. Open `src/caching-strategies-redis/STATE.md` and fill in the `topic`, `title`, `updated` date, and the **Goal** (one-line BLUF + audience). Leave `status: research`.

Your workspace now looks like:
```
src/caching-strategies-redis/
├── STATE.md         # progress tracker — your shared "memory" with the agents
├── article.md       # the draft (starts as a stub)
├── references.md    # sources (starts empty)
└── assets/images/   # diagrams / screenshots
```

**Human Checkpoint ✅** — Does the Goal capture the single most important takeaway and the right audience? Fix it now; everything downstream builds on it.

---

## Step 2 — Research

Hand the topic to the research agent. It searches the internet, keeps only trustworthy sources (official docs first), and writes a synthesis plus a numbered source list into `references.md`.

```bash
@blog-researcher Research the topic in src/caching-strategies-redis/. Focus on cache-aside vs write-through strategies in Redis 7 and record sources in references.md.
```

**How sources are tracked** — every source gets a stable id:
- **Confirmed:** `[REF-1] Redis Caching Docs — https://redis.io/docs/latest/develop/...`
- **Placeholder (gap):** `[REF-2] TODO: verify — need a benchmark for write-through latency`

The agent **never invents a URL**. Gaps stay as `TODO` placeholders for you or a later research pass to fill. When it finishes, it sets `STATE.md` to `status: drafting`.

**Human Checkpoint ✅** — Skim `references.md`. Are the confirmed links real and relevant? Note which `TODO` placeholders you still need. Sketch or approve an outline from the synthesis.

---

## Step 3 — Draft

The writing agent turns the research and outline into a scannable article at `article.md`, citing claims with `[REF-N]` markers that match `references.md`.

```bash
@blog-writer Using references.md in src/caching-strategies-redis/, write article.md for intermediate backend developers. Include a code example for cache-aside in Python.
```

What you get: a pain-point **hook**, short paragraphs, `H2`/`H3` structure with `---` between sections, bolded key terms, language-tagged code blocks, a **Next Steps** call to action, and correct SEO frontmatter. The agent sets `STATE.md` to `status: review`.

**Human Checkpoint ✅** — Read it as a reader would. Is the technical narrative right? Is the code correct? Adjust the prompt and re-run for sections that miss.

---

## Step 4 — Review

The review agent does a multi-layered edit — logical flow, active voice, grammar, code/link sanity — and a **reference-completeness check** that flags any `[REF-N]` still pointing at a `TODO` placeholder or any marker with no matching source.

```bash
@blog-reviewer Review src/caching-strategies-redis/article.md for grammar, clarity, style-guide adherence, and reference completeness. Provide Reviewer Notes.
```

You get a revised draft plus **Reviewer Notes** explaining *why* each change was made. The agent sets `STATE.md` to `status: publish` (clean) or `status: revise` (with blockers listed).

> **Do not skip the SME pass.** Send code and technical claims to a subject-matter expert. AI accelerates the work; a human engineer confirms accuracy.

**Human Checkpoint ✅** — Accept/reject edits, resolve every remaining `TODO` reference, and make sure the voice still sounds human.

---

## Step 5 — Publish

Run the pre-publish checklist before shipping:

- [ ] `title` and `description` frontmatter filled in.
- [ ] All internal and external links work.
- [ ] All code snippets tested and running.
- [ ] A clear Call to Action (Next Steps) at the bottom.
- [ ] **Zero unresolved `[REF-N] TODO` placeholders** — every source confirmed and mirrored into the `*Sources Consulted:*` block.
- [ ] `STATE.md` set to `status: published`.

Then commit the `src/caching-strategies-redis/` folder (docs-as-code) or paste the rendered Markdown into your CMS.

---

## Tips

- **Keep prompts constrained.** Always name the audience and the specific angle — "intermediate backend developers", "cache-aside vs write-through" — so tone and depth land right.
- **`STATE.md` is your resume point.** Coming back to a half-finished topic? Open its `STATE.md` — the **Next step** line tells you (and the next agent) exactly where to pick up.
- **One topic, one folder.** Never scatter drafts or sources outside `src/<topic-slug>/`.
- **Add visuals** with the `tech-image-generation` skill; save prompts under the topic's `assets/images/`.
- **Fix recurring errors at the source.** If an agent keeps making the same mistake, update the relevant `SKILL.md` so the fix sticks for every future post.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Agent won't proceed — "no STATE.md" | You skipped Step 1. Copy `src/_template/` first. |
| A citation links nowhere | It's a `TODO` placeholder — supply the real source in `references.md`, or re-run `@blog-researcher`. |
| Draft tone is off | Re-run `@blog-writer` with a sharper audience in the prompt. |
| Reviewer flags orphaned `[REF-N]` | A marker in the prose has no entry in `references.md` — add the source or remove the marker. |
