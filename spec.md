# Decode My Boss — Product Specification

## One-liner
Вставь сообщение от начальника — получи перевод на человеческий язык + red flags + toxicity score.

---

## WifiCard

- **Персона:** Даша, 29 лет. Тимлид в среднем корпорейте. Получила от директора письмо: "As per our earlier discussion, I'd like to revisit the timeline. Let me know your thoughts." Не понимает — это угроза, дедлайн сдвинулся, или просто вежливость?
- **Функция:** AI-декодирование корпоративных сообщений: перевод подтекста, выявление red flags, оценка токсичности.
- **Враг:** Подруга в чате ("да забей, наверное ничего страшного") + собственная тревожность (перечитывает 5 раз). Angry Email Translator делает ОБРАТНОЕ — помогает писать, а не читать.
- **Эмоция:** Тревога ("что он имел в виду?!") -> ясность ("аааа, это power move, а не просьба").
- **Результат:** Четкое понимание: что сказано, что подразумевается, какие red flags, и что делать дальше.
- **Скриншот-момент:** Карточка: "Toxicity: 7.2/10 | Decoded: Your boss is blaming you for his missed deadline" — шлет коллегам.

---

## Persona

**Даша, 29, Москва.** Тимлид в IT-компании на 200 человек. Хороший специалист, но плохо читает корпоративную политику. Директор пишет "Let's align on priorities" — Даша не знает: это нормальный синк или её проект убивают. Подруга говорит "не парься", но у подруги другой босс. Даша хочет не параноить, а ПОНИМАТЬ. Перечитывает сообщения по 5 раз, гуглит "per my last email meaning". Идеальный пользователь: получила сообщение, вставила, за 5 секунд получила расшифровку + план действий.

---

## User Flow

```
1. Открыл страницу
2. Видит: текстовое поле + выпадающий выбор контекста (Corporate Email / Slack / Teams / HR Letter / Performance Review / Other)
3. Вставляет сообщение от начальника/HR/коллеги
4. Нажимает "DECODE"
5. Ждёт 3-5 сек (анимация: текст "расшифровывается" как секретное послание)
6. Получает результат в 4 блоках:
   a. TRANSLATION — оригинал слева, "настоящий смысл" справа (side-by-side или построчно)
   b. RED FLAGS — обнаруженные паттерны: passive aggression, gaslighting, blame shifting, etc.
   c. TOXICITY SCORE — оценка 1-10 с прогресс-баром
   d. PLAYBOOK — 2-3 конкретных совета что делать дальше
7. Кнопка "Share Decode" — генерирует карточку-картинку для соцсетей/чатов
```

---

## Технический стек

### Архитектура: Single-file HTML + BYOK

```
[Browser] ---> [AI API напрямую]
   |
   index.html (весь код)
```

### Детали

- **Файл:** один `index.html`, self-contained
- **AI Provider:** пользователь вставляет свой API key (поддержка: OpenAI, Anthropic, Google Gemini)
- **API key хранение:** localStorage (с предупреждением что key остаётся в браузере)
- **Стили:** встроенный CSS, тёмная тема, корпоративно-детективная палитра
- **JS:** vanilla, никаких фреймворков
- **Шрифт:** system fonts (без загрузки)
- **Share:** Canvas API для генерации PNG-карточки

### API интеграция

```javascript
const PROVIDERS = {
  openai: { url: 'https://api.openai.com/v1/chat/completions', model: 'gpt-4o-mini' },
  anthropic: { url: 'https://api.anthropic.com/v1/messages', model: 'claude-sonnet-4-20250514' },
  google: { url: 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent', model: 'gemini-2.0-flash' }
};
```

---

## System Prompt (ядро продукта)

```
You are "The Corporate Decoder" — a sharp organizational psychologist who reads between the lines of workplace communication.

You've analyzed 50,000 corporate messages. You know every tactic: passive aggression disguised as politeness, blame shifting framed as "collaboration", gaslighting wrapped in "feedback". You are NOT a comedian — you are an incisive analyst. Calm, precise, clinical.

Context: {context} (Corporate Email / Slack / Teams / HR Letter / Performance Review / Other)

Analyze this message and respond in EXACTLY this JSON format:
{
  "translation": [
    {
      "original": "As per our earlier discussion",
      "decoded": "I told you once and you didn't do it",
      "tone": "passive_aggressive"
    },
    {
      "original": "I'd like to revisit the timeline",
      "decoded": "The deadline you set is unacceptable to me",
      "tone": "power_move"
    }
  ],
  "red_flags": [
    {
      "type": "passive_aggression",
      "evidence": "As per our earlier discussion",
      "severity": "medium",
      "explanation": "References a past conversation to imply you failed to act on it"
    }
  ],
  "toxicity_score": 6.5,
  "toxicity_breakdown": {
    "passive_aggression": 7,
    "blame_shifting": 5,
    "gaslighting": 3,
    "power_dynamics": 8,
    "exclusion_signals": 2
  },
  "verdict": "Your boss is reasserting control over the project timeline without directly saying your plan was wrong.",
  "playbook": [
    "Reply with a specific revised timeline, don't just acknowledge",
    "CC the project sponsor to create visibility",
    "Document this exchange — it may be referenced later as 'we agreed'"
  ]
}

Rules:
- Be analytical, NOT dramatic. You are a decoder, not a roaster.
- Don't catastrophize. Score honestly: 1-3 = normal/benign, 4-6 = yellow flags, 7-8 = concerning, 9-10 = toxic/hostile.
- A score of 1-2 is common. Not every message is toxic. If it's genuinely fine, say so.
- Red flag types: passive_aggression, gaslighting, blame_shifting, power_move, exclusion_signal, false_urgency, weaponized_positivity, plausible_deniability, scope_creep_trap, credit_theft.
- Tone per phrase: neutral, passive_aggressive, power_move, blame_shift, gaslighting, genuine, ambiguous.
- Playbook must be ACTIONABLE — specific reply strategies, not generic "communicate better".
- If the message is genuinely benign, say "No red flags detected. This appears to be a straightforward message." Score 1-2.
- Adapt language to input: Russian message = Russian response. English = English.
- Translation must cover EVERY sentence/phrase — don't skip parts.
- Max 6 red flags. Quality > quantity.
```

---

## Visual Identity

### Палитра — "Corporate Intelligence"

НЕ fire/roast. Тон: секретный аналитический отчет, разведывательная записка.

- **Background:** `#0f1419` (почти чёрный, "classified document")
- **Card background:** `#1a2332` (тёмно-синий, "corporate night")
- **Primary accent:** `#4a9eff` (холодный синий, "analytical clarity")
- **Warning:** `#f59e0b` (янтарный, для medium red flags)
- **Danger:** `#ef4444` (красный, для high-severity flags)
- **Safe:** `#22c55e` (зелёный, для low toxicity / benign)
- **Text:** `#e5e7eb`
- **Muted text:** `#6b7280`
- **Toxicity bar:** градиент зелёный (1) -> жёлтый (5) -> красный (10)

### Настроение

Шрифт: monospace для "decoded" текста (как рассекреченный документ). Тонкие линии. Никаких эмоджи в UI. Строгость. Иконки: лупа, замок, щит. Анимация загрузки: текст "расшифровывается" посимвольно.

---

## UI/UX Specification

### Layout

```
+------------------------------------------+
|     DECODE MY BOSS                       |
|     "What they say vs what they mean"     |
+------------------------------------------+
|                                          |
|  [Context: Corporate Email v]            |
|                                          |
|  +------------------------------------+  |
|  |                                    |  |
|  |   Paste the message here...        |  |
|  |   (textarea, min 5 rows)           |  |
|  |                                    |  |
|  +------------------------------------+  |
|                                          |
|  [ DECODE ]  (синяя кнопка)              |
|                                          |
+------------------------------------------+

--- РЕЗУЛЬТАТ ---

+------------------------------------------+
|  TOXICITY: 6.5/10  [========----]        |
|  "Your boss is reasserting control"      |
+------------------------------------------+
|  DECODED TRANSLATION                     |
|  Original          |  What they mean     |
|  "As per our..."   | "I told you once.." |
|  "Revisit the..."  | "Your plan is..."   |
+------------------------------------------+
|  RED FLAGS                               |
|  [!] Passive Aggression (7/10)           |
|  [!] Power Move (8/10)                   |
+------------------------------------------+
|  PLAYBOOK                                |
|  1. Reply with specific timeline...      |
|  2. CC the project sponsor...            |
|  3. Document this exchange...            |
+------------------------------------------+
|  [Share Decode]  [Decode Another]        |
+------------------------------------------+
```

### Первый запуск (нет API key)

```
+------------------------------------------+
|  To decode messages, you need an         |
|  AI API key.                             |
|                                          |
|  [OpenAI]  [Anthropic]  [Google]         |
|                                          |
|  API Key: [__________________________]   |
|                                          |
|  Your key stays in your browser.         |
|  We never see it. [How to get a key?]    |
|                                          |
|  [Save & Start Decoding]                 |
+------------------------------------------+
```

---

## Share Card

```
+-------------------------------+
|  DECODE MY BOSS               |
|                               |
|  Toxicity: 6.5 / 10          |
|  ========================     |
|                               |
|  Red Flags:                   |
|  * Passive Aggression         |
|  * Power Move                 |
|                               |
|  "Your boss is reasserting    |
|   control over the timeline"  |
|                               |
|  decodemyboss.com             |
+-------------------------------+
```

Карточка НЕ содержит оригинальный текст сообщения (приватность). Только: score, найденные red flags, verdict, URL.

---

## Acceptance Criteria

### AC-1: Ввод сообщения
- [ ] Textarea принимает текст от 10 до 5,000 символов
- [ ] Dropdown контекста: Corporate Email, Slack, Teams, HR Letter, Performance Review, Other
- [ ] Кнопка "Decode" неактивна если текст < 10 символов
- [ ] При нажатии показывается анимация расшифровки

### AC-2: API Key Management
- [ ] Поддержка 3 провайдеров: OpenAI, Anthropic, Google Gemini
- [ ] Key сохраняется в localStorage
- [ ] При невалидном key показывается понятная ошибка
- [ ] Кнопка "Change key" в настройках
- [ ] Ссылка "How to get a key?" с инструкциями для каждого провайдера

### AC-3: Decoded Translation
- [ ] Каждое предложение/фраза оригинала отображается рядом с расшифровкой
- [ ] Тон каждой фразы помечен цветным тегом (neutral=серый, passive_aggressive=жёлтый, power_move=красный, genuine=зелёный)
- [ ] Если фраза genuinely benign — тег "genuine" зелёный

### AC-4: Red Flags
- [ ] Каждый red flag содержит: тип, цитата-доказательство из текста, severity (low/medium/high), объяснение
- [ ] Severity визуально отличается цветом (зелёный/жёлтый/красный)
- [ ] Максимум 6 red flags. Если 0 — показывается "No red flags detected"
- [ ] Типы red flags: passive_aggression, gaslighting, blame_shifting, power_move, exclusion_signal, false_urgency, weaponized_positivity, plausible_deniability, scope_creep_trap, credit_theft

### AC-5: Toxicity Score
- [ ] Score 1.0-10.0 отображается с одним десятичным знаком
- [ ] Прогресс-бар с градиентом зелёный->жёлтый->красный
- [ ] Breakdown по 5 категориям: passive_aggression, blame_shifting, gaslighting, power_dynamics, exclusion_signals
- [ ] Verdict — одно предложение под score

### AC-6: Playbook
- [ ] 2-3 конкретных совета что делать
- [ ] Советы actionable (конкретные действия, не "communicate better")
- [ ] Если сообщение benign — playbook содержит "No action needed. This is a straightforward message."

### AC-7: Share
- [ ] Кнопка "Share Decode" генерирует PNG карточку через Canvas API
- [ ] Карточка содержит: toxicity score, red flags (типы), verdict, URL продукта
- [ ] Карточка НЕ содержит оригинальный текст (privacy)
- [ ] Скачивается или открывается Web Share API (если поддерживается)

### AC-8: Responsive
- [ ] Работает на мобильном (360px+)
- [ ] Работает на десктопе (1024px+)
- [ ] Side-by-side translation на десктопе, stacked на мобильном

### AC-9: Honesty Guard
- [ ] Benign сообщения получают score 1-2 и "No red flags detected"
- [ ] AI НЕ выдумывает toxicity где её нет — промпт явно требует честности
- [ ] Если сообщение содержит < 2 предложений, показывается disclaimer: "Short messages are hard to decode — context matters"

### AC-10: Локализация и Privacy
- [ ] Если текст на русском — AI отвечает на русском
- [ ] Если на английском — на английском
- [ ] Никакие данные не отправляются никуда кроме выбранного AI API
- [ ] API key не покидает localStorage
- [ ] Нет analytics, трекеров, cookies (кроме localStorage)

---

## Что НЕ входит в v1

- История декодированных сообщений
- Декодирование цепочки писем (thread)
- Распознавание конкретного "стиля" босса (обучение на нескольких сообщениях)
- Генерация ответа (это другой продукт)
- Бэкенд / прокси
- Встроенный бесплатный режим без API key

---

## Расширяемость (v2+)

Серия **"Decode My ___"** — тот же движок, другой контекст:

| Продукт | Персона | Контекст |
|---------|---------|----------|
| **Decode My HR** | Сотрудник получил HR-письмо | "We're restructuring for efficiency" = увольнения. PIP-письма, "voluntary separation". |
| **Decode My Landlord** | Арендатор | "We're updating the building policy" = повышение аренды. Юридические уловки. |
| **Decode My Ex** | Человек после расставания | "I just want you to be happy" = манипуляция. Emotional red flags. |
| **Decode My Doctor** | Пациент | "We should run a few more tests" = что это значит на самом деле. Медицинский жаргон -> простой язык. |
| **Decode My Client** | Фрилансер | "We love it, but can we explore other directions?" = переделывай бесплатно. Scope creep detection. |
| **Decode My Teacher** | Студент/родитель | "Your child is very energetic" = что учитель НА САМОМ ДЕЛЕ говорит о вашем ребёнке. |

### v2 фичи для core продукта:
- **Thread Mode** — вставить всю переписку, AI анализирует динамику и эскалацию
- **Boss Profile** — после 5+ сообщений от одного человека AI строит профиль: communication style, favorite tactics, predictability
- **Reply Suggestions** — не просто "что делать", а конкретные шаблоны ответов (осторожно — не пересекаться с Angry Email Translator)
- **Toxicity Trend** — график toxicity score по времени (если пользователь сохраняет историю)

---

## Метрики успеха

- Пользователь получает decode за < 10 сек
- Benign сообщения честно получают 1-2/10 (AI не параноит)
- Red flags содержат конкретные цитаты из текста (не generic)
- Playbook содержит действия, которые можно выполнить сегодня
- Share карточка генерируется корректно, не содержит оригинал сообщения

---

## Конкурентный ландшафт (из разведки Молота)

| Инструмент | Что делает | Почему НЕ конкурент |
|-----------|-----------|---------------------|
| Angry Email Translator | Переводит ИЗ злого В вежливое | Обратное направление — помогает ОТПРАВЛЯТЬ, не ПОЛУЧАТЬ |
| LeveliU | Тон-полишер для своих сообщений | Опять — про исходящие, не входящие |
| ChatGPT/Claude напрямую | "Explain this email" | Generic, нет score, нет red flags taxonomy, нет share card, нет структуры |

**Ниша "декодирование входящих корпоративных сообщений" = абсолютно пуста.**
"Per my last email" — мем на миллиарды просмотров. Аудитория = каждый офисный работник на планете.

---

## Следующие шаги

1. Саня (продакт) -> передаёт спеку на дизайн
2. Дизайн -> code в single-file HTML
3. Audit -> fix -> re-audit
4. Деплой в sandbox

---

*Spec by Саня, 2026-03-29*
*Research by Молот, 2026-03-29 (score 29/30)*
