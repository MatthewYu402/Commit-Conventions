# Commit Conventions

A personal standard for writing commits, naming branches, and keeping a clean Git history across all of my projects.

It is built on **[Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)**, the industry-standard specification used by Angular, Electron, and many other open-source projects. On top of that spec, it adds rules for message quality, commit scope, branching, and tooling.

---

## Table of Contents

1. [Why This Matters](#why-this-matters)
2. [Message Format](#message-format)
3. [Types](#types)
4. [Scope](#scope)
5. [Subject Line Rules](#subject-line-rules)
6. [Body](#body)
7. [Footer](#footer)
8. [Breaking Changes](#breaking-changes)
9. [Examples](#examples)
10. [What Makes a Good Commit](#what-makes-a-good-commit)
11. [Branching](#branching)
12. [Workflow](#workflow)
13. [Versioning](#versioning)
14. [Tooling](#tooling)
15. [Cheat Sheet](#cheat-sheet)
16. [References](#references)

---

## Why This Matters

A commit history is documentation. A good one lets you (or anyone else):

- **Understand why** code exists, not just what it does, via `git log` and `git blame`.
- **Find the commit that broke something** quickly with `git bisect`.
- **Scan the history** and immediately see which commits are features, fixes, or housekeeping.
- **Generate changelogs and version numbers automatically** from commit types.
- **Revert a single change cleanly** without dragging unrelated work with it.

---

## Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

| Part | Required? | Purpose |
|---|---|---|
| `type` | **Yes** | The kind of change (see [Types](#types)). |
| `scope` | Optional | The area of the codebase affected. |
| `subject` | **Yes** | A short summary of the change. |
| `body` | Optional | What changed and **why**. |
| `footer` | Optional | Breaking changes, issue references, co-authors. |

The header (`type(scope): subject`) is a single line. The body and footer are each separated from the line above by **one blank line**.

---

## Types

| Type | Use for | Example |
|---|---|---|
| `feat` | A new feature or capability | `feat(cpu): implement 8XY4 add with carry` |
| `fix` | A bug fix | `fix(display): clip sprites at screen edge` |
| `docs` | Documentation only | `docs: add opcode table to README` |
| `style` | Formatting with **no change in behavior** (whitespace, indentation, semicolons) | `style: normalize indentation in Chip8.cpp` |
| `refactor` | Restructuring code with **no change in behavior** | `refactor(cpu): split decode into per-family helpers` |
| `perf` | A change that improves performance | `perf(display): skip redraw when frame is unchanged` |
| `test` | Adding or fixing tests | `test: add opcode test for FX33 BCD` |
| `build` | Build system or dependencies (Makefile, CMake, packages) | `build: add SDL2 flags to Makefile` |
| `ci` | Continuous integration configuration | `ci: run build on pull requests` |
| `chore` | Maintenance that doesn't fit elsewhere | `chore: add build/ to .gitignore` |
| `revert` | Reverting a previous commit | `revert: feat(cpu): implement BNNN quirk` |

### Choosing between similar types

- **`style` vs. `refactor`:** `style` is purely cosmetic (formatting). `refactor` changes code structure: renaming, extracting functions, reorganizing files. Neither changes behavior.
- **`refactor` vs. `fix`:** if users or tests would notice a difference, it's a `fix` (or `feat`), not a `refactor`.
- **`chore` vs. `build`:** if it touches how the project is compiled or packaged, use `build`. Use `chore` only when nothing more specific fits.
- **When a commit seems to need two types,** it's usually two commits. See [atomic commits](#what-makes-a-good-commit).

---

## Scope

The scope names **the part of the codebase** a commit touches, in parentheses after the type.

- Use a **short, lowercase noun**: `cpu`, `display`, `input`, `loader`, `makefile`.
- **Keep a consistent set per project.** List the scopes in each project's README or `CONTRIBUTING.md` so they don't drift (`display`, `screen`, and `gfx` should not all appear for the same thing).
- **Omit the scope** when a change is project-wide or doesn't belong to one area: `docs: fix typos`.

---

## Subject Line Rules

1. **Use the imperative mood:** write it as a command.
   - ✅ `add ROM size check`
   - ❌ `added ROM size check` / `adds ROM size check` / `adding ROM size check`

   **Test:** the subject should complete the sentence *"If applied, this commit will ___."*

2. **Keep the whole header to 50 characters or fewer** where possible, with a hard limit of 72. GitHub and many tools truncate longer lines.
3. **Start with a lowercase letter** after the colon: `fix: handle empty ROM files`.
4. **No period at the end.**
5. **Be specific.** Describe the change itself, not the activity.
   - ✅ `fix(loader): reject ROMs larger than 3584 bytes`
   - ❌ `fix: bug fix` / `update code` / `changes` / `wip`

---

## Body

Include a body whenever the subject alone doesn't explain **why** the change was made.

- **Explain what and why, not how.** The diff already shows *how*. The body is for the reasoning: the problem, the motivation, the alternatives you rejected.
- **Wrap lines at 72 characters.** Git doesn't wrap text for you, so long lines are hard to read in a terminal.
- **Separate it from the subject with one blank line.** Git and its tools rely on this to tell the subject from the body.
- **Bullet points are fine** for listing several related changes.

```
fix(cpu): write VF after VX in arithmetic opcodes

When X is F, writing the flag first meant the result immediately
overwrote it. Compute the flag, write VX, then write VF, so the
flag survives. Caught by the Timendus flags test.
```

---

## Footer

The footer holds metadata in `Token: value` form, one per line.

| Footer | Purpose |
|---|---|
| `BREAKING CHANGE: <description>` | Describes a change that breaks compatibility (see below). |
| `Closes #12` / `Fixes #12` | Closes a GitHub issue automatically when merged. |
| `Refs #12` | Links to an issue without closing it. |
| `Co-authored-by: Name <email>` | Credits a collaborator. GitHub shows them as a co-author. |

---

## Breaking Changes

A **breaking change** is one that forces users of the code to change something: a removed function, a renamed command-line flag, a changed file format.

Mark it in either or both of these ways:

1. Add `!` after the type or scope: `feat(cli)!: rename --rom flag to --file`
2. Add a `BREAKING CHANGE:` footer explaining what broke and how to migrate.

```
feat(cli)!: rename --rom flag to --file

BREAKING CHANGE: the --rom flag has been removed. Use --file instead,
for example: chip8 --file roms/pong.ch8
```

---

## Examples

**Simple, subject only:**
```
docs: add build instructions to README
```

**With scope:**
```
feat(loader): load ROM into memory at 0x200
```

**With a body:**
```
fix(debug): print uint8_t values as numbers, not characters

uint8_t is a character type, so operator<< printed 0x4F as 'O'.
Cast each byte to int before printing so std::hex formats it.
```

**With a body and footer:**
```
feat(cpu): add configurable shift quirk for 8XY6/8XYE

Classic COSMAC VIP programs expect VX = VY before shifting, while
CHIP-48 programs shift VX in place. Add a flag to select either
behavior; default to CHIP-48.

Closes #7
```

**Revert:**
```
revert: feat(cpu): add configurable shift quirk

This reverts commit 3f2a9c1. The quirk broke the IBM logo test.
```

### Anti-patterns

| ❌ Avoid | Why | ✅ Instead |
|---|---|---|
| `fixed stuff` | No type, vague, past tense | `fix(input): map V key to keypad F` |
| `feat: Add font.` | Capitalized, trailing period | `feat: add font` |
| `update` | Says nothing | `refactor(cpu): extract opcode field decoding` |
| `feat: add timers, fix display bug, reformat` | Three unrelated changes | Three separate commits |
| `wip` | Not meaningful history | Squash before pushing (see [Workflow](#workflow)) |

---

## What Makes a Good Commit

### Atomic

**One commit = one logical change.**

- It does one thing, which its subject describes completely.
- It can be **reverted on its own** without breaking or undoing anything unrelated.
- If the subject needs the word **"and,"** consider splitting the commit.

### Complete

- **Every commit should build** (and pass tests, when there are tests). This keeps `git bisect` usable, because every point in history can be checked out and run.
- Don't commit half of a change and leave the project broken until the next commit.

### Clean

- **No unrelated changes.** Don't sneak a formatting fix into a feature commit; make it a separate `style` commit.
- **No generated files or secrets.** Build output, editor files, `.env` files, and API keys belong in `.gitignore`.
- **No debugging leftovers** such as stray print statements or commented-out code, unless they are intentional.

---

## Branching

`main` always holds working code. Everything else happens on short-lived branches.

### Branch names

```
<type>/<short-description>
```

- Use the **same types as commits**: `feat/`, `fix/`, `docs/`, `refactor/`, `chore/`, and so on.
- Use **lowercase words separated by hyphens.**
- Optionally include an issue number: `fix/12-sprite-wraparound`.

| ✅ Good | ❌ Bad |
|---|---|
| `feat/sdl-display` | `new-stuff` |
| `fix/vf-flag-ordering` | `Matthew_Branch` |
| `docs/opcode-table` | `test2` |

### Rules

- **Never commit directly to `main`** for anything beyond trivial documentation fixes.
- **Keep branches short-lived.** Merge within days, not weeks, to avoid painful conflicts.
- **Delete branches after merging.**

---

## Workflow

### Staging intentionally

- Use `git add -p` to review and stage changes **piece by piece**. This is the easiest way to split mixed work into atomic commits.
- Run `git diff --staged` before committing to see exactly what will be included.

### Cleaning up before pushing

Local history can be messy while you work. Clean it up **before** pushing:

- `git commit --amend`: fix the most recent commit's message or add a forgotten file.
- `git rebase -i <base>`: reorder, combine (`squash` / `fixup`), or reword commits.
- `git commit --fixup <hash>` followed by `git rebase -i --autosquash <base>`: attach a fix to an earlier commit automatically.

### The golden rule

> **Never rewrite history that has been pushed to a shared branch.**

Amending, rebasing, and force-pushing are fine on your **own** branches before others build on them. On `main` or any branch others use, history is permanent; fix mistakes with new commits (or `git revert`). If you must force-push your own branch, use `git push --force-with-lease`, which refuses to overwrite work you haven't seen.

### Merging

- Prefer **squash merging** for small feature branches, so `main` gets one clean, conventional commit per feature.
- Use a **regular merge or rebase** when the branch's individual commits are each meaningful on their own.
- Whichever method you use, the commit that lands on `main` must follow this convention.

### Pull requests

- **Title:** written in the same format as a commit header, since it often becomes the squashed commit message.
- **Description:** what changed, why, and how it was tested. Link related issues with `Closes #N`.

---

## Versioning

Conventional Commits maps directly to **[Semantic Versioning](https://semver.org/)** (`MAJOR.MINOR.PATCH`):

| Commits since the last release include… | Version bump | Example |
|---|---|---|
| A breaking change (`!` or `BREAKING CHANGE:`) | **MAJOR** | `1.4.2` → `2.0.0` |
| A `feat` | **MINOR** | `1.4.2` → `1.5.0` |
| Only `fix` or `perf` | **PATCH** | `1.4.2` → `1.4.3` |
| Only `docs`, `style`, `refactor`, `test`, `build`, `ci`, `chore` | None | — |

Before `1.0.0`, the project is considered unstable, and breaking changes may bump only the MINOR version.

Tag releases with an annotated tag: `git tag -a v1.0.0 -m "v1.0.0"`.

---

## Tooling

These are optional, but they make the convention easy to follow and hard to break.

### Commit message template

Create `~/.gitmessage`:

```
# <type>(<scope>): <subject>   (50 chars max, imperative, no period)
# |<----  Using a maximum of 50 characters  ---->|

# Body: explain WHAT and WHY, not how. Wrap at 72 characters.
# |<----   Try to limit each line to a maximum of 72 characters   ---->|

# Footer: BREAKING CHANGE: ..., Closes #N, Co-authored-by: ...

# Types: feat fix docs style refactor perf test build ci chore revert
```

Then enable it globally:

```
git config --global commit.template ~/.gitmessage
```

Every `git commit` (without `-m`) now opens your editor with this reminder. Lines starting with `#` are stripped automatically.

### Useful Git settings

```
git config --global core.editor "code --wait"       # or vim, nano, etc.
git config --global init.defaultBranch main
git config --global pull.rebase true                 # rebase instead of merge on pull
git config --global push.autoSetupRemote true        # first push sets upstream automatically
```

### Enforcing the format (optional)

- **[commitlint](https://commitlint.js.org/)** with `@commitlint/config-conventional` checks every commit message against the spec and rejects ones that don't comply.
- **[Husky](https://typicode.github.io/husky/)** or **[pre-commit](https://pre-commit.com/)** runs checks like commitlint automatically as Git hooks.
- **[release-please](https://github.com/googleapis/release-please)** or **[semantic-release](https://semantic-release.gitbook.io/)** generates changelogs and version bumps from your commit history.

For small personal projects, the template alone is usually enough. Add enforcement once a project has collaborators or automated releases.

### Viewing history

```
git log --oneline --graph --decorate     # compact visual history
git log --grep="^fix"                    # all fixes
git log -p -- path/to/file               # full history of one file
```

---

## Cheat Sheet

```
<type>(<scope>): <subject>
                                         ← blank line
<body: what and why, wrapped at 72>
                                         ← blank line
<footer: BREAKING CHANGE / Closes #N>
```

- **Types:** `feat` · `fix` · `docs` · `style` · `refactor` · `perf` · `test` · `build` · `ci` · `chore` · `revert`
- **Subject:** imperative ("add," not "added"), lowercase, no period, ≤ 50 characters
- **Test:** "If applied, this commit will ___."
- **One commit = one logical change.** Every commit builds.
- **Branches:** `<type>/<short-description>`
- **Breaking change:** `type!:` and/or a `BREAKING CHANGE:` footer
- **Never rewrite pushed shared history.**

---

## References

- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/): the specification this guide is based on
- [Angular Commit Message Guidelines](https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit): the convention Conventional Commits grew out of
- Chris Beams, [How to Write a Git Commit Message](https://cbea.ms/git-commit/): the classic guide to subject lines and bodies
- [Semantic Versioning 2.0.0](https://semver.org/)
- [Pro Git book](https://git-scm.com/book/en/v2), especially the chapters on rewriting history and branching workflows
