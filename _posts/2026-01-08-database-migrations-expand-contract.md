---
layout: post
title:  "Database Migrations - Expand Before You Contract"
date:   2026-01-08 09:40:00 +0530
comments: True
categories: [Software, Architecture]
excerpt_separator: "<!--more-->"
---

The first planning week of the year is when everybody is brave. Roadmaps are fresh, the release calendar has that new-calendar smell, and someone always puts "clean up the user table" on the board like it is a half-day task. Renaming a column sounds trivial. It is one line of SQL. What could it possibly do?

Quite a lot, as I learned on an otherwise ordinary release train. The rename itself took milliseconds. The rolling deploy around it took down nothing and broke something subtler: for about ten minutes, old pods and new pods were serving traffic against the same database, disagreeing about what a column was called. Nobody paged. The errors just quietly piled up in the logs until support started forwarding screenshots. That morning taught me the rule I have followed ever since: never change the schema in one step when two versions of your code will ever meet it.

<!--more-->

### The rename that bit us

The table was nothing exotic, a users table behind a Spring Boot service, the kind every shop has. We wanted `phone` split into `phone_country_code` and `phone_number`, because international onboarding had finally forced the issue. The migration renamed the column in place, the code went out in the same release, and on paper the two matched perfectly.

In practice, Kubernetes does rolling updates. At any moment during that deploy, version N and version N+1 were both alive, both talking to the same database. The new code asked for columns that did not exist yet on some pods; the old code asked for a column that no longer existed on others. Each side was correct on its own and wrong together. In a bank, where every release already passes a change board that asks "what is the rollback plan", my answer that day was embarrassingly thin: roll everything back at once and hope the data survived the round trip.

The fix was not a better tool. It was an older idea I should have applied from the start: expand before you contract.

### Expand, then contract

The pattern goes by expand-contract, sometimes parallel change, and it is the foundation of every zero-downtime migration guide worth reading. Instead of jumping from the old schema to the new one in a single leap, you live in a middle state for a while where both shapes work side by side. That compatibility window is what lets old and new code run against the same database without disagreeing.

For our phone split, done properly, it looks like this:

- **Expand** — add the new columns. Old code ignores them completely, which is exactly the point. `ALTER TABLE users ADD COLUMN phone_country_code ...` breaks nothing.
- **Dual-write** — deploy code that writes to both the old column and the new ones, but still reads from the old one. Every row created from this moment on exists in both shapes.
- **Backfill** — copy existing data into the new columns in batches, with pauses, until every old row has a new-shape twin.
- **Switch reads** — deploy code that reads from the new columns. The old column is now write-only legacy.
- **Contract** — days later, not minutes, stop the dual writes and drop the old column.

Five steps instead of one, and every step is independently deployable and independently reversible. That last property is the real prize. At no point are you one bad deploy away from an incident you cannot roll back from, which is precisely the question the change board keeps asking.

### Backfills deserve their own deploy

The step teams most often skip is the backfill, usually by running it inside the migration itself as one giant `UPDATE`. On staging with ten thousand rows it takes seconds. On production with a hundred million it holds locks, lags replicas, and turns your deploy window into a war room. I now treat backfills as batch jobs, not migrations: loop over small batches, sleep briefly between them, run them from a script or a scheduled job, and stop when a batch affects zero rows.

Two database-specific habits belong in the same paragraph. On Postgres, build new indexes with `CREATE INDEX CONCURRENTLY` and add constraints as `NOT VALID` first, validating later, so the heavy scan never blocks writes; always set a short lock timeout so a stuck migration fails loudly instead of queueing behind traffic. On MySQL, large alters go through an online schema-change tool like gh-ost or pt-online-schema-change rather than a bare `ALTER TABLE`. None of this is new technology. All of it predates my career by years, which is exactly why there is no excuse left for skipping it. Versioned migration tools like Flyway and Liquibase orchestrate the steps, but no tool removes the need to think in phases. A good overview of the whole pattern, with the lock-level details, is this schema migration guide that I keep bookmarked: https://www.jusdb.com/blog/schema-versioning-and-migration-strategies-for-scalable-databases — and the tool homepages https://flywaydb.org and https://www.liquibase.org document the mechanics.

### The checklist I run now

Before any migration leaves my laptop, it has to survive four questions:

- Is every step backward-compatible with the currently deployed code, not just the code I am about to deploy?
- Do schema changes and code changes ship in separate deploys whenever the change is breaking?
- Is there a backup taken before the contract phase, since `DROP COLUMN` is the one step that cannot be undone with a redeploy?
- Can I state the rollback for each phase in one sentence? If not, the migration is still one big step wearing a trench coat.

If I had to choose one habit that separates calm teams from brave ones, it would be this: treat the database schema as a public API with two live consumers during every deploy, because that is literally what it is. Expand first, migrate patiently, contract only when nobody is looking at the old shape anymore. Renaming a column is still one line of SQL. Shipping it safely is five small boring steps, and boring is everything I want my release trains to be this year.
