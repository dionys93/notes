# CS50 in a Nutshell
### The 24-hour course, distilled to what actually sticks

A quick map before you dive in: CS50 is really two arcs. Lectures 0–5 teach you to *think like a computer* using C — a "close to the metal" language that forces you to understand memory. Lectures 6–9 (plus AI, SQL, and web) show you how the real world builds software fast, on top of those foundations. The pain of C early on is the whole point: everything after feels easy because you know what's happening underneath.

---

## Lecture 0 — Scratch
**Big idea: computing is just turning inputs into outputs with an algorithm in the middle.**

- Everything a computer stores is **binary** — 0s and 1s (bits). Numbers are binary directly; letters use **ASCII/Unicode** (e.g. `A` = 65); colors use **RGB** (three numbers for red/green/blue); images are grids of those; video is images in sequence. It's all just numbers underneath.
- **Algorithms** are step-by-step instructions. A good algorithm isn't just *correct*, it's *efficient* — the phone-book demo (tearing the book in half repeatedly vs. flipping one page at a time) previews the whole course.
- **Abstraction** = hiding complexity so you can think at a higher level. You don't rebuild "make a sandwich" every time; you call it.
- Scratch (drag-and-drop blocks) introduces the building blocks you'll use in *every* language: **functions**, **loops**, **variables**, **conditionals**, **boolean expressions**, and **events**. The syntax changes later; these ideas don't.

**Takeaway:** if you understand loops, conditionals, and functions here, you already know 80% of the *concepts* — the rest of the course is learning to express them in harder languages.

---

## Lecture 1 — C
**Big idea: source code you write gets compiled into machine code the computer runs.**

- CS50 grades on three axes worth internalizing for life: **correctness** (does it work?), **design** (is it well-built, not repetitive?), and **style** (is it readable?).
- Core C syntax: `#include` for libraries, `int main(void)` as the entry point, `printf` with **format codes** (`%i`, `%s`, `%f`, `%c`).
- **Data types matter in C** — you must declare them (`int`, `float`, `char`, `bool`, `long`, `double`). This is annoying now but teaches you what's really happening.
- Two classic gotchas that reveal how computers actually work:
  - **Integer overflow** — an `int` runs out of bits and wraps around (famously why old games glitch at high scores, and behind the Y2K panic).
  - **Floating-point imprecision** — computers can't store `1/3` perfectly, so `0.1 + 0.2` isn't exactly `0.3`. Finite bits, infinite real numbers.

**Takeaway:** the computer does *exactly* what you say, including your mistakes. Precision is the whole game.

---

## Lecture 2 — Arrays
**Big idea: memory is a long row of boxes, and arrays let you store data contiguously.**

- What "compiling" really means, in four steps: **preprocessing** → **compiling** (to assembly) → **assembling** (to machine code) → **linking** (stitching in libraries).
- **Arrays** store multiple values of one type back-to-back, accessed by index starting at **0**.
- **Strings are just arrays of characters** ending in a special `\0` (null terminator) byte. This single fact explains a *lot* of C weirdness.
- **Command-line arguments** via `argc` (count) and `argv` (the values), so programs can take input when you run them.
- Programs return an **exit status** (0 = success, non-zero = error) — the thing scripts check to know if something worked.
- Debugging is a taught skill here: `printf` debugging, `debug50`, and **rubber-duck debugging** (explaining your code aloud to something dumb until you spot the bug yourself).

**Takeaway:** strings aren't magic — they're arrays with a hidden `\0` on the end. Remember that and C strings stop being scary.

---

## Lecture 3 — Algorithms
**Big idea: how fast an algorithm runs matters more than raw computer speed.**

- **Searching:** linear search (check each item, slow) vs. **binary search** (repeatedly halve a *sorted* list — the phone-book trick). Sorting first pays off big.
- **Big O notation** describes how running time grows as input grows. Learn these by feel:
  - `O(1)` — constant, instant
  - `O(log n)` — binary search, gorgeously fast
  - `O(n)` — linear
  - `O(n log n)` — good sorting (merge sort)
  - `O(n²)` — nested loops, gets ugly fast
- **Sorting algorithms** as a lesson in trade-offs: **bubble sort** and **selection sort** are simple but slow (`O(n²)`); **merge sort** is fast (`O(n log n)`) but uses more memory. There's usually a **time vs. space** trade-off.
- **Recursion** — a function that calls itself, shrinking the problem each time until a **base case** stops it. Elegant, and the mental model behind merge sort.

**Takeaway:** `O(log n)` beats `O(n)` beats `O(n²)`. When something's slow, this is usually why.

---

## Lecture 4 — Memory
**Big idea: variables live at addresses, and pointers let you work with those addresses directly.** *(This is the hardest lecture — and the payoff.)*

- Memory addresses are written in **hexadecimal** (base 16). A **pointer** is a variable that holds an address. `&x` = "address of x"; `*p` = "the value at pointer p."
- **`malloc`** grabs memory manually; **`free`** gives it back. Forget to free and you get a **memory leak**; `valgrind` catches them.
- Memory is laid out in regions: **text** (your code), **data**, **heap** (grows down, `malloc` lives here), and **stack** (grows up, function calls live here). Crash them into each other and you get a **stack overflow**.
- Now the earlier weirdness clicks: strings are really **pointers to the first character** of a char array. That's why `s1 == s2` compares *addresses* not text (use `strcmp`), and why copying a string means copying it character by character (`strcpy` / `memcpy`), not just `s2 = s1`.
- **Structs** let you bundle related variables into your own custom type — the seed of objects and databases later.

**Takeaway:** once pointers click, you understand what *every* higher-level language is hiding from you. Push through this one.

---

## Lecture 5 — Data Structures
**Big idea: how you organize data determines what your program can do quickly.**

- **Abstract data types** — describe *behavior*, then pick an implementation:
  - **Stack** — Last In, First Out (like a stack of plates)
  - **Queue** — First In, First Out (like a line)
- **Linked lists** — chains of structs connected by pointers. Easy to grow/shrink, but you lose instant indexing (must walk the chain). The classic array-vs-linked-list trade-off: **fast lookup vs. easy resizing.**
- **Trees** and **binary search trees** — keep data sorted for `O(log n)` search *and* flexible size.
- **Hash tables** — a **hash function** maps a key to a slot for near-instant `O(1)` lookup. **Collisions** (two keys, one slot) are handled by chaining. This is how real dictionaries/maps work.
- **Tries** — trees optimized for lookups by prefix (great for autocomplete/spellcheck): blazing fast, but memory-hungry.

**Takeaway:** every structure trades something. Hash tables buy speed with memory and complexity. Choosing the right one *is* the engineering.

---

## Lecture 6 — Python
**Big idea: trade a little runtime speed for a huge gain in developer speed.**

- All the C pain disappears: **no manual memory management** (no pointers, no `malloc`/`free` — it's automatic), **dynamic typing** (no declaring `int`/`char`), and clean, readable syntax.
- Rich built-in types: **lists**, **dictionaries** (hash tables for free!), **tuples**, **sets** — the structures you hand-built in C are now one line.
- **Libraries** do the heavy lifting — `import` and you inherit thousands of hours of other people's work. This is the real superpower.
- Error handling with **`try` / `except`** instead of checking return codes everywhere.
- Comfortable with text-wrangling: file I/O, CSV parsing, and **regular expressions** (pattern-matching for strings).

**Takeaway:** Python is "C with the tedium removed." Slower to run, dramatically faster to write — which is the right trade for most real work.

---

## AI (interlude)
**Big idea: modern AI is pattern-recognition learned from data, not hand-coded rules.**

- Early "AI" was just clever **conditionals and rules** (decision trees). It works but doesn't scale to messy real-world problems.
- **Machine learning** flips it: instead of writing rules, you show the system many examples and let it learn patterns. **Neural networks** stack simple math into layers that recognize increasingly complex features.
- **Large language models (LLMs)** predict the next word/token, trained on enormous text. Powerful, but they **hallucinate** — they generate plausible text, not guaranteed truth.
- Practical thread: **prompt engineering** (how you ask shapes what you get) and using AI as a coding partner — CS50 even has its own AI "rubber duck" tutor that guides rather than hands over answers.

**Takeaway:** treat AI as a confident, fast, occasionally-wrong assistant. Verify; don't outsource your judgment.

---

## Lecture 7 — SQL
**Big idea: when data outgrows spreadsheets, you need a real database.**

- Flat files (CSV) break down at scale — slow to search, easy to corrupt, no relationships. **Relational databases** (SQLite here) fix this.
- The four operations, **CRUD**: `INSERT` (create), `SELECT` (read), `UPDATE`, `DELETE`.
- Data lives in **tables** with typed columns; **primary keys** uniquely ID each row; **foreign keys** link tables together. **JOINs** combine related tables in one query.
- **Indexes** (often B-trees under the hood) make lookups dramatically faster — the database equivalent of the sorting-first lesson from Lecture 3.
- Two real-world dangers introduced here:
  - **Race conditions** — two operations touching the same data at once; solved with transactions/locks.
  - **SQL injection** — untrusted input smuggled into a query. Always use **placeholders**, never glue user input into SQL strings.

**Takeaway:** databases are just data structures (Lecture 5) at industrial scale — with real security stakes.

---

## Lecture 8 — HTML, CSS, JavaScript
**Big idea: the front end is three languages doing three jobs — structure, style, behavior.**

- How the internet works underneath: **IP** addresses locate machines, **DNS** turns names into IPs, **TCP/IP** moves the packets, **HTTP(S)** is the request/response conversation (with **status codes** like `200 OK` and `404 Not Found`).
- **HTML** = structure/content (tags and elements — the skeleton).
- **CSS** = presentation (colors, layout, fonts; **selectors** target elements). Frameworks like **Bootstrap** save you writing it from scratch, and **responsive design** adapts to phone vs. desktop.
- **JavaScript** = behavior. It manipulates the **DOM** (the live tree of the page) and responds to **events** (clicks, typing, submits) to make pages interactive without reloading.

**Takeaway:** structure (HTML), style (CSS), behavior (JS). Keep those three jobs separate in your head and web dev stays organized.

---

## Lecture 9 — Flask
**Big idea: the back end runs on the server, stores your data, and decides what each user sees.**

- **Flask** (Python) ties the whole course together into a working web app.
- **Routes** map URLs to functions; **HTTP methods** split intent — `GET` to fetch/display, `POST` to submit/change data.
- **Templates** (Jinja) inject dynamic data into HTML, so pages are generated per-user instead of hand-written.
- **Sessions** and **cookies** remember who's logged in across requests (HTTP itself is forgetful/stateless).
- The app talks to a **SQL database** (Lecture 7) to persist data, and can expose or consume **APIs** to exchange data with other services.

**Takeaway:** this is the payoff lecture — Python + SQL + HTML/CSS/JS combine into a real, full-stack app. Everything before this was the toolkit.

---

## The 60-second version
1. **It's all numbers** — binary underneath everything.
2. **Algorithms have costs** — Big O tells you how fast they grow; `log n` good, `n²` bad.
3. **Memory is physical** — pointers (Lecture 4) are the hard-won key to understanding everything above them.
4. **Data structures are trade-offs** — speed vs. memory vs. simplicity; pick deliberately.
5. **Higher-level languages hide the pain** — Python gives you C's power without the bookkeeping.
6. **Real apps are layers** — database (SQL) + server (Flask) + browser (HTML/CSS/JS), stitched together.
7. **The struggle is the curriculum** — C is hard *so that* everything after it feels manageable.
