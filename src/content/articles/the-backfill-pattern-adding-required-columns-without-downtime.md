---
title: "The Backfill Pattern: Adding Required Columns Without Downtime"
date: 2026-07-22
---

Adding a required column to a table sounds like the simplest task in the world. Then you try it on a table that's already in production, with code writing to it every second, and you find out why it isn't.

There are two ways people usually try this, and both break.

Add the column as `NOT NULL` right away, and the migration fails outright, or it locks the table while it scans millions of existing rows that have no value to give it.

Add it as nullable instead and move on, and nothing enforces the constraint at all. Six months later half the rows are still null, and the column is "required" only in your head.

The way out is a boring four step dance.

## The four steps

1. **Create the column as nullable.** The migration is instant and safe. Nothing breaks, because nothing is required yet. Old code keeps writing rows without the field, and that's fine.
2. **Backfill.** Fill the column for all existing rows, in batches, in the background. Batches matter here: updating millions of rows in one statement can lock the table and hurt the same production traffic you were trying to protect.
3. **Start sending the field from the API.** Every new row now arrives with a value. From this point on, the data only gets more complete, never less.
4. **Run a second migration making the column `NOT NULL`.** By now every row has a value, the old ones from the backfill and the new ones from the API. The constraint applies cleanly, with no failures and no locks.

## Why the order matters

Each step only works because of the one before it. Enforce `NOT NULL` before the backfill, and the migration explodes on old rows. Backfill before the API sends the field, and new rows keep arriving empty while you chase your own tail. The sequence is the whole pattern, not just a checklist.

## The bigger idea

This shape shows up everywhere once you notice it: expand first, migrate the data, switch the writers, then contract. Renaming columns, splitting tables, moving data between services, it's always the same four beats.

I used to think senior engineers had some secret trick for risky migrations. Turns out the trick is mostly patience, applied in the right order.
