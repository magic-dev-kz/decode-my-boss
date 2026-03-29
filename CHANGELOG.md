# Decode My Boss — Changelog

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
