# CS50 — Lecture 2: Arrays
### What "compile" really means, and how strings are secretly arrays

Lecture 1 handed you a black box: you typed `make hello`, and a runnable program appeared. Lecture 2 opens that box — you learn the four real steps a compiler takes — and then introduces **arrays**: a way to store many values of the same type in a row. The lecture's big reveal is that **strings were arrays all along**, which explains a lot of C's quirks and sets up the harder Memory lecture to come. It all culminates in writing actual cryptography.

---

## 1. What "compile" really means — the four stages

Back in Lecture 1, "compiling" was one word for turning source code into a runnable program. In reality it's a **pipeline of four stages** — and this pipeline *is* the orthodox model: it's identical whether you invoke `clang`, `gcc`, or any other C toolchain. Historically each stage was even a separate Unix program (`cpp`, the compiler proper, `as`, `ld`); today one compiler command coordinates all four for you.

1. **Preprocessing** — anything starting with `#` is handled first. `#include <stdio.h>` is literally replaced with the relevant contents of that header file (the declarations for `printf` and friends), and `#define` macros get substituted. The preprocessor is essentially find-and-replace that runs before real compilation.
2. **Compiling** — your preprocessed C source is translated down into **assembly language**: low-level, CPU-specific instructions (`mov`, `push`, `call`, etc.) that are much closer to what the processor actually does. (Strictly, *this* stage alone is what "the compiler" does; the word is also used loosely for the whole pipeline.)
3. **Assembling** — the assembly is translated into **machine code**: the raw 0s and 1s (**object code**, typically `.o` files) the CPU executes. Now it's binary.
4. **Linking** — your object code is stitched together with the pre-compiled machine code of any libraries you used, producing one final executable. This is why pulling in an outside library adds a linker step: the linker resolves the reference and merges the library's actual compiled code in. (CS50's training-wheels library adds `-lcs50` for exactly this reason.)

Knowing the stages demystifies error messages, because they tell you *which* stage failed. A **compiler error** means an early stage rejected your source — bad syntax, an unknown type. A **linker error** (e.g., *undefined reference*) means stage 4 failed: the earlier stages understood your code fine, but the linker couldn't find the actual implementation of a function you called. Different stage, different fix.

---

## 2. Debugging like a programmer

Lecture 2 formalizes debugging into three tools, from crudest to most powerful:

- **`printf` debugging** — temporarily print the value of a variable to see what your program *actually* thinks is going on versus what you assumed. Simple, universal, always available.
- **The debugger (`debug50`)** — a tool that runs your program in slow motion. You set a **breakpoint** on a line, and execution pauses there so you can **step** through one line at a time, watching each variable's value change live. Far more surgical than scattering print statements.
- **Rubber-duck debugging** — explain your code aloud, line by line, to an inanimate object. The act of articulating *why* each line should work often surfaces the flawed assumption on its own.

The mindset shift the lecture pushes: bugs aren't failures, they're the normal state of programming. The skill isn't avoiding them — it's finding them efficiently.

---

## 3. Memory and arrays

Every variable lives in **memory** (RAM), and each type takes up a known amount of space — a direct callback to bits and bytes from Lecture 0:

| Type | Size |
|------|------|
| `char` | 1 byte |
| `int` | 4 bytes |
| `float` | 4 bytes |
| `long` / `double` | 8 bytes |
| `bool` | 1 byte |

Now suppose you want to store three test scores. You *could* declare `int score1, score2, score3;` — but that doesn't scale, and you can't loop over separately named variables. The fix is an **array**: a block of memory holding multiple values of the **same type**, laid out **contiguously** (back-to-back).

```c
int scores[3];        // reserve room for 3 ints, side by side in memory
```

The key facts about arrays:

- They're **indexed from 0**. For an array of size 3, the valid indices are `0`, `1`, `2` — *not* 1, 2, 3.
- Access an element with square brackets: `scores[0]`, `scores[1]`, `scores[2]`.
- The elements sit next to each other in memory, which is exactly what makes them fast and loop-friendly.

---

## 4. Working with arrays

Declare and fill in one step, or assign after:

```c
int scores[3] = {72, 73, 33};   // initialize all at once

scores[0] = 72;                 // or assign individually
scores[1] = 73;
scores[2] = 33;
```

Because array indices are just numbers, a **loop** can walk the whole thing — this is the entire reason arrays and loops are taught together:

```c
int sum = 0;
for (int i = 0; i < 3; i++)
{
    sum += scores[i];
}
```

### Off-by-one errors
The most common array bug is running off the end. An array of size `n` has indices `0` through `n − 1`. Writing `scores[3]` on a size-3 array reaches *past* the array into memory you don't own — sometimes garbage, sometimes a crash. C won't stop you; it trusts you to stay in bounds. Watch your loop conditions (`i < n`, not `i <= n`).

### Avoiding magic numbers
Hard-coding `3` all over your program is fragile — change the size and you must hunt down every `3`. Name the constant once instead:

```c
const int N = 3;          // a constant variable
#define N 3               // or a preprocessor macro (substituted before compiling)

int scores[N];
for (int i = 0; i < N; i++) { ... }
```

Now the size lives in one place. `const` makes a variable unchangeable; `#define` is a preprocessor substitution (recall step 1 above) that swaps `N` for `3` everywhere before compilation.

---

## 5. The big reveal: strings are arrays of characters

Here's the idea that reframes everything from Lecture 1. A **string is just an array of `char`s.** When you write:

```c
string s = "HI!";
```

what you really have is an array where:

```
s[0] = 'H'
s[1] = 'I'
s[2] = '!'
```

The `string` type from the CS50 library was hiding this. You can index into a string exactly like any array — `s[0]` is `'H'` — because that's literally what it is. (In **Lecture 4**, `string` gets stripped down even further to reveal what's underneath; for now, "array of chars" is the crucial upgrade.)

---

## 6. The null terminator: why "HI!" takes 4 bytes

If a string is just an array of characters sitting in memory, there's a problem: how does C know where the string *ends*? The array doesn't carry its own length around.

The answer is a **sentinel value** at the end — the **null terminator**, written `\0`: a single byte in which every bit is 0. Every C string is silently ended with it:

```
s[0] = 'H'
s[1] = 'I'
s[2] = '!'
s[3] = '\0'   ← the invisible terminator
```

So `"HI!"` occupies **4 bytes**, not 3. The `\0` is why C can tell where a string stops — anything that processes a string just reads characters until it hits `\0`. Forget that the terminator exists and off-by-one string bugs multiply. This single convention explains a huge amount of C's string behavior.

---

## 7. Working with strings

Because a string ends in `\0`, its length can be computed by counting characters up to that terminator. The `strlen` function (from `<string.h>`) does exactly that:

```c
#include <string.h>

string s = get_string("Name: ");
printf("%lu\n", strlen(s));      // number of chars, not counting '\0'
```

And you can iterate over a string like any array:

```c
for (int i = 0; i < strlen(s); i++)
{
    printf("%c\n", s[i]);
}
```

**A performance subtlety CS50 points out:** calling `strlen(s)` in the loop *condition* means it re-counts the entire string on every single iteration — turning a simple walk into wasted work. Compute it once instead:

```c
for (int i = 0, n = strlen(s); i < n; i++)
{
    printf("%c\n", s[i]);
}
```

It's a small change with an early lesson in efficiency — the kind of thing Big O (Lecture 3) makes rigorous.

---

## 8. Characters are numbers: ASCII manipulation

Recall from Lecture 0 that a `char` is stored as a number via ASCII (`'A'` = 65, `'a'` = 97). That means you can do **arithmetic on letters**. Notice that every lowercase letter sits exactly 32 above its uppercase counterpart:

```
'A' = 65      'a' = 97
'B' = 66      'b' = 98      → always a gap of 32
```

So you can convert a lowercase letter to uppercase by subtracting 32:

```c
char upper = c - 32;     // 'a' (97) becomes 'A' (65)
```

But hard-coding 32 is exactly the kind of magic number to avoid. The `<ctype.h>` library gives you clear, safe helpers instead:

- `toupper(c)` / `tolower(c)` — convert case
- `isupper(c)` / `islower(c)` / `isalpha(c)` / `isdigit(c)` — test what a character is

These are more readable and handle edge cases (non-letters) for you. The deeper point stands, though: to the computer, a character *is* a number, so text processing is just arithmetic.

---

## 9. Command-line arguments: argc and argv

Until now, programs got input only by prompting mid-run with `get_string`. But you can also accept input *when the program is launched*, straight from the command line. To do that, `main` takes two parameters:

```c
int main(int argc, string argv[])
```

- **`argc`** ("argument count") — how many words were typed to run the program, **including the program's own name**.
- **`argv[]`** ("argument vector") — an **array of strings** holding those words.

Run `./greet David` and you get:

```
argc      = 2
argv[0]   = "./greet"     ← the program name is always first
argv[1]   = "David"
```

So a program can read `argv[1]` to greet whoever was named on the command line:

```c
int main(int argc, string argv[])
{
    if (argc == 2)
    {
        printf("hello, %s\n", argv[1]);
    }
    else
    {
        printf("hello, world\n");
    }
}
```

Checking `argc` first matters — reaching for `argv[1]` when the user didn't provide it is another out-of-bounds read.

---

## 10. Exit status

Every program returns an integer **exit status** when it finishes — a status code the operating system (and other programs) can check:

- **`0`** conventionally means **success**.
- Any **non-zero** value signals **an error** (and the specific number can indicate *which* error).

That's why `main`'s return type is `int`. Return a non-zero value to bail out when something's wrong:

```c
int main(int argc, string argv[])
{
    if (argc != 2)
    {
        printf("Usage: ./greet name\n");
        return 1;              // non-zero: something went wrong
    }
    printf("hello, %s\n", argv[1]);
    return 0;                  // success
}
```

You normally don't *see* the status, but scripts and tools rely on it constantly to know whether a step succeeded before moving on.

---

## 11. Bringing it together: cryptography

The lecture's capstone combines arrays, strings, ASCII math, and command-line arguments into real **cryptography** — scrambling a message so only someone with the key can read it. The vocabulary:

- **Plaintext** — the original, readable message.
- **Key** — the secret used to scramble and unscramble it.
- **Cipher** — the algorithm that does the scrambling.
- **Ciphertext** — the scrambled output.

The classic example is the **Caesar cipher**: shift every letter forward by a fixed number (the key). With a key of 1, `A → B`, `B → C`, `HELLO → IFMMP`. To decrypt, shift back by the same key.

Look at how naturally the whole lecture converges on this one problem:

- The **key** arrives as a **command-line argument** (`argv[1]`).
- You **iterate over the string** of plaintext (arrays + `strlen`).
- You shift each letter with **ASCII arithmetic** (`ctype.h` to detect and preserve case, then add the key).
- You return an **exit status** to report bad input (e.g., a missing or non-numeric key).

That convergence is the point: every piece introduced in Lecture 2 is a component of a genuinely useful program. (Real modern cryptography is vastly more sophisticated — the Caesar cipher is trivially breakable — but the plaintext → cipher → ciphertext model is exactly right.)

---

## The big picture
Lecture 2's throughline is **"peek under the abstraction."**

1. **Compiling is four steps** — preprocess, compile, assemble, link — not magic.
2. **Arrays** store same-type values contiguously, indexed from 0, made powerful by loops.
3. **Strings are arrays of `char`s** — the CS50 `string` type was hiding that.
4. **The `\0` null terminator** marks a string's end, which is why `"HI!"` is 4 bytes and how `strlen` works.
5. **Characters are numbers**, so text manipulation is arithmetic.
6. **`argc` / `argv`** let programs take input at launch; the **exit status** lets them report success or failure.

Notice the recurring theme: things you took on faith in Lecture 1 (compiling, strings) turn out to be simpler structures underneath. That's the exact move Lecture 4 (Memory) makes next — pulling back the `string` curtain entirely to reveal the pointers beneath. Arrays are the on-ramp to that.
