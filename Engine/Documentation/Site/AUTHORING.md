# Writing Mowo docs

The user and developer documentation lives here, in the engine repo, next to the
code it describes. mowoengine.com renders it: on every build the site copies
`Engine/Documentation/Site/` and builds the reference pages from
`Engine/Documentation/Generated/`. Change the code and its page in the same commit.

```
Engine/Documentation/
  Site/<section>/<page>.mdx   guides, written by hand (this folder)
  Generated/                  written by `mbt docs`, never by hand:
    api/<module>.json         every public header (→ /docs/api/)
    console.json              console variables and commands (→ /docs/reference/console-*/)
    cli.json                  every mbt command and option (→ /docs/reference/mbt-cli/)
    facts.json                version, modules, plugins, templates, RHIs, file types, counts
                              (→ <Fact>, /docs/reference/file-types/)
```

`ROADMAP.md`, `ARCHITECTURE.md`, the audits and `Security/` stay internal and are
never published.

## Pages

A page is `Site/<section>/<page>.mdx` and renders at `/docs/<section>/<page>/`.
Two levels only: a section folder with `index.mdx` (the section overview) and
flat pages beside it. `reference/console-variables`, `reference/console-commands`,
`reference/mbt-cli` and `reference/file-types` are generated; do not create files
with those names.

## Names

Use the public names in prose, titles and headings: **Global Illumination** and
**Visual Scripting**; no internal codenames. Code identifiers stay literal and in
backticks where the reader has to type or search them: the `MowoEngine.VisualScripting`
module, `.vscript` files, class names, console variables such as `r.GI.BakedSdf`.

## Frontmatter

```yaml
---
title: Install Mowo            # page H1 and sidebar label
sidebarTitle: Install          # optional shorter sidebar label
description: One sentence, under 160 characters. Used for SEO and the page lead.
order: 2                       # sidebar order inside the section (index.mdx orders the section)
icon: Rocket                   # index.mdx only: Rocket, LayoutDashboard, Sun, Gamepad2, Workflow, PanelsTopLeft, Network, FolderInput, PackageCheck, Hammer, FlaskConical, Braces, Boxes, Layers, FileText
status: in-development         # optional: shows an "In development" chip under the title
sources:                       # required: the code this page's claims come from (repo-relative, must exist)
  - Engine/Source/Runtime/EngineLoop/Private/Core/Engine.cpp
verifiedAt: 1a2b3c4d5          # required: the engine commit the page was last checked against
---
```

- `sources` lists code, config and data files, never other docs. When one of them
  changes after `verifiedAt`, `mbt docs --lint` lists the page as stale: re-read
  the page against the change, fix it, and move `verifiedAt` to the new commit.
- `verifiedAt` is the commit you checked the page against (`git rev-parse --short HEAD`).

Do not add an H1 in the body; the page renders `title`. Start with a short
paragraph, then `##` sections (`###` for subsections). The table of contents is
built from `##`/`###`.

## Components (no imports needed)

- `<Callout type="tip|note|warning|danger|engine">…</Callout>`. Use `engine` for "Engine note" (how the engine behaves internally).
- `<Steps>` with `<Step title="…">markdown</Step>` children.
- `<CardGrid>` with `<Card title="…" href="/docs/…/" icon="Sun">one line</Card>`.
- `<Flow steps={["A", "B", "C"]} highlight={1} caption="…" branch={{ from: "D3D11", label: "…" }} />` for pipeline diagrams.
- `<Tabs>` with `<Tab label="Windows">…</Tab>` children, for platform-specific instructions.
- `<CodeTabs>` wrapping several fenced blocks, each with `tab="C++"` in the fence meta.
- `<Source symbol="FEngine::Initialize" />` cites a public API symbol; the site resolves the file and line from `Generated/api`. Use the qualified name as the API reference shows it (`Type`, `Type::Member`, or a free function name).
- `<Source path="Engine/Source/…/Private/File.cpp" />` cites a file that has no public symbol to name. Do not add `line=`: line numbers go stale with every edit.
- `<Fact id="engine.version" />` prints a value from `Generated/facts.json` so numbers never go stale. Paths are dotted: `engine.version`, `engine.versionShort`, `counts.consoleVariables`, `counts.publicTypes`, `plugins.length`, `templates.length`, `modules.length`, `rhiBackends` (a list prints comma-separated).
- `<Kbd>F5</Kbd>`, `<Status />` (renders "In development").
- `<Screenshot id="editor-viewport-toolbar" alt="…" caption="…" />`. Every screenshot is a placeholder the team captures later, so put one wherever a reader would benefit from seeing the UI. Make `alt` a precise capture instruction: what window, what state, what is selected. `id` must be unique across the docs (kebab-case, prefixed with the section). Keep an existing id when the page still shows the same UI: ids key the capture queue.

Fenced code: ` ```cpp title="MyActor.cpp" `. Languages: `cpp`, `hlsl`, `csharp`, `json`, `mdf`, `ini`, `bash`, `powershell`, `text`.
Escape `{`, `}` and `<` in prose (MDX): write `\{`, `\}`, `&lt;`, or put them in backticks.

## Links

Link with the trailing slash: `/docs/rendering/shadows/`. Generated targets:
`/docs/api/<module>/`, `/docs/reference/console-variables/`,
`/docs/reference/console-commands/`, `/docs/reference/mbt-cli/`, `/docs/reference/file-types/`.

## Rules

1. Every claim comes from the code. Never invent features, numbers, flags, class names or paths. When the code and an older text disagree, the code wins.
2. Things in progress get `status: in-development`, `<Status />` or a `warning` callout. Say what does not work yet.
3. C++ samples must compile against the public headers. From mowoengine.com, run `node scripts/check-code-samples.mjs [section]` (clang, `-fsyntax-only`).
   - A full translation unit compiles as written.
   - Add `wrap` to the fence meta for statement snippets: `#include` lines are hoisted and the rest is wrapped in a function.
   - Add `nocompile` only for deliberate fragments (declarations that need a `.cpp` body elsewhere, pseudo-code). Prefer compiling samples.
4. Voice: plain, direct, second person, short sentences. No marketing adjectives. Sentence-case headings.

## Checks

```bash
./mbt.cmd docs            # regenerate Generated/ after a code change (CI does it on dev)
./mbt.cmd docs --lint     # frontmatter, sources, <Source>, <Fact>, links; lists stale pages
```

CI runs `mbt docs --lint --require-verified` on every merge request and on `dev`.
To preview the site, run `npm run dev` in `../mowoengine.com` (it syncs from this
checkout, or from `MOWO_ENGINE_ROOT`).
