---
name: meal-prep
description: >-
  Plan weekly meals, generate consolidated grocery/shopping lists, and maintain a
  growing personal recipe database as markdown. Use this whenever the user wants to
  plan what to cook this week, do batch/meal prep, build a dinner lineup, make a
  shopping or grocery list from recipes, add or save a recipe, mark a recipe as a
  favorite, or set up dietary preferences — even if they don't say the words "meal
  prep." Triggers include "what should I cook this week," "plan my dinners," "make
  me a grocery list," "save this recipe," "I liked that curry," and similar.
---

# Meal Prep

Help the user plan meals, shop for them, and accumulate a recipe collection that
gets better over time. The whole point is that this isn't one-shot: the user's
profile and recipe database persist, so each week's plan should feel personalized
and the database should grow with recipes they actually liked.

There are three things you do, often together:
1. **Plan a week** of meals tailored to the user's profile.
2. **Generate a grocery list** consolidated across the planned recipes.
3. **Maintain the recipe database** — add new recipes, update usage, mark favorites.

## First: find (or set up) the data directory

The user's data lives *outside* this skill so the skill stays portable and the data
survives. Before doing anything, resolve the data root in this order and stop at the
first hit:

1. **An explicit path in the request** ("my recipes are in ~/cooking") — use it.
2. **The saved config** at `~/.config/meal-prep/data_dir` — if this file exists, its
   contents are the path. Use it.
3. **Common locations** — check, in order, `./meal-prep`, `~/meal-prep`,
   `~/Documents/meal-prep`. If one exists and looks like a data dir (has a
   `profile.md` or `recipes/`), use it.
4. **Nothing found** — ask the user where they'd like it, defaulting to `~/meal-prep`.
   Create the folder, then **save the chosen absolute path** to
   `~/.config/meal-prep/data_dir` (create `~/.config/meal-prep/` if needed) so future
   runs find it automatically. Don't skip saving — that's what makes "ask once" work.

The data directory has this shape:

```
<data-dir>/
├── profile.md          # dietary preferences & constraints (read every run)
├── recipes/
│   ├── index.md        # table of all recipes, ⭐ marks favorites
│   ├── sheet-pan-chicken-fajitas.md
│   └── ...
└── plans/
    └── 2026-06-21-week.md   # dated weekly plans you generate
```

If `profile.md` doesn't exist yet, that's expected on first use — see "Setting up the
profile" below. Create `recipes/` and `plans/` lazily when you first write to them.

## Read the profile before planning

`profile.md` carries the constraints that make a plan personal: household size,
allergies, dislikes, diet style, time budget, equipment, and favorite cuisines. Read
it at the start of every planning task and let it shape every choice. Two rules are
non-negotiable because they're about trust and safety:

- **Allergies are hard avoids.** Never put an allergen in a plan, even as "optional."
- **Servings come from the profile** unless the user overrides for this plan.

Everything else (dislikes, cuisines, time budget) is a strong preference — honor it,
but it's fine to stretch occasionally for variety and call it out when you do.

### First run: walk the user through setup

When there's no `profile.md` yet, **don't drop a pre-filled template and don't guess
defaults.** Walk the user through a short, friendly setup conversation and build the
profile from *their* answers. The recipe database also starts **empty** — never seed
it with example or dummy recipes. It fills up only with recipes the user actually
plans or saves, so everything in it is real to them.

Run the setup as a brief interview. Lead with the few questions that most change the
output, fold in anything they already told you in their request (don't re-ask it),
and keep it to a couple of rounds — this should feel like a quick chat, not a form:

- **Who's eating?** → servings per meal (and any kids, since that affects spice/effort)
- **Any allergies?** → hard avoids, treated as never-include
- **Diet style and hard dislikes?** → vegetarian/vegan/omnivore/etc., plus foods to skip
- **Weeknight cooking time?** → the active-time budget that bounds recipe choices
- **Anything you love?** → favorite cuisines, go-to proteins (nice-to-have, not required)

Use `assets/profile-template.md` only as the *structure* for the file you write once
you have their answers — fill it with what they actually said, leaving blanks where
they didn't care rather than inventing values. Confirm the saved profile back to them
in a sentence, and mention they can edit `profile.md` anytime. You'll refine it
naturally over weeks as you learn their tastes — the profile is a living file, not a
one-time questionnaire.

## Building a weekly plan

**Default scope: 7 dinners.** It's the most common meal-prep need and easy to reason
about. Override freely when asked — "dinners and lunches," "5 days," "lunches are
leftovers," "two big-batch meals stretched across the week." When leftovers are in
play, plan a couple of larger-batch dishes and assign their extra servings to lunches
rather than inventing separate lunch recipes.

### Where recipes come from

Balance fresh variety against the user's proven favorites:

1. **Read `recipes/index.md`** to see what's already in the database.
2. **Favor liked recipes.** Recipes marked `liked: true` (⭐ in the index) are ones
   the user has endorsed — work a couple into each plan so the week feels familiar,
   unless they ask for "all new." This is the payoff of the database growing.
3. **Generate new recipes** for the remaining slots to keep things interesting and to
   fit the week's constraints. Generated recipes get **saved into the database** (see
   below) so today's good idea is reusable next month.
4. **Avoid monotony.** Don't repeat a protein or cuisine back-to-back unless the
   profile says they like that. Spread cooking effort across the week — don't stack
   three ambitious dishes on weeknights.

### Save new recipes as you plan

Every brand-new recipe you put in a plan becomes a file in `recipes/`:

- **Filename:** kebab-case of the name, e.g. `miso-glazed-salmon.md`.
- **Format:** follow `assets/recipe-template.md` exactly — frontmatter
  (`name, servings, time_minutes, tags, liked, times_used, last_used, source`) then
  Ingredients / Steps / Prep & storage / Notes.
- **Ingredients must be scalable.** Write every ingredient as `qty unit ingredient`
  (e.g. `1.5 lb chicken thighs`, `2 cup jasmine rice`). This is what lets you combine
  them into an accurate grocery list and rescale servings later. Avoid vague lines
  like "some garlic."

For any recipe used in a plan (new or existing), bump `times_used` and set
`last_used` to the plan's date, then update `index.md`.

### Plan output format

Write the plan to `plans/<YYYY-MM-DD>-week.md` (date = the week's start, or today if
unspecified) and also summarize it in the chat. Use this structure:

```markdown
# Week of <Mon DD, YYYY>

_Profile: <servings> servings · <one-line note on constraints applied>_

## The plan
| Day | Meal | Recipe | Time | Notes |
|-----|------|--------|------|-------|
| Mon | Dinner | [Sheet-Pan Chicken Fajitas](../recipes/sheet-pan-chicken-fajitas.md) ⭐ | 30 min | makes extra for Tue lunch |
| ... |

## Prep & batch notes
- <what to cook together / ahead, in what order, to save time>
- <storage reminders for anything batched>

## Grocery list
<consolidated list — see next section>
```

Mark favorites with ⭐ in the table so the user sees their endorsed recipes recurring.

## Generating the grocery list

The grocery list is the part that saves the user real effort, so make it shopping-
ready, not just a dump of recipe ingredients:

- **Consolidate across recipes.** If three recipes need onions, add up the quantity
  into one line ("4 onions"), don't list it three times. Combine compatible units
  thoughtfully (e.g. convert to a sensible common unit; if two amounts genuinely
  can't be combined, list both rather than guessing wrong).
- **Group by store section** so the user shops aisle by aisle. Default sections:
  **Produce, Protein, Dairy & eggs, Pantry & dry goods, Frozen, Other**. Respect a
  custom section order if the profile specifies one.
- **Assume staples on hand, but flag them.** Things like salt, pepper, oil, and
  common spices usually don't need buying — list them under a short "Check you have"
  line rather than the main list, so the user can glance and confirm.

Format:

```markdown
### Produce
- [ ] 4 onions
- [ ] 2 bell peppers
### Protein
- [ ] 2 lb chicken thighs
### Pantry & dry goods
- [ ] 2 cup jasmine rice

_Check you have: olive oil, salt, pepper, cumin, soy sauce._
```

Use checkbox bullets (`- [ ]`) — it's a list people tick off while shopping.

## Liking and managing recipes

The database earns its keep when the user tells you what worked. Handle these cleanly:

- **"I liked the chicken curry" / "favorite that" / "we loved Monday's dinner"** →
  find the recipe file, set `liked: true` in its frontmatter, and add a ⭐ next to it
  in `index.md`. Confirm briefly. Liked recipes get preference in future plans.
- **"I didn't like X" / "don't make that again"** → set `liked: false` and add a short
  note in the recipe's Notes (or a `disliked: true` flag) so you steer clear of it
  going forward. Don't delete it unless they ask.
- **"Save this recipe: …" / pasted recipe or a link** → parse it into the template,
  save it to `recipes/`, and add it to the index. If it's a URL, fetch and extract the
  real ingredients and steps rather than paraphrasing from the name.
- **"What's in my recipe database?" / "show me my chicken recipes"** → read
  `index.md` and filter by tag/keyword.

### index.md format

Keep `recipes/index.md` as a scannable table, regenerating or appending as recipes
change:

```markdown
# Recipe Index

| Recipe | Tags | Time | ⭐ | Used |
|--------|------|------|----|----|
| [Sheet-Pan Chicken Fajitas](sheet-pan-chicken-fajitas.md) | chicken, mexican, weeknight | 30 min | ⭐ | 3 |
| [Miso-Glazed Salmon](miso-glazed-salmon.md) | salmon, japanese | 25 min |  | 1 |
```

## Keeping it human

This is someone's actual dinner, not a database exercise. A few habits make the
output feel thoughtful rather than mechanical:

- **Lead with the plan, not the process.** The user wants to know what they're eating
  and what to buy. Show that first; mention file paths briefly at the end.
- **Explain trade-offs you made**, in a sentence — "kept Thursday light since it's a
  weeknight," "reused your favorite chili so the week isn't all-new."
- **Ask before assuming on the big stuff** (allergies, household size) but don't
  interrogate — sensible defaults beat a long questionnaire.
