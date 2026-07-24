# CS50 — Lecture 5: Data Structures
### Where structs, pointers, and malloc finally build something

Lecture 4 was the wall; Lecture 5 is the reward for climbing it. Every tool you earned there — **structs** to bundle data, **pointers** to link things together, **malloc** to grow memory on demand — now combines to build structures that arrays alone can't. And the lens for judging them is Lecture 3's: each structure trades **time against space**, and choosing the right one for the job *is* the engineering. There's no "best" data structure — only the right one for what you're doing.

---

## 1. Structs: your own custom types

An array holds many values of *one* type. A **struct** does the opposite — it bundles values of *different* types into a single custom type you define:

```c
typedef struct
{
    string name;
    int age;
}
person;
```

Now `person` is a type you can use like any other. Access its fields with **dot notation**:

```c
person p;
p.name = "David";
p.age = 50;
```

A struct is a *container for related data* — a "person" is one thing made of a name and an age. That idea — grouping fields into a unit — is the seed of every structure in this lecture. The magic comes from one more move: **letting a struct hold a pointer, including a pointer to its own type.**

---

## 2. The limitation of arrays

Why not just use arrays for everything? Because an array's size is **fixed at creation**. It sits in one contiguous block of memory, and there's no guarantee the bytes right after it are free. So to "grow" an array you can't just extend it — you must:

1. `malloc` a new, bigger block,
2. copy every existing element over,
3. `free` the old one.

That copy is `O(n)` every time you outgrow the array. Arrays give you instant `O(1)` random access by index — their great strength — but they're rigid. What we want is a structure that can **grow and shrink cheaply**. That requires giving up contiguous memory, and the tool for that is the pointer.

---

## 3. Linked lists

A **linked list** stitches together individual **nodes** that can live anywhere in memory, each one pointing to the next. The node is a struct that contains its data *and* a pointer to another node of the same type:

```c
typedef struct node
{
    int number;
    struct node *next;   // a pointer to the next node
}
node;
```

One genuine subtlety: notice the struct is *named* `struct node`, not anonymous. It has to be, because the `next` field refers to the struct's own type *before* the `node` alias finishes being defined — so you can't use the short name yet inside itself.

Working with nodes introduces the **arrow operator** `->`, which is just shorthand for "dereference, then access a field." `n->number` means exactly `(*n).number`:

```c
node *n = malloc(sizeof(node));   // make a node on the heap
if (n == NULL)
{
    return 1;
}
n->number = 5;      // set its data   —  same as (*n).number
n->next = list;     // point it at the current front of the list
list = n;           // and make it the new front
```

Prepending like that is **`O(1)`** — no shifting, no copying, however long the list is. And the list grows one node at a time with no wasteful reallocation. To visit every element you *traverse*, following `next` until you hit `NULL`:

```c
for (node *tmp = list; tmp != NULL; tmp = tmp->next)
{
    printf("%i\n", tmp->number);
}
```

**The trade-off** is the mirror image of the array's. You gained cheap growth; you *lost* random access. There's no `list[5]` — to reach the sixth node you must walk the five before it, so search is `O(n)` and you can't binary-search even if the data's sorted (binary search needs instant indexing). You also spend extra memory on a pointer per node. Array vs. linked list is the foundational trade-off of the whole lecture: **fast lookup vs. easy resizing.**

---

## 4. Abstract data types: stacks and queues

Sometimes what matters isn't *how* a structure is built but *how it behaves*. An **abstract data type (ADT)** describes behavior — the rules for putting things in and taking them out — and leaves the implementation open (you could build either of these on an array *or* a linked list):

- **Stack** — **LIFO**, Last In, First Out. Like a stack of plates: you add to the top and remove from the top. Operations: **push** (add) and **pop** (remove the most recent). Think browser back button, or undo.
- **Queue** — **FIFO**, First In, First Out. Like a line at a store: first to arrive is first served. Operations: **enqueue** (add to the back) and **dequeue** (remove from the front).

The point of an ADT is the separation between the *contract* (a stack is LIFO) and the *mechanism* (array or linked list underneath). Same behavior, swappable guts — which is exactly the abstraction idea from Lecture 0, applied to data.

---

## 5. Trees and binary search trees

A linked list points forward in a single line. Let each node point in **two** directions and you get a **tree**. The star example is the **binary search tree (BST)**: every node has a `left` and `right` child, arranged so that everything to the left is smaller and everything to the right is larger.

```c
typedef struct node
{
    int number;
    struct node *left;
    struct node *right;
}
node;
```

That ordering is the whole trick. To search, you start at the top (**root**) and go left or right by comparing — the same halving idea as binary search from Lecture 3, but now in a structure you can also **grow cheaply** like a linked list. A BST is the best of both worlds it was chasing: `O(log n)` search *and* dynamic size.

The catch — an honest one: that `O(log n)` holds only if the tree is **balanced** (roughly even on both sides). Insert already-sorted data naively and the tree degenerates into a single long line — effectively a linked list — and search collapses back to `O(n)`. Keeping trees balanced is a real topic of its own (and why databases use self-balancing B-trees, Lecture 7).

---

## 6. Hash tables

The **hash table** is the workhorse of practical programming — the structure behind Python's dictionaries and JavaScript's objects. Its goal is the holy grail: near-instant, `O(1)` lookup.

It works by combining an array with a **hash function** — a function that takes a key and computes an array index from it:

```c
// A trivially simple hash: first letter of a word → bucket 0–25
unsigned int hash(string word)
{
    return toupper(word[0]) - 'A';
}
```

To store "apple," you hash it to an index and drop it in that slot (**bucket**). To look it up later, you hash it *again* — landing at the same slot instantly, without scanning. That's the `O(1)`.

The complication is **collisions**: two different keys hashing to the *same* bucket ("apple" and "avocado" both starting with A). The standard fix is **separate chaining** — each bucket isn't a single slot but the head of a **linked list**, so colliding entries just chain together.

Here's the whole thing assembled — and notice it's built entirely from the pieces you already have, an array of the linked-list nodes from section 3:

```c
#define BUCKETS 26        // one bucket per starting letter, A–Z
#define LENGTH  45        // max word length

// Each node stores its OWN copy of the word, plus a pointer to the next node in the chain
typedef struct node
{
    char word[LENGTH + 1];
    struct node *next;
}
node;

// The table: an array of BUCKETS chains. As a global, every bucket starts NULL (empty).
node *table[BUCKETS];

// Hash a word to a bucket number in the range 0–25, by its first letter
unsigned int hash(string word)
{
    // toupper(word[0]) is the first letter, forced to uppercase: 'A'..'Z'.
    // But letters are ASCII numbers ('A' is 65, 'Z' is 90), and our array only
    // has indices 0..25. Subtracting 'A' shifts a letter to its alphabet position:
    //     'A' - 'A' == 0,   'B' - 'A' == 1,   ...   'Z' - 'A' == 25
    // giving a valid bucket index. (Plain char arithmetic — see Lecture 2.)
    return toupper(word[0]) - 'A';
}

// Insert a word into the table
bool insert(string word)
{
    node *n = malloc(sizeof(node));   // a node on the heap
    if (n == NULL)                    // malloc can fail — always check
    {
        return false;
    }
    strcpy(n->word, word);            // copy the word INTO the node (Lecture 4: a real copy)

    unsigned int i = hash(word);      // find its bucket...
    n->next = table[i];               // ...point the node at the chain's current head...
    table[i] = n;                     // ...and make it the new head — O(1) prepend
    return true;
}

// Is a word in the table?
bool contains(string word)
{
    unsigned int i = hash(word);                                 // hash once to pick the bucket
    for (node *tmp = table[i]; tmp != NULL; tmp = tmp->next)     // then walk only that chain
    {
        if (strcmp(tmp->word, word) == 0)   // compare CONTENTS, not addresses (Lecture 4: strcmp, not ==)
        {
            return true;
        }
    }
    return false;
}
```

Trace the speed through `contains`: you hash the word *once* to jump straight to one bucket, then search only that bucket's short chain — never the whole dataset. If the hash spreads words evenly, each chain stays tiny and lookup is effectively `O(1)`. Two details are Lecture 4 paying off directly: `strcpy` gives each node its *own* copy of the word (so nothing dangles), and `contains` uses `strcmp` rather than `==`, because `==` would compare addresses instead of characters.

Notice the payoff: a hash table is *made of* the earlier structures — an array of linked lists. Its performance lives and dies by the hash function: a good one spreads keys evenly and keeps chains short (fast); a bad one dumps everything into one bucket, degrading to a single `O(n)` linked list (which is why this first-letter hash is only a teaching toy — real hash functions mix in every character). The trade-off here is **speed bought with memory and complexity** — you allocate a big array and accept a fiddlier implementation in exchange for that `O(1)`.

---

## 7. Tries

The **trie** (from re*trie*val, usually said "try") pushes the speed/space trade-off to its extreme. It's a tree where the *path* spells out the key: each level corresponds to one character of the key, and each node holds an array of pointers — one for every possible next character.

To look up a word, you follow one pointer per letter. The remarkable result: lookup time depends only on the **length of the key**, *not* on how many items are stored. A trie with a hundred words and one with ten million words take the same time to find a five-letter word — effectively `O(1)` in the number of entries. Nothing beats it for speed.

The price is brutal **memory** usage. Every node carries a full array of child pointers (26 for lowercase letters, more if you allow apostrophes, capitals, etc.), the vast majority sitting empty. A trie can consume enormous space to hold a modest dictionary. It's the purest illustration of the lecture's thesis: you can almost always buy more speed with more memory, and vice versa — the art is knowing which you can afford.

---

## The big picture
Lecture 5 is where the course's two big threads converge: Lecture 4's *mechanics* (structs, pointers, malloc) build the structures, and Lecture 3's *analysis* (Big O, time vs. space) tells you which to reach for. Every structure is a different point on the same trade-off curve:

| Structure | Search | Insert | The trade |
|-----------|--------|--------|-----------|
| Array (unsorted) | `O(n)` | `O(1)`* | instant indexing, but fixed size |
| Sorted array | `O(log n)` | `O(n)` | fast search, slow to modify |
| Linked list | `O(n)` | `O(1)` | grows freely, but no indexing |
| Binary search tree | `O(log n)`† | `O(log n)`† | fast *and* flexible — if balanced |
| Hash table | `O(1)` avg | `O(1)` avg | blazing fast, hungry for memory |
| Trie | `O(k)` | `O(k)` | speed independent of size, huge memory |

<sub>*at the end, ignoring resizes.  †when balanced; degrades to `O(n)` otherwise. `k` = length of the key.</sub>

The lasting lesson: **there is no universally best data structure.** A hash table's speed costs memory; a linked list's flexibility costs lookup; a trie's constant-time search costs a fortune in space. Choosing well means knowing your actual workload — how often you search versus insert, how much memory you have, whether the data is sorted.

The reach beyond this lecture is everywhere. **Lecture 6 (Python)** hands you these structures pre-built and polished — `list`, `dict`, `set` are a dynamic array, a hash table, and a hash set you get for free. **Lecture 7 (SQL)** scales the tree idea into the B-tree indexes that make databases fast. You spent Lecture 4 learning to build these by hand precisely so that, when a language hands you a `dict`, you understand exactly what you're holding.
