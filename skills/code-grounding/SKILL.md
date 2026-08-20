---
name: code-grounding
description: >-
  Use when an issue needs to be tied to the code that implements it — locating
  the symbols and files involved, and identifying what else calls them — before
  writing a root cause hypothesis or recommended fix. Prefers semantic LSP
  queries when a language server is available and falls back to text search
  when it is not.
---

# Code Grounding

An analysis that names real symbols and real files can be checked. One that
describes the change abstractly cannot, and it reads as authoritative whether
or not it is correct. This skill is the procedure for getting from an issue
description to verified code locations, and for stating plainly when that
could not be done.

## Tools reminder

You have the `Bash` tool. Use `ls`, `cat`, `grep`, and `rg` for text search.
If an LSP tool is available in this sandbox, prefer it for symbol resolution
and reference lookup — it understands types, interfaces, and call chains that
text search cannot.

## Step 0 — Establish whether semantic lookup is available

Do not assume. Check once, at the start:

- If an LSP tool is exposed, issue one cheap query (for example `documentSymbol`
  on a known source file). If it reports the server is still starting, wait a
  few seconds and retry **once**. Language servers can be slow to warm up on
  large repositories.
- If there is no LSP tool, or the retry fails, continue with text search. This
  is a normal outcome, not an error — record which mode you used, because it
  changes how much confidence your findings carry.

## Step 1 — Extract candidates from the issue

Before touching the repository, list what to look for:

- **Explicit mentions** — symbol names, file paths, error strings, flags, and
  environment variables quoted in the issue body, comments, or logs.
- **Component hints** — the area or component the issue names, mapped to a
  directory if the repository's layout makes that obvious.
- **Behavioral phrases** — what the reporter says happens. These often match
  log lines or function names closely enough to search for.

An issue that yields no candidates at all is a signal in itself: say so rather
than inventing plausible ones.

## Step 2 — Locate the candidates

With LSP: resolve symbols directly, then open the definition.

Without LSP: search for the distinctive strings first — an exact error message
is a far better query than a guessed function name. Narrow by directory once
the area is clear. Prefer one targeted search over broad recursive listing.

Stop when you have located the code the issue concerns. Reading more of the
repository than the issue requires costs time this stage does not have.

## Step 3 — Read enough to be right

Open the definition and the code immediately around it. Read the repository's
own guidance — `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, and any relevant
`docs/` page — before describing how a change should be made, so that what you
propose matches how this project already does things.

## Step 4 — Establish the blast radius

The question is what else is affected, and it is the question text search
answers worst.

With LSP: find references to the symbol, and implementations of the interface
if one is involved. Those call sites are the change's real surface.

Without LSP: grep for the identifier and treat the result as a lower bound —
it will miss interface satisfaction, embedded types, and indirect calls. Say
that it is a lower bound.

## Step 5 — State findings so they can be checked

Every path you name must be one you actually opened, or a clearly marked new
path in a directory that already exists. Never write a path you have not
verified — a plausible-looking path that does not exist is worse than no path,
because a reader has no way to tell the difference without checking.

If the repository could not be inspected, or the candidates could not be
located, write that explicitly instead of describing the change in general
terms. "I could not locate the code that handles X" is a useful finding.
An unlocated guess dressed as analysis is not.

When the change would follow an existing convention — an established hook, a
registration point, an existing test layout — name it. Do not propose a new
framework, test runner, or directory layout where the repository already has
one, and if a deviation is genuinely required, say why.

## Budget

This stage has a short timeout. Spend it on the code the issue actually
concerns:

- Skip grounding entirely for issues that will not reach implementation —
  duplicates, questions, spam, and issues blocked on other work.
- Prefer one good search over many speculative ones.
- If you have not located the relevant code after a handful of targeted
  attempts, stop and report what you looked for and where. A partial finding
  with an honest boundary is more useful than a broad guess.
