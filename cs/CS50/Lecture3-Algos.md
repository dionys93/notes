# CS50 — Lecture 3: Algorithms (Deep Dive)
### Why *which* algorithm matters more than how fast your computer is

Lecture 0 previewed this with the phone book — tearing it in half beats flipping one page at a time. Lecture 3 makes that intuition rigorous. You get a vocabulary (**Big O**) for describing how an algorithm's cost grows as its input grows, you analyze searching and sorting through that lens, and you meet **recursion** — a function that calls itself. The single most important takeaway: a better algorithm can beat a faster computer by an unbounded margin, so the *choice of algorithm* is where the real leverage is.

---

## 1. The problem: searching

Start with a concrete task: given an array, is a particular value in it, and where? How you search depends entirely on whether the data is **sorted**, which turns out to be the whole story.

Think of lockers with numbers hidden inside, or names in a list. The naive approach and the clever approach differ enormously in cost — and that gap is what the lecture is really about.

---

## 2. Linear search

The straightforward method: check each element, one at a time, from the start until you find the target (or run out).

```
For each element, left to right:
    If it's the value I want, stop — found it.
If I reach the end, it's not here.
```

- It always works, sorted or not.
- **Worst case:** the value is last, or absent — you check all `n` elements.
- **Best case:** the value is first — you check just one.

Linear search is correct and simple, but it doesn't exploit any structure in the data. If the data *is* organized, we can do far better.

---

## 3. Binary search

If the array is **sorted**, you can use the phone-book trick: jump to the middle, decide which half the target must be in, throw the other half away, and repeat.

```
Look at the middle element.
If it's the target, done.
If the target is smaller, repeat on the left half.
If the target is larger, repeat on the right half.
```

Each step **halves** what's left to search. On a million-element array, linear search might take a million steps; binary search takes about **20**. Doubling the array size adds only *one* step.

**The catch — and it's the essential trade-off:** binary search requires the data to be **sorted first**. That precondition isn't free, and it dominates the rest of the lecture: sorting is the price of admission for fast searching.

---

## 4. Measuring efficiency: running time

To compare algorithms honestly, you don't time them with a stopwatch — hardware speed varies. Instead you count **how the number of steps grows as the input size `n` grows.** A stopwatch measures your laptop; step-growth measures the *algorithm*, which is what actually transfers.

The insight: for large inputs, only the **dominant term** matters. An algorithm taking `n² + 3n + 100` steps is, for big `n`, essentially `n²` — the lower-order terms and constants become noise. So we describe algorithms by that dominant growth shape, and nothing else.

---

## 5. Big O, Omega, and Theta

These three symbols formalize "how does cost grow." Orthodox definitions first, then the shorthand CS50 uses:

- **Big O — `O(...)`** — an **asymptotic upper bound** on growth: the cost grows *no faster than* this. In practice it's used to describe the **worst case**.
- **Big Omega — `Ω(...)`** — a **lower bound**: the cost grows *at least* this fast. Used to describe the **best case**.
- **Big Theta — `Θ(...)`** — a **tight bound**: used when the upper and lower bounds are the *same*, so best and worst behave alike.

A precise note worth keeping (you asked for orthodoxy over convention): strictly, O, Ω, and Θ are bounds on the *growth of a function*, not synonyms for "worst" and "best" case. You can put a Big-O bound on a best-case running time. CS50's "O = worst case, Ω = best case" is a teaching shorthand that works because the course pairs each case with its tightest bound. It's a fine mental model — just know the symbols themselves describe *bounds*, and the case (best/worst) is a separate axis.

---

## 6. The hierarchy of running times

These are the growth rates you'll meet constantly, from best to worst. Learn them by feel:

| Notation | Name | Feel |
|----------|------|------|
| `O(1)` | constant | instant, regardless of input size |
| `O(log n)` | logarithmic | gorgeous — binary search; doubling input adds one step |
| `O(n)` | linear | proportional — linear search |
| `O(n log n)` | linearithmic | the best *general* sorting can do |
| `O(n²)` | quadratic | nested loops; degrades fast |

The gaps are enormous at scale. For `n` = 1,000,000: `O(log n)` ≈ 20 steps, `O(n)` = a million, `O(n²)` = a *trillion*. This is why algorithm choice can dwarf hardware — no processor upgrade closes a gap like that.

Applying it to searching:
- **Linear search:** `O(n)`, `Ω(1)`
- **Binary search:** `O(log n)`, `Ω(1)`

---

## 7. Sorting: paying for fast search

Binary search needs sorted data — so how expensive is sorting? This is where the lecture spends its energy, because sorting is a perfect showcase for comparing algorithms that all produce the *same result* at wildly different costs.

The setup: given an unsorted array, arrange it in order. Three classic approaches follow, and they're worth studying not to memorize but to see *how* the same goal admits better and worse strategies.

The first two sorts share one small move — **swapping two array elements** — so it's worth seeing on its own. You need a temporary variable, because assigning one slot directly onto the other would clobber a value before you could rescue it:

```c
int tmp = list[j];        // stash the first value
list[j] = list[j + 1];    // overwrite it with the second
list[j + 1] = tmp;        // put the stashed value in the second slot
```

(In real code you'd factor this into a reusable `swap()` function — but doing that cleanly needs pointers, which arrive in Lecture 4. For now it's inlined.)

---

## 8. Bubble sort

Repeatedly walk the list, comparing **adjacent** pairs and swapping any that are out of order. Big values "bubble" toward the end pass by pass. Keep passing until a full pass makes no swaps.

- **Worst case:** `O(n²)` — up to `n` passes, each comparing ~`n` pairs.
- **Best case:** `Ω(n)` — if the list is already sorted, one clean pass with no swaps confirms it (assuming the "stop when no swaps" optimization).

```c
void bubble_sort(int list[], int n)
{
    for (int i = 0; i < n - 1; i++)
    {
        bool swapped = false;
        for (int j = 0; j < n - 1 - i; j++)   // last i elements are already in place
        {
            if (list[j] > list[j + 1])         // adjacent pair out of order?
            {
                int tmp = list[j];             // swap them
                list[j] = list[j + 1];
                list[j + 1] = tmp;
                swapped = true;
            }
        }
        if (!swapped)   // a whole pass with no swaps → already sorted, stop early
        {
            break;
        }
    }
}
```

The `swapped` flag is what earns the `Ω(n)` best case: on already-sorted input, the first pass makes zero swaps and the `break` bails out after a single sweep.

Simple to understand, but quadratic — impractical for large data.

---

## 9. Selection sort

Repeatedly **select the smallest** remaining element and swap it into the next sorted position. Scan the unsorted region for the minimum, place it, shrink the region, repeat.

- **Worst case:** `O(n²)`
- **Best case:** `Ω(n²)` — and here's the instructive part: it's *also* `n²` even on already-sorted input, because it always scans the entire remaining region to be *sure* it found the minimum. It can't shortcut. So selection sort is `Θ(n²)` — the same cost no matter what.

```c
void selection_sort(int list[], int n)
{
    for (int i = 0; i < n - 1; i++)
    {
        int min = i;                        // assume the smallest is at i...
        for (int j = i + 1; j < n; j++)     // ...then scan the rest to be sure
        {
            if (list[j] < list[min])
            {
                min = j;
            }
        }
        int tmp = list[i];                  // swap the true minimum into place
        list[i] = list[min];
        list[min] = tmp;
    }
}
```

Notice there's no early-exit flag here, and there *can't* be: the inner loop always runs to the end, because the only way to know you've found the minimum is to check every remaining element. That's why sorted input doesn't help — hence `Θ(n²)`.

The lesson from comparing 8 and 9: two algorithms can share a worst case yet differ in best case. Bubble sort can get lucky; selection sort never does.

---

## 10. Merge sort

The breakthrough. Instead of nibbling at the problem, **divide and conquer**:

```
If the list is 1 element, it's already sorted (stop).
Otherwise:
    Sort the left half.
    Sort the right half.
    Merge the two sorted halves into one sorted list.
```

The clever move is the **merge** step: combining two *already-sorted* halves is cheap and linear — just compare the fronts and take the smaller repeatedly.

```c
// Merge the two sorted halves list[lo..mid] and list[mid+1..hi] back together
void merge(int list[], int lo, int mid, int hi)
{
    int temp[hi - lo + 1];      // scratch space — the "extra memory" merge sort needs
    int i = lo, j = mid + 1, k = 0;

    while (i <= mid && j <= hi)          // take the smaller of the two fronts
    {
        if (list[i] <= list[j])
        {
            temp[k++] = list[i++];
        }
        else
        {
            temp[k++] = list[j++];
        }
    }
    while (i <= mid)                     // drain whatever remains in the left half
    {
        temp[k++] = list[i++];
    }
    while (j <= hi)                      // ...or the right half
    {
        temp[k++] = list[j++];
    }
    for (int x = 0; x < k; x++)          // copy the merged run back into place
    {
        list[lo + x] = temp[x];
    }
}

// Sort list[lo..hi]. Initial call: merge_sort(list, 0, n - 1)
void merge_sort(int list[], int lo, int hi)
{
    if (lo >= hi)               // base case: 0 or 1 elements are already sorted
    {
        return;
    }
    int mid = (lo + hi) / 2;
    merge_sort(list, lo, mid);          // sort the left half
    merge_sort(list, mid + 1, hi);      // sort the right half
    merge(list, lo, mid, hi);           // then merge the two sorted halves
}
```

- **Cost:** `Θ(n log n)` — the halving gives `log n` levels of division, and each level does `n` work to merge. That's dramatically better than `n²`.
- **The trade-off:** merge sort needs **extra memory** — that `temp` scratch array in `merge` is a second copy of the data being combined. This is the course's recurring **time vs. space** tension: you buy speed with memory. (Bubble and selection sort rearrange in place, using almost no extra space, but pay in time.)

Notice how much shorter and simpler `merge_sort` itself is than the actual work — it just splits, recurses, and delegates the real effort to `merge`. That's the divide-and-conquer shape, and it's exactly the recursion the next section formalizes.

`n log n` is provably the best any *comparison-based* sort can do in the general case — merge sort hits that ceiling.

---

## 11. Recursion

Merge sort works by calling itself on smaller halves — that's **recursion**: a function defined in terms of itself. Every correct recursion needs two parts:

- **Base case** — the simplest input, where the function stops calling itself and just returns. (In merge sort: a 1-element list is already sorted.) Without a base case, recursion never ends — the equivalent of an infinite loop, which eventually exhausts memory and crashes (a *stack overflow*, previewed here and explained in Lecture 4).
- **Recursive case** — the function does a little work, then calls itself on a **smaller** version of the problem, trusting that call to handle the rest.

A cleaner minimal example than sorting:

```c
int factorial(int n)
{
    if (n <= 1)          // base case
    {
        return 1;
    }
    return n * factorial(n - 1);   // recursive case: smaller problem
}
```

The mental shift recursion demands: instead of spelling out every step, you define the problem in terms of a smaller copy of *itself* and one stopping condition. It's the natural language for anything that repeatedly subdivides — like halving an array.

---

## 12. The real question: is sorting worth it?

Put the pieces together and a genuine engineering judgment emerges. Sorting costs `O(n log n)`; binary search costs `O(log n)`; linear search costs `O(n)`.

- **Searching once?** Don't bother sorting. `O(n)` linear search beats paying `O(n log n)` to sort *plus* `O(log n)` to search.
- **Searching many times?** Sort once up front, then every future lookup is a cheap `O(log n)` binary search. The one-time sorting cost amortizes away.

This is exactly why databases (Lecture 7) build sorted **indexes**: they pay the sorting cost once so that the thousands of queries afterward are each blazing fast. The trade-off you learn here scales straight up to real systems.

---

## 13. Zooming out: how algorithms are categorized

*(This section reaches past the lecture itself to situate what you just learned in the wider field — CS50 doesn't formalize this taxonomy, but the ideas below are where the discipline organizes it.)*

The lecture taught a handful of *specific* algorithms. Step back, and the whole field gets sorted along a few independent axes. The most illuminating one is by **design paradigm** — the underlying *strategy* — because there's a surprisingly short, stable list of them, and nearly every algorithm is an application or combination of these:

- **Brute force** — try every possibility. Simple, often slow. (Linear search is this.)
- **Divide and conquer** — split the problem into smaller copies, solve each, combine the results. *Both merge sort and binary search from this lecture are divide and conquer* — so you've already met the paradigm twice.
- **Greedy** — at each step, take the choice that looks best *right now* and hope it sums to the global best. Works for some problems (Dijkstra's shortest paths, Huffman compression), provably fails for others.
- **Dynamic programming** — break a problem into *overlapping* subproblems and store each result so it's never recomputed. (The cure for slow recursive Fibonacci, which recalculates the same values exponentially many times.)
- **Backtracking** — build a solution incrementally and abandon a path the instant it can't lead anywhere. (Sudoku solvers, the N-queens puzzle, maze-solving.)
- **Randomized** — deliberately use randomness as part of the method (randomized quicksort, probabilistic primality tests).

That's roughly the whole core toolkit — about half a dozen strategies. This is the "limited set" the field really does have.

A second axis is by **problem domain** — what *kind* of thing the algorithm operates on: sorting, searching, **graph** algorithms (networks, routes, dependencies), **string** algorithms (search, matching, alignment), **numerical** (arithmetic, linear algebra), **geometric** (shapes, collisions), **cryptographic**, and **machine-learning** algorithms. Same strategies, applied to different worlds.

A third axis — the deepest — is by **computational difficulty**, which sorts *problems* rather than algorithms:

- **P** — problems solvable in a reasonable (polynomial) amount of time. The "efficient" tier.
- **NP-complete** — problems where a proposed answer is quick to *check* but no known method finds one efficiently. The traveling salesman problem is the classic. Whether these secretly have efficient solutions (**P vs NP**) is the most famous open question in computer science.
- **Undecidable** — problems for which *no algorithm can ever exist*, proven impossible. The **halting problem** (deciding, in general, whether an arbitrary program will eventually stop or loop forever) is the canonical example.

### A finite toolkit, but unbounded solutions
Here's the subtle part, and it resolves the natural assumption that algorithms are a closed set waiting to be fully catalogued. Two things are true at once:

- The **strategies** are limited — that short paradigm list is stable and rarely grows.
- The **specific algorithms** are effectively unbounded and still expanding. Researchers publish genuinely new ones constantly, especially in newer domains like machine learning and quantum computing.

So it's *not* a periodic table you finish filling in. There are infinitely many possible problems, and — because of undecidability — some of them have no algorithm at all and provably never will. The accurate picture is: a small set of reliable strategies, an ever-growing library of specific solutions built from them, and a hard frontier beyond which no solution can exist.

---

## The big picture
Lecture 3 is where computer science stops being about *making the computer do something* and starts being about *making it do so efficiently*.

1. **Sorted data unlocks binary search** (`O(log n)`), which crushes linear search (`O(n)`) at scale.
2. **Big O / Ω / Θ** describe how cost *grows*, ignoring constants and hardware, so comparisons transfer between machines.
3. **Only the dominant term matters** for large `n`.
4. **Sorting is a masterclass in trade-offs** — bubble, selection, and merge sort reach the same result at `n²` vs `n log n` cost, and merge sort buys its speed with extra memory.
5. **Recursion** — base case plus a smaller self-call — is the natural tool for divide-and-conquer.
6. **Efficiency is a decision**: whether to sort first depends on how often you'll search.

The through-line to the rest of the course: the time-vs-space trade-off introduced by merge sort becomes the organizing question of **Lecture 5 (Data Structures)**, and the recursion and pointers hinted at here get their full treatment in **Lecture 4 (Memory)**. Algorithms is the lens; everything after is applying it.
