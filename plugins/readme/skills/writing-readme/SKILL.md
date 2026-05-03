---
name: writing-readme
description: Use when writing or generating a README from scratch for a software project.
---

# Writing a README

This skill writes a README **from scratch** by reading the codebase first, then assembling the file according to opinionated rules.

## Step 1: Read the codebase first

You cannot write a good README without grounding in real code. Gather:

- **Name, tagline, license** from the package manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`). Reuse the manifest's `description` field as the tagline if it's real (not boilerplate). When you have to write a tagline from scratch, follow the pattern *"X is a Y for Z that does W"* — name + category + audience + value.
- **Project type**: library, CLI, application/service, or internal tool. Signals are in the manifest, entry point, dependencies, and deployment config. If signals are mixed, pick the type matching the *primary* user experience.
- **Install command** from real evidence — the registry the project actually publishes to, or `git clone` + bootstrap if it doesn't.
- **Usage example** from `examples/`, tests, or the entry point. Copy verbatim. Never invent code.

If something cannot be found, never guess.

## Step 2: Identify the audience

- **Public-facing** (open source, published packages, customer-facing): readers are *prospective users*. Keep the README tight; push detail to documentation.
- **Internal / onboarding-primary** (private repos, no separate docs site): readers are *future maintainers and oncall*. Going longer is fine and often required — operational detail (IPC contracts, schemas, runbooks) belongs in the README when it's the only documentation.
- **Mixed**: pick the larger audience and serve them. Trying to satisfy both produces a README that's too long for users and too marketing-y for contributors.

## Step 3: Section order

1. **Title** (H1, matches the manifest name)
2. **Tagline** — one line, < 120 chars. Always render *something* visible; the tagline must not be left as an HTML comment placeholder. If the manifest has no real description, write a factual structural tagline grounded in what the code actually is (its dependencies, entry point, project type) rather than guessing what the project *does*. "A Python scaffold built on the Claude Agent SDK" is grounded; "An AI-powered resume tailor" is invented.
3. **Picture** — screenshot, GIF, or runnable code block. If the codebase has none, an HTML-comment placeholder is acceptable here (unlike the tagline, the picture can be invisible without sinking the README).
4. **## About** — two short paragraphs maximum. What it does, why it exists, what makes it different. Skip if the tagline + picture already do this work.
5. **## Setup** — one-line requirements only if real prerequisites exist, then copy-pasteable install commands. For substantial configuration (multiple env vars or a real config file), use a table. For a single env var, fold it into the install block. If install requires a user-specific path (e.g. `git clone … ~/some-dir`), use a generic placeholder, not the author's machine path, and follow the block with a one-line *"Replace `<placeholder>` with …"* sentence.
6. **## How to use** — smallest runnable example, copied from real code. Show expected output when meaningful — readers shouldn't have to install just to see what the tool does. CLIs: add 3–5 idiomatic invocations. Libraries with a small API surface (< ~30 methods): inline reference may live here.
7. **## FAQ** — *only* if real recurring questions exist. Don't invent FAQs.
8. **## Contributing** — *only* if `CONTRIBUTING.md` (or a contribution template) exists. Link to it in one paragraph.
9. **## License** — *only* if BOTH of these are true: (a) there's a real license signal (LICENSE file exists, manifest has a non-empty `license` field, project is on a public registry, or the codebase clearly signals open-source intent) AND (b) the owner is identifiable (manifest `author` field, organization in the repo path, or a name the user has explicitly given you).

**Never include a section whose only content is a `<!-- TODO -->` placeholder.** An empty section header is worse than no section. If you can't write real content for a section, omit the section and add the missing information to the placeholder list in your final response.

## The above-the-fold rule

By the time a reader has scrolled ~30 lines (one laptop screen), they should see: name, tagline, picture, About, and the start of Setup. That's the entire pitch. This is the single most important rule in this skill — anything that fights for that real estate must be removed.

## Tone

Default to developer-to-developer prose: terse, neutral, factual. The reader is a peer, not a customer.

- No marketing superlatives — avoid "powerful", "elegant", "blazing fast", "delightful".
- No filler adverbs — avoid "simply", "just", "easily", "effortlessly".
- No exclamation marks. Ever.
- Imperative for instructions ("Run `npm install`"), not "you can run".
- Cite specifics. "Stores data in a single SQLite file" beats "robust storage layer."
- Third person about the project, not first person from the author. "Skills get added over time" beats "I add skills over time." Even on a personal repo, the README is the project's voice, not the maintainer's.

For internal tools, tone shifts further toward operational — the reader is a future maintainer, not a prospective user.

## GitHub rendering

The README will be rendered on GitHub. Use **relative links** for in-repo files (`LICENSE`, `CONTRIBUTING.md`, `docs/...`) — they work in clones and across branches; absolute GitHub URLs don't.

Do **not** use GitHub callouts (`> [!NOTE]`, `> [!WARNING]`, etc.) unless the user explicitly asks for them. They are not a substitute for a tagline or About paragraph and they should never appear at the top of the README as a "status line" — that's the most common misuse.

## Project-type variations

The structure is the same; emphasis differs.

- **Library** — picture is usually a runnable code block (the output is what matters). Show one canonical usage example. Inline API reference for small surfaces; link to docs for large ones.
- **CLI** — picture is a terminal recording or GIF (placeholder if missing). List install commands for every package manager the project actually ships to. "How to use" includes 3–5 idiomatic invocations.
- **Application / service** — picture is a hero screenshot. Setup may have two install paths (cloud signup + self-host); make the easier path more visible. Deploy buttons if the project supports them.
- **Internal tool** — operational tone. Add a "Maintainers" section after About (team, oncall, contact). Link runbooks and dashboards. Going long is fine. License: `© <Company>. Internal use only.`
- **Plugin / extension** — a package that loads into a host tool (Claude Code plugin, VS Code extension, framework plugin). Picture and "How to use" usually aren't needed; the host activates the plugin, and the README's job is to get the reader from clone to working install. Replace "How to use" with an inventory section — a table of what the plugin provides (commands, skills, components). Setup typically has two steps: clone/download, then register/activate inside the host.
- **Doesn't fit** (monorepo, RFC, demo, theme) — keep the above-the-fold rule and tone. Add or drop sections as the project demands.

## Verify before finishing

Read the README end-to-end as a stranger would. In particular:

- **Path and name references must agree** across code blocks and prose. If the install snippet uses `~/foo` and a "Replace X" sentence still references `~/bar`, the steps don't work end-to-end. Check every command block, every "Replace X" sentence, and any inline references against each other.
- **Code fences open and close on their own lines.** Closing a fence on the same line as the last command will silently break rendering.
- **Every command runs as written.** A reader pasting the steps verbatim should land in a working state without inferring substitutions the README never asked for.

## Anti-patterns

- **Burying the lede** — the first sentence must say what the project does.
- **Inventing usage examples** — broken or made-up code destroys trust faster than anything else.
- **Inline API dump for large surfaces** — link to `docs/` instead.
- **"Contributions welcome!" with no instructions** — link to a real CONTRIBUTING.md or omit the section.
- **Mismatched names** across README title, manifest `name`, and GitHub repo description — reconcile before writing.
- **License section with wrong info** — don't write "MIT" if there's no LICENSE file. Don't fabricate an owner ("© <Project> authors") to fill the slot. If there's no real license + owner signal, omit the section entirely.
- **Empty sections** — a header followed only by a TODO placeholder is worse than no header. Omit the section.
- **Status callouts at the top of the README** — no `> [!NOTE]` block to announce "early stage", "scaffold", or any other status. If the project's status matters, work it into the tagline or About in plain prose. Callouts at the top read as caution tape and the user does not want them.

## What not to include

The README is for whoever is reading it *now*, not a documentation dump.

- **Tech-stack roll calls.** Mention a stack item only if it changes how the user installs, runs, or extends the project.
- **Build tool internals** — esbuild configs, eslint settings. They live in the config files or CONTRIBUTING.md.
- **Internal architecture** for *public-facing* READMEs (IPC contracts, DB schemas). For internal READMEs these may belong — check the audience.
- **Author bios or origin stories.**
- **Roadmap / "coming soon"** unless it's load-bearing.
- **Aspirational install commands** — only document what works today.
- **Invented FAQs** — only real recurring questions belong.
