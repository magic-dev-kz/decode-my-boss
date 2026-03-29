# Decode My Boss -- Audit Report

**Auditor:** Nash (QA)
**Date:** 2026-03-29
**Version:** v1 (initial audit)

---

## Overall Score: 8.2 / 10

Solid first-pass. The code is clean, well-structured, and covers most spec requirements. ZERO critical bugs. Strong security hygiene (escapeHtml with all 5 characters, localStorage wrapped in try/catch, CORS warning for Anthropic). Share card correctly omits original text. Main issues are a few contrast problems and missing edge-case handling.

---

## P1 -- Critical (0)

None.

---

## P2 -- Major (3)

### P2-1: toxicity_breakdown values treated with `||0` -- score 0 is falsy
**File:** index.html, line 539 and 635
**Code:** `const v=r.toxicity_breakdown[k]||0;`
**Problem:** This works for zero (maps falsy 0 to 0), BUT if any breakdown value is explicitly `null` or `undefined` from AI, it silently becomes 0 instead of being flagged as missing data. More importantly, the same pattern on line 635 (canvas share card) uses `||0`, which is consistent, but the REAL issue is: `parseFloat(parsed.toxicity_score)||1` on line 482 -- if AI returns `toxicity_score: 0`, this becomes 1.0. The code then `Math.max(1, ...)` clamps to 1 anyway, but the parseFloat fallback masks a 0 score. The spec says "1-10" so minimum 1 is arguably correct, but the parseFloat fallback for non-numeric garbage also defaults to 1 (which makes invalid responses look benign instead of erroring).
**Impact:** A completely broken AI response with garbage in toxicity_score silently gets score=1.0 ("benign"), misleading the user.
**Fix:** If parseFloat returns NaN, throw a parse error or show a warning, don't silently default to 1.

### P2-2: No timeout on AI API calls
**File:** index.html, `callAI` / `callOpenAI` / `callGemini` / `callAnthropic`
**Problem:** No AbortController timeout on fetch calls. If the AI API hangs (rate limit, network issue), user stares at the decrypt animation forever. The spec says "< 10 sec" for decode time -- there is no enforcement.
**Impact:** UX dead-end on slow/hanging API. User has to reload the page.
**Fix:** Add `AbortController` with ~30s timeout. On abort, show a clear "Request timed out, try again" message instead of hanging.

### P2-3: `#f59e0b` (warning/amber) text on `#0f1419` background -- WCAG AA borderline
**File:** index.html, CSS `.cors-warn` color and `.short-warn` color
**Contrast:** `#f59e0b` on `#0f1419` = approximately 8.6:1 -- actually this passes. Let me recalculate.
**Correction:** `#f59e0b` on `#0f1419` is ~8.6:1 -- PASSES AA. However, `#6b7280` (--text-muted) on `#0f1419` (--bg) = approximately **4.2:1** -- this PASSES AA for normal text (4.5:1 threshold) only if we calculate precisely.

Let me recalculate `#6b7280` on `#0f1419`:
- `#6b7280` luminance: R=107, G=114, B=128 -> relative luminance ~0.153
- `#0f1419` luminance: R=15, G=20, B=25 -> relative luminance ~0.0053
- Contrast ratio: (0.153 + 0.05) / (0.0053 + 0.05) = 0.203 / 0.0553 = **3.67:1** -- FAILS AA (4.5:1 required for normal text)

**Affected elements:** header subtitle (line 149 `<p>` with `--text-muted`), `.char-count`, `.flag-severity`, `.loading-sub`, `.prov-label`, `.disclaimer`, settings dropdown label. All use `--text-muted: #6b7280` at small font sizes on dark backgrounds.

**Impact:** Users with low vision may struggle to read muted text elements.
**Fix:** Lighten `--text-muted` to at least `#8b929a` (approximately 5.0:1) or similar.

---

## P3 -- Minor (8)

### P3-1: `--text-sec: #9ca3af` on `--bg: #0f1419` -- verify contrast
- `#9ca3af` luminance ~0.325, `#0f1419` luminance ~0.0053
- Ratio: (0.325 + 0.05) / (0.0553) = 0.375/0.0553 = **6.78:1** -- PASSES AA. OK.

Wait, recalculating more carefully: R=156, G=163, B=175.
- sRGB: 0.612, 0.639, 0.686 -> linear: ~0.336, ~0.365, ~0.428
- Luminance: 0.2126*0.336 + 0.7152*0.365 + 0.0722*0.428 = 0.0714 + 0.261 + 0.0309 = 0.363
- Ratio: (0.363+0.05)/(0.0053+0.05) = 0.413/0.0553 = **7.47:1** -- PASSES. OK, not a bug.

### P3-1 (actual): `--text-muted` on `--card: #1a2332` -- even worse contrast
- `#6b7280` on `#1a2332` (card background, used in settings dropdown `.prov-label`)
- `#1a2332` luminance: R=26, G=35, B=50 -> ~0.012
- Ratio: (0.153+0.05)/(0.012+0.05) = 0.203/0.062 = **3.27:1** -- FAILS AA
- This affects `.prov-label` in settings dropdown and `.settings-dd .prov-label`.

### P3-2: No focus trap in settings dropdown
**File:** index.html, settings dropdown (`#settingsDd`)
**Problem:** The dropdown has `role="menu"` with `role="menuitem"` buttons, which is correct semantically. However, there is no focus trap -- Tab can escape the dropdown to background elements. The Escape key closes it (line 705), which is good, but Arrow key navigation between menuitems is missing (ARIA menu pattern requires it).
**Impact:** Keyboard-only users have suboptimal navigation in the menu.
**Fix:** Add arrow key navigation between menuitems and trap Tab within the dropdown when open.

### P3-3: Settings dropdown does not get focus when opened
**File:** index.html, line 317
**Code:** `gearBtn.addEventListener('click',e=>{e.stopPropagation();settingsDd.classList.toggle('open')});`
**Problem:** When dropdown opens, focus stays on gear button. First menuitem should receive focus for keyboard accessibility.
**Fix:** After opening, focus the first menuitem (`ddChangeKey`).

### P3-4: Service Worker cache-first without revalidation
**File:** sw.js
**Problem:** The SW caches everything on first load and serves from cache forever. There is no stale-while-revalidate strategy, no cache versioning beyond manual bump. Users who loaded v1 will never see updates unless they manually clear cache or the developer bumps `CACHE_NAME`.
**Impact:** Users stuck on old version after updates.
**Fix:** Implement stale-while-revalidate or add cache-busting via versioned `CACHE_NAME` plus checking for new SW on `navigator.serviceWorker.register`.

### P3-5: Manifest icons -- separate `any` and `maskable` (good), but SVG data URI with emoji
**File:** manifest.json
**Problem:** The icons use `data:image/svg+xml` with an emoji character (magnifying glass). Emoji rendering in SVG is platform-dependent -- some platforms may show a blank square. The `any` and `maskable` are correctly separated into two entries (good -- previous products had this wrong). However, the SVG uses an HTML entity `&#128269;` inside a data URI, which may not render correctly everywhere.
**Impact:** PWA icon may display incorrectly on some devices.
**Fix:** Use a simple SVG shape (e.g., a circle with a line for magnifying glass) instead of emoji in SVG.

### P3-6: No `inert` attribute on other sections when settings dropdown is open
**Problem:** When the settings dropdown opens, the rest of the page remains interactive. Screen readers may navigate to elements behind the dropdown.
**Impact:** Minor accessibility issue for screen reader users.

### P3-7: `alert()` for error messages
**File:** index.html, line 358
**Code:** `alert('Error: '+errMsg);`
**Problem:** Native `alert()` is blocking, not styled, and poor UX. It also doesn't match the app's visual design. Consider an inline error banner in the input section.
**Impact:** Jarring UX on errors.
**Fix:** Replace with an inline error message element with appropriate ARIA live region.

### P3-8: Share button not disabled during card generation
**File:** index.html, line 560
**Problem:** `generateShareCard()` is async but the share button is not disabled during execution. Fast double-clicks could trigger two share dialogs.
**Impact:** Minor -- double share attempt.
**Fix:** Disable button during generation, re-enable in finally block.

---

## Acceptance Criteria Verification

### AC-1: Message Input
- [x] Textarea accepts 10-5000 chars (maxlength="5000", decode disabled if <10)
- [x] Context dropdown: Corporate Email, Slack, Teams, HR Letter, Performance Review, Other
- [x] Decode button disabled if text < 10 characters
- [x] Decrypt animation shown during loading
- **Note:** The 10-char minimum is enforced client-side only, which is fine for a BYOK app.

### AC-2: API Key Management
- [x] 3 providers: OpenAI, Anthropic, Google Gemini
- [x] Key saved in localStorage (with try/catch!)
- [x] Invalid key shows error (from API response)
- [x] "Change key" in settings dropdown
- [ ] **MISSING:** "How to get a key?" link with instructions for each provider. The spec requires this (AC-2). There is no help link anywhere.

**Upgrading to P2:**

### P2-4: Missing "How to get a key?" help link
**File:** index.html, setup section
**Problem:** Spec AC-2 requires: "Link 'How to get a key?' with instructions for each provider." No such link exists in the UI.
**Impact:** Users who don't have an API key have no guidance on obtaining one.
**Fix:** Add a "How to get a key?" link/button that shows provider-specific instructions.

### AC-3: Decoded Translation
- [x] Each phrase displayed with original + decoded side by side
- [x] Tone tags with color coding (passive_aggressive, power_move, genuine, neutral, etc.)
- [x] Genuine = green tag
- **Note:** `weaponized_positivity` tone has CSS class (line 97) -- good attention to detail.

### AC-4: Red Flags
- [x] Each flag has: type, evidence quote, severity, explanation
- [x] Severity visually distinct (color-coded borders + text)
- [x] Max 6 flags enforced in `parseResult`
- [x] 0 flags shows "No red flags detected"
- [x] All 10 flag types supported in prompt

### AC-5: Toxicity Score
- [x] Score displayed with one decimal (`.toFixed(1)`)
- [x] Progress bar with gradient green->yellow->red
- [x] Breakdown by 5 categories
- [x] Verdict shown under score

### AC-6: Playbook
- [x] 2-3 steps (capped at 3 in `parseResult`)
- [x] Actionable (enforced by prompt)
- [x] Benign handling in prompt ("No action needed")

### AC-7: Share
- [x] Canvas PNG generation
- [x] Contains: toxicity score, red flags, verdict, URL
- [x] **NO original text in share card** -- VERIFIED. The canvas draws only score, verdict, flags, breakdown, and URL. Original message is never rendered.
- [x] Web Share API with `canShare({files})` check
- [x] Download fallback with `toBlob` + `toDataURL` fallback chain

### AC-8: Responsive
- [x] Mobile-friendly (max-width:600px media query)
- [x] Desktop works (720px container)
- [ ] **PARTIAL:** Spec says "side-by-side translation on desktop, stacked on mobile." Current implementation is always stacked (original above, decoded below). No side-by-side layout on desktop.

### P3-9: No side-by-side translation layout on desktop
**Problem:** Spec AC-8 calls for side-by-side translation on desktop. Implementation is always stacked.
**Impact:** Minor -- the stacked layout works well and is arguably more readable.

### AC-9: Honesty Guard
- [x] Prompt explicitly requires honest scoring (1-3 = normal, benign = 1-2)
- [x] Prompt says "If genuinely fine, say so"
- [x] Short message disclaimer (<50 chars, shown when >=10 and <50)
- **Note:** Spec says "< 2 sentences", implementation checks character count (<50). This is a reasonable approximation.

### AC-10: Localization and Privacy
- [x] Prompt instructs: "Russian input -> Russian output"
- [x] No analytics, trackers, or third-party scripts
- [x] API key stays in localStorage
- [x] Data only sent to chosen AI API
- **Note:** Google Gemini sends API key as URL parameter (not header). This is Google's API design, but means the key appears in server logs and potentially CDN/proxy logs. Not a code bug, but worth noting.

---

## What's Good

1. **localStorage wrapped in try/catch** -- FIRST TIME Ilya's team nailed this from day one. `lsGet`, `lsSet`, `lsRemove` all have try/catch. No more "Ilya's systemic localStorage problem."

2. **escapeHtml covers all 5 characters** -- `&`, `<`, `>`, `"`, `'`. Properly used for both content AND attribute-adjacent contexts (tone class names are escaped before insertion).

3. **CORS-specific error handling for Anthropic** -- TypeError detection + specific message about CORS. This is exactly the lesson from Roast My Listing applied.

4. **Share card privacy** -- Verified: canvas draws score, verdict, flags, breakdown, and URL. Zero original text. Good privacy design.

5. **prefers-reduced-motion** -- Present on line 13. All animations get `duration:0.01ms` and `iteration-count:1`. Correct pattern (not `animation:none` which would break JS handlers).

6. **Canvas toBlob fallback chain** -- toBlob -> toDataURL fallback. Web Share API with canShare check. Complete fallback chain.

7. **parseResult robustness** -- Handles missing fields, caps arrays, clamps score to 1-10. Strips markdown code fences from AI response.

8. **Focus management** -- Sections have `tabindex="-1"` and receive focus on transition. Escape closes dropdown. Gear button gets focus back after dropdown close.

9. **Semantic HTML** -- `role="tablist"` / `role="tab"` / `aria-selected` for provider tabs. `role="menu"` / `role="menuitem"` for settings. `aria-label` on sections. `aria-live="polite"` on dynamic content.

---

## Summary of Issues

| ID | Priority | Description |
|----|----------|-------------|
| P2-1 | Major | parseFloat fallback masks broken AI response as score=1 |
| P2-2 | Major | No timeout on API calls -- user stuck on hanging request |
| P2-3 | Major | `--text-muted` (#6b7280) contrast ~3.67:1 on bg, ~3.27:1 on card -- FAILS AA |
| P2-4 | Major | Missing "How to get a key?" help link (spec AC-2) |
| P3-1 | Minor | text-muted on card bg even worse (~3.27:1) |
| P3-2 | Minor | No focus trap / arrow keys in settings dropdown |
| P3-3 | Minor | Settings dropdown doesn't focus first item on open |
| P3-4 | Minor | SW cache-first without revalidation |
| P3-5 | Minor | Manifest SVG icon with emoji may not render everywhere |
| P3-6 | Minor | No inert on background when dropdown open |
| P3-7 | Minor | alert() for errors instead of inline message |
| P3-8 | Minor | Share button not disabled during generation |
| P3-9 | Minor | No side-by-side translation on desktop |

**Total: 0 P1, 4 P2, 9 P3**

---

## Recommendations for 9.0+

1. Fix `--text-muted` contrast (lighten to ~#8b929a or higher)
2. Add AbortController timeout (~30s) on all API fetches
3. Handle NaN toxicity_score as a parse error, not silent default
4. Add "How to get a key?" modal/link with per-provider instructions
5. Replace alert() with inline error element
6. Focus first menuitem when settings dropdown opens

---

*Audit by Nash, 2026-03-29*
