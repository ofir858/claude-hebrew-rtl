---
name: hebrew-rtl
description: |
  AUTO-INVOKE before producing ANY response containing Hebrew characters, in any
  project, in any conversation. The Claude Desktop and Web chat renderers have bidi
  (bidirectional text) rendering bugs that misplace final punctuation and scramble
  mixed Hebrew/English word order; this skill enforces four writing principles that
  work around those bugs.

  TRIGGER (strong, project-independent): any response that will contain Hebrew
  characters — Hebrew prose, Hebrew mixed with English code identifiers, file
  paths, URLs, technical terms, or even a one-line Hebrew reply. The trigger is
  the presence of Hebrew in the OUTPUT, not the user's phrasing of the request.

  A user who installs this skill is a Hebrew speaker. Do not wait for them to
  explicitly invoke the skill — every Hebrew response should already follow the
  rules. If you detect any Hebrew in what you are about to write, invoke this
  skill first.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
---

# Hebrew/RTL writing — 4 core principles

**The problem:** The Claude Desktop and Claude Web chat renderers have bidi (bidirectional text) bugs. When a Hebrew paragraph contains English tokens (code identifiers, technical terms, filenames), the renderer often scrambles word order, misplaces final punctuation, or flips adjacent characters. These rules enforce a writing pattern that works around the bugs.

**Discovered and refined via A/B testing with Ofir Sharon (April 2026).** Every rule below has been verified against live output in Claude Desktop.

## The four principles (in priority order)

### 1. Punctuation → logical START of the line
Every `.`, `?`, and end-of-line `:` moves to the BEGINNING of the string, not the end.

- Natural (broken): `זה עובד טוב.`
- Fixed: `.זה עובד טוב`

- Natural: `רוצה לעדכן?`
- Fixed: `?רוצה לעדכן`

- Natural: `הסיבות:`
- Fixed: `:הסיבות`

Mid-sentence colons (between Hebrew words, not at line-end) stay in place.

### 2. English tokens → always wrapped in `⁦...⁩`
Every English word, identifier, filename, acronym, or URL inside Hebrew prose wraps in Unicode bidi isolates (LRI U+2066 ... PDI U+2069). **Markdown backticks do NOT isolate** — they style but don't direct the bidi algorithm. Always use `⁦...⁩` on English, even inside backticks, section headers, bold labels, or parentheses.

- Broken: `נקרא bidi algorithm`
- Fixed: `נקרא ⁦bidi algorithm⁩`

### 3. Hebrew word BEFORE and AFTER every English token — ON THE SAME LINE

**This single rule replaces "Hebrew at start" and "Hebrew at end" rules.** It's easier to apply mechanically: for every English token, just look left and right ON THE SAME LINE — both sides must have a Hebrew word (or Hebrew prefix attached with hyphen).

**⚠️ SAME LINE IS MANDATORY.** Hebrew on the next line does NOT count. Every individual line is rendered independently by the bidi algorithm. A bullet title that ends in English still breaks even if the description below it starts in Hebrew.

- Broken: `- .ציור בטעינה דרך ⁦draw-in on mount⁩`  (description is on next line — doesn't help)
- Fixed: `- .ציור בטעינה דרך ⁦draw-in on mount⁩ הטבעי`  (Hebrew word on SAME line)
- Also fixed: merge the description into the same line so there's Hebrew after.

**When scanning, look at each line in isolation.** Do not let yourself rationalize "the next line has Hebrew, so it's fine" — that rationalization is THE bug this rule exists to prevent.

**Examples:**

- Broken: `.⁦Linear⁩ משתמשים ב-⁦Inter⁩`  (nothing before ⁦Linear⁩)
- Fixed: `.חברת ⁦Linear⁩ משתמשת ב-⁦Inter⁩ היומיומי`  (Hebrew before and after each)

- Broken: `.הפונקציה נקראת ⁦getUsers()⁩`  (nothing after ⁦getUsers()⁩)
- Fixed: `.הפונקציה ⁦getUsers()⁩ מחזירה מערך`  (Hebrew after)

- Broken: `.⁦setUsers()⁩ ⁦getUsers()⁩ שתי פונקציות`  (no Hebrew between the two English tokens)
- Fixed: `.⁦setUsers()⁩ והפונקציה ⁦getUsers()⁩ הן שתי פונקציות`  (Hebrew between)

**Exemptions:**

- Code inside fenced blocks (```) is exempt — it stays pure LTR.
- Hebrew prefixes attached with hyphen count as a Hebrew word: `ב-⁦Inter⁩`, `ה-⁦dev server⁩`, `ל-⁦API⁩` are all valid.
- Pure English bullets in a list can stand alone IF the surrounding context is Hebrew (header before the list, paragraph after).
- **URLs are NEVER wrapped in `⁦...⁩` isolates.** The invisible isolate characters get copied together with the URL and break the link when pasted in a browser. Instead: put URLs on their own line, in a fenced code block, or as a proper markdown link `[text](url)`. Example of the bug: `⁦https://github.com/ofir858/repo⁩` — looks fine in chat but breaks when copied. Instead write the URL on its own line with Hebrew context on the line above, or use: `הקישור נמצא ב-[GitHub](https://github.com/ofir858/repo)`.

**Fixed opener vocabulary — use these when a line would start with English:**

- **כך / זהו** — neutral general openers
- **ראשית / שנית / שלישית** — for numbered enumeration
- **הסוג הראשון / הסוג השני** — for category lists
- **חברת / הפונט / הכלי / הקובץ / הפונקציה / הרכיב** — restate the type of the English token

**Why this is faster than two separate rules (3 and 3b):** one unified check instead of two. Just scan each English token for Hebrew neighbors in both directions. If either direction is missing Hebrew — add a word or rephrase.

### 4. Meaning ALWAYS wins over mechanics
If rules 1-3 require adding a Hebrew word that changes the meaning of the source, **reject the addition**. Instead do one of:

a. **Rephrase naturally** so a Hebrew word ends the line without distorting meaning.
b. **Code-chip**: put the English on its own line, then write a Hebrew sentence referring to it.
c. **Split**: break into two shorter sentences so no single line has to force-end in English.

**Never** use these as fillers to meet rule 3 (they change meaning):
- "הראשי" (implies hierarchy)
- "בלבד" (adds restriction)
- "הרגיל" (implies variants)
- "לחלוטין" / "בדיוק" / "ממש" (intensifiers)
- "אוטומטית" / "מיידית" / "ישירות" (manner/timing adverbs)

**Safe neutral endings** (only if they fit grammatically AND match real meaning):
- "עצמו" / "עצמה" — emphatic reflexive
- "הזה" / "הזאת" — demonstrative
- Restating the subject: "הקוד", "הקובץ", "הפונקציה"

## ⚠️ Highest-risk pattern: ANY line ending in a technical token

**This is the #1 place where the rule gets dropped.** Any line whose natural ending is the English token — bullets, questions, headlines, paragraph closers, list items — is high-risk. The token IS the point of the sentence, so the natural phrase finishes there. Without explicit attention, the line ends in English.

This applies to ALL line types, not just bullets. Specifically including:

- **Bulleted items** describing technical facts
- **Questions** ending with the technical subject ("?רוצה לעדכן את ⁦ManyChat⁩")
- **Headlines and section closers** ending with the topic name
- **Paragraph last sentences** that conclude with the term
- **Closing CTAs** in any output ("?לבנות עם ⁦Stripe⁩")

Examples that broke in real testing:

- Broken: `כל mutation עוברת דרך RLS וסקלה ל-tenant_id`  (bullet)
- Broken: `מסכים בלי 404`  (bullet, ends with number)
- Broken: `העלאת קובץ json`  (bullet, file extension)
- Broken: `?רוצה שנתכנן יחד את ה-flow של ManyChat`  (question)
- Broken: `הסיום מתבצע ב-Supabase`  (paragraph closer)

### Universal template (mechanical fix)

For every line that contains a technical token, force this shape:

```
[Hebrew description] ⁦[English token]⁩ [Hebrew qualifier word]
```

This works for bullets, questions, paragraphs — everywhere. The Hebrew qualifier at the end does the work. It must reflect actual reality, not be a meaningless filler.

### Fixed vocabulary for bullet endings

Pick from this list — these match common technical contexts without distorting meaning:

**After an acronym describing a permission/policy** (RLS, JWT, ACL, RBAC):
- "הנכון" — when it's the correct/specific one
- "הרלוונטי" — when it applies to this context
- "המתאים" — when it fits the use case

**After a technical term describing a feature/component** (middleware, hook, store):
- "המוגדר" — when it's defined/configured
- "הקבוע" — when it's set/persistent
- "הראשי" — when it's the primary one of its kind

**After a number or error code** (404, 500, 200):
- "כלשהן" / "כלשהו" — for "any", non-specific
- "הספציפי" — when referring to a particular one
- "הצפוי" — when it's expected behavior

**After a file path or filename** (config.ts, users.json):
- "הראשי" — main/primary file
- "הייעודי" — purpose-built file
- "המקורי" — original/source file
- "הקיים" — existing file

**After a UI element or screen route** (/dashboard, /settings):
- "הראשי" — primary route
- "הייעודי" — purpose-built screen
- "המבוקש" — requested/target screen

### When to apply

Whenever ANY line — bullet, question, headline, paragraph closer — would naturally end with an English token, pick a qualifier from the vocabulary above. Don't think it through each time. Just pick the closest match and append.

**Special attention to closing questions and last lines.** The end of a long output is where discipline drops most. Run the 5-character check explicitly on the LAST line before sending — and on every closing question (`?...`) regardless of where it appears.

This converts the most error-prone pattern from "judgment call" to "dictionary lookup".

## Last-5-characters quick check (lightweight safety net)

**Before sending, glance at the last 5 characters of each line.**

If the last 5 characters end with an English letter, a closing isolate `⁩`, a number, or a URL fragment — that line is broken. Fix it by adding a Hebrew word before the punctuation or rewriting the sentence.

This is a one-second check per line, not a full scan. It catches the most common violation (line ending in English) without the heavy overhead of line-by-line scoring.

**What to look for at end of line:**

- Ends with Hebrew letter → ✅ good
- Ends with `⁩` (isolate close) → ❌ add Hebrew word after
- Ends with Latin letter → ❌ add Hebrew word after or rewrite
- Ends with `/`, `.css`, `.ts`, or similar path fragment → ❌ move to own line or rephrase
- Ends with number alone (like `2008`) → ⚠️ usually OK but check context

This check is kept permanent — even when the full rule set is fluently applied, this 5-character glance costs almost nothing and catches the one mistake I'm most likely to make.

## URLs in bullets — two valid forms only

**The combination of bullet + URL is uniquely fragile.** Even when every other rule is followed correctly, putting a URL inline with Hebrew text in a bullet often scrambles word order. Empirical testing showed that no inline pattern (`Hebrew + English + Hebrew + URL + Hebrew`) renders reliably in Claude Desktop. The bidi engine reorders adjacent runs unpredictably.

### Two valid forms only

Use one of these. Anything else is broken.

**Form 1 — URL alone in the bullet:**
```
- https://www.ynet.co.il
```

**Form 2 — Hebrew description on the bullet line, URL on the line below:**
```
- אתר ⁦Ynet⁩ הישראלי
  https://www.ynet.co.il
```

### Pre-bullet checks (must run for any bullet containing a URL)

1. **No bidi isolates around the URL.** No `⁦` or `⁩` touching the URL itself.
2. **One of the two valid forms above** — never inline mixed with Hebrew on both sides.

### Forbidden patterns (always break)

- `- אתר ⁦Ynet⁩ ב-https://www.ynet.co.il הישראלי` — Hebrew prefix `ב-` before URL with Hebrew after
- `- אתר ⁦Ynet⁩ בכתובת https://www.ynet.co.il` — Hebrew word before URL, end of line
- `- אתר ⁦Ynet⁩ — https://www.ynet.co.il הישראלי` — em-dash separator with Hebrew after URL
- `- ⁦https://www.ynet.co.il⁩` — URL wrapped in isolates (breaks click)

## Pre-send URL scan

**Before sending, scan the draft for `http://` or `https://` and confirm each URL sits in plain text or inside a code fence — never inside `⁦...⁩` isolates.**

The Last-5-characters check above does NOT catch this — URLs are usually mid-line, not line-final. And Principle 2's "wrap every English token" reflex routinely violates the URL exemption (Principle 3) unless this check is run explicitly.

If you find a wrapped URL: strip the `⁦` and `⁩` around it. The URL should sit free in the line, with Hebrew on at least one side per Principle 3.

Symptom of a missed wrap: the user clicks the link and gets a 404 with `%E2%81%A9` (PDI, U+2069) appended to the URL. Discovered the hard way during a Tier-2 dashboard handoff session (April 2026) — the rule existed but the active scan didn't, and Claude wrapped every URL in the message reflexively.

## The fast-write template

For any mixed Hebrew-English sentence, write in this shape:

```
.[Hebrew OPENER] ⁦[English token]⁩ [Hebrew bridge] [Hebrew ENDING]
```

Both the opener AND ending must be Hebrew words. Concrete examples:

- `.הפונקציה ⁦getUsers()⁩ מחזירה מערך נתונים`  (opener: "הפונקציה", ending: "נתונים")
- `.הסוג הראשון — ⁦UI typeface⁩ — מטפל בטקסט הפונקציונלי`  (opener: "הסוג", ending: "הפונקציונלי")
- `.חברת ⁦Linear⁩ משתמשת ב-⁦Söhne⁩ לכותרות`  (opener: "חברת", ending: "לכותרות")

This shape is always safe. If the content won't fit:
- More than 3 English tokens → split into multiple lines (one token per line)
- Sentence ends in English naturally → use code-chip (English on own line)
- Complex logic with multiple conditions → split into atomic sentences

## Decision tree (run mentally, don't re-derive)

```
For each line:

1. Wrap every English token in ⁦...⁩.

2. For EACH English token — scan left and right:
   ├── Hebrew word (or Hebrew prefix with hyphen) on BOTH sides?
   │     ├── YES → good, move on.
   │     └── NO  → add Hebrew, rephrase, or split/code-chip.

3. Move period/question/colon to logical start.

4. Done.
```

**Run mechanically, don't re-derive each rule on every line.**

## When to use Variation A vs Variation D

- **Variation A (default)**: single sentence with `⁦...⁩` isolates. Use when there are ≤3 English tokens and a single coherent thought.
- **Variation D (split)**: atomic sentences, one English token per line. Use when:
  - More than 3 English tokens would appear in one sentence
  - Multi-condition logic (if/then/else, multiple states)
  - Ordered sequence where step order matters
  - Critical info where misreading causes a bug

## Canonical before/after examples

```
SOURCE:  זה עובד טוב.
OUTPUT:  .זה עובד טוב

SOURCE:  רוצה לעדכן?
OUTPUT:  ?רוצה לעדכן

SOURCE:  הרכיבים:
OUTPUT:  :הרכיבים

SOURCE:  הפונקציה getUsers() מחזירה מערך.
OUTPUT:  .הפונקציה ⁦getUsers()⁩ מחזירה מערך

SOURCE:  הפתרון — reverse-order composition.
OUTPUT:  .reverse-order composition—הפתרון

SOURCE:  אכין תיקיית הפצה עם README ו-LICENSE.
OUTPUT:  .LICENSE ו README תיקיית הפצה אכין
```

## Principle ordering when they conflict

When rules collide: **4 > 3 > 1 > 2**.
- Meaning preservation (4) trumps everything.
- Hebrew ending (3) trumps punctuation position (1) when it would require a bad filler.
- Punctuation position (1) trumps isolation (2) when both apply to the same token.

## Why this works

The Claude Desktop / Claude Web renderer misplaces final punctuation and gets confused by unisolated English tokens in Hebrew paragraphs. By pushing punctuation to the logical start and isolating every English token, the bidi algorithm processes directions correctly per line. Ending in Hebrew ensures the paragraph direction is detected as RTL. Meaning preservation keeps the rewrite faithful to the source.

## When this skill applies

- Any Hebrew response in Claude Desktop or Claude Web chat
- Any line mixing Hebrew with English (code, technical terms, filenames)

Not needed for:
- Pure English responses
- Code written to files (bidi is not an issue in source code)
- Translation file content (that's a translation-quality concern, not a rendering one)

## Activation model — global, automatic, no per-project setup

This skill is designed to fire automatically whenever a response will contain Hebrew. There is no project-level setup, no memory file to create, no per-project ritual. The only mechanism is the YAML `description` at the top of this file, which Claude Code reads as the auto-invocation trigger.

**What you (as Claude) must do:**

Before composing any response, ask: *will this response contain Hebrew characters?* If yes, the rules in this file apply — apply them. The presence of Hebrew in the output is the trigger; the user does not need to name the skill, request it, or set anything up.

**Optional belt-and-suspenders for users who want extra reliability:** they can add a single line to their user-level `~/.claude/CLAUDE.md` (which applies globally across all projects):

```
For any response containing Hebrew, apply the rules from ~/.claude/skills/hebrew-rtl/SKILL.md before composing the response.
```

This is a documented option in the README, not a required action. Do not auto-create or modify `CLAUDE.md` — it is the user's file to edit.
