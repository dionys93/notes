# CS50 — Lecture 8: HTML, CSS, JavaScript (Deep Dive)
### Putting a program on the internet — structure, style, and behavior

Until now, your programs ran on one machine. This lecture is about the **web**: how data actually travels the internet, and the three languages that build everything you see in a browser. The key organizing idea is a clean division of labor — **HTML** provides structure, **CSS** provides style, and **JavaScript** provides behavior. Three languages, three jobs, kept separate. First, though, you need to know what happens between typing a URL and seeing a page.

---

## 1. How the internet actually works

A few protocols and systems make the web possible. The essentials:

- **IP addresses** — every device on the internet has a unique numeric address (like `192.0.2.10`), so data knows where to go.
- **DNS (Domain Name System)** — translates human-friendly names like `google.com` into those numeric IP addresses. It's a phone book for the internet — the exact same lookup idea from Lecture 0's phone-book demo, just at global scale.
- **TCP/IP** — the rules for chopping data into **packets**, addressing them, routing them across many networks, and reassembling them in order at the other end. IP handles *where*; TCP handles *reliable, in-order delivery* (and ports, which let one machine run many services at once).
- **HTTP / HTTPS** — HyperText Transfer Protocol, the language browsers and servers speak. Your browser sends a **request**; the server sends back a **response**. **HTTPS** is the same thing encrypted, so no one in between can read it.

Two pieces of HTTP you'll use constantly:

- **Methods** — chiefly **`GET`** (fetch a page or data) and **`POST`** (submit data, like a form). The distinction matters: `GET` puts parameters in the visible URL; `POST` sends them in the request body.
- **Status codes** — the server's summary of what happened: `200 OK`, `301/302` (redirect), `403 Forbidden`, the famous `404 Not Found`, and `500 Internal Server Error`. You've seen these in the wild without knowing they were HTTP.

---

## 2. HTML: structure

**HTML (HyperText Markup Language)** describes a page's *structure and content* — the skeleton. Crucially, it is **not a programming language**: there's no logic, no loops, no variables. It's *markup* — text wrapped in **tags** that label what each part is.

An element is usually an opening tag, content, and a closing tag: `<p>Hello</p>`. Tags can carry **attributes** for extra information: `<a href="https://example.com">link</a>`. Here's the standard skeleton every page shares:

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>My Page</title>
    </head>
    <body>
        <h1>Hello, world</h1>
        <p>Welcome to my site.</p>
    </body>
</html>
```

Notice the **nesting**: `<body>` contains an `<h1>` and a `<p>`; `<html>` contains `<head>` and `<body>`. That structure is a **tree** — the same data structure from Lecture 5 — and it has a name in the browser: the **DOM (Document Object Model)**, which becomes important the moment JavaScript enters. Common tags worth knowing: headings (`<h1>`–`<h6>`), paragraphs (`<p>`), links (`<a>`), images (`<img>`), lists (`<ul>`/`<ol>`/`<li>`), containers (`<div>`/`<span>`), and **forms** (`<form>`, `<input>`, `<button>`) — the last of which is how a page sends data back to a server.

---

## 3. CSS: style

Raw HTML is unstyled and plain. **CSS (Cascading Style Sheets)** controls *presentation* — colors, fonts, spacing, layout — keeping how a page *looks* separate from what it *is*. You write rules that target elements with **selectors** and set **properties**:

```css
h1 {
    color: navy;
    text-align: center;
}

.highlight {          /* a class selector — targets class="highlight" */
    background-color: yellow;
}

#logo {               /* an id selector — targets id="logo" */
    width: 200px;
}
```

The three selector types — by **element** (`h1`), by **class** (`.highlight`, reusable across many elements), and by **id** (`#logo`, a single unique element) — cover most needs. The "**cascading**" part is the rule for resolving conflicts: when multiple rules target the same element, *specificity* and order decide which wins.

Two practical notes the lecture emphasizes: keep CSS in a **separate file** (linked from the HTML) rather than tangled inline, so structure and style stay independent; and use **responsive design** (via media queries) so a page adapts to phone, tablet, and desktop. Frameworks like **Bootstrap** package professional-looking styles and a responsive grid you can drop in, so you don't hand-write everything.

---

## 4. JavaScript: behavior

HTML is structure and CSS is style — but both are static. **JavaScript** is the *programming* language of the browser, and it adds **behavior**: interactivity that responds to the user. Unlike the Python you'll write for the server (Lecture 9), JavaScript runs **client-side**, right inside the visitor's browser.

Its syntax is a familiar blend: C-family curly braces (Lecture 1) with dynamic, no-declared-types variables like Python (Lecture 6), using `let` and `const`. Its superpower is manipulating the **DOM** — that tree of page elements — *live*, after the page has loaded. It can find an element, change its text or style, or add and remove elements entirely, all without reloading the page.

It does this by responding to **events** — user actions like clicks, key presses, and form submissions:

```html
<button id="btn">Click me</button>

<script>
    document.querySelector('#btn').addEventListener('click', function () {
        alert('Hello!');
    });
</script>
```

That should feel oddly familiar: it's Scratch's **"when green flag clicked"** from Lecture 0, grown up. The whole course opened with event-driven blocks, and here the exact same idea — *when this happens, do that* — returns as real code driving real web pages. Everything dynamic on a modern site (live search, form validation, content updating in place) is JavaScript reacting to events and rewriting the DOM.

---

## 5. Three layers, three jobs

The unifying discipline of front-end development is keeping these three concerns **separate**:

- **HTML — structure.** The nouns: what content exists (a heading, a form, an image).
- **CSS — presentation.** The adjectives: how it looks (navy, centered, rounded).
- **JavaScript — behavior.** The verbs: what happens when the user acts (on click, submit, type).

You *can* tangle them together — inline styles, event handlers jammed into tags — but keeping each in its own place is what makes a site maintainable. Change the look without touching the structure; change the behavior without touching either. It's the abstraction and separation-of-concerns theme that's run through the whole course, now applied to the page.

---

## The big picture
Lecture 8 moves you from programs that run in one place to programs that live on the internet:

1. **The internet is protocols** — IP addresses locate machines, **DNS** maps names to them (a phone book), **TCP/IP** ferries the packets, and **HTTP(S)** carries the request/response conversation (with methods like `GET`/`POST` and status codes like `404`).
2. **HTML** is structure — a tree of tags (the **DOM**), and not a programming language.
3. **CSS** is style — selectors and properties, kept separate and responsive.
4. **JavaScript** is behavior — client-side code that manipulates the DOM in response to **events**.
5. **Three layers, three jobs** — structure, style, behavior — kept apart for sanity.

Notice how much of this rhymes with earlier lectures: DNS is the phone book, the DOM is a tree, JavaScript's events are Scratch's events, and its syntax is C wearing Python's flexibility. But there's a gap left open on purpose. Everything here is the **front end** — what runs in the browser. When that `<form>` gets submitted, *something on a server* has to receive it, look things up in a database (Lecture 7), and send back a response. That server-side half is the finale: **Lecture 9 (Flask)**, where Python ties the browser to the database and the whole application comes together.
