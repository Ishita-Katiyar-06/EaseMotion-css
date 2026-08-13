# Bug Fix: `ease-in-out` Easing Value in Motion Engine Parser

## 1. What does this fix?

In `easemotion/engine/parser.js`, the `'ease-in-out'` key in `EASING_MAP` was mapped to
`cubic-bezier(0.4, 0, 0.2, 1)` — the **exact same cubic-bezier value** as the default `'ease'`
curve. This is a copy-paste bug that makes the `ease-in-out` token completely indistinguishable
from `ease` at runtime.

### Broken behaviour

Using `em="fade-in ease-in-out"` produced **identical motion** to `em="fade-in ease"` because
both resolved to the same timing function. The distinction intended by the developer is silently
discarded.

## 2. The fix

Change the `'ease-in-out'` entry in `EASING_MAP` to the standard W3C symmetric ease-in-out curve:

```diff
// easemotion/engine/parser.js — EASING_MAP
- 'ease-in-out': 'cubic-bezier(0.4, 0, 0.2, 1)',   // BUG: duplicate of 'ease'
+ 'ease-in-out': 'cubic-bezier(0.42, 0, 0.58, 1)', // FIX: symmetric ease-in-out (CSS spec)
```

The value `cubic-bezier(0.42, 0, 0.58, 1)` matches the W3C CSS Animations Level 1 definition of
`ease-in-out` and is what developers expect when they write that token.

## 3. How is it used?

```html
<!-- em="" attribute — ease-in-out now produces a distinctly different curve from ease -->
<div em="slide-up 600ms ease-in-out">Slides in with symmetric acceleration</div>
<div em="fade-in 400ms ease">Fades with the default curve (unchanged)</div>
```

## 4. Why does this fit EaseMotion CSS?

EaseMotion's core promise is **human-readable** motion tokens. When a developer writes
`ease-in-out`, they expect the standard symmetric timing curve — not a silent fallback to `ease`.
This fix makes the engine honest: every named easing token does exactly what it says.

The fix is a single-line change with zero visual regression on existing `ease`, `ease-in`,
`ease-out`, `linear`, `spring`, `bounce`, and `snap` tokens, which are all unaffected.

## 5. Demo

Open `demo.html` directly in a browser — no server required.

Click **▶ Play Animation** to see both boxes slide across the track simultaneously.
- 🔴 Left box uses the **buggy** `cubic-bezier(0.4, 0, 0.2, 1)` (same as `ease`)
- 🟢 Right box uses the **fixed** `cubic-bezier(0.42, 0, 0.58, 1)` (true `ease-in-out`)

The difference is most visible at the start and end of the motion — the fixed curve has
a noticeably smoother, more symmetric acceleration and deceleration.

## 6. Affected file

| File | Change |
|---|---|
| `easemotion/engine/parser.js` | Line 23: change `cubic-bezier(0.4, 0, 0.2, 1)` → `cubic-bezier(0.42, 0, 0.58, 1)` for `'ease-in-out'` |

> **Note for maintainer:** The existing unit test in `tests/engine.test.js` only checks that
> `ease-out` parses to `cubic-bezier(0, 0, 0.2, 1)` and `spring` to its value — there is no
> test for `ease-in-out` specifically. A test asserting
> `expect(parse('fade-in ease-in-out').easing).toBe('cubic-bezier(0.42, 0, 0.58, 1)')` would
> prevent this class of regression from recurring.
