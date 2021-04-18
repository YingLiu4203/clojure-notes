# Calva

## 1 Basic Command

- Jack in: `ctrl+alt+c ctrl+alt+j`
- Load and evaluate current file: `ctrl+alt+c enter`

- Evaluate current form whe cursor behind a form: `ctrl+enter`
- Evaluate current top level form: `alt+etner`
- Dismiss the display of results: `escape`
- Instrument for debugging: `ctrl+alt+c i`

- Evaluate current form and add result as comment: `ctrl+alt+c c`
- Evaluate current top form and add result as comment: `ctrl+alt+c space`
- Evaluate code and replace it in the editor, inline: `ctrl+alt+c r`

- Paste the current form to REPL window: `ctrl+alt+c ctrl+alt+e`
- Paste the current top-level form to REPL window: `ctrl+alt+c ctrl+alt+space`
- Copy last evaluation results to buffer: `ctrl+alt+c ctrl+c`

- Toggle pretty printing: `ctrl+alt+c p`

## 2 Paredit

LISP isn't line or chracter oriented, it is based on S-expressions, a.ak.a forms.

- `backspace`: deletes chars backward while keeping the structure.
- `delete` delete chars forward while keeping the structure.
- `alt+backspace` or `shift+backspace`: delete backward without keeping structure.
- `alt+delete` or `shift+delete`: delete forward without keeping structure.

The default strict mode keeps structures balanced. Use `ctrl+alt+p ctrl+alt+m` to toggle it. The Paredit commands are sorted into Navigation, Selection, and Edit. strings are treated in much the same way as lists are.

For visual effect, check [Calva Paredit](https://calva.io/paredit/)

### 2.1 Navigation

- Forward Sexp: `alt+right`
- Backward Sexp: `alt+left`

- Forward down Sexp: `ctrl+down`
- Backward down Sexp: `ctrl+alt+up`
- Forward up Sexp: `ctrl+alt+down`
- Backward up Sexp: `ctrl+up`

- Foward to list end: `ctrl+end`
- Backward to list start: `ctrl+home`

### 2.2 Selecting

- Select the current form, then expan to enclosing forms: `ctrl+w`
- Shrink selection: `ctrl+shift+w`
- Select top level form: `ctrl+alt+w space`

Add a `shift` to the same navigation key combination to select:

- Select forward sexp: `shift+alt+right`
- Select backward sexp: `shift+alt+left`

- Select forward down sexp: `shift+ctrl+down`
- Select backward down sexp: `shift+ctrl+alt+up`
- Select forward up exp: `shift+ctrl+alt+down`
- Select backward up exp: `shift+ctrl+up`

- Select forward to list end: `ctrl+shift+end`
- select backward to list start: `ctrl+shift+home`

### 2.3 Editing

- Slurp forward: `ctrl+alt+right`
- Barf forward: `ctrl+alt+left`
- Slurp backward: `shift+ctrl+alt+left`
- Barf backward: `shift+ctrl+alt+right`

- Splice Sexp: `ctrl+alt+s`
- Split Sexp: `shift+ctrl+s`

- Join Sexps/Forms: `ctrl+shift+j`
- Raise Sexp: `ctrl+alt+p ctrl+alt+r`
- Transpose (swap) Sexps/Forms surrounding the cursor: `ctrl+alt+t`

- Drag Sexp backward: `ctrl+alt+shift+b`
- Drag Sexp forward: `ctrl+alt+shift+f`
- Drag Sexp backward up: `ctrl+alt+shift+u`
- Drag Sexp forward down: `ctrl+alt+shift+d`
- Drag Sexp forward up: `ctrl+alt+shift+k`
- Drag Sexp backward down: `ctrl+alt+shift+j`

- Convolute (swap two outer forms): `ctrl+shift+c`

- Kill Sexp forward: `ctrl+shift+delete`
- Kill Sexp backward: `ctrl+shift+backspace`
- Kill list forward: `ctrl+delete`
- Kill list backward: `ctrl+backspace`
- Splice killing list forward: `ctrl+alt+shift+delete`
- Splice killing list backward: `ctrl+alt+shift+backspace`

- Wrap around with `()/[]/{}/""`: `ctrl+alt+shift+p/s/c/q`
- Rewrap around with `()/[]/{}/""`: `ctrl+alt+r ctrl+alt+p/s/c/q`

## 3 Output, Test and Format

### 3.1 REPL output window

- `ctrl+alt+c o`: open the REPL window or open the file of current namespace.
- `ctrl+alt+c alt+n`: sync to current namespace.
- `alt+enter`: evaluate expression.
- `alt+up`: prev in history.
- `alt+down`: next in history.
- `cmd + click`: on the filename/symbol to open/peek definition

REPL output is saved to a session-temprary file `.calva/output-window` in the project root.

### 3.2 Test

- Debug: `ctrl+alt+c i`
- Run ns tests: `ctrl+alt+c t`
- Run all tests: `ctrl+alt+c shift+t`
- Run current tests: `ctrl+alt+c ctrl+alt+t`
- Run previously failing tests: `ctrl+alt+c ctrl+t`

### 3.3 Format

Calva uses [cljfmt](https://github.com/weavejester/cljfmt) to format file according to [Bozhidar Batsov's Clojure Style Guide](https://github.com/bbatsov/clojure-style-guide).

It formats forms as you type/paste code. Format the select form when you hit `tab`.
