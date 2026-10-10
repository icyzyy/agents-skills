---
name: commit-code
description: Use when creating a git commit — whenever you or a user are about to run git commit, write a commit message, or review commit messages for a change
---

# commit-code

## Overview

Every commit message MUST follow the Conventional Commits v1.0.0 specification:
https://www.conventionalcommits.org/en/v1.0.0/#specification

A commit without a type prefix is an invalid commit.

## Required format

Every commit message MUST match this template:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]

On-behalf-of: <name> <<email>>
```

The first line is REQUIRED and always has the shape `type:` or `type(scope):` —
a noun type, optional scope in parentheses, optional `!`, then `: ` and a short
description.

The `On-behalf-of` footer is REQUIRED on every commit. Resolve the values from
git config at commit time:

```bash
git commit -m "$(printf 'feat(auth): add Google OAuth login\n\nOn-behalf-of: %s <%s>' \
  "$(git config user.name)" "$(git config user.email)")"
```

If `git config user.name` or `user.email` is unset, ask the user for them —
never guess or omit the footer.

## Types

| Type | Use for |
|------|---------|
| `feat` | a new feature (SemVer MINOR) |
| `fix` | a bug fix (SemVer PATCH) |
| `docs` | documentation only |
| `style` | formatting, no logic change |
| `refactor` | code change that neither fixes nor adds |
| `perf` | performance improvement |
| `test` | adding or correcting tests |
| `build` | build system or dependencies |
| `ci` | CI configuration |
| `chore` | maintenance, tooling |
| `revert` | reverting a previous commit |

`feat` and `fix` are mandatory for features and bug fixes. Other types are
optional but must still be valid spec types.

## Before writing the message: analyze the change

Composing a message without reading the change is forbidden. REQUIRED steps,
in order:

1. Run `git diff` and read the actual changes (not a description of them, not
   what you remember doing — the diff itself).
2. Analyze what the diff does, THEN pick the commit type from that analysis:
   adds behavior → `feat`, fixes a bug → `fix`, neither → the other types in
   the table. Never pick a type first and fit the diff to it.
3. Scan for breaking changes: removed or renamed public functions/classes,
   changed function signatures, changed runtime behavior contracts (e.g. an
   exception no longer thrown), config/CLI/env-var changes, dependency major
   bumps. If found, apply the breaking-change rules below.
4. If the diff mixes concerns (e.g. a fix plus a feature), keep them in ONE
   commit but give EACH concern its own conventional-commit line — one
   `type(scope): description` line per concern, separated by a blank line,
   then the footers. See "Multiple concerns" under Rules.
5. Only now compose the message and commit.

## Rules

- **Scope:** optional noun describing the code section, e.g. `fix(parser):`.
- **Multiple concerns (fix + feat etc.):** allowed in ONE commit — never
  split. Give each concern its own `type(scope): description` line, separated
  by a blank line, e.g.:

  ```
  fix(parser): prevent crash when node value is null

  feat(parser): add array parsing support
  ```

  Order: `fix` lines first, then `feat`, then other types. Every line MUST
  independently satisfy the format rules above.

  Note: strict Conventional Commits parsers (commitlint, standard-version)
  read only the FIRST line as the subject and will treat subsequent concern
  lines as body text. This is an intentional project-wide extension of the
  spec — readable history is prioritized over tooling compatibility.
- **Description:** imperative mood, concise, no trailing period, starts right
  after `: `.
- **Subject length:** every `type(scope):` line — the first and any additional
  concern lines — stays at 72 characters or fewer.
- **Breaking changes:** append `!` before the colon (`feat(api)!: ...`) OR add
  a footer `BREAKING CHANGE: <description>` (always uppercase). Any type can be
  breaking, and `!` + footer may be combined when the footer explains the
  migration. Breaking changes map to a SemVer MAJOR bump regardless of type.
  In a multi-concern commit, put `!` only on the concern line whose change is
  breaking; if several are, put it on each.
- **Body:** optional, begins one blank line after the description (or after
  the last concern line when there are multiple). Paragraphs separated by
  blank lines.
- **Footers:** placed one blank line after the body — or directly after the
  description if there is no body. Format `Token: value` or `Token #value`,
  e.g. `Refs: #123`, `Reviewed-by: Z`. Footer tokens use `-` instead of
  whitespace (`Acked-by`); `BREAKING CHANGE` is the one exception and is
  always uppercase. Among other footers, `BREAKING CHANGE` comes first;
  the `On-behalf-of` footer MUST be last. Never start a
  body paragraph line with `Word:` — the blank-line separation is what keeps
  parsers from mistaking body text for a footer.

## Examples

```
feat(auth): add Google OAuth login

On-behalf-of: Jane Doe <jane@example.com>
```

```
fix(parser): prevent crash when node value is null

On-behalf-of: Jane Doe <jane@example.com>
```

```
feat!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
On-behalf-of: Jane Doe <jane@example.com>
```

```
fix(parser): prevent crash when node value is null

feat(parser): add array parsing support

The tokenizer previously assumed every node carried a text payload,
which crashed rendering on sparse trees. Null and undefined values now
render as empty strings.

Array support covers bracketed lists in both strict and lenient modes.
Parsing mode is inherited from the parent document unless explicitly
overridden per node.

Multi-dimensional arrays flatten to their first dimension for now; full
nested-array support is tracked separately.

BREAKING CHANGE: `render()` no longer throws on null values; callers
relying on the exception for control flow must check explicitly.

Reviewed-by: Z
Refs: #456
On-behalf-of: Jane Doe <jane@example.com>
```

Here all concern lines come first, then the body (one blank line after the
last concern line, paragraphs separated by blank lines), then footers
(one blank line after the body).

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Reviewed-by: Z
Refs: #123
On-behalf-of: Jane Doe <jane@example.com>
```

## Common mistakes

| Wrong | Right |
|-------|-------|
| `Add Google OAuth login support to auth module` | `feat(auth): add Google OAuth login support` |
| `Fix null-pointer crash in parser` | `fix(parser): prevent null-pointer crash in rendering` |
| `stuff: fix stuff` (invalid type) | use a real type from the table |
| missing or guessed `On-behalf-of` footer | resolve `git config user.name`/`user.email` and include it |
| splitting a fix + feature into separate commits | one commit, one `type(scope):` line per concern |
| picking a type from memory without running `git diff` | run `git diff`, analyze, then pick the type |
| `Fixed the login bug.` (past tense, period) | `fix(auth): prevent logout on token refresh` |

If the wrong type was already committed before merging, fix the history with
`git rebase -i` rather than adding a corrective commit.
