## Part 1 - Schema

### SQL `CREATE TABLE` statements:

```mysql
CREATE TABLE users (
    user_id INT NOT NULL AUTO_INCREMENT,
    full_name VARCHAR(50) NOT NULL,
    email VARCHAR(50) NOT NULL UNIQUE,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (user_id)
);
```

```mysql
CREATE TABLE venues (
    venue_id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    city VARCHAR(50) NOT NULL,
    state VARCHAR(50),
    country VARCHAR(2),
    capacity INT,
    type VARCHAR(25),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (venue_id)
);
```

```mysql
CREATE TABLE events (
    event_id INT NOT NULL AUTO_INCREMENT,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NULL,
    name VARCHAR(100) NOT NULL,
    venue_id INT NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (event_id),
    FOREIGN KEY (venue_id) REFERENCES venues(venue_id)
);
```

```mysql
CREATE TABLE ticket_listings (
    ticket_listing_id INT NOT NULL AUTO_INCREMENT,
    event_id INT NOT NULL,
    seller_user_id INT NOT NULL,
    seat_info VARCHAR(25),
    listed_price DECIMAL(10, 2) NOT NULL,
    status VARCHAR(15) NOT NULL,
    currency VARCHAR(3) NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (ticket_listing_id),
    FOREIGN KEY (event_id) REFERENCES events(event_id),
    FOREIGN KEY (seller_user_id) REFERENCES users(user_id),
    CHECK (listed_price > 0),
    CHECK (status IN ('active', 'sold', 'cancelled', 'expired'))
);
```

```mysql
CREATE TABLE orders (
    order_id INT NOT NULL AUTO_INCREMENT,
    buyer_user_id INT NOT NULL,
    status VARCHAR(15) NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (order_id),
    FOREIGN KEY (buyer_user_id) REFERENCES users(user_id),
    CHECK (status IN ('pending', 'paid', 'cancelled'))
);
```

```mysql
CREATE TABLE order_items (
    order_item_id INT NOT NULL AUTO_INCREMENT,
    order_id INT NOT NULL,
    ticket_listing_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price_at_sale DECIMAL(10, 2) NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (order_item_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (ticket_listing_id) REFERENCES ticket_listings(ticket_listing_id),
    CHECK (quantity > 0)
);
```

### Indexes:

```mysql
-- improve efficiency vs full-table lookups for all tickets for event listed from cheapest price
CREATE INDEX active_event_listings
ON ticket_listings (event_id, status, listed_price)
```

```mysql
-- quick lookup for a user's orders which very likely to be frequent on a ticket purchasing site
CREATE INDEX user_orders
ON orders(buyer_user_id, created_at)
```

```mysql
-- instead of scanning all listings, can get venue_id for specific city, good for searching tickets in my area/city
CREATE INDEX city_events
ON venues(city)
```

```mysql
-- supports joins and aggregates on order items by order
CREATE INDEX order_items_order_id
ON order_items(order_id);
```

```mysql
-- supports joins from order_items back to listings
CREATE INDEX order_items_listing_id
ON order_items(ticket_listing_id);
```

## Part 2 - Queries

### active listings for an event

```mysql
SELECT ticket_listing_id, seller_user_id, seat_info, listed_price
FROM ticket_listings
WHERE event_id = ? -- bind the target event_id
  AND status = 'active'
ORDER BY listed_price ASC;
```

### top 5 events by total resale revenue over last 30 days

```mysql
SELECT e.event_id,
       e.name AS event_name,
       v.name AS venue_name,
       SUM(oi.unit_price_at_sale * oi.quantity) AS revenue

FROM order_items oi

INNER JOIN orders o ON o.order_id = oi.order_id
INNER JOIN ticket_listings tl ON oi.ticket_listing_id = tl.ticket_listing_id
INNER JOIN events e ON e.event_id = tl.event_id
INNER JOIN venues v ON v.venue_id = e.venue_id

WHERE o.created_at > NOW() - INTERVAL 30 DAY
  AND o.status = 'paid'
  AND tl.status IN ('sold', 'expired') -- sold/consumed listings only

GROUP BY e.event_id, e.name, v.name

ORDER BY revenue DESC
LIMIT 5;
```

### inactive buyers

```mysql
SELECT users.user_id, users.full_name
FROM users
WHERE
EXISTS (SELECT 1
        FROM orders
        WHERE orders.buyer_user_id=users.user_id)
AND NOT EXISTS (SELECT 1
                FROM orders
                WHERE orders.buyer_user_id=users.user_id
                AND orders.created_at >= NOW() - INTERVAL 90 DAY)
```

## Part 3 - Thought process & explanation

In order to demonstrate learning ability and understanding, no ai was used.

### 1. How I approached the problem

At first, I reasoned that I needed a refresher and lesson of SQL. I referred to SQLBolt for a quick brief on SQL queries with hands on exercises, as well as remembered about 3NF that I learned in class so went into functional dependencies as it would be needed for table creation.

I first though outside of the scope of SQL, what information would I want to store within the tables:

- users:

  - user_id (primary key + unique)
  - full_name
  - email (unique)
  - created_at
  - updated_at

- venues

  - venue_id (primary key + unique)
  - name
  - city
  - state
  - country (ISO 3166-1 alpha-2 so VARCHAR(2))
  - capacity (optional?)
  - type (stadium, arena, theatre)
  - created_at
  - updated_at

- events

  - event_id (primary key + unique)
  - start_time (timestamp, normalize?, NON-NULL)
  - end_time (optional?)
  - name (event name)
  - venue_id (foreign key, have to know where event is so NON-NULL)
  - created_at
  - updated_at

- ticket_listings

  - ticket_listing_id (primary key)
  - event_id (foreign key)
  - seller_user_id (foreign key->user_id, NON-NULL)
  - seat_info (section/row/seat)
  - listed_price (NON-NULL, >= 0)
  - status (active, sold, cancelled, expired)
  - currency (ISO 4217 standard currency)
  - created_at
  - updated_at

- orders

  - order_id (primary key)
  - buyer_user_id (foreign key->user_id, NON-NULL)
  - status (pending, paid, cancelled)
  - created_at (NON-NULL)
  - updated_at

- order_items
  - order_item_id (primary key)
  - order_id (foreign key, NON-NULL)
  - ticket_listing_id (foreign key->listing_id, NON-NULL)
  - quantity (>0)
  - unit_price_at_sale (NON-NULL, Decimal(10, 2))
  - created_at
  - updated_at

### 2. Learning process

#### Approach:

I utilized online resources and went through lessons, exercises, as well as articles (etc: https://support.microsoft.com/en-us/office/database-design-basics-eb2159cf-1e30-401a-8084-bd4f9c9ca1f5). I wanted to display my learning capability without using ai at all.

I first went through some SQL lessons through W3 schools just to see where I was having gaps and needed to learn. After concluding that I had difficulty recalling just about anything, I went online to find resources to have a more in-depth learning where I found SQLBolt with lessons + hands-on exercises. I also utilized good old stack overflow for code snippets and question answerings requiring me to reason and apply examples instead of being hand-fed answers. Rather than going in blindly, I googled and searched creating database tables/queries with production in mind. As well as utilized SQL cheat sheets and examples that I could find and refer to. Recognizing that with my lack of depth of knowledge on SQL, creating tables/queries for production was currently beyond my capabilities.

The ordering of my approach was first to design the system even without SQL fields or syntax in mind, to gauge what I need to store, how they'd relate, and what the system would look like. Afterwards, I went into creating the tables based off the initial brainstorming I did of the design. Afterwards I glanced at the queries and picked a few scenarios and attributes that could be utilized for the indexes. While I was doing all this, in between I jotted down notes of my approach/understanding and explanations so I could freshly write them all down in the moment to display my reasoning and learning.

Given that this assignment was to showcase my learning ability, I didn't go too in depth/detailed as I did with the coding take-home assignment, given that my initial expertise level vastly differed and rather focused on learning the fundamentals and showcasing my learnings.

After creating the tables and running them in my local mysql + phpadmin docker container. I moved onto learning the best practices for creating Indexes as well as considering what we would want to look up like high-volume queries. Some that initially came to mind are:

- Searching tickets for an event based on lowest price
- Displaying all orders for a user
- Checking events by city

Afterwards, I recognized that these indexes can be used for a more efficient queries, especially for part 2. But although after looking into what queries I could make, I was unsure how I could actually utilize these queries. Do I have to specify the indexes that I created themselves? But that in itself doesn't seem efficient in production as a developer would have to check every single index created to see whether they can reference and use.

While doing query 2, I was quickly guided toward the JOIN statement for looking up attributes across various tables and the various types.

I was initially stumped on query 2 but through a process of running my queries via my MySQL docker container and debugging the errors, I was guided into GROUP BY's with aggregation's like SUM and finally got it to at leasty be syntactically correct.

#### Understanding/Application:

Initially unfamiliar with mysql syntax, referring to the documentation and examples I found online I created the tables with primary key at the table level (as I found that it's a better practice since in the future it's easier to find primary keys as well as potentially store multiple) and types (datetime reference: https://stackoverflow.com/questions/50603953/how-to-add-created-at-and-updated-at-columns).

Using the design guides I considered what exactly I wanted to store before going into normalization rules. I also found it standard practice within enterprises to store created_at + updated_at fields as well which I promptly added.

Understanding that indexes improve from full-table search O(n) to a binary search O(logn), I created multi-column indexes for potential queries like looking up tickets for an event. I also learned that SQL uses leftmost prefix as index for multiple comumn indexes which allowed me to optimize further.

Upon a deeper dive of how indexes are used, it was a lot simpler than I anticipated as they are automatically utilized in queries when possible for a more efficient lookup! I also learned that when creating indexes they are bound to a single table (i.e., I thought of how I could just simply select commonly used attributes from a variety of tables) but quickly learned that indexes were built for within a single table.

Given that my baseline knowledge of SQL was extremely rusty, I definitely learned a lot through this real hands on assignment. Thus, my understandings of how to use SQL in the first place developed and utilzing it with production in mind for more optimized and efficient tables, queries, and indexes changed for the better.

As well, for the more complex query 2, I had initially chosen FROM events as I naively figured that since I'm querying events I start from the events table. But after going through a loop of failures in my test db and some debugging from online resources, I realized that my FROM statement should stem from order_items as ultimately I'm querying revenue resale data.

### 3. Schema decisions

#### Table structure:

I wanted to keep somewhat of a standard instead of just VARCHAR everything so I looked into standard formatting for things like countries and found ISO 3166-1 format and opted for the alpha-2 (2 char) for easier lookups if necessary.

#### Indexes:

With the understanding of leftmost prefix indexing for MySQL, for indexes like active_event_listings, I specified the order of lookups from event_id->status (active)->listed_prices. I also created indexes with the queries in part 2 in mind, or at least tried to thinking how instead of a full-table lookup how could the attributes that I access be indexed for a more efficient lookup from a sorted structure like a b-tree.

#### Real System Thoughts:

Although for this assignment, I assumed normalization of things like cities (since it follows 3NF as non transitive dependency). Though if I were to maybe join cities for analytics or needed locations to adhere to a strict formatting I would create and integrate those tables. Same goes for certain fields like currency, countries, and so on.

## 4. Query reasoning

### inactive buyers

```mysql
SELECT users.user_id, users.full_name
FROM users
WHERE
EXISTS (SELECT 1
        FROM orders
        WHERE orders.buyer_user_id=users.user_id)
AND NOT EXISTS (SELECT 1
                FROM orders
                WHERE orders.buyer_user_id=users.user_id
                AND orders.created_at >= NOW() - INTERVAL 90 DAY)
```

The tables read from are users and orders. Given that we are querying for user data we select FROM users. Afterwards, we have two WHERE statements with EXIST and NOT EXISTS checking for whether there exists 1 order where the buyer_user_id==user_id meaning that this user has purchased at least one ticket. We afterwards check again same FROM and WHERE statement but this time checking if the orders.created_at which is checking whether the TIMESTAMP is within a 90 day range but since it's wrapped in a NOT EXISTS it's a negation so no orders in past 90 days.
