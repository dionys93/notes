# CS50 — Lecture 4: Memory
### The notorious wall — and the lecture that makes everything before it click

This is the hardest lecture in CS50, and it's hard on purpose. Everything so far has been standing on a trapdoor: `string` was a polite fiction, arrays "somehow" kept their changes when passed to functions, and `==` mysteriously failed on text. Lecture 4 opens the trapdoor. You learn that **variables live at numbered addresses in memory, and a pointer is just a variable that holds an address.** Once that lands, a dozen earlier mysteries resolve at once — and you finally understand what every higher-level language has been hiding from you. Push through this one slowly; it's worth it.

---

## 1. Hexadecimal: the notation for addresses

Before addresses, a detour into how they're written. Programmers describe memory in **hexadecimal** — base 16 — so it's worth understanding why.

Hex has sixteen digits: `0`–`9`, then `A`–`F` for ten through fifteen. The reason it's everywhere isn't aesthetics — it's that **one hex digit maps exactly to four bits**, so two hex digits are exactly one byte. That clean alignment makes hex a compact, readable stand-in for binary:

```
binary   hex
0000  =   0
1010  =   A
1111  =   F
11111111 = FF   (one byte, 255)
```

In C (and most languages), hex numbers are prefixed with `0x` to distinguish them: `0x1A` is 26, not "one-A". You've actually seen hex already — RGB color codes like `#FF0000` (pure red) from Lecture 0 are hexadecimal bytes. Memory addresses are conventionally shown in hex too, which is the reason this comes first.

---

## 2. Memory as addressed bytes

Here's the foundational picture: **RAM is one enormous array of bytes, and every byte has a numbered address.** When you declare a variable, the computer sets aside room for it somewhere in that array, at some address.

So a variable has two things worth talking about:

- its **value** — what's stored (e.g., `50`), and
- its **address** — *where* it's stored (e.g., `0x7ffe34a8`).

Until now you've only cared about values. Lecture 4 is what happens when you start caring about addresses — because manipulating *where* things live is the key to strings, dynamic memory, and every data structure in Lecture 5.

---

## 3. Pointers: `&` and `*`

Two operators let you work with addresses:

- **`&` — "address of."** `&n` gives you the address where `n` lives.
- **`*` — "dereference," or "go to."** `*p` gives you the value stored *at* the address `p` holds.

They're inverses: `&` takes a value's location, `*` follows a location back to its value. Here's the canonical example:

```c
int n = 50;          // an ordinary int
int *p = &n;         // p holds the ADDRESS of n

printf("%p\n", p);   // prints n's address, e.g. 0x7ffe34a8
printf("%i\n", *p);  // prints 50 — follow p to the value there
```

A **pointer** is exactly this: a variable whose value is an **address**. `p` doesn't hold `50`; it holds *where 50 lives*. The `%p` format code prints an address; `*p` dereferences it back to the `50`.

---

## 4. Declaring a pointer

In `int *p`, the `*` marks `p` as a **pointer to an int** — a variable that stores the address of an int. Read it as "`p` is a pointer to int," or "`*p` is an int."

A small but classic gotcha: the `*` binds to the *variable*, not the type. So

```c
int *p, q;    // p is a pointer to int; q is just an int (NOT a pointer)
```

To declare two pointers you'd write `int *p, *q;`. This trips people up constantly.

A pointer's own size is the machine's address width — 8 bytes on a 64-bit system — regardless of what it points *to*. A pointer to a `char` and a pointer to a giant struct are the same size, because both just hold an address.

---

## 5. The big reveal: a string is a pointer

This is the moment the whole course has been building toward. The CS50 `string` type is defined, roughly, as:

```c
typedef char *string;
```

That's it. **`string` is just `char *` — a pointer to a character.** When you write:

```c
string s = "HI!";
```

`s` doesn't "contain" the text. `s` holds a single **address**: the location of the first character, `'H'`. The rest of the characters sit in the bytes immediately after it, ending in the `\0` terminator from Lecture 2. So a string is nothing but *the address of its first byte* plus the convention "keep reading until `\0`."

```
Address:   0x123   0x124   0x125   0x126
Value:      'H'     'I'     '!'     '\0'
            ↑
   s holds 0x123  (the address of the first char)
```

Every quirk of C strings falls out of this one fact. The next few sections are just consequences.

---

## 6. Pointer arithmetic and the array connection

Because a string is an address, you can do *math* on it to reach later characters. `s[i]` is, by definition, shorthand for `*(s + i)` — "start at `s`, move `i` positions over, and dereference":

```c
string s = "HI!";
printf("%c\n", s[1]);     // 'I'
printf("%c\n", *(s + 1)); // 'I' — exactly the same thing
```

This reveals the deep truth teased back in Lecture 2: **arrays and pointers are two views of the same idea.** An array's name behaves like a pointer to its first element, and indexing with `[]` is pointer arithmetic wearing friendlier clothes. Adding `1` to a pointer advances it by the *size of the type* it points to (one byte for `char`, four for `int`), so the arithmetic always lands on the next element, not the next byte.

This is also why passing an array to a function lets the function change it (a mystery from Lecture 1): you're not copying the array, you're handing over its *address*. The function reaches back through that pointer to the original.

---

## 7. Why `==` fails on strings

Try to compare two strings the obvious way and it breaks:

```c
string s = get_string("s: ");
string t = get_string("t: ");
if (s == t)   // BUG: this is almost never true
```

Type the same word twice and this is *still* false. Why? Because `s` and `t` are pointers — **`s == t` compares their addresses**, not their contents. The two inputs are stored at two different places in memory, so their addresses differ even when the text matches.

To compare the actual characters, use `strcmp` (from `<string.h>`), which walks both strings and returns `0` when they're identical:

```c
if (strcmp(s, t) == 0)   // correct: compares contents, char by char
```

(The `0`-means-equal convention feels backwards, but it lets `strcmp` also report *which* string sorts first via negative/positive returns.)

---

## 8. Why copying a string isn't `t = s`

Same trap, other direction:

```c
string s = "HI!";
string t = s;      // does NOT copy the text
```

This copies the *pointer*, not the string. Now `s` and `t` hold the **same address** — they point at the *same* single copy of "HI!" in memory. Change a character through one and it changes for "both," because there's really only one string.

To make a genuine, independent copy, you must **allocate new memory** and copy the characters into it:

```c
#include <stdlib.h>
#include <string.h>

char *t = malloc(strlen(s) + 1);   // +1 for the '\0'
strcpy(t, s);                      // copy the characters over
// ... use t ...
free(t);                           // release it when done
```

The `+ 1` is easy to forget and a classic bug — you need room for the terminator, not just the visible characters. Which brings us to `malloc`.

---

## 9. Dynamic memory: `malloc` and `free`

Sometimes you don't know how much memory you'll need until the program runs. **`malloc`** ("memory allocate") requests a block of memory of a given size and hands back a **pointer** to it:

```c
int *arr = malloc(10 * sizeof(int));   // room for 10 ints
if (arr == NULL)                       // malloc returns NULL if it fails
{
    return 1;                          // always check!
}
```

Two rules that will save you hours of pain:

- **Always check for `NULL`.** `malloc` can fail (out of memory), and using a `NULL` pointer crashes.
- **Every `malloc` needs a matching `free`.** Memory you allocate is *not* automatically reclaimed; you must hand it back with `free(arr)` when you're done. Forgetting is a **memory leak** (section 11).

This manual allocate-and-release is exactly what Python, JavaScript, and friends do *for* you automatically (Lecture 6). C makes you do it by hand — which is tedious, but it's why you now understand what "automatic memory management" is actually managing.

---

## 10. The memory map: stack and heap

Where does all this memory live? A running program's memory is organized into regions:

- **Machine code** — your compiled program's instructions.
- **Globals** — global variables.
- **Heap** — the pool `malloc` draws from. It grows **downward** as you allocate more.
- **Stack** — where **function calls** and their **local variables** live. Each function call adds a **stack frame**; it grows **upward**.

The heap and stack grow *toward each other*. The important consequence: if one grows too far and collides with the other, you're out of usable memory. Infinite (or just very deep) **recursion** is the classic cause — each call piles another frame on the stack until it overflows, a **stack overflow** (yes, that's where the website's name comes from). This is why the recursion in Lecture 3 needs a *base case*: without one, the stack grows forever.

Local variables living on the stack also explains a subtle bug: a pointer to a local variable becomes invalid once that function returns, because its stack frame is gone. Memory that must outlive the function that created it belongs on the **heap** (via `malloc`).

---

## 11. When memory goes wrong

Working directly with memory means new ways to fail — and the tools to catch them:

- **Segmentation fault ("segfault")** — you touched memory you're not allowed to: dereferencing a `NULL` or garbage pointer, reading past the end of an array, or using memory you already `free`d. The program is killed on the spot.
- **Memory leak** — you `malloc`'d but never `free`d, so the memory stays reserved but unreachable. Leak in a loop and a long-running program slowly devours all available RAM.
- **Garbage values** — uninitialized memory holds whatever bits were left there by whatever used it last. Reading a variable before setting it gives unpredictable results.
- **`valgrind`** — a tool that runs your program and reports leaks and invalid memory accesses, pointing to the exact line. In C, `valgrind` is your best friend.

None of these errors *exist* in higher-level languages — not because those languages are smarter, but because they pay a runtime cost to handle memory for you. Learning them here is learning what that cost buys.

---

## 12. Files are just bytes

The lecture closes by applying pointers to **file I/O** — reading and writing files. A file is opened through a `FILE *` (a pointer, naturally):

```c
FILE *file = fopen("data.txt", "r");   // "r" read, "w" write, "a" append
if (file == NULL)                      // fopen returns NULL on failure
{
    return 1;
}
// ... fgetc / fputc / fread / fwrite / fprintf ...
fclose(file);                          // always close what you open
```

The deeper lesson mirrors Lecture 0: a file is *just bytes*. A JPEG, a BMP, a text file — all sequences of bytes you can read and manipulate directly. (This is what powers CS50's image problem sets: recovering deleted JPEGs by scanning raw bytes, or applying filters by editing pixel values one `struct` at a time.) File formats often describe their structure with **`struct`s** — custom compound types bundling related fields — which get their full treatment as the backbone of Lecture 5.

---

## 13. A wider lens: where pointers came from, and why they outlive C

It's natural to wonder whether all this pointer machinery is just a C-specific tax — something you'll never touch again once you're "off the metal." It isn't. The pointer is one of the most universal ideas in computing; what's specific to C is only how *nakedly* it exposes it.

The concept didn't originate with C. One memory cell holding the *address* of another is as old as the stored-program computer itself — indirect addressing existed in machine code and assembly before high-level languages did. Harold Lawson is generally credited with making the pointer a first-class *language* feature, in PL/I in the mid-1960s, and C (1972) inherited and popularized it rather than inventing it. The truly timeless idea underneath the syntax is **indirection**: referring to something by *where it is* rather than carrying a copy of it around. That idea is so foundational there's a well-worn aphorism for it, usually credited to David Wheeler — *any problem in computer science can be solved with another layer of indirection.*

Here's the part that answers your question directly: **higher-level languages didn't abolish pointers — they renamed them "references" and made them automatic and safe.** In Java, Python, JavaScript, C#, and Ruby, every object variable is a reference, which is a managed pointer under the hood. And because of that, the two traps you just learned reappear *verbatim* in languages that claim not to have pointers:

- The `t = s` trap — copying the address, not the data — is exactly why `b = a` on a list in Python makes both names refer to the *same* list, so mutating one changes the other. That's pointer aliasing wearing a friendly name.
- The `== compares addresses` trap is why Python distinguishes `is` (same object — identity, address-like) from `==` (same value), and why Java splits `==` from `.equals()`. It's the `strcmp` lesson, reincarnated.
- C's `NULL` is Python's `None`, Java's `null`, JavaScript's `null`/`undefined` — and the crashes they cause are the managed descendants of the segfault. Tony Hoare, who introduced null references back in 1965, later called it his "billion-dollar mistake," precisely because the problem propagated into nearly every language that followed.

So understanding pointers doesn't just help you write C — it gives you the *accurate mental model* for a whole class of bugs (shallow vs. deep copies, shared mutable state, identity vs. equality, null errors) in languages that supposedly spare you from them. You stop being surprised, because you can see the references underneath.

And pointers stay fully *explicit* well outside C, anywhere performance or control matters: C++ (with `unique_ptr`/`shared_ptr` "smart pointers" that automate the `free`), Go (explicit `&` and `*`, but with garbage collection), and most tellingly **Rust** — a modern language whose entire design, its ownership-and-borrowing system, is a direct reckoning with the exact hazards this lecture introduced (dangling pointers, use-after-free, leaks). Rust's whole pitch is essentially "pointers without the footguns." The fact that a language in wide use today is organized *around* solving these problems is the strongest evidence that they never went away — they just got managed.

---

## The big picture
Lecture 4 is the pivot of the whole course. Its one idea — **memory is addressed, and a pointer holds an address** — retroactively explains everything that came before:

1. **Hexadecimal** is just a compact, byte-aligned way to write the addresses.
2. **`&`** finds a variable's address; **`*`** follows an address to its value.
3. **A string is a `char *`** — the address of its first character. Every string quirk follows from this.
4. **`==` compares addresses**, so strings need `strcmp`; **`t = s` copies an address**, so real copies need `malloc` + `strcpy`.
5. **`malloc`/`free`** give you memory on the **heap** that outlives function calls; the **stack** holds function frames and overflows on runaway recursion.
6. **Segfaults, leaks, and garbage values** are the failure modes; **`valgrind`** catches them.
7. **Files are byte sequences**, accessed through `FILE *` pointers.

Here's the real payoff. The reason C forces all of this on you is so that the rest of the course makes sense. **Lecture 5 (Data Structures)** builds linked lists, trees, and hash tables — and every one of them is just structs connected by pointers, allocated with `malloc`. You can't build those without this lecture. And **Lecture 6 (Python)** will feel like a holiday precisely because it hides everything you just learned to do by hand. The wall was the point: climb it, and the view is the whole rest of programming.
