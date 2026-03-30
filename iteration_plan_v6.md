# Decode My Boss v6 — Iteration Plan

**Дата:** 2026-03-29
**Текущий статус:** v5.0, score 24/30, live на GitHub Pages
**Контекст:** Deferred API setup, Draft Response (уникальная фича), glass border fix (0.08->0.12) уже сделаны

---

## ПЛАНЁРКА — Высказывания агентов

### Саня (продакт)

1. **Пользователь и боль четко определены** — работник, который получает мутные сообщения от руководства и не понимает, что от него реально хотят. Это страх + фрустрация. Продукт попадает точно в нерв. Но мы должны думать о retention: человек использует раз, получает результат — зачем возвращаться?

2. **Draft Response — наше конкурентное преимущество.** Ни один "tone decoder" не дает готовый ответ. Это то, что делает продукт незаменимым: не просто диагноз, а action. Нужно сделать Draft Response более заметным в UI — сейчас он спрятан внизу, после всех результатов.

3. **Tone Slider — мощная фича, но её ценность неочевидна.** Пользователь не понимает, зачем крутить ползунок ДО анализа. Нужен tooltip или микро-copy: "Как жестко вам ответить?"

4. **Context selector ("Corporate Email", "Slack") — правильный ход,** но не хватает "1-on-1 Meeting Notes" и "LinkedIn Message" — два самых тревожных контекста.

5. **Куда двигать:** "Decode My Team" — серия декодеров. Но пока надо дожать core product до 28+ у Leo.

---

### Скай (дизайн)

1. **Courier New в 2026 — архаика.** `--mono: 'Courier New', Courier, monospace` (строка 59). Нужно подключить JetBrains Mono через Google Fonts. Это единственный шрифт, который одновременно читается, выглядит "хакерски" и не кричит "1998". Это Leo-рекомендация номер один.

2. **8+ разных box-shadow — хаос.** Сейчас в CSS: `0 8px 32px rgba(0,0,0,.3)`, `0 12px 40px rgba(0,0,0,.5)`, `0 4px 16px`, `0 4px 20px`, `0 4px 24px`, `0 6px 28px`, `0 2px 12px`, `0 20px 60px`. Нужно свести к 4 уровням через CSS-переменные: `--shadow-sm`, `--shadow-md`, `--shadow-lg`, `--shadow-xl`. Это делает систему предсказуемой.

3. **Toxicity score без анимации — упущенная возможность.** Число 7.2 должно "набираться" от 0 до 7.2 за 1.2 секунды (countUp). Это wow-moment, который пользователь запомнит и покажет коллегам. Визуальная анимация bar уже есть, но число статично.

4. **Mobile padding тесно.** `.container{padding:16px 20px}` на мобилке становится `12px 14px` — карточки прижаты к краям. Нужно минимум `16px` на мобилках, а `20px 24px` на десктопе.

5. **Первое впечатление:** пустой textarea с placeholder "Paste your boss's message here..." — рабочий паттерн. Но example chips недостаточно заметны. Стоит добавить один "featured example" прямо в placeholder или сделать пульсирующую подсказку на первом визите.

---

### Хаус (аналитик)

1. **Весь проект в одном HTML-файле (~1300+ строк).** Это монолит. Для v6 пока терпимо, но CSS (~960 строк), HTML (~160 строк) и JS (~340+ строк) в одном файле — это технический долг. Пока не рефакторим, но фиксируем.

2. **parseResult() (строка 1425) — единственная точка отказа.** Если AI возвращает кривой JSON, мы падаем. Есть базовая валидация (toxicity_score), но нет fallback для malformed translation array. Если `translation[0].original` undefined — рендер молча сломается.

3. **Нет debounce на decode button.** Двойной клик = два API-вызова = двойная трата денег пользователя. Нужен простой lock: `decodeBtn.disabled = true` во время запроса (частично есть через showSection, но кнопка не disabled явно).

4. **History хранит только snippet (80 chars), не full result.** Это by design (privacy disclaimer в строке 1752), но это значит повторный decode = повторный API-вызов. Стоит хранить full result в sessionStorage (не localStorage) с TTL 1 час.

5. **AbortController timeout 30s** (строки 1375, 1390, 1410) — для Gemini нормально, для OpenAI может быть tight на gpt-4o-mini при длинных сообщениях. Рекомендация: 45s или адаптивный timeout от длины input.

---

### Нэш (QA)

1. **Edge case: пустой red_flags с непустым toxicity_score > 7.** AI может вернуть `red_flags: []` при score 8 — UI покажет "No red flags detected" зеленой галочкой рядом с красным 8.0. Нужна валидация: если score > 6 и flags пусто — добавить предупреждение "AI не выявил конкретных флагов, но общий тон тревожный".

2. **Accessibility: toxicity bar не имеет aria-label.** Цветовая шкала (зеленый-желтый-красный) без текстовой альтернативы — незрячий пользователь не получит информацию. Нужен `role="progressbar"` с `aria-valuenow`, `aria-valuemin`, `aria-valuemax` и `aria-label`.

3. **Share canvas font hardcoded.** В `generateShareCard()` (строка 1530+) используются системные шрифты. На Linux share-картинка будет выглядеть иначе. Некритично, но стоит знать.

4. **XSS через tone tag.** `escapeHtml(t.tone)` используется в имени класса `tone-${tone}` — если AI вернет tone с пробелами или спецсимволами, это сломает CSS-класс. Нужна санитизация: `.replace(/[^a-z_]/g, '')`.

5. **Feedback widget (строка 1105) — кнопки не делают ничего кроме UI.** Нет backend, нет analytics event. Feedback теряется. Если нет плана на analytics — убрать виджет, он создает ложные ожидания.

---

### Игорь (маркетинг)

1. **Headline "AI-powered corporate message decoder" — generic.** Каждый второй AI-tool так себя описывает. Нужно: **"Paste what your boss wrote. See what they really meant."** — это action-oriented, конкретно, и создает интригу.

2. **OG-image ссылается на `og-image.png`** (строка 14) — нужно убедиться что он существует и выглядит shareable. Формула: крупный toxicity score + "What they said vs what they meant" + брендинг. Должен работать как standalone.

3. **Вирусный потенциал — share card.** Canvas-генерация есть, но формат 2400x1260 — нестандарт для Stories (1080x1920). Добавить вертикальный формат для Instagram/TikTok share.

4. **Meta description хорошая** ("Paste a message from your boss. Get the real meaning, red flags, toxicity score, and what to do next.") — но можно добавить urgency: "Free. Private. No sign-up."

5. **Название "Decode My Boss" — отличное.** Запоминается, googlится, домен-friendly. Не менять.

---

### Олег (консультант)

1. **Честный ответ: продукт уже в запускаемом состоянии.** v5 с score 24/30 — это solid. Дальнейшая итерация дает diminishing returns, если не менять что-то принципиальное.

2. **Но 24/30 vs 28/30 — это разница между "нормальным tool" и "тем, что хочется показать коллегам".** Рекомендации Leo (шрифт, тени, анимация, mobile) — это именно те polish-моменты, которые отделяют хорошее от отличного.

3. **Draft Response — единственная причина итерировать.** Это то, что делает продукт utility, а не novelty. Нужно максимально раскрыть эту фичу: сделать её visually prominent, добавить "Copy & paste into email" flow.

4. **Риски дальнейшей итерации:** over-engineering монолитного HTML-файла. Каждая следующая фича увеличивает хрупкость. Рекомендация: v6 — финальный polish по Leo, v7 — разделение на файлы (если будет v7).

5. **Вердикт: итерировать v6 как "finish line".** Закрыть все 4 оставшихся Leo-пункта + 2-3 quick wins из QA/аналитики. Не добавлять новых фич. Цель: 28/30.

---

## РЕШЕНИЕ — Plan v6

### Приоритет 1: Leo-рекомендации (цель: +4 балла)

| # | Задача | Сложность | Влияние |
|---|--------|-----------|---------|
| 1 | **JetBrains Mono** — подключить через Google Fonts, заменить `--mono` | 5 min | Высокое — убирает визуальный архаизм |
| 2 | **Консолидация shadows** — 4 переменные (`--shadow-sm/md/lg/xl`), заменить все 8+ инлайн-теней | 20 min | Среднее — чистит систему, снижает визуальный шум |
| 3 | **Toxicity score countUp анимация** — число набирается от 0 до N за 1.2s | 15 min | Высокое — wow-moment |
| 4 | **Mobile padding** — `container` 16px min, карточки 20px на mobile | 5 min | Среднее — убирает ощущение тесноты |

### Приоритет 2: Quick wins из планёрки

| # | Задача | Сложность | Влияние |
|---|--------|-----------|---------|
| 5 | **Decode button disable во время запроса** — явный lock | 5 min | Высокое — предотвращает двойные API-вызовы |
| 6 | **Toxicity bar aria-attributes** — `role="progressbar"`, `aria-valuenow` | 5 min | Среднее — accessibility fix |
| 7 | **Tone tag sanitization** — `.replace(/[^a-z_]/g, '')` для CSS-класса | 2 min | Низкое — но предотвращает edge case XSS |
| 8 | **Subtitle update** — "AI-powered corporate message decoder" -> "Paste what your boss wrote. See what they really meant." | 2 min | Среднее — маркетинговый punch |

### НЕ делаем в v6 (backlog)

- Разделение на файлы (CSS/JS отдельно) — v7 если будет
- Vertical share card format — nice-to-have, не core
- SessionStorage для full results — privacy concern
- Новые context options — не влияет на score
- Feedback widget backend — нет analytics infrastructure
- Featured example / onboarding pulse — over-engineering

### Порядок выполнения

```
1. JetBrains Mono (шрифт)
2. Shadow consolidation (CSS vars)
3. Mobile padding fix
4. Toxicity countUp animation
5. Button disable lock
6. Aria-attributes на toxicity bar
7. Tone tag sanitization
8. Subtitle copy update
```

### Критерий успеха

Leo re-audit: **28/30** (текущий 24, цель +4 через 4 Leo-пункта + polish)

---

*OpenClaw team, 2026-03-29*
