# PrepWise AI – Smart Study Planner

A lightweight, single-page study planner that turns your subject, topics, and exam date into a day-by-day schedule. No build step, no server required — open `index.html` in a browser and start planning.

---

## Features

- **Study form** — Subject, comma-separated topics, and exam date
- **Auto-generated schedule** — Topics spread across every day from today until the exam
- **Saved plans** — Stored in `localStorage` (key: `study_plans`); survives page refresh
- **Live date logic** — All dates read from the system clock via `lib/dateUtils.js`; nothing hardcoded
- **Smart plan list** — Upcoming exams first (soonest first), past exams greyed out with a **Past** badge
- **Mobile responsive** — Tailwind CSS layout works on phone and desktop
- **Cursor-ready** — `/* CURSOR PROMPT: ... */` comments mark where to plug in Groq, Supabase, or Next.js

---

## Project structure

```
website/
├── index.html          # Main app (form, plan display, saved plans)
├── lib/
│   └── dateUtils.js    # Single source of truth for all date logic
└── README.md
```

---

## Quick start

1. Open `index.html` in any modern browser (Chrome, Firefox, Edge, Safari).
2. Fill in **Subject**, **Topics** (comma-separated), and **Exam Date**.
3. Click **Generate Study Plan**.
4. Open **View Saved Plans** to see everything you've created.

> **Tip:** For the best experience, serve the folder with a local server instead of `file://`:
> ```bash
> npx serve .
> ```
> Then visit `http://localhost:3000`.

---

## How it works

### Plan generation

When you submit the form, the app:

1. Validates all fields and blocks past exam dates
2. Counts days until the exam using `getDaysUntil()`
3. Distributes your topics evenly across each study day
4. Adds a final review day on the exam date
5. Saves the plan to `localStorage`

Each day in the schedule includes:

| Field   | Example                          |
|---------|----------------------------------|
| `day`   | `1`, `2`, `3`…                   |
| `date`  | `2026-06-19`                     |
| `topic` | `Organic Chemistry – Alkanes`    |
| `tasks` | Read notes, practice questions…  |

### Date utilities (`lib/dateUtils.js`)

All date math lives in one file. **No other file should call `new Date()` directly.**

| Function | Returns | Example |
|----------|---------|---------|
| `getTodayISO()` | Today as `YYYY-MM-DD` (local timezone) | `"2026-06-19"` |
| `getDaysUntil(examDateISO)` | Whole days until exam | `5` (negative = passed) |
| `formatFriendlyDate(dateISO)` | Human-readable date | `"19 June 2026"` |
| `getCurrentMonthYear()` | Current month + year | `"June 2026"` |
| `buildAIContext(examDateISO)` | Prompt string for AI | See below |
| `getExamCountdownLabel(examDateISO)` | Card countdown text | `"3 days until exam"` |
| `sortPlansByExamDate(plans)` | Sorted plan array | Upcoming first |

**AI context string** (logged to the browser console on generate):

```
Today's date is 2026-06-19, currently June 2026. The exam is on 2026-06-25,
which is 6 days away. Build a day-by-day schedule starting from today.
```

This prevents an LLM from assuming a stale date from its training data.

---

## Saved plans behaviour

Every time you open **View Saved Plans**, the app recalculates from today's real date:

- **Upcoming** — Colourful card, live countdown (e.g. `"4 days until exam"`)
- **Today** — Shows `"Exam is today"`
- **Past** — Greyed out with a **Past** badge and `"Exam passed"`

Sorting: upcoming plans first (soonest exam on top), then past plans.

---

## Dev debug line

When running locally (`localhost`, `127.0.0.1`, or `file://`), a debug line appears at the bottom of the home page:

```
[DEV] getTodayISO() = 2026-06-19 · getCurrentMonthYear() = June 2026
Remove this debug line before final deployment.
```

Use this to confirm dates are live, not hardcoded.

**Remove before shipping:** delete the `#dev-debug` element in `index.html` and the `initDevDebug()` function.

---

## Upgrading to Next.js + Groq + Supabase

The app is structured to map cleanly to a full `prepwise-lite` Next.js project:

| Current file | Next.js equivalent |
|--------------|------------------|
| `index.html` (form section) | `components/StudyForm.tsx` |
| `index.html` (plan cards) | `components/PlanCard.tsx` |
| `index.html` (home page) | `app/page.tsx` |
| `index.html` (saved plans) | `app/plans/page.tsx` |
| `generatePlan()` | `app/api/generate/route.ts` |
| `lib/dateUtils.js` | `lib/dateUtils.ts` |
| `localStorage` | Supabase `study_plans` table |

### Supabase table SQL

```sql
CREATE TABLE study_plans (
  id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  subject    text NOT NULL,
  exam_date  date NOT NULL,
  schedule   jsonb NOT NULL,
  created_at timestamptz DEFAULT now()
);
```

### Environment variables (`.env.local`)

```env
GROQ_API_KEY=your_groq_api_key_here
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url_here
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key_here
```

Set `USE_LOCAL_GENERATOR = false` in `index.html` (or remove it in Next.js) and wire `generatePlan()` to `POST /api/generate` using the Groq model `llama-3.3-70b-versatile`.

---

## Deploying (static version)

Upload the `website/` folder to any static host:

- [Netlify Drop](https://app.netlify.com/drop)
- [GitHub Pages](https://pages.github.com/)
- [Vercel](https://vercel.com/) (static deploy)

No environment variables needed for the local planner version.

---

## Browser support

Works in all modern browsers that support:

- `localStorage`
- `crypto.randomUUID()`
- ES6+ JavaScript
- CSS Grid / Flexbox

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Exam date picker shows wrong min date | Hard-refresh the page; dates come from `getTodayISO()` on load |
| Plans disappeared | `localStorage` is per-browser — clearing site data removes plans |
| `dateUtils.js` not found | Keep `lib/dateUtils.js` in the same relative path as `index.html` |
| Debug line won't go away | Remove `initDevDebug()` before deploying to a public URL |

---

## License

Personal / educational project. Free to use and modify.
