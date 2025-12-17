# Database Exercise

We use relational (SQL) databases in our day-to-day work on a ticket resale marketplace.

This exercise is meant to see how you **learn and apply** basic database concepts in a domain close to what we do: events, venues, tickets, and resale orders.

You may use any resources you like (documentation, tutorials, AI, etc.), **as long as you understand what you submit**. We may ask you to walk through your solution later.

Please keep your total time investment reasonable. We do not expect a perfect solution.

---

## Domain

Imagine a simplified ticket resale platform with the following concepts:

- **Users** - people using the platform. A user can buy and/or resell tickets.
- **Venues** - places where events happen (stadiums, arenas, theatres).
- **Events** - concerts, games, shows, etc.  
  Each event:
  - happens at a single venue  
  - has a start time
- **Ticket listings** - tickets that users list for resale on our platform.  
  Each listing:
  - belongs to one event  
  - is created by one seller (a user)  
  - has seat information (for example, section/row/seat or a free-text field)  
  - has a listed price  
  - has a timestamp when it was listed
- **Orders** - when a buyer purchases one or more tickets from listings.  
  Each order:
  - belongs to one buyer (a user)  
  - has a creation timestamp
- **Order items** - individual line items inside an order.  
  Each order item:
  - belongs to one order  
  - references a ticket listing  
  - has a quantity (usually 1, but model it as a number)  
  - stores the **unit price at the time of sale** (resale price can change over time, so we want to keep history)

You do **not** need to model every possible real-world edge case. Keep it simple but sensible.

---

## What to deliver

Create a file named **`solution.md`** in the root of your repository with three sections:

1. `## Part 1 - Schema`  
2. `## Part 2 - Queries`  
3. `## Part 3 - Thought process & explanation`

Use fenced code blocks (for example, <code>```sql</code>) for any SQL.

If you use AI at any point, also create a separate file named **`ai.md`** (details below).

---

## Part 1 - Schema (relational / SQL)

Design a relational schema for this ticket resale system.

In `solution.md`, under `## Part 1 - Schema`, provide:

1. **SQL `CREATE TABLE` statements** for the tables you think are needed.  
   At a minimum, you should model:

   - `users`  
   - `venues`  
   - `events`  
   - `ticket_listings`  
   - `orders`  
   - `order_items`  

   For each table, include:

   - a primary key  
   - foreign keys  
   - reasonable column types (you can assume PostgreSQL or MySQL; exact flavor doesn’t matter)

2. At least **two indexes** you would create, with a short comment for each on why you chose it.

   Example format (not necessarily the exact index you should use):

   ```sql
   -- Example: to speed up queries for events at a venue by start time
   CREATE INDEX idx_events_venue_start_time
     ON events(venue_id, start_time);
   ```

You can include the schema in a single big code block or several smaller ones.

---

## Part 2 - Queries

Under `## Part 2 - Queries` in `solution.md`, write SQL queries (PostgreSQL- or MySQL-style syntax is fine) for the following:

### 1. Active listings for an event

Given an `event_id`, return all **currently active** ticket listings for that event, ordered from lowest price to highest.

For each listing, return at least:

- listing id  
- seller user id  
- seat information  
- listed price  

You can assume there is some way to know if a listing is “active” vs sold/expired (for example, a `status` column). Design something reasonable and use it in your query.

### 2. Top events by resale revenue (last 30 days)

Return the **top 5 events by total resale revenue** over the last 30 days.

For each event, return:

- event id  
- event name  
- associated venue name  
- total resale revenue in that period  

Total resale revenue should be based on the prices stored in order items (for example, `quantity * unit_price_at_sale`), not the current listing price.

### 3. Inactive buyers

Return all users who:

- have **bought** at least one ticket in the past (at any time), but  
- have **not placed any orders** in the last 90 days.

You can choose the exact format of the result (for example, just user id and name).

If you’re unsure about perfect SQL syntax, focus on getting the overall structure right (joins, filters, grouping). You can add comments if something is approximate.

---

## Part 3 - Thought process & explanation

Under `## Part 3 - Thought process & explanation` in `solution.md`, write short answers (bullet points or short paragraphs are fine) to the following:

### 1. How you approached the problem

- How did you break the task down?  
- In what order did you work on the parts (schema vs queries vs explanations), and why?  
- What trade-offs did you consider when deciding how detailed to make the schema?

### 2. Learning process

- What did you have to look up or learn for this task?  
- How did you approach learning it (docs used, tutorials, tools, etc.)?  
- Did your understanding change while working on the task? If so, how?

### 3. Schema decisions

- Why did you choose this table structure?  
- Why did you choose the indexes you defined?  
- Is there anything you *didn’t* model that you would add in a real system?

### 4. Query reasoning

Pick **one** of your queries (any of the three) and explain in words how it works:

- Which tables does it read from?  
- How do the joins work?  
- How do the filters and ordering/grouping work?

Please keep this section honest and straightforward. We care more about seeing how you think than about getting everything “perfect”.

---

## AI usage (`ai.md`)

If you use AI (for example, ChatGPT, Copilot, etc.) at any point in this exercise, create a separate file named **`ai.md`** in the root of your repository.

In `ai.md`, briefly describe:

1. **Where you used AI**
   - Which parts of the task (schema, queries, explanations, something else)?  
   - Paste or summarize any prompts that were especially helpful.

2. **How you evaluated AI output**
   - How did you check that the AI’s suggestions made sense?  
   - Did you change or simplify anything the AI produced? Why?

3. **What was actually yours**
   - Which parts do you feel you fully understand and could reproduce without AI?  
   - Are there any parts you’re still not fully confident about?

We are not testing whether you can avoid AI. We are testing whether you stay in control of the solution and learn from the tools you use.

If you do **not** use any AI tools for this exercise, you can skip `ai.md`.

---

## Submission

1. Create a new Git repository (GitHub, GitLab, etc.).  
2. Add a file named **`solution.md`** in the root containing:
   - Part 1 - Schema  
   - Part 2 - Queries  
   - Part 3 - Thought process & explanation  
3. If you used AI, add a file named **`ai.md`** in the root describing how you used it.  
4. Commit and push your work.  
5. Send us the **repository link** by email.
