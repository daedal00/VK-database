# Production SQL Readiness Assignments

MySQL 8 as default; add Postgres equivalents where noted. Focus on correctness, safety, and observability over syntax perfection. Keep artifacts in version control (DDL, queries, explain plans, notes).

## 0) Base Dataset
- Create a reusable seed that fits the ticket resale domain (users, venues, events, listings, orders, order_items). Target: ~50 venues, ~500 events, ~50k listings, ~200k order_items with realistic timestamps and prices.
- Deliverables: `sql/seeds/seed.sql` (or `.sql.gz`) plus a short README on how to load into MySQL (and a Postgres note: switch `AUTO_INCREMENT` to `GENERATED ALWAYS AS IDENTITY`).

## 1) Schema Hardening
- Add NOT NULL, CHECK constraints (MySQL: use CHECK with sql_mode=strict or triggers; Postgres: native CHECK), unique keys, and FK ON DELETE/UPDATE choices. Add a status enum-like constraint for listings/orders.
- Deliverables: migration script adding constraints; a note explaining each constraint and expected failure cases.

## 2) Query Accuracy + Performance
- Take 5 real questions (e.g., active listings per event, revenue per venue per day, buyer inactivity, average ticket price trend, top sellers). Write baseline queries, then optimize them.
- Measure with `EXPLAIN FORMAT=JSON` (MySQL) or `EXPLAIN (ANALYZE, BUFFERS)` (Postgres). Show plan deltas after changes.
- Deliverables: before/after SQL, explain outputs, and a 3–5 bullet summary of what improved and why.

## 3) Window Functions & Analytics
- Implement: running revenue per event over time, DENSE_RANK top sellers per week, LAG-based price change alerts, sessionization (group events by 30-minute gaps), retention cohorts (signup vs purchase week).
- Deliverables: SQL snippets and sample result snippets (first 10 rows) proving correctness; note any MySQL vs Postgres syntax differences (e.g., MySQL requires window functions; no DISTINCT ON).

## 4) Index Design Lab
- Design 4–6 indexes: covering index for active listings by event ordered by price, compound index for orders by buyer and created_at, partial/filtered index (Postgres) for active listings, functional index (Postgres) on lower(email), and a JSON/expression index if applicable.
- For MySQL, compare compound index orderings (event_id,status,listed_price) vs (status,event_id); show when index-only scans occur. For Postgres, include a partial index example and show planner picking it.
- Deliverables: DDL for each index, explain plans showing usage, and a short table of index size vs speed tradeoff.

## 5) Transactions, Isolation, and Deadlocks
- Script two concurrent sessions that update the same listings/orders in different order to force a deadlock; capture error and resolution.
- Show the difference between READ COMMITTED and REPEATABLE READ for a phantom-read scenario (MySQL default is REPEATABLE READ; Postgres default is READ COMMITTED). Document mitigation (consistent ordering, SELECT ... FOR UPDATE SKIP LOCKED, retry logic).
- Deliverables: reproducible scripts (or psql/mysql shell transcripts), plus a mitigation checklist.

## 6) Safe Upserts and Backfills
- Implement idempotent upserts for a derived table (e.g., daily_event_revenue) using `INSERT ... ON DUPLICATE KEY UPDATE` (MySQL) and `INSERT ... ON CONFLICT ... DO UPDATE` (Postgres).
- Backfill a new nullable column (e.g., listing_fee_pct) without long locks: add column nullable, backfill in batches ordered by PK with LIMIT + sleep, then add NOT NULL + default.
- Deliverables: migration SQL, backfill script, and notes on lock behavior observed (include `performance_schema.events_statements_history_long` or Postgres `pg_locks` snapshot if available).

## 7) Partitioning / Sharding Simulation
- Partition a large events table by day or month (MySQL RANGE; Postgres PARTITION BY RANGE). Load data and verify partition pruning via EXPLAIN.
- Compare query times for recent vs old partitions; show maintenance tasks (detach/drop old partitions) and impact on inserts.
- Deliverables: partition DDL, EXPLAIN showing pruning, timings comparison, and a brief ops playbook for managing partitions.

## 8) Observability & Ops
- Build monitoring queries: top 20 slow queries from `performance_schema.events_statements_summary_by_digest` (MySQL) or `pg_stat_statements` (Postgres); table/index bloat check (Postgres: `pg_stat_all_tables`; MySQL: INFORMATION_SCHEMA sizes); lock wait diagnostics.
- Create a lightweight health dashboard query pack (connection count, replication lag if available, longest running queries).
- Deliverables: a `.sql` file of diagnostic queries and example outputs, plus a short “what to page on” note.

## 9) Data Quality & Tests
- Write assertions: no duplicate emails, FK consistency, price > 0, currency in allowed set, event start_time < end_time. Add failing test cases to prove constraints fire.
- Add row-count and aggregate reconciliations (e.g., sum of order_items matches orders total if you add a total column).
- Deliverables: test SQL with expected pass/fail results and a checklist for pre-deploy data validation.

## 10) Mini Capstone
- Design and run an experiment: choose a suspected slow query, hypothesize a fix (index, rewrite, cache), apply it, measure before/after, and summarize impact and risks.
- Deliverables: 1-page writeup with hypothesis, change, measurements, and rollback plan.
