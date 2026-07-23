# CS50 — Lecture 0: Scratch (Deep Dive)
### What computing actually *is*, before any real code

Lecture 0 looks like the easy one — it ends with drag-and-drop cartoon blocks. Don't be fooled. It quietly installs every mental model the rest of the course depends on: how information is represented, what an algorithm is, why efficiency matters, and the core building blocks of *all* programming. Nail this and the hard lectures later have somewhere to land.

---

## 1. What is computer science?

CS is fundamentally about **solving problems**. The model for the entire field fits in three boxes:

```
INPUT  →  [ ALGORITHM ]  →  OUTPUT
```

You put something in, a set of steps transforms it, and you get something out. That "black box" in the middle is where all the interesting work lives. Everything in CS50 — sorting a list, searching a database, rendering a webpage — is some version of this diagram.

But before a computer can process anything, it has to *represent* it. And computers only have one thing to work with: electricity that's either on or off. So the first real question is: how do you represent the whole world with just "on" and "off"?

---

## 2. Representation: it's all just numbers

### Binary — counting with two symbols
Humans use **decimal** (base-10): ten digits, 0–9, because we have ten fingers. Computers use **binary** (base-2): two digits, 0 and 1, because a switch is either off (0) or on (1). Each 0 or 1 is a **bit**.

Decimal works by place values that are powers of 10 (ones, tens, hundreds). Binary is the same idea with powers of 2 (ones, twos, fours, eights…):

```
Binary place values:   8   4   2   1
Binary number:         0   1   0   1   =  4 + 1  =  5
```

Counting up in binary with 3 bits:

```
000 = 0     100 = 4
001 = 1     101 = 5
010 = 2     110 = 6
011 = 3     111 = 7
```

Notice you run out of patterns fast — 3 bits only counts to 7. That's the seed of a big idea later (**integer overflow** in Lecture 1): with a fixed number of bits, there's a biggest number you can hold, and going past it wraps around.

### So what *is* a bit, exactly?
A bit is easy to confuse with the hardware that holds it, but the two are distinct:

- A **bit** is a unit of *information* — a single binary digit, one `0` or one `1`. The word is literally "**bi**nary dig**it**." It's a *value*, not a place.
- The **hardware** that stores that value is a physical component — a tiny switch (a transistor, or a **memory cell**) that's either off (`0`) or on (`1`).

In short, a bit is the *what* (the value), and the memory cell is the *where* (the storage). Everyday speech blurs the two — "a bit of memory" — which is harmless, but underneath, the bit is the information and the transistor is the hardware holding it. That separation of *value* from *location* pays off in **Lecture 4 (Memory)**, where addresses (the "where") become a topic in their own right, distinct from the values stored at them.

### Why bundle bits into bytes?
One bit on its own can only distinguish **two** things: on or off, yes or no, `0` or `1`. That's almost never enough — you can't spell a letter or count past 1 with a single yes/no. The fix is to **group bits**, and every bit you add *doubles* the number of things you can represent:

```
1 bit   →  2 values      (2¹)
2 bits  →  4 values      (2²)
3 bits  →  8 values      (2³)
...
8 bits  →  256 values    (2⁸)   ← a byte
```

**A byte** is 8 bits grouped together, giving 256 possible combinations (0–255). That's the smallest *practically useful* chunk — 256 is exactly enough to cover every character in ASCII (see below), which is no coincidence. Bytes are the standard "unit" of memory: when you hear a file is "5 kilobytes" or your RAM is "16 gigabytes," those are counts of bytes — billions of those on/off cells.

### Why 8 bits, specifically?
There's no law of physics forcing a byte to be 8 bits — early computers used 6-bit, 7-bit, and 9-bit bytes. Eight won out for three reasons:

- **Text set the floor.** ASCII needs 7 bits to cover its 128 characters (letters, digits, punctuation, control codes). Anything smaller couldn't represent written English, so 7 was the practical minimum.
- **Powers of two are convenient.** 8 is 2³, and hardware handles powers of two cleanly — for addressing memory, and for splitting and grouping data. Rounding 7 up to 8 made the math tidy, and the spare 8th bit wasn't wasted: early on it served as a **parity bit** (a simple check to catch a flipped bit), and later it doubled the character set to 256 for "extended ASCII" (accented letters, symbols).
- **IBM standardized it.** IBM's System/360 mainframe (1964) was dominant enough that its 8-bit byte became the industry default, and nearly everything since has followed. (The word "byte" was coined in 1956 by IBM engineer Werner Buchholz, spelled with a *y* so it wouldn't be misread as "bit.")

So the answer is a stack of reasons rather than one: 7 bits was the floor for text, 8 was the nearest tidy power of two with a useful spare bit, and IBM's market dominance locked it in — part engineering, part math, part historical accident.

### Letters — ASCII and Unicode
If a computer only stores numbers, how does it store the letter `A`? By **agreeing on a code**. **ASCII** is that agreement for English:

```
A = 65     a = 97
B = 66     b = 98
C = 67     ...
```

So the word `HI!` is really stored as the numbers `72 73 33`. The letter is an illusion — context (this is text, not a number) tells the computer to *display* 72 as `H`.

ASCII only had room for 256 characters, which can't cover every language on Earth, let alone emoji. **Unicode** extends the idea with far more bits, giving every character — Chinese, Arabic, math symbols, 😀 — its own number. Emoji are literally just big numbers with an agreed-upon picture. (Skin-tone variations and combined emoji are multiple codes glued together, which is why they sometimes render oddly on older devices.)

### Colors, images, video, sound
The same trick scales to everything:

- **Colors** use **RGB** — three numbers for the amount of **R**ed, **G**reen, **B**lue, each 0–255. `(255, 0, 0)` is pure red. Three bytes = one dot of color.
- **Images** are grids of those colored dots (**pixels**). A photo is just a very long list of RGB numbers. Zoom in far enough and you see the squares.
- **Video** is a rapid sequence of images (frames) — motion is an illusion built from stills.
- **Sound/music** is numbers too: which note, how loud, how long.

**The unifying insight of the whole section:** there is no fundamental difference between a number, a letter, a color, and a song inside a computer. It's *all* bits. What makes 72 mean "the letter H" versus "the number 72" is nothing but **context** — the agreed-upon convention for interpreting those bits.

---

## 3. Abstraction

**Abstraction** is the single most important idea in the course, and it appears in Lecture 0 so you start using it early. It means **hiding complexity behind a simple name so you can stop thinking about the details.**

Analogy: when you tell someone "make me a sandwich," you don't re-specify how to slice bread, unscrew the jar, and spread the peanut butter every time. "Make a sandwich" is an abstraction — a single label wrapping many smaller steps. Once someone knows how to do it, you just invoke the name.

In programming, you'll bundle a sequence of steps into a **function**, give it a name, and call it whenever you need it — without re-thinking the internals. Every layer of computing is built by hiding the layer below it. You'll see this pattern climb the whole course: bits → numbers → data types → data structures → libraries → whole applications, each one an abstraction over the last.

---

## 4. Algorithms and efficiency

An **algorithm** is a set of step-by-step instructions for solving a problem. But *how you write the steps* massively changes how fast it runs — and this is the demo that defines CS50.

### The phone book
Imagine a 1,000-page phone book, alphabetized, and you want to find "Mike Smith."

- **Approach 1 — one page at a time:** turn to page 1, page 2, page 3… Correct, but agonizingly slow. Worst case: 1,000 steps.
- **Approach 2 — two pages at a time:** faster, but now it's *buggy* — you might skip right past Mike's page. A fast algorithm that's occasionally wrong is worthless. **Correctness first.**
- **Approach 3 — split in half:** open to the middle. Is Mike's name earlier or later than the current page? Throw away the half that can't contain him, and repeat on what's left. Then repeat again. Each step **halves** the problem.

The third approach is dramatically better. Doubling the phone book to 2,000 pages adds *1,000 steps* to Approach 1, but only **one more step** to Approach 3. That gap — linear vs. logarithmic growth — is previewed here and formalized as **Big O notation** in Lecture 3. It's the reason "which algorithm" matters more than "how fast is the computer."

### Pseudocode
Before writing real code, you write **pseudocode** — the algorithm in plain, structured English:

```
1  Open to the middle of the phone book
2  Look at the name on the page
3  If the person is on this page
4      Call them
5  Else if the person is earlier in the book
6      Open to the middle of the left half
7      Go back to step 2
8  Else if the person is later in the book
9      Open to the middle of the right half
10     Go back to step 2
11 Else
12     Quit
```

This is worth studying because it exposes the building blocks hiding inside every program:
- **Functions / actions** — the verbs: *open, look, call* (things that *do* something).
- **Conditionals** — the *if / else if / else* forks (decisions).
- **Boolean expressions** — the yes/no questions the conditionals ask (*is the person earlier?*), which evaluate to true or false.
- **Loops** — "go back to step 2" (repetition).

Those four ideas — actions, conditionals, booleans, loops — are the grammar of *every* programming language. The rest of Lecture 0 just gives you a friendly place to practice them.

---

## 5. Scratch — programming without the syntax

**Scratch** (built at MIT) is a visual, drag-and-drop language. You snap colored **blocks** together instead of typing code, which removes syntax errors entirely so you can focus purely on *logic*. Everything you build here maps directly onto C, Python, and JavaScript later — only the appearance changes.

### The environment
- **Sprites** — the characters/objects (the default is a cat).
- **Stage** — the canvas where sprites act.
- **Blocks palette** — categorized puzzle pieces (Motion, Looks, Sound, Events, Control, etc.) you drag into scripts.

### The building blocks (same ones as the pseudocode)

**Functions / statements** — blocks that perform an action, like `say [Hello]` or `move (10) steps`. Some take **arguments** (inputs) — `say` needs to know *what* to say; `move` needs *how far*.

**Variables** — named containers that store a value you can change, like a `score` or `counter`. (`set score to 0`, `change score by 1`.)

**Boolean expressions** — questions that resolve to true/false, like `mouse down?` or `(x) > (10)`.

**Conditionals** — act on those booleans:
```
if <touching edge?> then
    play sound [meow]
```
and the two-way `if / else`.

**Loops** — repetition, so you don't copy-paste a block 100 times:
- `forever` — repeat endlessly
- `repeat (10)` — repeat a set number of times
- `repeat until <condition>` — repeat while a boolean stays false

**Events** — "when *this* happens, do *that*": `when green flag clicked`, `when [space] key pressed`, `when this sprite clicked`. Events are how programs respond to the user — a preview of the click/keypress handling you'll do in JavaScript in Lecture 8.

### Abstraction, revisited
The payoff move in Scratch is **making your own block** — bundling several blocks into a new one you name yourself (e.g., a custom `cough` block that's really `say [cough] wait play sound`, repeated). The moment you do that, you've *created an abstraction*: a new command that hides its own complexity. That's the exact same instinct behind writing a function in every language for the rest of the course.

---

## Why Lecture 0 matters more than it looks
By the end you've quietly learned:

1. **Everything is bits**, and meaning comes from agreed-upon context (ASCII, RGB, Unicode).
2. **Fixed bits = limits** — the setup for overflow and imprecision later.
3. **Problems = input → algorithm → output**, and the algorithm you choose determines speed (phone book → Big O).
4. **The four universal building blocks**: functions/actions, conditionals, booleans, and loops — plus variables and events.
5. **Abstraction** — naming complexity so you can forget it — which is the ladder you'll climb through the entire course.

Scratch is the sandbox. From Lecture 1 onward, you're doing the exact same things — just typing them in C, where the training wheels come off.
