---
layout: post
title:  "Object Storage - Storing Is Cheap, Getting It Out Is the Bill"
date:   2026-04-18 10:15:00 +0530
comments: True
categories: [Software, Cloud]
excerpt_separator: "<!--more-->"
---

Somewhere in our account there is an S3 bucket that nobody owns and everybody uses. Our Spring Boot services dump daily statements, audit exports, and application logs into it, versioning switched on years ago by someone careful, lifecycle rules configured by nobody. Last quarter I opened Storage Lens out of curiosity and just stared. One prefix held three years of files nobody had opened since the week they were written, all sitting in Standard storage, all billed every single month.

Storing data feels free because the per-GB number is tiny. Keeping it there forever, and then moving it around carelessly, is where the real money goes.

<!--more-->

### The bucket nobody owns

This is how it happens in every team I have seen. The application needs a place to put exports, so someone creates a bucket. Versioning gets turned on because an overwrite scare during a release taught everyone a lesson. Then time passes. Each deploy writes more objects. Old versions pile up silently because nobody wrote a rule to expire noncurrent versions. A few large uploads fail halfway through the night and leave multipart parts behind, which S3 bills you for until you explicitly abort them.

Nobody notices because the line item says something vague like "S3 storage" and everyone splits the bill in their head. Storage Lens finally put a number on our guilty prefix, and suddenly the cleanup had a business case.

### Lifecycle rules are the janitor

S3 lifecycle rules are the closest thing to hiring a janitor for the bucket. You describe what should happen to objects as they age, and S3 does it without anyone running a cron job. The transitions I actually use are boring and that is the point:

- Current versions move out of Standard after 30 days. If I do not know the access pattern, Intelligent-Tiering is my default, because it watches access per object and slides things down to cheaper tiers by itself.
- Noncurrent versions expire after 30 to 90 days, depending on how paranoid compliance is feeling that quarter.
- Incomplete multipart uploads get aborted after 7 days. This one rule alone deleted a shocking amount of garbage.
- Anything with a one-year retention need goes to a Glacier tier, and I pick the tier based on how fast we would ever need it back.

Intelligent-Tiering deserves a word because it quietly got good. It was launched back in 2018, and the shape of it today is simple: objects untouched for 30 days drop to the Infrequent tier at roughly 40% off, untouched for 90 days drop further to Archive Instant at roughly two-thirds off, and access at any point pulls them back to Frequent automatically. There are opt-in archive tiers below that for data you can wait hours for, saving up to 95%, at the cost of needing a restore first. There are no retrieval fees inside it and no minimum durations, just a small per-object monitoring charge, which is why tiny objects under 128 KB are excluded from auto-tiering. One gotcha worth knowing: since September 2024, lifecycle rules by default skip objects smaller than 128 KB entirely, so a bucket full of tiny JSON files will not tier the way you expect. And Glacier-side minimums still bite, 90 days for Flexible Retrieval and 180 for Deep Archive, plus a small per-object overhead, so do not archive something you will delete next month.

### Getting data out costs

Here is the part that stung us. Storage was only half the bill. Uploads into S3 are free and same-region reads from our services are free, so everyone assumed movement was free. Then our reporting job, running in a different region because of course it was, started pulling terabytes of exports across regions every night at roughly two cents a gigabyte. Separately, a customer-facing download feature served files over presigned URLs straight from the bucket, which meant every download was internet egress: the first 100 GB a month free, then around nine cents a gigabyte for the first 10 TB, with volume tiers below that.

None of those numbers is large on its own. Multiply by nightly jobs and a few thousand downloads and it stops being background noise. Our fix was unglamorous:

- Moved the reporting readers into the same region as the bucket. Same-region traffic between S3 and our services costs nothing.
- Put CloudFront in front of the downloads. Origin fetches from S3 are free, edge delivery is cheaper than direct S3 egress, and cache hits mean repeat downloads never touch the bucket at all.
- Added a gateway VPC endpoint for same-region S3 access so the traffic stopped hairpinning through the NAT gateway, which charges its own processing fee per gigabyte.
- Compressed the exports before writing them. Fewer bytes stored, fewer bytes moved, smaller everything.

### What I would do on day one

If I could start the bucket over, and I mostly get to say this because I cannot, the checklist fits on a sticky note. Versioning on, obviously. Lifecycle on from the first week: tier down at 30 days, expire noncurrent versions, abort failed uploads, archive what compliance wants kept. Alarms on the storage metric per prefix so growth pages someone before finance does. And one rule I now treat as law: no cross-region reads by default and no direct-to-internet downloads at scale without a CDN in front.

Object storage is the cheapest shelf I have ever rented. But the shelf charges rent forever, and the loading dock charges by the truck. Write the lifecycle rules early, keep the readers close to the data, and check which prefix is actually eating your budget before you blame the cloud for being expensive.
