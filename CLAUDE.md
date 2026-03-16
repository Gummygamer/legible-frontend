# CLAUDE.md — Legible Frontend Framework

## Project Overview

A **web frontend framework** written in the **Legible** programming language (`.lbl` files). The framework generates HTML, CSS, and JavaScript from Legible source code, served by the Legible HTTP server builtins. It follows a **server-side rendering** model where the backend generates complete HTML pages and handles interactivity through form submissions and HTMX-style partial page updates.

This is a companion to `../legible-backend/` and relies on the same interpreter at `../legible/`.

## Architecture: Server-Rendered Components

Since Legible is synchronous and has no browser runtime, the framework uses a **server-side component model**:

1. **Components** are functions that take props (a record) and return HTML text
2. **Pages** compose components into full HTML documents
3. **Interactivity** uses HTML forms + server round-trips, with optional HTMX attributes for partial updates
4. **State** lives on the server, passed through request/response cycles
5. **Styling** uses a utility-first CSS approach generated from Legible data structures

This is similar to PHP/Jinja templates or Rails ERB, but with Legible's type safety and pipeline ergonomics.

## Project Structure

```
legible-frontend/
├── CLAUDE.md
├── lib/                      # Framework library modules (.lbl)
│   ├── html.lbl              # HTML element builder functions
│   ├── css.lbl               # CSS utility class generation
│   ├── page.lbl              # Full HTML page scaffold (head, body, scripts)
│   ├── component.lbl         # Component composition helpers
│   ├── form.lbl              # Form builder with CSRF and validation
│   ├── htmx.lbl              # HTMX attribute helpers for partial updates
│   └── escape.lbl            # HTML/URL escaping for XSS prevention
├── examples/
│   ├── hello_page.lbl        # Minimal HTML page
│   ├── todo_app.lbl          # Full CRUD todo app with forms
│   └── dashboard.lbl         # Multi-component dashboard page
└── tests/
    └── fixtures/             # Test programs and expected outputs
```

## Interpreter Dependencies

The framework uses these existing builtins from `../legible/`:

### Required (already exist)
- **HTTP**: `http_start`, `http_next_request`, `http_respond_with_headers`, `http_stop`
- **JSON**: `json_parse`, `json_encode`
- **Text**: `replace`, `substring`, `contains_text`, `index_of`, `split`, `join`, `trim`, `starts_with`, `ends_with`, `text_length`, `uppercase`, `lowercase`
- **Collections**: `length`, `append`, `concat`, `contains`, `filter`, `map`, `reduce`, `range`, `keys`, `values`, `has_key`, `get`, `put`
- **File I/O**: `read_file`, `file_exists`
- **Utilities**: `current_time_ms`, `log`, `to_text`

### Needed (to be added to interpreter)
- `url_decode(str: text): text` — Percent-decode URL-encoded strings (for form bodies)
- `random_hex(length: integer): text` — Generate random hex string (for CSRF tokens)

## Core Design Principles

1. **HTML as text** — Components return `text` containing HTML. No virtual DOM, no diffing. Simple string concatenation with `++` and pipelines.
2. **Security by default** — All user content is HTML-escaped before rendering. Raw HTML must be explicitly opted into.
3. **Composable functions** — Components are just functions. Compose with pipelines and higher-order functions.
4. **No client-side Legible** — JavaScript is used sparingly and only as inline strings for progressive enhancement.
5. **Pipeline-friendly API** — Element builders chain naturally: `div(attrs, children |> join(""))`.

## Framework Modules

### `html.lbl` — HTML Element Builders

Core function for creating HTML elements:

```legible
public function element(tag: text, attrs: a mapping from text to text, content: text): text
  intent: create an HTML element with the given tag attributes and content
  -- produces: <tag attr="val" ...>content</tag>
end

public function void_element(tag: text, attrs: a mapping from text to text): text
  intent: create a self-closing HTML element
  -- produces: <tag attr="val" ... />
end
```

Convenience wrappers for common elements:
- Block: `div`, `section`, `article`, `header`, `footer`, `nav`, `main_content`, `aside`
- Inline: `span`, `strong`, `em`, `a_link`, `code_element`
- Text: `h1` through `h6`, `p`, `pre`, `blockquote`
- Lists: `ul`, `ol`, `li`
- Media: `img`, `video`, `audio`
- Table: `table`, `thead`, `tbody`, `tr`, `th`, `td`
- Form: `form`, `input`, `textarea`, `select`, `option`, `button`, `label`

### `css.lbl` — CSS Utilities

```legible
public function style_block(rules: a list of text): text
  intent: wrap CSS rules in a style element
end

public function class_list(classes: a list of text): text
  intent: join class names with spaces for the class attribute
end
```

Provides a small set of utility CSS rules as text constants for common patterns (spacing, flex, grid, colors).

### `page.lbl` — Page Scaffold

```legible
public function page(title: text, head_extra: text, body_content: text): text
  intent: create a complete HTML5 page with doctype head and body
end

public function page_with_htmx(title: text, head_extra: text, body_content: text): text
  intent: create an HTML5 page that includes the HTMX library
end
```

### `component.lbl` — Component Composition

```legible
public function render_list(items: a list of text, wrapper: fn(text): text): text
  intent: render each item through a component function and join results
end

public function render_if(condition: boolean, content: text): text
  intent: conditionally render content or return empty string
end

public function render_each(items: a list of T, renderer: fn(T): text): text
  intent: map items through a renderer function and join the HTML
end
```

### `form.lbl` — Form Builders

```legible
public function form_post(action: text, fields: text): text
  intent: create a POST form with CSRF token and fields
end

public function text_input(name: text, label_text: text, value: text): text
  intent: create a labeled text input field
end

public function parse_form_body(body: text): a mapping from text to text
  intent: parse URL-encoded form body into key-value pairs
end
```

### `htmx.lbl` — HTMX Attribute Helpers

```legible
public function hx_get(url: text): a mapping from text to text
  intent: create HTMX attributes for a GET request
end

public function hx_post(url: text): a mapping from text to text
  intent: create HTMX attributes for a POST request
end

public function hx_target(selector: text): a mapping from text to text
  intent: create HTMX target attribute
end

public function hx_swap(strategy: text): a mapping from text to text
  intent: create HTMX swap strategy attribute
end
```

### `escape.lbl` — Security

```legible
public function html_escape(content: text): text
  intent: escape HTML special characters to prevent XSS
  -- Escapes: & < > " '
end

public function url_encode_value(value: text): text
  intent: encode a value for safe inclusion in URLs
end
```

## Development Workflow

1. Make interpreter changes in `../legible/` if needed
2. Run `cd ../legible && cargo test` to verify interpreter
3. Run `cd ../legible && cargo run -- run ../legible-frontend/examples/hello_page.lbl` to test
4. Framework `.lbl` files reference each other via `use` imports (module = filename)

## Running Examples

```bash
# From the legible interpreter directory:
cd ../legible

# Run a frontend example:
cargo run -- run ../legible-frontend/examples/hello_page.lbl

# Then open http://localhost:8080 in a browser
```

## Conventions

- Follow all Legible language conventions from `../legible/CLAUDE.md`
- Every function has an `intent:` line
- Use pipelines for composing HTML fragments
- All user-provided content must go through `escape.html_escape()` before rendering
- Records are immutable — use `with` for updates
- Component functions: `(props: SomeRecord): text` signature (returns HTML)
- Page handlers: `(method: text, path: text, body: text, query: text, headers: a mapping from text to text): Response` signature
