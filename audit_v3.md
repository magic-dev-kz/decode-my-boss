# Decode My Boss v2 — QA Audit (Nash)
**Date:** 2026-03-29
**File:** `index.html` (884 lines), `sw.js` (29 lines)
**Auditor:** Nash, QA @ OpenClaw

---

## Score: 7.5 / 10

Solid v2 release with good XSS hygiene and accessibility foundations, but a significant privacy gap in history storage contradicts the stated design goal.

---

## P1 — Critical

### 1. History stores full AI result, NOT just snippets
**Location:** `addToHistory()`, line ~816-822
```js
const entry = {
  message_snippet: msgSnippet.substring(0,80),
  // ...
  full_result: result   // <-- entire decoded analysis persisted
};
```
**Problem:** The v2 spec says "privacy: only snippets stored," but `full_result` contains the complete `translation` array (with all original sentences), `red_flags` (with `evidence` quotes), `verdict`, `playbook`, and `toxicity_breakdown`. On a shared or corporate machine, another user opening DevTools > Application > localStorage sees the full decoded analysis including all original boss messages (split by sentence in `translation[].original`).

**Fix:** Store only display-critical fields: `message_snippet`, `context`, `toxicity_score`, `timestamp`. Remove `full_result`. If re-viewing full results is needed, store a hash/ID and re-decode, or explicitly warn the user that full results are cached locally.

### 2. `trans-header` column labels invisible on mobile, but also invisible on first desktop load
**Location:** line 581
```html
<div class="trans-pair trans-header" style="display:none; ...">
```
The inline `style="display:none"` is overridden by the `@media(min-width:768px)` rule using `!important` -- so on desktop it appears. However, the `!important` only applies to the `.trans-pair.trans-header` selector. If CSS load order changes or a future refactor drops `!important`, headers silently vanish. Fragile pattern.

**Fix:** Use a CSS class (e.g., `.desktop-only`) instead of inline `display:none` + `!important` override. Cleaner, less brittle.

---

## P2 — Medium

### 3. History `loadHistoryItem` re-renders full stored result without re-validation
**Location:** line 863-868
```js
function loadHistoryItem(idx){
  const hist = getHistory();
  if(!hist[idx] || !hist[idx].full_result) return;
  lastResult = hist[idx].full_result;
  renderResult(lastResult);
```
**Problem:** If localStorage is tampered with (XSS from another origin on same eTLD+1, or browser extension injection), malformed `full_result` data is fed directly into `renderResult`. While `escapeHtml` protects against HTML injection in rendered output, the `tox-bar-fill` width is set via `(score*10)+'%'` -- a non-numeric score that passes `parseFloat` as `NaN` would produce `NaN%` width (harmless but broken UI). Stored results bypass the `parseResult` validation that live API responses go through.

**Fix:** Run `parseResult`-style validation on loaded history items, or at minimum validate `toxicity_score` is a finite number before rendering.

### 4. `getHistory` / `saveHistory` / `historyClear` bypass the `lsGet`/`lsSet`/`lsRemove` wrappers
**Location:** lines 812-813, 871-874
**Problem:** The codebase defines `lsGet`, `lsSet`, `lsRemove` with try/catch (lines 292-294), but the history functions implement their own inline try/catch instead of using those helpers. This is an inconsistency -- if the error-handling strategy changes (e.g., adding telemetry to `lsSet`), history operations will be missed.

**Fix:** Refactor `getHistory`/`saveHistory`/`historyClear` to use the `lsGet`/`lsSet`/`lsRemove` helpers.

### 5. WCAG AA contrast concern: `.tone-neutral` tag text
**Location:** line 117
```css
.tone-neutral{background:rgba(107,114,128,.2);color:#9ca3af}
```
**Problem:** `#9ca3af` on the effective background (dark card `#1a2332` blended with 20% gray overlay ~ `#273240`) yields approximately **4.0-4.2:1** contrast ratio. The tone tags are rendered at `.68rem` (roughly 11px) which is "small text" under WCAG -- requires **4.5:1 minimum** for AA compliance.

**Fix:** Bump to `#b0b8c1` or lighter (~5:1+). Same audit recommended for `.tone-ambiguous` (`#93c5fd` is fine at ~7:1).

---

## P3 — Low / Nits

### 6. Arrow key navigation in settings dropdown: Enter handler is redundant
**Location:** line 800
```js
else if(e.key==='Enter' && focused && focused.getAttribute('role')==='menuitem'){
  e.preventDefault(); focused.click();
}
```
**Problem:** Buttons already activate on Enter natively. The `e.preventDefault()` + manual `.click()` is harmless but redundant. Wrap-around logic (lines 798-799) is correct and clean.

**Verdict:** No bug. Clean implementation.

### 7. History shows max 5 items (`HISTORY_SHOW=5`) but stores 15 (`HISTORY_MAX=15`)
**Location:** lines 806-807
**Problem:** No UI to view items 6-15. They silently exist in localStorage with no way to access them, consuming storage for no user benefit.

**Fix:** Either add "Show more" / pagination, or reduce `HISTORY_MAX` to 5.

### 8. Service worker caches API responses
**Location:** `sw.js`, line 22-24
```js
caches.match(e.request).then(cached => cached || fetch(e.request).then(resp => {
  const clone = resp.clone();
  caches.open(CACHE_NAME).then(c => c.put(e.request, clone));
```
**Problem:** The SW caches ALL GET requests. API calls to OpenAI/Gemini/Anthropic are POST (so they're excluded -- good), but any future GET-based API integration would be cached incorrectly. Also, third-party resources (fonts, analytics) would be cached forever with no expiry or size limit.

**Verdict:** Not a current bug (API calls are POST), but the cache-everything strategy is a latent risk.

### 9. Mobile responsive layout -- no regressions found
The `@media(max-width:600px)` breakpoint correctly stacks result actions and simplifies tox-breakdown to single column. The side-by-side grid at `min-width:768px` does NOT apply on mobile, so `.trans-pair` remains block layout. No regression.

### 10. Share card -- no regressions found
`generateShareCard` still works correctly. Uses `lastResult` directly. Canvas drawing, blob export, and Web Share API fallback chain is intact.

---

## Regression Check Summary

| Feature | Status | Notes |
|---------|--------|-------|
| AI fetch (OpenAI/Gemini/Anthropic) | OK | AbortController timeout, error handling intact |
| Error handling (inline errors) | OK | CORS, timeout, safety block messages all present |
| Share card generation | OK | Canvas rendering, blob/download fallback chain works |
| PWA / Service Worker | OK | Cache v2, old cache cleanup, POST excluded |
| Escape key closes dropdown | OK | Line 792 |
| Provider tab switching | OK | CORS warning for Anthropic shown |
| XSS protection | OK | `escapeHtml` used consistently in all `innerHTML` paths |
| prefers-reduced-motion | OK | Line 35 |

---

## Verdict

**Ship with P1-1 fix.** The full_result privacy leak is a direct contradiction of the stated privacy design ("only snippets stored") and must be resolved before launch. P2 items are advisable for the release but not blocking. P3 items can go into backlog.
