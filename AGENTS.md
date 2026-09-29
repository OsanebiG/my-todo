# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

A single-page todo app with per-task notes, deployed as a static site on Vercel (free Hobby plan). The whole app lives in `index.html`: HTML, CSS, and JavaScript in one file. There is no build step, no framework, and no package manager.

## Structure

```
index.html   The entire app
AGENTS.md    This file
```

## Features

- Add, complete, edit, and delete tasks
- Per-task notes, priority (none, low, medium, high), and due date
- Overdue and due-today highlighting, with a banner count
- Search across titles and notes
- Filters: All, Open, Done
- Sort: manual order, due date, priority
- Drag to reorder (manual order, All filter, no search only); Move up and Move down buttons for touch screens
- "My list" and "Shared list" tabs (Shared appears only when the Claude runtime is available)

## Conventions

- Keep everything in `index.html` unless the user asks to split it up.
- Plain JavaScript (ES5-style `var` and functions). No frameworks, no bundlers, no npm dependencies.
- Colors are CSS variables on `:root`, with dark mode under `prefers-color-scheme` and a `data-theme` override. Use the variables rather than hardcoded colors.
- Keep the viewport meta tag and safe-area padding on `:root`.
- Text is set with `textContent`, never `innerHTML` with user data.
- Only external resource: Google Fonts (Bricolage Grotesque), with a system font fallback.

## Storage model

Tasks are objects: `{id, text, done, note, pri, due, order}`. `due` is `YYYY-MM-DD` or an empty string.

There are two store implementations with the same interface (`sub`, `add`, `upd`, `del`):

- `localStore()` saves to `localStorage` under the key `todo-notes-v2`. This is the default and the only store that works on Vercel.
- `dbStore(col)` uses the `window.claude` runtime (`db` and `user` capabilities). It only works when the page is published as a Claude artifact. It powers cross-device sync and the shared list.

On Vercel, `window.claude` does not exist, so the page silently stays on `localStore`. Do not remove that fallback. Keep every call to `claude.use(...)` inside the guarded init block at the bottom of the script.

## Adding sync and sharing outside Claude

To get sync and shared lists on Vercel, write a third store with the same interface backed by a hosted service such as Supabase or Firebase, and add sign-in. Keep `localStore` as the signed-out fallback. Never commit API secrets. Public anon keys are fine, but access rules must be enforced on the backend.

## Testing

No test suite. To check changes:

1. Open `index.html` in a browser, or run `npx serve .`
2. Add a task, open Details, set priority, due date, and notes, then reload and confirm they persist.
3. Check search, all three filters, all three sorts, and reordering.
4. Check dark mode and a narrow (phone-width) window.
5. Confirm there are no errors in the browser console.

## Deployment

Vercel deploys automatically on every push to `main`. Framework preset is "Other", with no build command and no output directory. `index.html` must stay at the repository root.

## Do not

- Add a build step or dependencies without asking.
- Use `innerHTML` with user-provided text.
- Store secrets or personal data in the repo.
- Break the `localStorage` fallback.
