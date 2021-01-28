# Calva

## 1 Basic Command

- Jack in: `ctrl+alt+c ctrl+alt+j`

- Grow/expand slection: `ctrl+w`
- Select current form: `ctrl+alt+c ctrl+s`
- Select current top form: `ctrl+alt+w space`

- Evaluate current form whe cursor behind a form: `ctrl+enter`
- Evaluate current top level form: `alt+etner`
- Dismiss the display of results: `escape`
- Evaluate current form and add result as comment: `ctrl+alt+c c`
- Evaluate current top form and add result as comment: `ctrl+alt+c space`
- Evaluate code and replace it in the editor, inline: `ctrl+alt+c r`

- Paste the current form to REPL window: `ctrl+alt+c ctrl+alt+e`
- Paste the current top-level form to REPL window: `ctrl+alt+c ctrl+alt+space`
- Copy last evaluation results to buffer: `ctrl+alt+c ctrl+c`

- Load and evaluate current file: `ctrl+alt+c enter`
- Toggle pretty printing: `ctrl+alt+c p`

## 2 Paredit

LISP isn't line or chracter oriented, it is based on S-expressions, a.ak.a forms.

- `backspace`: deletes chars backward while keeping the structure.
- `delete` delete chars forward while keeping the structure.
- `alt+backspace` or `shift+backspace`: delete backward without keeping structure.
- `alt+delete` or `shift+delete`: delete forward without keeping structure.

The default strict mode keeps structures balanced. Use `ctrl+alt+p ctrl+alt+m` to toggle it. The Paredit commands are sorted into Navigation, Selection, and Edit. strings are treated in much the same way as lists are.

## 3 Output, Test and Format

### 3.1 REPL output window

REPL history

- up: `alt+up`
- down: `alt+down`

REPL output is saved to a session-temprary file `.calva/output-window` in the project root.

- Use `cmd + click` on the filename/symbol to open/peek definition

### 3.2 Test

- Debug: `ctrl+alt+c i`
- Run ns tests: `ctrl+alt+c t`
- Run all tests: `ctrl+alt+c shift+t`
- Run current tests: `ctrl+alt+c ctrl+alt+t`
- Run previously failing tests: `ctrl+alt+c ctrl+t`

### 3.3 Format

Calva uses [cljfmt](https://github.com/weavejester/cljfmt) to format file according to [Bozhidar Batsov's Clojure Style Guide](https://github.com/bbatsov/clojure-style-guide).

It formats forms as you type/paste code. Format the select form when you hit `tab`.
