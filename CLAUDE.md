# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not a software project** — it's an **Obsidian vault** (personal knowledge base) belonging to a backend/Java/Spring developer (이태성 / taesung). Content is almost entirely Markdown notes, written primarily in **Korean**, covering study notes, project logs, troubleshooting write-ups, and tool references. There is no application code, no package manifest, and **no build, lint, or test tooling** — don't go looking for one, and don't introduce one unless the user explicitly asks.

This is the **public** half of the user's vault: a private vault (with sensitive info) exists separately, and content is filtered into this repo before publishing.

## Working with file paths in this repo

Filenames and directory names routinely contain **emoji, spaces, Korean characters, and symbols like `&`** (e.g. `project/🔫 이슈 분석 & 트러블슈팅.md`, `짧은 키워드/`, `lectur/🏫 강의 & 자격증.md`). The Bash tool's POSIX path handling mangles these (multi-byte/emoji paths get corrupted, `&` breaks command parsing). **Prefer the `Glob`/`Grep`/`Read`/`Edit` tools over Bash `find`/`grep`/`cat` for any path in this vault** — they handle these filenames correctly where shell commands don't.

Note: the `lectur/` directory is spelled that way (missing the final "e") — this is the real directory name, not a typo to fix.

## Repository structure

Content is organized by category, each with an index ("MOC", map-of-content) note that links out to the actual notes via wikilinks. The root index is [`🏠 taesung's Blog.md`](🏠%20taesung's%20Blog.md), which links to each category's MOC.

- **`project/`** — Development project logs. Sub-folders per project (some literally named `New Project` — that's the AnoniChat project folder, never renamed): `New Project` (AnoniChat: infra, CI/CD, ELK, NGINX, monitoring — also holds non-prose assets, see "Non-prose special files" below), `hello Batch` (Spring Batch refactor), `hello marketing` (AOP-based analytics module), `MeloMeter Project`, `note CI_CD` (Obsidian→Netlify publishing writeup). Indexed by [`project/💻 프로젝트.md`](project/💻%20프로젝트.md); the issue-tracking subset is indexed separately by [`project/🔫 이슈 분석 & 트러블슈팅.md`](project/🔫%20이슈%20분석%20&%20트러블슈팅.md).
- **`Issue_TroubleShooting/`** — Standalone issue/root-cause write-ups (flat folder, no sub-index of its own; linked from the project-level issue MOC above).
- **`study/`** — Study notes, in sub-folders: `CS` (computer science / Spring / infra concepts — itself a mix of narrative essays, content-free stub sub-MOCs like `🍃 Spring.md`/`📔 DataBase.md`, and a couple of 회고/retrospective seminar recaps), `Coding Test`, `Dev Seminar` (despite the MOC calling these "회고," they're technical postmortems, not personal-reflection writeups), `정보처리기사` (Korean IT engineer certification prep, split into `필기`/written and `실기`/practical — **not linked from `study/📕 공부.md`**; only reachable via [`lectur/🏫 강의 & 자격증.md`](lectur/🏫%20강의%20&%20자격증.md)). Indexed by [`study/📕 공부.md`](study/📕%20공부.md).
- **`Tools/`** — Reference notes on tools (AWS, Docker, Redis, Jenkins, k8s, Obsidian itself). Indexed by [`Tools/🪓 도구.md`](Tools/🪓%20도구.md).
- **`lectur/`** — Notes following a paid course (Spring Core Basic by 김영한/inflearn), numbered sequentially (`2.4.1 전체 흐름 정리.md`) rather than emoji-prefixed. Indexed by [`lectur/🏫 강의 & 자격증.md`](lectur/🏫%20강의%20&%20자격증.md), but that MOC only links top-level chapters — chapters 4–8 are forward-referenced links to notes that don't exist on disk yet, while the sub-numbered files that DO exist (2.1, 2.2, 2.3...) aren't linked from the MOC at all and are only reachable by browsing the folder.
- **`짧은 키워드/`** — Flat glossary of short technical-term write-ups (one concept per file), cross-linked from wherever the term comes up rather than from a single index. Noticeably terser than every other category — see "Note conventions" below.
- **`사진 및 문서/`** — Attachments (pasted screenshots, images, misc documents). This is Obsidian's configured `attachmentFolderPath` (see `.obsidian/app.json`) — new pasted images land here automatically.
- **`.obsidian/`** — Obsidian app config and community plugins. Treat as vault configuration, not content; avoid editing unless the user specifically asks about Obsidian settings/plugins.
- **`custom-head-content.html`** — Injected into the exported static site (dark theme forcing, floating home/back/scroll-to-top buttons). Only relevant when working on the publishing pipeline, not note content.

## Note conventions

When creating or editing notes, match the existing style. Most categories (`project/`, `Issue_TroubleShooting/`, `study/CS`, `study/Dev Seminar`, `Tools/`, `lectur/`) share a baseline, but treat it as an organic habit, not a strict template — see the two genre exceptions below before assuming it applies everywhere.

**Baseline pattern:**
- **Filename = H1 title**, usually prefixed with an emoji (`🍧 JPA의 영속성 상태와 데이터 Log 이슈.md` → `# 🍧 JPA의 영속성 상태와 데이터 Log 이슈`).
- **Tags** as a hashtag line right under the title (`#SQL #JPA #이슈 #log #로그`), not YAML frontmatter.
- **`---` horizontal rules** separate major sections.
- **Wikilinks** (`[[Note Name]]`) for cross-references — link by note title, not by relative path; Obsidian resolves it. A recurring variant is an arrow-style relational link to a parent/related note, e.g. `보러가기 ◀ [[...]]` at the top or `▶ [[...]]` at the bottom, used in `Issue_TroubleShooting/` and `짧은 키워드/`.
- **Image/attachment embeds** use `![[Pasted image 20250520142253.png]]`, optionally with a width: `![[image.png|700]]`.
- **Callouts** (`>[!info]`, `>[!note]`, `>[!danger]`, `>[!caution]`, `>[!question]`, `>[!tip]`, `>[!warning]`, `>[!bug]`) for asides, warnings, and Q&A within a note.
- Inline HTML shows up for extra styling the user relies on: `<u>`, `<font color="#hex">`, `<br/>` — preserve this pattern rather than converting to pure Markdown.
- Longer troubleshooting/project notes tend toward an overview→analysis→conclusion arc, but the actual headings vary by author mood: `개요→분석→결론`, `작업 목적→작업 계획`, `문제→원인분석→해결방법→Reference`, `현상→원인분석→수정내역→재발방지대책`, or several independent numbered "이슈" mini-arcs in one note. Don't force a note into a rigid 개요/분석/결론 template if the content doesn't naturally split that way.
- Fenced code blocks (` ```java `, ` ```sql `, etc.) are used for snippets and config dumps. The language tag is often chosen loosely/incorrectly (YAML or conf content tagged ```python or ```c) — this is a consistent author habit, not a mistake to silently "fix."
- Paired main+appendix notes (e.g. `...리펙토링` / `...리펙토링 부록`) exist but link inconsistently — sometimes one-way (main → appendix only), sometimes bidirectional with the appendix opening on a "먼저 보기" (read this first) link back. Match whichever direction the specific pair already uses.
- Category MOC files follow the same shape: a `>[!note]`/`>[!info]` callout describing the category, then `##`/`######` grouped bullet lists of `[[wikilinks]]`. When adding a new note under an existing category, also add a link to it from that category's MOC (and from the root blog page's "새로 추가된 글" section, if it's a recent/notable addition).

**Genre exceptions — don't apply the baseline here:**
- **`짧은 키워드/`** entries are much terser: an H1 usually phrased as a question (`~란?`, `~일까?`) followed by a handful of short bullets or a small table/code block, and *no* tags line and *no* wikilinks in most entries — the opposite of the baseline's "always tag, always link" habit.
- **`study/정보처리기사/` (필기 & 실기)** notes are a flashcard/Q&A drill format, not prose: each card is `# <question>` followed by an answer using `==highlighted==` Korean mnemonic acronyms, and multiple-choice options rendered as task-list checkboxes (`- [x]` marks the correct answer). 실기 notes further split into keyword drills, SQL problems (table + query + 해설), and code-trace problems (code block → step trace → `답:`).

**Non-prose special files** (mainly in `project/New Project/`, but the pattern can recur anywhere Excalidraw or raw configs are attached):
- **`*.excalidraw.md`** files are Excalidraw-plugin diagrams, not prose: frontmatter `excalidraw-plugin: parsed`, a `## Text Elements` section (searchable diagram labels), then a `## Drawing` fenced ` ```compressed-json ` blob encoding the actual diagram geometry. Don't hand-edit the compressed-json blob; open/edit these in Obsidian's Excalidraw view instead.
- **Bare config-dump notes** (e.g. `anoniChat-docker-compose.yml.md`, `NGINX-docker-compose.yml.md`, `anoniChat-logstash.conf.md`) have no H1, no tags, just one fenced code block. Some are explicitly linked from a parent post as a "comment-stripped reference copy"; others are orphaned drafts that have already drifted from the config actually described in the prose post (e.g. `NGINX-docker-compose.yml.md` doesn't match `☘ ANONI Chat - NGINX...md`, and `anoniChat-logstash.conf.md` still filters on a leftover `"auction_log"` type from a different project). **Treat these as historical/reference snapshots, not a guaranteed-current source of truth** — check the linking prose note for the author's own caveats before relying on one.

## Known vault quirks

- **MOC coverage lags reality.** Some MOC links point to chapters/notes that don't exist as files yet (forward references, e.g. `lectur` chapters 4–8); some existing notes aren't linked from any MOC at all (e.g. `lectur`'s own sub-numbered files, `study/정보처리기사`); and some notes are cross-listed under a MOC category that doesn't match their physical folder (e.g. the AWSKRUG/Tech Friends Mixer 회고 notes physically live under `study/CS/` but are listed under "Company Seminar" in `study/📕 공부.md`). **The MOC is the source of truth for a note's logical category — its folder location can lag behind or simply not match.** Don't be alarmed by a dead-looking MOC wikilink with no file behind it, and don't "fix" categorization by moving files to match folders.
- **Stub, placeholder, and near-duplicate files are normal, pre-existing clutter**, not something to silently clean up unless asked: some notes are intentional placeholders (`👽 Algorithm.md` is just "추가예정.."), some are completely empty (`짧은 키워드/RESTful API.md`), and some are accidental near-duplicates (`짧은 키워드/GitOps.md` vs `GitOps란.md` — near-identical content, and the former is missing its H1 entirely).

## Git history

Nearly all commits are auto-generated by the Obsidian Git plugin on a timer/save ("`vault backup: <timestamp>`") — this is not a hand-curated commit history and shouldn't be read as meaningful development milestones. Manual commits are rare exceptions (e.g. the README addition).

## Publishing pipeline (context only)

This repo holds the Markdown source only — there is nothing to build here. The user's actual publish flow (documented in [`project/note CI_CD/👻 Obsidian 정적호스팅 CICD 과정.md`](project/note%20CI_CD/👻%20Obsidian%20정적호스팅%20CICD%20과정.md)) is: edit in Obsidian → export static HTML via the `webpage-html-export` community plugin → push the exported HTML to a *separate* repo → Netlify auto-deploys that repo. `custom-head-content.html` is injected into that export.
