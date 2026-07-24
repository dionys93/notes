# CS50 — Lecture 9: Flask
### The back end — where the whole course finally assembles into one application

This is the finale, and it's a convergence. Lecture 8 gave you the **front end** — the HTML, CSS, and JavaScript a visitor sees. But a page that submits a form needs *something on a server* to catch that submission, think about it, maybe save it to a database, and send a response back. That something is the **back end**, and you build it in **Flask**, a Python web framework. Flask is the wire that connects the browser (Lecture 8) to the database (Lecture 7) using the language you already know (Lecture 6). Everything you've learned meets here.

---

## 1. The client–server model

The web runs on a simple conversation. The **client** (the browser) sends an HTTP **request**; the **server** (your Flask program) sends back a **response**. The front end is what the client renders; the back end is the program on the server that decides *what* to send.

This split matters because the two run in different places and do different jobs. JavaScript (Lecture 8) runs on the *client*, in the visitor's browser. Flask/Python runs on the *server*, where you control the logic and hold the data. The back end is where secrets live (database passwords, other users' data) precisely because the client can't see it — only the response it's handed.

---

## 2. Flask and routes

A Flask app maps **URLs to Python functions**. Each mapping is a **route**, declared with a decorator above a function:

```python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("index.html")
```

When a browser requests `/`, Flask runs `index()` and sends back whatever it returns. Add more routes (`/about`, `/search`) and you've defined every page your app answers. This is just **functions** (Lecture 1) wearing a new hat: a URL comes in, a function runs, a response goes out — the input → algorithm → output model from Lecture 0, now spanning the internet.

---

## 3. GET vs POST: handling forms

Recall the two HTTP methods from Lecture 8: **`GET`** fetches a page, **`POST`** submits data. A single route can handle both — show a form on `GET`, process it on `POST`:

```python
@app.route("/register", methods=["GET", "POST"])
def register():
    if request.method == "POST":
        name = request.form.get("name")   # read what the user submitted
        # ...save it, look something up, etc...
        return render_template("success.html", name=name)
    # otherwise it's a GET — just show the blank form
    return render_template("register.html")
```

The `request` object is how the back end reads what the front end sent — `request.form.get("name")` pulls the value the user typed into the form field named `name`. This is the missing half from Lecture 8: that `<form>` you built now has somewhere to go.

---

## 4. Templates: generating HTML from Python

You could return HTML as a giant Python string, but that's a mess. Flask uses **templates** (via the **Jinja** engine) to keep HTML in its own files and inject data into them — separating logic (Python) from presentation (HTML), the same separation-of-concerns discipline from Lecture 8.

`render_template` passes Python values into an HTML file, where **`{{ }}`** inserts a value and **`{% %}`** runs logic:

```python
return render_template("greet.html", name="David", items=["a", "b", "c"])
```

```html
<!-- templates/greet.html -->
<h1>Hello, {{ name }}</h1>

<ul>
    {% for item in items %}
        <li>{{ item }}</li>
    {% endfor %}
</ul>
```

Look closely at that `{% for %}` loop: it's building HTML *dynamically*, one `<li>` per item. The loops and conditionals from Lecture 0 — the very first building blocks, first met as Scratch blocks — are now generating web pages on a server. The whole course has been circling back to those primitives, and here they are producing a live site.

---

## 5. Sessions and cookies: remembering users

Here's a subtle problem: **HTTP is stateless.** Each request is independent — the server has no built-in memory that *this* request came from the same person as the last one. So how does a site keep you logged in across many pages?

The answer is **cookies**: on your first visit, the server hands your browser a small token, and your browser sends it back with every subsequent request. That token lets the server recognize you. Flask wraps this in **sessions**, which let you store per-user data server-side:

```python
from flask import session

session["user_id"] = 1        # remember this user...
# ...and on a later request:
if session.get("user_id"):
    # they're logged in
```

Cookies and sessions are the clever workaround that makes a fundamentally forgetful protocol *feel* like it remembers you. Every "stay logged in" checkbox you've ever ticked is this.

---

## 6. Storing data: Flask meets SQL

A real app needs data that outlives a single request — accounts, posts, orders. That's the **database** from Lecture 7, now driven from Flask. Your route runs SQL to read and write persistent data:

```python
# store a new registration
db.execute("INSERT INTO users (name) VALUES (?)", name)

# look users back up later
rows = db.execute("SELECT * FROM users")
```

Notice the **placeholder** (`?`) — the parameterized query from Lecture 7, guarding against SQL injection. This is the moment the two back-end lectures fuse: Flask handles the request, SQL handles the storage, and the `?` keeps user input from becoming an attack.

---

## 7. APIs: talking to other programs

Sometimes a server's job isn't to return a *page* for a human but **data** for another program. An **API (Application Programming Interface)** is an endpoint that returns structured data — usually **JSON** — instead of HTML:

```python
from flask import jsonify

@app.route("/api/users")
def api_users():
    rows = db.execute("SELECT name FROM users")
    return jsonify(rows)     # returns data, not a web page
```

This is how apps talk to each other: a weather widget fetching a forecast, a phone app pulling your feed, JavaScript on the front end quietly requesting fresh data without reloading the page. Same request/response machinery — just program-to-program instead of server-to-human.

---

## The big picture: the whole course, assembled
Lecture 9 is where the layers stack into a complete application:

1. **Client–server** — the browser requests, the Flask server responds.
2. **Routes** map URLs to Python functions — functions from Lecture 1, spanning the internet.
3. **`GET`/`POST`** let a route show a form and then process it via the `request` object.
4. **Templates (Jinja)** generate HTML dynamically — the loops and conditionals from Lecture 0, building pages.
5. **Sessions and cookies** work around stateless HTTP to remember users.
6. **SQL from Flask** persists data, with placeholders guarding against injection (Lecture 7).
7. **APIs** return JSON so programs can talk to programs.
