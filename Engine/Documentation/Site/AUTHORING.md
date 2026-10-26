# Writing Mowo docs

The documentation lives here, in the engine repo, next to the code it describes.
mowoengine.com renders it: on every build the site copies
`Engine/Documentation/Site/` and builds the reference pages from
`Engine/Documentation/Generated/`. Change the code and its page in the same commit.

**Site is for users of the installed editor build.** The engine is closed source:
a reader installs the editor, makes games with content, Visual Scripting and a C++
game module, and ships them. Write for that reader, as Unity's manual does. Engine
internals go to `Internal/`, which uses the same format and is never published:
building the engine from source, the checkout layout (`Engine/Source/...`,
`Engine/Intermediate`, ThirdParty vendoring), MBT and header-tool internals, the
module DAG and engine architecture, engine tests (`mbt test`, the gate, CI),
contributing, internal tools and design-export rows. A published page never cites
an engine `.cpp` or a repo path the installed build does not ship; cite a public
header where it helps a C++ game programmer, or nothing.

```
Engine/Documentation/
  Site/<section>/<page>.mdx      published guides, written by hand (this folder)
  Internal/<section>/<page>.mdx  unpublished pages about the engine itself
  Generated/                     written by `mbt docs`, never by hand:
    api/<module>.json            the scripting API: allowlisted public headers and
                                 every reflected type (→ /docs/api/)
    console.json                 user-facing console variables and commands (→ /docs/reference/console-*/)
    cli.json                     the mbt commands a project uses (→ /docs/reference/mbt-cli/)
    facts.json                   version, API modules, plugins, templates, RHIs, file types, counts
                                 (→ <Fact>, /docs/reference/file-types/)
```

What the API, console and CLI references publish is decided in MBT
(`ScriptingApi` and `PublishedReference`; see the MBT `CLAUDE.md`). A console
variable whose help starts with `Debug:` stays off the site.

`ROADMAP.md`, `ARCHITECTURE.md`, the audits, `Security/` and `Internal/` stay
internal and are never published.

## Pages

A page is `Site/<section>/<page>.mdx` and renders at `/docs/<section>/<page>/`.
Two levels only: a section folder with `index.mdx` (the section overview) and
flat pages beside it. An `Internal/` page keeps the same shape, must not share a
route with a published page, and may link to published pages; a published page
never links into `Internal/`. When a page moves to `Internal/`, add a
`redirects.json` entry from its old URL to the nearest published page. `reference/console-variables`, `reference/console-commands`,
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
sources:                       # shown on the page: public headers, Engine/Config, Templates, Generated
  - Engine/Source/Runtime/Game/Public/Game/AActor.h
tracks:                        # not published: the implementation the claims come from
  - Engine/Source/Runtime/Game/Private/Game/AActor.cpp
verifiedAt: 1a2b3c4d5          # required: the engine commit the page was last checked against
---
```

- A published page needs `sources`, `tracks` or both. The site lists `sources` on
  the page, so they may only name what the installed build ships: public headers
  (`Engine/Source/<Runtime|Editor>/<Module>/Public|Classes/...`), `Engine/Config/`,
  `Templates/`, `Samples/`, `Engine/Documentation/Generated/`, and plugin manifests,
  config and public headers. Everything else the page was checked against
  (`.cpp` files, MBT, shaders) goes under `tracks`, which `mbt docs --bundle`
  strips before publishing.
- An `Internal/` page lists everything under `sources` and has no `tracks`.
- Both list code, config and data files, never other docs. When one of them
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
- `<Source symbol="AActor::BeginPlay" />` cites a symbol of the published API; the site resolves the file and line from `Generated/api`. Use the qualified name as the API reference shows it (`Type`, `Type::Member`, or a free function name). A symbol outside the published API is an error on a published page.
- `<Source path="Engine/Source/Runtime/Game/Public/Game/AActor.h" />` cites a file with no symbol to name. On a published page it must be something the installed build ships (see `sources`); an `Internal/` page may cite any file, and `path=` beside `symbol=` for a private symbol. Do not add `line=`: line numbers go stale with every edit.
- `<Fact id="engine.version" />` prints a value from `Generated/facts.json` so numbers never go stale. Paths are dotted: `engine.version`, `engine.versionShort`, `counts.consoleVariables`, `counts.publicTypes`, `plugins.length`, `templates.length`, `modules.length` (the API modules), `rhiBackends` (a list prints comma-separated).
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
./mbt.cmd docs --lint     # both trees: frontmatter, sources/tracks, <Source>, <Fact>, links; lists stale pages
```

CI runs `mbt docs --lint --require-verified` on every merge request and on `dev`.
To preview the site, run `npm run dev` in `../mowoengine.com` (it syncs from this
checkout, or from `MOWO_ENGINE_ROOT`).
