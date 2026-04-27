# Hebrew RTL for Claude Code

A Claude Code skill that fixes Hebrew/RTL rendering bugs in Claude Desktop and Claude Web chat output.

## The problem

When Claude responds in Hebrew (or any RTL language) mixed with English technical terms — code identifiers, filenames, URLs, acronyms — the chat renderer often scrambles word order, misplaces final punctuation, or flips adjacent characters. A sentence like `אני משתמש ב-React.` may render with the period floating in the wrong place, or with words appearing in reverse order after a line wraps.

This is a **bidi (bidirectional text) bug** in how Claude's chat UI applies the Unicode Bidi Algorithm to mixed-content paragraphs.

## The fix

Four core writing principles, enforced mechanically by Claude when this skill is loaded:

1. **Punctuation at the logical start of the line** — every `.`, `?`, and end-of-line `:` moves to the beginning of the string.
2. **English in `⁦...⁩` isolates** — every English token wrapped in Unicode bidi isolates (U+2066 / U+2069). Markdown backticks alone do NOT isolate.
3. **Hebrew word before AND after every English token — on the same line** — both sides of every English token must have a Hebrew word (or Hebrew prefix with hyphen) on the same visual line.
4. **Meaning preservation wins** — never add filler Hebrew words that change the meaning of the source. Rephrase, split, or code-chip instead.

Every rule has been discovered and verified via A/B testing against live Claude chat output.

## Install

```bash
git clone https://github.com/ofir858/claude-hebrew-rtl.git ~/.claude/skills/hebrew-rtl
```

Restart Claude Code (or start a new session). The skill auto-loads from `~/.claude/skills/`.

## How activation works

**No per-project setup. No memory file to create. Install once, works everywhere.**

The skill auto-fires whenever Claude is about to produce Hebrew output, in any project, in any conversation. The trigger is the YAML `description` at the top of `SKILL.md` — Claude Code reads it and treats *any response containing Hebrew characters* as the activation signal. Users who installed this skill are Hebrew speakers; the skill assumes that every Hebrew response should already follow the rules.

### Optional: global fallback (for extra reliability)

If you ever notice Hebrew responses skipping the rules, add a single line to your **user-level** memory file `~/.claude/CLAUDE.md` (which applies globally across every project, not just one):

```
For any response containing Hebrew, apply the rules from ~/.claude/skills/hebrew-rtl/SKILL.md before composing the response.
```

This is a one-time, one-line edit. It belongs in your user-level CLAUDE.md, not a per-project file. After this, the rules are belt-and-suspenders guaranteed.

## What you'll see

Before installing — typical broken output:
```
הפונקציה getUsers() מחזירה מערך.    ← period ends up on the wrong side
אכין תיקיית הפצה עם README ו-LICENSE.    ← word order scrambles
```

After installing:
```
.הפונקציה ⁦getUsers()⁩ מחזירה מערך
.LICENSE ו README תיקיית הפצה אכין
```

The isolates are invisible. The rendered output reads naturally right-to-left for Hebrew readers, with English tokens staying LTR where they should.

## Why this exists

I'm a Hebrew-speaking developer who found Claude's RTL rendering frustrating enough to reverse-engineer the bugs and write around them. After a long session of A/B testing with Claude, these four rules cover every bidi-breaking pattern I've hit.

If the Claude team fixes the underlying renderer, this skill becomes unnecessary. Until then, it works.

Also supports Arabic and Persian (same RTL script family — the rules apply identically).

## ⚠️ About the GitHub bidi-text warning

When you view `SKILL.md` on GitHub, you'll see a warning banner:

> *This file contains bidirectional or hidden Unicode text that may be interpreted or compiled differently than what appears below.*

**This is expected and safe.** The warning is GitHub's automatic flag for any file containing Unicode bidi control characters (LRI `U+2066` and PDI `U+2069`). It was introduced after the *Trojan Source* CVE in 2021 to warn against malicious code that hides its real behavior behind invisible characters.

In our case, those invisible characters are exactly what the skill is about — they're the bidi isolates the rules instruct Claude to wrap around English tokens. The file contains documentation and writing rules only; there is no executable code, so there is no Trojan Source risk.

The warning will continue to appear on every commit because the characters are intentional and central to the skill. You can safely ignore it.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for the version history. Each release is documented with what changed, what was added, and the real-world bug or test case that motivated the change.

## Contributing

Issues and PRs welcome. If you hit a rendering bug this skill doesn't catch, open an issue with a reproduction and I'll add a rule.

## License

MIT — see LICENSE.
