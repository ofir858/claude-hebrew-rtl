# Changelog

All notable changes to this skill are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.0] — 2026-05-06

### Added
- **Silent self-install on first activation (Claude Code only).** The first time the skill activates in any Claude Code session, it now appends a single marked block to the user-level `~/.claude/CLAUDE.md` referencing itself. After that one-time write, every future Claude Code session in every project automatically picks up the rules with zero user input.
- **Marker-based idempotency.** The self-install uses an HTML-comment marker (`<!-- hebrew-rtl auto-apply -->` ... `<!-- /hebrew-rtl auto-apply -->`) so subsequent activations detect the existing install and skip the write entirely. No duplicate writes across versions, projects, or sessions.

### Changed
- Activation model section in `SKILL.md` rewritten to document the self-install steps (read → check marker → append → continue silently) with explicit "do not announce, do not ask permission" language.
- README "How activation works" section rewritten to describe the install-once-works-everywhere model and the silent self-install behavior.

### Why
A user who installs this skill is a Hebrew speaker who wants the rules applied in every project, automatically, without per-project ceremony. v2.0.0 relied entirely on the YAML description as the auto-invocation trigger, which is not always reliable for output-styling skills. The self-install converts an uncertain description-trigger into a guaranteed user-level memory rule that applies across every project the user opens. After one near-invisible action on first run, the skill is fully transparent forever.

The self-install is intentionally limited to Claude Code (and Cowork, which inherits the same filesystem access). `claude.ai` and Claude Desktop run in a sandbox without filesystem write tools, so the same approach is impossible there — those environments require manual setup.

## [2.0.0] — 2026-04-27

### Initial public release

The skill ships with four core writing principles, discovered and verified via A/B testing against live Claude Desktop output during April 2026.

#### The four principles
1. **Punctuation → logical START of the line** — every `.`, `?`, and end-of-line `:` moves to the beginning of the string.
2. **English in `⁦...⁩` isolates** — every English token wrapped in Unicode bidi isolates (U+2066 / U+2069). Markdown backticks alone do NOT isolate.
3. **Hebrew word BEFORE and AFTER every English token, on the same line** — both sides of every English token must have a Hebrew word (or Hebrew prefix with hyphen). Same line is mandatory.
4. **Meaning preservation wins** — never add filler Hebrew words that change the meaning of the source. Rephrase, split, or code-chip instead.

#### Activation
- **Global, automatic, project-independent.** The skill auto-fires whenever a response will contain Hebrew, in any project, in any conversation. No per-project memory file, no setup ritual. The trigger is the YAML `description`, which Claude Code reads as the auto-invocation signal.
- **Optional global fallback** — users can add a single line to their user-level `~/.claude/CLAUDE.md` for belt-and-suspenders reliability across all projects.

#### URL handling
- URLs are exempt from `⁦...⁩` wrapping (the invisible bidi characters break copy-paste into a browser).
- URLs in bullets must use one of two safe forms only — URL alone in the bullet, or Hebrew description on the bullet line with the URL on the line below. Inline mixing scrambles unpredictably.
- Pre-send URL scan checks every `http://` and `https://` to confirm no URL sits inside isolates.

#### Reading and writing aids
- **Last-5-characters quick check** on every line before sending — catches the most common violation (line ending in English) without per-token scoring.
- **Highest-risk pattern** documented — any line whose natural ending is the English token (bullets, questions, headlines, paragraph closers).
- **Vocabulary lookup tables** for common bullet endings (acronyms, components, file paths, UI routes, error codes).
- **Universal fast-write template**: `[Hebrew description] ⁦[English token]⁩ [Hebrew qualifier word]`.
- **Variation A vs Variation D decision rule** — single sentence with isolates by default; atomic split when more than three English tokens, multi-condition logic, or critical correctness.

[2.1.0]: https://github.com/ofir858/claude-hebrew-rtl/releases/tag/v2.1.0
[2.0.0]: https://github.com/ofir858/claude-hebrew-rtl/releases/tag/v2.0.0
