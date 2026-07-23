# CS50 — Lecture 1: C 
### Same ideas as Scratch — now you type them, and the training wheels come off

Lecture 1 takes every building block from Scratch — functions, conditionals, booleans, loops, variables — and re-teaches them in **C**, a real, text-based programming language. Nothing conceptually new is happening; you already know *what* a loop is. What's new is **syntax** (the exact punctuation a computer demands), **compiling** (turning your text into something the machine can run), and the unforgiving precision C requires. That precision is the entire reason C is taught first: it forces you to see what higher-level languages hide.

---

## 1. From blocks to text

In Scratch you snapped a `say [hello, world]` block onto the stage. In C, the equivalent program is:

```c
#include <stdio.h>

int main(void)
{
    printf("hello, world\n");
}
```

Every line here is a building block you already met, now written as text:

- `printf(...)` is a **function** that prints text — the typed cousin of Scratch's `say`.
- `"hello, world\n"` is the **argument** you hand it.
- `main` is where every C program begins running.

The catch: unlike Scratch, a computer won't forgive a missing semicolon or bracket. Which brings us to syntax.

---

## 2. Anatomy of the program (syntax)

C is picky. Every character below is doing a job:

- **`#include <stdio.h>`** — pull in a **header file**. `stdio.h` ("standard input/output") is what gives you `printf`. Without including it, C doesn't know what `printf` means. Think of it as importing a toolbox before using its tools.
- **`int main(void)`** — declares the starting function. `main` is special: it's the entry point. (`int` and `void` here describe what goes in and comes out — that clicks in the Functions section below.)
- **Curly braces `{ }`** — mark where a block of code begins and ends. Everything between them belongs to `main`.
- **Semicolons `;`** — end a statement. Forgetting one is the single most common beginner error. Every instruction needs its period.
- **Double quotes `" "`** — wrap a **string** (text) so C treats it as data, not code.

### Escape sequences
That `\n` at the end of `"hello, world\n"` is a **newline** — it moves the cursor to the next line, like pressing Enter. The backslash starts an **escape sequence**, a way to type characters you otherwise couldn't put directly in quotes:

- `\n` — new line
- `\t` — tab
- `\"` — a literal double-quote (so C doesn't think the string ended)
- `\\` — a literal backslash

---

## 3. Compiling: from source code to machine code

You wrote text a human can read. The CPU can't run that — it only understands **machine code**, raw binary instructions (0s and 1s). **Compiling** is the translation step in between.

- The text you write is **source code**.
- A **compiler** translates it into **machine code** the computer can execute. "Compiler" is a *category* of program, not one specific tool: `gcc` (the long-standing GNU standard) and `clang` (the LLVM-based compiler, default on macOS) are the two ubiquitous C compilers, and either does the job identically. CS50 happens to use `clang` — but nothing about the concepts depends on that choice.
- You rarely invoke the compiler by hand for anything real — a **build tool** runs the command for you so you don't retype it every time. `make` is the classic Unix one (CS50 uses it: `make hello`, then `./hello` to run the result), but it's one convention among several; the orthodox idea is simply "a tool that automates the compile command."

The takeaway to carry forward: **there are two files** — your readable source code and the compiled program — and every time you change the source, you must recompile before the change takes effect. This edit → compile → run loop is the rhythm of C, whatever compiler and build tool you use. (The full four-stage compilation pipeline gets unpacked in **Lecture 2**.)

---

## 4. Correctness, design, style

CS50 evaluates code — and asks you to evaluate your own — on three axes worth adopting permanently:

- **Correctness** — does it actually do what it's supposed to? Non-negotiable and comes first.
- **Design** — is it well-built? Efficient, not needlessly repetitive, not doing in twenty lines what five would do. This is the artful part.
- **Style** — is it readable? Consistent indentation, clear names, helpful comments. Code is read by humans far more often than it's written, including future-you.

A program can be correct but ugly, or elegant but broken. The goal is all three.

---

## 5. Printing and format codes

`printf` is more powerful than just printing fixed text — it can splice in **variables** using **format codes** (placeholders). Each `%` marker says "insert a value of this type here":

```c
printf("hello, %s\n", name);      // %s → a string
printf("You are %i years old\n", age);   // %i → an integer
```

The common format codes:

- `%s` — string (text)
- `%i` — integer (CS50 uses `%i`; `%d` also works)
- `%f` — float (decimal number)
- `%c` — a single character
- `%li` — a long integer

Getting the code wrong (e.g., `%i` for a decimal) produces garbage output — a small preview of how strict C is about **types**, which is the next big idea.

---

## 6. Variables and data types

In Scratch a variable just held "a number" or "a word." In C, you must declare **what kind** of data a variable holds — its **type** — because the type determines how many bits it takes and how they're interpreted (a direct payoff of the "it's all bits, meaning comes from context" idea in Lecture 0).

Core types:

| Type | Holds | Typical size |
|------|-------|--------------|
| `int` | whole numbers | 4 bytes (~ −2 billion to +2 billion) |
| `char` | a single character | 1 byte |
| `float` | decimal numbers | 4 bytes |
| `double` | decimals, higher precision | 8 bytes |
| `long` | big whole numbers | 8 bytes |
| `bool` | `true` / `false` | 1 byte |
| `string` | text (a CS50 convenience) | — |

Note `string` isn't really a built-in C type — it's an abstraction the **CS50 library** provides to keep things gentle early on. Under the hood it's something more primitive, and that curtain gets pulled back in **Lecture 4 (Memory)**.

### Declaring vs. initializing
```c
int counter;        // declare: reserve a box of type int
counter = 0;        // initialize: put a value in it
int counter = 0;    // do both at once
```

### The CS50 library and getting input
To read input from the user, CS50 provides helper functions via `#include <cs50.h>`:

```c
#include <cs50.h>

string name = get_string("What's your name? ");
int age = get_int("How old are you? ");
```

`get_string`, `get_int`, `get_float`, and friends prompt the user and hand back a value of the right type — training wheels so you can focus on logic before wrestling with raw input handling.

### Shorthand for changing variables
```c
counter = counter + 1;   // the long way
counter += 1;            // syntactic sugar
counter++;               // even shorter, for adding 1
```

These all do the same thing. `counter++` shows up constantly, especially in loops.

---

## 7. Conditionals

Same idea as Scratch's `if` blocks — decisions based on **boolean expressions** (questions that resolve to true or false):

```c
if (x > 0)
{
    printf("positive\n");
}
else if (x < 0)
{
    printf("negative\n");
}
else
{
    printf("zero\n");
}
```

The comparison and logical operators you'll use:

- `>`  `<`  `>=`  `<=` — greater/less than (or equal)
- `==` — **equal to** (two equals signs; a single `=` means *assign*, a classic bug)
- `!=` — not equal to
- `&&` — logical AND (both must be true)
- `||` — logical OR (either can be true)

That `==` vs `=` trap catches nearly everyone once: `if (x = 5)` silently *assigns* 5 to x instead of *checking* it. Watch for it.

---

## 8. Loops

Three ways to repeat, each suited to a different situation:

**`while`** — repeat as long as a condition holds:
```c
while (i < 3)
{
    printf("meow\n");
    i++;
}
```

**`do…while`** — same, but always runs at least once (check the condition *after* the first pass). Handy for input validation — ask, then re-ask if the answer was bad:
```c
do
{
    n = get_int("Size: ");
}
while (n < 1);
```

**`for`** — the go-to when you know how many times to loop. It bundles setup, condition, and update into one line:
```c
for (int i = 0; i < 3; i++)
{
    printf("meow\n");
}
```
Read that as: start `i` at 0; keep going while `i < 3`; add 1 to `i` each pass.

A deliberate **infinite loop** is `while (true)`, escaped with a `break` when some condition is met. An *accidental* infinite loop (forgetting to update your counter) is a rite of passage.

---

## 9. Functions

Functions let you name a chunk of work and reuse it — the same **abstraction** move you made in Scratch by building a custom block. The guiding principle is **DRY: Don't Repeat Yourself.** If you're copy-pasting the same lines, or repeating a formula you'd rather define once, wrap it in a function.

The most useful mental model is the input→output box from Lecture 0: a function takes values in, and hands a computed value *back*. Here's one that converts a temperature — it earns its place by putting the conversion formula in exactly one spot, so there's only one thing to fix if it's ever wrong:

```c
#include <cs50.h>
#include <stdio.h>

// Definition: take a value in, hand a computed value back
float to_fahrenheit(float celsius)
{
    return celsius * 9.0 / 5.0 + 32.0;
}

int main(void)
{
    float c = get_float("Temperature in C: ");
    printf("%.1f F\n", to_fahrenheit(c));
}
```

Three things to understand:

- **Parameters (inputs)** — the values in the parentheses. `to_fahrenheit(float celsius)` takes one `float` and names it `celsius` inside the function.
- **Return type (output)** — the word *before* the name says what the function hands back. Here it's `float`, and the `return` statement is what actually sends the value back to whoever called it. (`void` would mean "returns nothing" — a function that only causes a side effect, like printing.) This is also where `int main(void)` finally makes full sense: `main` returns an `int` and takes no input (`void`).
- **Definition order** — C reads top to bottom and won't call a function it hasn't seen yet. Defining `to_fahrenheit` *above* `main`, as here, is the simplest way to satisfy that: the definition doubles as the declaration, so nothing extra is needed.

### The alternative: prototypes
You'll often see the function defined *below* `main` instead, with a one-line **prototype** up top:

```c
float to_fahrenheit(float celsius);   // prototype: name, parameters, return type

int main(void)
{
    ...
}

float to_fahrenheit(float celsius)    // definition, further down
{
    return celsius * 9.0 / 5.0 + 32.0;
}
```

A prototype is just a declaration — it tells C the function's signature (name, parameter types, return type) ahead of the call, and the real definition follows later. It's genuinely needed in three cases: when you want `main` to read first, top-down, with helpers below it; when two functions call *each other* (mutual recursion), so one is unavoidably defined second; and when a function is shared across multiple files, where its declaration lives in a header (`.h`). For a small single-file program, though, defining above `main` is cleaner — one signature instead of two to keep in sync.

A subtlety worth knowing: whether a *missing* declaration actually stops compilation depends on the C standard. Older C (pre-C99) would silently assume the function returns `int` and build anyway; modern compilers usually just warn; C23 makes it a hard error. CS50 compiles with strict flags that treat warnings as errors, so in *that* setup it does fail the build — but "plain" C is often more forgiving.

### A note on declaration order across languages
The deeper idea — a call has to be connected to a definition — is universal, but *when* each language resolves it differs, which is why some need forward declarations and some don't:

- **C** does a single top-to-bottom pass and must know a function's **signature** before it sees the call, hence define-above-use or a prototype.
- **JavaScript** **hoists** function declarations: the engine scans the whole file first and effectively lifts every `function foo() {}` to the top, so you can call `foo()` on a line *above* where it's written. (Caveat: only classic declarations hoist. A function stored in a variable — `const foo = () => {}` — follows variable rules and is *not* callable before its line.)
- **Python** doesn't hoist either, but you rarely notice, because a name is only looked up when the call actually *runs*. As long as every function is defined by the time execution reaches the call — typically because calls live inside other functions that run later — file order is flexible. Call a function at the top level before its `def`, though, and you get a `NameError`.
- **Java / C++** land in between: within a class, Java resolves methods regardless of textual order, so no forward declaration is needed; C++ inherits C's model and still uses prototypes (usually tucked into header files).

---

## 10. Where C bites: the gotchas that teach how computers really work

These aren't trivia — they're the reason C is the teaching language. Each exposes a truth that higher-level languages paper over.

### Integer overflow
An `int` has a fixed number of bits (4 bytes), so there's a largest value it can hold (~2.1 billion). Add 1 past the maximum and it **wraps around** to a negative number — the odometer rolling over. This is the direct consequence of the "fixed bits = a ceiling" idea from Lecture 0, and it's behind real-world disasters (arcade games glitching at high scores, the Y2K scare, aircraft systems needing periodic reboots). Fix: use a bigger type like `long`.

### Floating-point imprecision
Computers have finite bits but there are infinitely many real numbers, so decimals can't always be stored exactly. Ask C to print `0.1 + 0.2` to 20 decimal places and you won't get a clean `0.3` — you'll get something like `0.30000000000000004`. The lesson: never assume floating-point math is perfectly exact, especially with money.

### Integer division / truncation
Divide two integers and C throws away the remainder — the result is a whole number:
```c
float result = 1 / 3;   // result is 0.0, NOT 0.333...
```
Because `1` and `3` are both ints, C computes `1 / 3` as integer division (= 0) *before* storing it as a float. The fix is **type casting** — telling C to treat a value as another type:
```c
float result = (float) 1 / 3;   // now 0.3333...
```

---

## 11. Debugging

Bugs are not failure — they're the job. CS50 teaches three tools, in increasing sophistication:

- **`printf` debugging** — sprinkle print statements to see the actual values of variables at each step. Crude but effective.
- **`debug50`** — a proper **debugger** that pauses your program at a **breakpoint** and lets you step through line by line, watching variables change in real time.
- **Rubber-duck debugging** — explain your code out loud, line by line, to something that can't help you (a rubber duck). Articulating it forces you to spot the flaw yourself. It sounds silly and it works.

---

## The big picture
Lecture 1 doesn't introduce new *concepts* — it introduces **rigor**. The same six building blocks from Scratch, now expressed in a language where:

1. **Syntax is exact** — semicolons, braces, and quotes all matter.
2. **You compile before you run** — source code becomes machine code via `clang`/`make`.
3. **Types are explicit** — you declare what kind of data every variable holds, because underneath it's all just bits.
4. **The machine is literal** — overflow, imprecision, and truncation are consequences of finite hardware, not bugs in C.
5. **Good code is correct, well-designed, and readable** — all three.

Everything C forces you to confront here — types, memory sizes, precision — is the groundwork for the harder lectures. Push through the strictness now and Arrays, Memory, and Data Structures have a foundation to stand on.
