# Decode My Boss — Changelog

## v11.0 (2026-03-29) — Micro-improvements

### Changed
- **Welcome banner animated gradient border** — banner now has a continuously shifting gradient border (blue-purple-lavender) via `::after` pseudo-element with `welcomeBorderShift` keyframes cycling at 4s
- **Template chips stronger hover lift** — hover translateY increased from -1px to -3px with scale(1.03); shadow deepened to 8px/20px spread for more tactile feel
- SW cache bumped to `decode-my-boss-v11.0`

### Preserved
- All v10.x features intact (Gemini default, draft response glow, welcome banner steps)

---

## v10.0 (2026-03-29) — Micro-improvements

### Changed
- **Gemini as default provider** — Google Gemini (free tier) pre-selected as default provider instead of OpenAI; reduces friction for new users who can get a free API key from Google AI Studio
- **Draft Response more prominent** — Draft a Response button now has animated gradient glow (`draftGlow` keyframes) and gradient background shift; tooltip "AI writes your reply — choose your tone" shown on hover via `.draft-btn-wrap` wrapper

### Added
- **"How it works" welcome banner** — first-visit onboarding banner with 3 steps: "Get a Free Key", "Paste Message", "See Decoded"; dismissible via close button; state persisted in localStorage (`dmb_welcome_dismissed`); not shown on repeat visits

### Changed
- SW cache bumped to `decode-my-boss-v10.0`

### Preserved
- All v9.x features intact (message stats bar, share on X, export history)
- All AI integration (OpenAI, Gemini, Anthropic) untouched
- Share card canvas generation unchanged
- Feedback widget unchanged

## v9.0 (2026-03-29) — Pre-launch Polish

### Added
- **Message Stats Bar** — real-time word count and character count displayed under the textarea; warning "Short messages may give less accurate results" shown when word count is between 1 and 9 words
- **Share on X** — "Share on X" button in result actions opens Twitter intent URL in a new tab with tweet text: "My boss's message scored X/10 on the toxicity scale [unlock emoji] Check yours: [URL]"
- **Export History** — "Export History" button in the history section header downloads all decode history as a formatted JSON file via Blob + URL.createObjectURL

### Changed
- Textarea input handler refactored into reusable `updateMsgStats()` function called from all text-filling paths (examples, templates, URL share, re-decode, new analysis)
- SW cache bumped to `decode-my-boss-v9.0`

### Preserved
- All v8.x features intact (templates library, history summary cards, share as link)
- All AI integration (OpenAI, Gemini, Anthropic) untouched
- Share card canvas generation unchanged
- Feedback widget unchanged

## v8.0 (2026-03-29) — Enhanced Decode Experience

### Added
- **Message Templates Library** — collapsible "Common Boss Messages" section with 11 pre-built templates across 3 categories: Passive-Aggressive (4), Power Moves (4), Vague Threats (3); clicking a template fills textarea and auto-decodes if API key is configured
- **Decode History Enhancement** — clicking a history item now shows a compact summary card with toxicity score, verdict, top 2 red flags (with severity coloring), and first decoded phrase; "Decode Again" button re-fills textarea and triggers new analysis; "Back" button returns to input
- **Share as Link** — new "Share as link" button in result actions copies URL with `?example=encoded-text` to clipboard; opening the URL auto-fills the textarea with the decoded text
- History entries now store additional data: `top_flags` (top 2 flags with type/severity), `first_decoded` (first decoded phrase), `full_message` (up to 300 chars for re-decode)

### Changed
- URL `?example=` parameter now supports both chip slugs (v7 compat) and URL-encoded full text (v8)
- SW cache bumped to `decode-my-boss-v8.0`

### Preserved
- All v7.x features intact (PWA install, keyboard shortcut, existing URL share for examples)
- All AI integration (OpenAI, Gemini, Anthropic) untouched
- Share card canvas generation unchanged
- Feedback widget unchanged

## v7.0 (2026-03-29) — PWA Install, Keyboard Shortcut & URL Share

### Added
- **PWA Install Prompt** — fixed bottom banner shown after 2+ visits via `beforeinstallprompt` event; "Install" triggers native prompt, "Not now" dismisses and persists choice in localStorage (`dmb_pwa_dismissed`)
- **Keyboard Shortcut** — `Ctrl/Cmd+Enter` triggers decode when textarea is focused and button is enabled
- **URL Share for Examples** — clicking an example chip adds `?example=slug` to URL via `replaceState`; loading page with `?example=per-my-last-email` auto-fills textarea with matching example text

### Changed
- SW cache bumped to `decode-my-boss-v7.0`

### Preserved
- All v6.x features intact (decode, draft response, tone slider, copy, examples, history, share card, feedback)

## v6.0 (2026-03-29) — Polish & Accessibility

### Added
- **JetBrains Mono** — connected via Google Fonts for monospace elements (code, scores, evidence)
- **Shadow design tokens** — `--shadow-sm`, `--shadow-md`, `--shadow-lg`, `--shadow-xl` CSS variables with dual-layer (ambient + direct) shadows; all inline box-shadows consolidated
- **Toxicity countUp animation** — score animates from 0 to final value over 1.2s with ease-out easing using `requestAnimationFrame`
- **Toxicity bar ARIA** — `role="progressbar"`, `aria-valuenow`, `aria-valuemin="0"`, `aria-valuemax="10"`, `aria-label="Toxicity score"` on the toxicity bar element
- **Tone tag sanitization** — CSS class names from AI tone labels sanitized via `.replace(/[^a-z_]/g, '')` to prevent class injection / XSS

### Changed
- **Mobile padding fix** — container padding increased from 12px to 16px on mobile; card padding minimum 20px on mobile breakpoint
- **Decode button disable** — button disabled during API request with `opacity: 0.6` + `cursor: not-allowed`; re-enabled in `finally` block
- **Subtitle updated** — tagline changed to "Paste what your boss wrote. See what they really meant."
- SW cache bumped to `v6.0`

### Preserved
- All v5.x features intact (decode, draft response, tone slider, copy, examples, history, share card, feedback)
- All AI integration (OpenAI, Gemini, Anthropic) untouched
- `prefers-reduced-motion` respected

## v5.0 (2026-03-29) — Full Cycle: Decode + Respond

### Added
- **Draft Response (P0)** — after decoding, click "Draft a Response" to get AI-generated professional reply addressing the underlying concerns with clear boundaries. Unique feature no competitor offers
- **Copy Response** button — one-click copy of the drafted response to clipboard
- **Tone Intensity Slider** — 5-level slider (Very Diplomatic / Diplomatic / Neutral / Direct / Very Direct) controls the tone of the drafted response
- Separate loading state for draft generation
- Draft card with purple accent theme to distinguish from decode results

### Changed
- Context selector values now also influence the draft response prompt
- SW cache bumped to `v5.0`

### Preserved
- All v4.x features intact (decode, copy, examples, history, share card, feedback)
- All AI integration (OpenAI, Gemini, Anthropic) untouched for decode
- Draft uses same BYOK infrastructure and provider as decode

## v4.0 (2026-03-29) — Functional Upgrade

### Added
- **Copy Decoded** button — one-click copy of decoded translation to clipboard (`navigator.clipboard.writeText` with `execCommand` fallback)
- **Pre-built Examples** — 5 clickable example boss messages ("Per my last email...", "Let's take this offline", etc.) that auto-fill textarea and trigger decode if API key is configured
- **Tone color coding** — new CSS classes for `professional` (blue), `threatening` (dark red), and `encouraging` (green) tone tags
- **Accessibility improvements**:
  - `aria-label` on Copy Decoded button and Clear History button
  - `role="alert"` preserved on inline errors
  - `aria-live="polite"` preserved on result content

### Changed
- SW cache bumped to `v4.0`

### Preserved
- All AI integration (OpenAI, Gemini, Anthropic) untouched
- All v3.0 visual design intact
- Existing tone tag colors unchanged

## v3.0 (2026-03-29) — Premium Visual Redesign

### Added
- **Google Fonts (Inter)** — connected Inter (400-900) for modern typography
- **Gradient backgrounds** — deep multi-stop body gradient with ambient radial light effects
- **Glassmorphism** — all cards, dropdowns, modals use translucent backgrounds with `backdrop-filter: blur(16px)` and soft inner-glow borders
- **Micro-animations**:
  - `fadeInUp` on section transitions and staggered result blocks
  - `scaleIn` on dropdown/modal opens
  - `gradientShift` on header title, loading text, accent elements
  - Shimmer sweep on decode button hover and loading bar
  - `loadPulse` on loading subtitle
  - Gear icon rotates on hover; modal close rotates on hover
  - History items and flag items slide right on hover
  - Playbook steps indent on hover; tone tags scale on hover
- **Soft shadows with colored glow** — accent-colored box-shadow on buttons, severity-colored text-shadow on toxicity score, glow on tone tags and history badges
- **Gradient border accents** — animated gradient top-line on result blocks; gradient `border-image` on decoded translations
- **Loading screen enhanced** — gradient text fill with glow filter, background shimmer on loading bar
- **Custom scrollbar** — thin translucent scrollbar
- **Selection color** — accent-tinted text selection

### Changed
- **Visual hierarchy** — header title uses animated gradient text with drop-shadow; result headings use gradient text; bolder font weights throughout
- **Border radius** — 10px to 14px globally
- **Input fields** — darker backgrounds, focus ring (`box-shadow: 0 0 0 3px`) with outer glow
- **Select** — custom SVG chevron, removed native appearance
- **Buttons** — gradient backgrounds with colored shadows; hover lift effect
- **Decode button** — shimmer sweep on hover via `::before`
- **Toxicity bar** — thicker (12px), shimmer animation overlay on fill
- **Toxicity score** — larger (2.8rem, weight 900), text-shadow glow
- **Provider tabs** — active uses gradient + lift + shadow
- **Flag items** — inner glow by severity, slide-right hover
- **Playbook numbers** — gradient background with shadow

### Mobile (v3)
- Tighter container padding, smaller header, reduced card padding
- Adjusted toxicity score and textarea sizes
- Full-width result action buttons

### Preserved
- All JavaScript untouched
- All `id`, `data-*`, `aria-*` attributes unchanged
- `prefers-reduced-motion` respected
- Dark mode structure maintained

## v2.0 (2026-03-29)
### Added
- **Decode History** — past decodes saved locally (max 15), last 5 displayed
- **Side-by-Side Translation** — desktop 2-column view (original vs decoded)
- Arrow key navigation in settings dropdown

### Fixed
- Privacy: history stores only message snippets, not full text

### Changed
- SW cache bumped

## v1.1 (2026-03-29)
### Fixed
- parseFloat NaN → error instead of silent fallback to 1
- AbortController 30s timeout on all 3 AI providers
- --text-muted contrast improved (#6b7280 → #8b929a)
- "How to get a key?" collapsible with provider-specific instructions
- Dropdown focus management
- alert() replaced with inline error (role="alert")
- Share button disabled during generation

## v1.0 (2026-03-29)
- Initial release
- BYOK: OpenAI, Google Gemini, Anthropic
- Decoded Translation with tone tags
- Red Flags detection (max 6)
- Toxicity Score (1-10, 5-category breakdown)
- Playbook (actionable steps)
- Canvas share card (privacy: no original text)
- Decrypt animation
- PWA (service worker + manifest)
