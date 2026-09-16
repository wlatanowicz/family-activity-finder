# Backlog

Execution-ordered implementation backlog for the Family Activity Finder MVP,
derived from [`docs/discovery_summary.md`](../discovery_summary.md). Each
numbered file is one deliverable, full-stack task (backend + frontend + data,
where applicable) that can be built, reviewed, and shipped on its own.

## Ordering rationale

1. **01–02** replace the template's demo `items` app with the real `Activity`
   domain model and the first two screens (browse + detail), and introduce
   client-side routing.
2. **03–07** layer on the MVP filters called out in the discovery summary
   (indoor/outdoor + duration, age, location, "near me", weather), one
   filter/column per task so each migration and query stays small.
3. **08–09** add the engagement loop (favorites, community ratings) that the
   discovery summary lists as MVP scope, on top of the auth already scaffolded
   by the template.
4. **10–11** add child profiles and wire them into the age filter for the
   "decision engine, not a directory" personalization the discovery summary
   calls out as the opportunity.
5. **12** gives the team a way to create/edit/delete activities without
   touching the database directly — the direct answer to the "cold start
   (content)" challenge named in the discovery summary.
6. **13–16** close the gaps between this backlog and
   [`IDEAS.md`](IDEAS.md) found during validation: a map view (13), a
   gender-suitability filter alongside the existing age filter (14), photo
   attachments on reviews alongside the existing text comment (15), and
   linking a saved child profile to a review for anonymized display context
   (16). Each depends on the earlier task it extends (06, 04, 09, and 09+10
   respectively) and is otherwise independent of the others.

## Status tracking

Every numbered task file carries a `status` field in its YAML frontmatter:
`todo` (not started) or `done` (shipped). Update it as part of the PR that
completes the task — this file's table is the human-readable index, the
frontmatter is the source of truth.

## Deferred (not in this backlog yet)

These are listed under "Future Features" in the discovery summary and need
their own discovery/spike before they can be broken into implementation
tasks — they depend on decisions (which AI model/provider, which event
sources to scrape/license, which booking APIs to integrate) that aren't
resolved yet:

- AI / personalized recommendations beyond simple age matching
- Event aggregation from external sources
- Ticket booking integrations
- Weekend planner (multi-activity itineraries)

"Rainy-day suggestions" *is* included — see task 07 — because it turned out
to be a thin, cheap layer on top of the weather-suitability filter rather
than a separate system.

## Task list

| # | Task | Depends on |
|---|------|------------|
| 01 | Replace demo app with Activity model and browse page | — |
| 02 | Client-side routing and activity detail page | 01 |
| 03 | Indoor/outdoor and duration filters | 01–02 |
| 04 | Age range filter | 01–02 |
| 05 | City location filter | 01–02 |
| 06 | "Near me" geolocation distance sort | 01–02 |
| 07 | Weather-suitability filter and rainy-day shortcut | 01–02 |
| 08 | Favorites | 01–02, auth (template) |
| 09 | Community ratings and reviews | 01–02, auth (template) |
| 10 | Child profiles | auth (template) |
| 11 | Personalized "for my child" recommendations | 04, 10 |
| 12 | Admin activity management (create/edit/delete) | 03–07 |
| 13 | Activity map view | 01–02, 06 |
| 14 | Gender-suitability filter | 01–02, 04 |
| 15 | Review photo attachments | 09 |
| 16 | Anonymized child context on reviews | 09, 10 |
