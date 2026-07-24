# CS50 — Lecture 7: SQL
### When data outgrows a file — storing, relating, and querying at scale

Lecture 6 gave you Python, and with it the ability to work with files and data. This lecture starts from a familiar-looking place — data sitting in a flat file, like a spreadsheet or a **CSV** (comma-separated values) — and confronts a problem: **files don't scale.** A spreadsheet is fine for a hundred rows and miserable for a million — slow to search, easy to corrupt, and hopeless when many people read and write at once. The fix is a **database**, and the language for talking to one is **SQL**. Along the way, the data structures from Lecture 5 quietly return at industrial scale, and you meet two real-world dangers — race conditions and SQL injection — that separate toy programs from production ones.

---

## 1. Why not just use a file?

You *can* store data in a CSV, but it breaks down fast:

- **Searching is slow.** Finding a row means scanning the whole file top to bottom — `O(n)`, every time.
- **No relationships.** A flat file can't cleanly express "this order belongs to that customer."
- **Concurrency corrupts it.** Two programs writing the same file at once can mangle it.
- **It all lives in memory.** To work with it, you load the entire file — impossible past a certain size.

A **database** is purpose-built to fix all of this: fast lookups, structured relationships, safe concurrent access, and efficient storage that doesn't require loading everything at once.

---

## 2. Databases, SQL, and SQLite

**SQL** (Structured Query Language, often said "sequel") is the language for querying **relational databases** — databases that organize data into tables with defined relationships. The core SQL you learn transfers across systems: **SQLite** (what CS50 uses — a lightweight database that lives in a single file), **MySQL** and **PostgreSQL** (heavier client/server databases powering much of the web), and others.

The distinction worth knowing: SQLite is a *file-based* database (great for learning, mobile apps, and small sites), while MySQL/PostgreSQL run as a *server* that many clients connect to (built for large, high-traffic systems). The SQL you write is nearly the same either way.

---

## 3. Tables

Data in a relational database lives in **tables** — rows and columns, like a spreadsheet but with enforced structure. Each **column** has a fixed **type**, and each **row** is one record. SQLite keeps its types simple:

- `INTEGER` — whole numbers
- `REAL` — decimals
- `TEXT` — strings
- `BLOB` — raw binary data
- (`NUMERIC` — for things like dates/booleans)

You define a table's shape once with `CREATE TABLE`:

```sql
CREATE TABLE people (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    age INTEGER
);
```

`NOT NULL` and `PRIMARY KEY` are **constraints** — rules the database enforces for you (a name can't be empty; each `id` must be unique). That enforcement is part of what makes a database safer than a file.

---

## 4. CRUD: the four operations

Almost everything you do to data is one of four actions, known as **CRUD** — Create, Read, Update, Delete — each with a SQL keyword:

```sql
-- CREATE: add a new row
INSERT INTO people (name, age) VALUES ('David', 50);

-- READ: retrieve rows
SELECT * FROM people;

-- UPDATE: change existing rows
UPDATE people SET age = 51 WHERE name = 'David';

-- DELETE: remove rows
DELETE FROM people WHERE name = 'David';
```

A word of caution the lecture stresses: `UPDATE` and `DELETE` **without a `WHERE` clause apply to every row.** `DELETE FROM people;` empties the entire table. The `WHERE` is what scopes the operation — forget it and you've changed everything.

---

## 5. SELECT, in depth

Reading data is where most of the work happens, and `SELECT` is far richer than "get everything." You narrow, sort, limit, and summarize:

```sql
SELECT name FROM people WHERE age > 40;          -- filter rows with WHERE
SELECT * FROM people ORDER BY age DESC;          -- sort (descending here)
SELECT * FROM people ORDER BY age DESC LIMIT 10; -- just the top 10
SELECT DISTINCT name FROM people;                -- drop duplicate values
```

And **aggregate functions** compute a single summary value across many rows:

```sql
SELECT COUNT(*) FROM people;        -- how many rows?
SELECT AVG(age) FROM people;        -- average age
SELECT MAX(age) FROM people;        -- oldest
```

`COUNT`, `AVG`, `SUM`, `MIN`, `MAX`, often paired with `GROUP BY` to summarize *per category* (e.g., average age per city), turn a database from mere storage into a tool for answering questions about your data.

---

## 6. Relationships: keys and multiple tables

The "relational" in relational database is the heart of it. Rather than cramming everything into one giant table (with facts repeated over and over), you **split data across tables and link them.** Two kinds of keys make the links work:

- A **primary key** uniquely identifies each row in its table — usually an `id`.
- A **foreign key** is a column in one table that holds the primary key of a row in *another* table, connecting them.

Say books each have an author. Instead of repeating the author's full details on every book row, you keep authors in their own table and reference them:

```sql
CREATE TABLE authors (
    id INTEGER PRIMARY KEY,
    name TEXT
);

CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT,
    author_id INTEGER,               -- foreign key...
    FOREIGN KEY(author_id) REFERENCES authors(id)   -- ...pointing to authors
);
```

This avoids redundancy (an author's name is stored *once*), keeps data consistent (fix a typo in one place), and models the real world's relationships. Notice the idea: a foreign key is essentially a **pointer** from one table to another — the Lecture 4 concept, reincarnated at the data level.

---

## 7. JOINs

If related data lives in separate tables, you need a way to recombine it in a query. That's a **JOIN** — it stitches rows from multiple tables together on a matching key:

```sql
SELECT books.title, authors.name
FROM books
JOIN authors ON books.author_id = authors.id
WHERE authors.name = 'J.K. Rowling';
```

Read it as: pull titles from `books` alongside names from `authors`, matching each book to its author via the `author_id`/`id` link, and keep only Rowling's. `JOIN` is what makes the split-into-tables design practical — you separate data to store it cleanly, then join it back together however each question requires.

---

## 8. Indexes: the return of the tree

By default, searching a table with `WHERE` scans every row — `O(n)`, the same slowness as a flat file. An **index** fixes that:

```sql
CREATE INDEX name_index ON people (name);
```

An index builds a separate, sorted structure — typically a **B-tree**, a balanced tree — over a column, so lookups on it drop from `O(n)` to roughly `O(log n)`. This is three earlier lectures converging in one feature: the `O(log n)` search of Lecture 3, the balanced trees of Lecture 5, and the exact "is sorting worth it?" calculation from Lecture 3 — you pay a one-time cost to build the index (and a little extra storage and slower writes) so that the thousands of reads afterward are each fast. Databases are, quite literally, the data structures you built by hand, scaled up and battle-tested.

---

## 9. When things go wrong: race conditions

Databases serve many users *at once*, and concurrency creates a subtle danger. A **race condition** happens when two operations read and modify the same data simultaneously and step on each other.

The classic example: a bank account with $100, and two withdrawals of $100 hitting at the same instant. Both read the balance ($100), both see enough funds, both subtract — and the account goes to −$100 because each acted on data the other was about to change. The bank pays out $200 it didn't have.

The fix is **transactions**: bundling operations so they happen **atomically** — all-or-nothing, and locked so no one else can interleave:

```sql
BEGIN TRANSACTION;
    -- read balance, check funds, subtract — as one indivisible unit
COMMIT;                 -- (or ROLLBACK to undo if something failed)
```

Locking guarantees the second withdrawal waits until the first fully finishes, so it sees the true, updated balance. Concurrency safety is a big part of why you use a real database instead of a file.

---

## 10. When things go wrong: SQL injection

The second danger is a security hole, and it's one of the most common serious vulnerabilities in the world. **SQL injection** happens when untrusted user input is pasted directly into a query, letting an attacker smuggle in their *own* SQL.

Imagine building a login query by gluing strings together:

```
"SELECT * FROM users WHERE username = '" + input + "'"
```

If someone types `' OR '1'='1` as their username, the query becomes a condition that's always true — and they're logged in without a password. Worse inputs can delete tables entirely. (This is the joke behind the famous "Bobby Tables" cartoon — a student named to trigger a `DROP TABLE`.)

The fix is **parameterized queries** (placeholders): you never concatenate input into the query. Instead you pass it separately, and the database library treats it strictly as *data*, never as executable SQL:

```python
# Python, using placeholders — the '?' is filled in safely
db.execute("SELECT * FROM users WHERE username = ?", username)
```

This is the industrial-scale version of a lesson that's run through the whole course: **never trust user input.** It's the same instinct as validating a Caesar cipher's key back in Lecture 2 — just with much higher stakes.

---

## The big picture
Lecture 7 is where data gets a permanent, structured, safe home:

1. **Files don't scale** — databases give fast search, relationships, concurrency, and efficient storage.
2. **SQL** queries relational databases; **tables** hold typed rows and columns, defined with `CREATE TABLE`.
3. **CRUD** — `INSERT`, `SELECT`, `UPDATE`, `DELETE` — is the core, and `WHERE` scopes what you touch.
4. **`SELECT`** filters, sorts, limits, and aggregates to actually *answer questions*.
5. **Primary and foreign keys** link tables (a foreign key is a pointer between tables), and **`JOIN`** recombines them.
6. **Indexes** are B-trees that restore `O(log n)` search — the data structures of Lecture 5 at scale.
7. **Race conditions** (fixed with transactions) and **SQL injection** (fixed with placeholders) are the real-world hazards.

The through-line: nearly every concept here is an old friend scaled up. Foreign keys are pointers; indexes are balanced trees; "should I index this?" is "is sorting worth it?"; parameterized queries are input validation. And the destination is now in sight — a database is the *storage* layer of a real application. **Lectures 8 and 9 (web)** add the other two layers: a front end people interact with, and a back end (in Python, via Flask) that ties the browser to this database and puts the whole thing on the internet.
