---
layout: post
title:  "Using EF Core Migrations and SQLite: Alter Column"
date:   2019-03-04 21:30:59 -0500
categories: ef-core sqlite
---

I've been utilizing [EF Core](https://docs.microsoft.com/en-us/ef/core/) on a side-project ([Arkive](https://arkive.io/)) with a [SQLite](https://www.sqlite.org/index.html) backend. So far, everything has been running smoothly. 

I am able to create an initial EF Core-based migration schema and the database is created automatically with `context.Database.Migrate()` which is fantastic for my project. I ran into an issue recently however where I noticed a `DateTime` property on one of my EF Models needed to be made nullable. No problem! I'll simply update the property and tell EF Core to generate a new migration.

So I thought...

Here's what EF Core generated for me. One table column is no longer `NOT NULL`.

![Bad Migration code](/img/posts/2019-03-04/before.png "This won't work")

Turns out, SQLite has a few limitations and EF Core isn't going to complain when you generate the migration to update the table. When you go to apply the migration however, you'll get `SQLite does not support this migration operation`.

![Migration Error](/img/posts/2019-03-04/error.png "Not something I expected")

What to do?

SQLite does not allow `ALTER` which is the ultimate problem here.

The easiest thing I found that you can do is simply modify the migration and execute hand-written SQL directly. The trick with SQLite, it turns out, is to create a new table with the correct schema (ex. MyTableTemp), copy the data over from the old table, then delete the original table (MyTable) which has the old schema. Finally, just rename the temporary table to the same as the original (MyTable). 

It's not the prettiest in the world, but it'll allow you to keep moving along. You just need to copy the CREATE TABLE EF Core code from the previous migration (or recreate it if it's composed of many migrations).

![Migration Error](/img/posts/2019-03-04/after.png "All set!")

The bigger question may arise later however if EF Core + EF Migrations + SQLite are even worth it. Time will tell for me, I'm a bit hesistent though, but we'll see. I've explored DBUp but ran into a few problems with the migrations.

The above example is simple; however, should you have Foreign keys, you'll likely have to drop/recreate several tables to reach the desired result which could be painful.
