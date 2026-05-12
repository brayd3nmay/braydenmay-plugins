# LaTeX Formatting Reference

Used by the refine-jakes-resume skill when the user shares a `.tex` source file rather than rendered PDF text. These are general LaTeX rules — they apply to any `.tex` document, not just Jake's Resume.

When editing a `.tex` file directly, follow these conventions or the document won't compile (or will render strangely).

## Special characters that must be escaped

**Before adding escapes, check whether the character is already escaped.** A `.tex` file you're editing may already contain `\%` or `\$` — adding another backslash (`\\%`) double-escapes and breaks compilation. Only escape unescaped characters; leave already-escaped ones alone.

| Character | Escape | Resume context |
|---|---|---|
| `%` | `\%` | `50\%` improvement |
| `$` | `\$` | `\$10K` raised |
| `&` | `\&` | `AT\&T`, `R\&D` |
| `#` | `\#` | `\#1` ranked |
| `_` | `\_` | `user\_data` (rare; most code identifiers don't appear on resumes) |
| `{` `}` | `\{` `\}` | rare in resume context |
| `\` | `\textbackslash` | rare in resume context |

**The big three: `%`, `$`, `&`.** These are the most common silent compile-breakers in resume bullets. Always escape them.

`+`, `=`, `*`, `<`, `>`, `/`, `(`, `)`, `[`, `]`, `?`, `!` do not need escaping. `C++` is fine as-is.

## Dashes

LaTeX has three different dash characters. Using the wrong one looks unprofessional.

| Want | Type | Example |
|---|---|---|
| Hyphen (intra-word) | `-` | `data-driven`, `full-stack`, `end-to-end` |
| En-dash (ranges) | `--` | `May 2024 -- Aug 2024`, `pp. 10--15` |
| Em-dash (parenthetical) | `---` | `the result --- a 50\% gain --- was clear` |

**For resumes: always use `--` (en-dash) in date ranges.** `2024 - 2025` renders as a too-short hyphen and looks wrong.

## Quotes — don't use `"`

Straight double-quotes render incorrectly in LaTeX (as two left-quotes or backwards). Use:

| Want | LaTeX | Renders as |
|---|---|---|
| Single open | `` ` `` (backtick) | ' |
| Single close | `'` (apostrophe) | ' |
| Double open | `` `` `` (two backticks) | " |
| Double close | `''` (two apostrophes) | " |

Apostrophes in possessives (`user's`, `Jake's`) use plain `'` — only quote pairs need the asymmetric syntax.

## Bold, italic, emphasis

| Command | Renders | Use for |
|---|---|---|
| `\textbf{text}` | **text** | Job titles, company names, project names |
| `\emph{text}` | *text* | Tech stack inline (e.g., `\emph{Python, Flask, React}`) |
| `\textit{text}` | *text* | Same as emph; slightly different semantics |
| `\underline{text}` | underlined | Hyperlink display text |
| `\texttt{text}` | `text` | Code/monospace (rare on resumes) |

**Prefer `\emph` over `\textit`** because `\emph` nests correctly — italic inside italic toggles back to upright, which is the right behavior.

## Hyperlinks

Requires `\usepackage{hyperref}` in the preamble (already present in Jake's Resume).

```latex
\href{https://github.com/jake}{github.com/jake}
\href{mailto:jake@x.com}{jake@x.com}
\href{https://linkedin.com/in/jake}{linkedin.com/in/jake}
```

First argument is the URL; second is the visible text. Usually paired with `\underline{...}` on the visible text to make it look like a link.

## Spacing and line breaks

| Goal | Syntax |
|---|---|
| Force line break (within paragraph) | `\\` |
| Paragraph break | Blank line in source |
| Non-breaking space ("Dr.~Smith") | `~` |
| Literal tilde | `\~{}` or `\textasciitilde` |
| Thin space | `\,` |
| Vertical space | `\vspace{6pt}` or `\vspace{-4pt}` |
| Horizontal space | `\hspace{1em}` |

Multiple spaces in source collapse to one in the rendered output. Use `\hspace{...}` or `\quad` / `\qquad` for deliberate horizontal spacing.

## Comments

```latex
% This is a comment, ignored when compiled
```

Useful on a resume source file for:
- Stashing alternate bullet wording for A/B testing
- Hiding old experience without deleting it
- Leaving notes to yourself about why a bullet is phrased a certain way

## Symbols commonly used on resumes

| Symbol | Syntax | Use |
|---|---|---|
| `\|` (vertical bar) | `$|$` | Separator between header fields (phone | email | LinkedIn) |
| `•` (bullet) | `\textbullet` | Inline bullet (rare; `itemize` handles list bullets automatically) |
| `–` (en-dash) | `--` | Date ranges (see Dashes section) |
| `—` (em-dash) | `---` | Parentheticals |
| `★` (star) | `\textstar` | Awards (requires extra package; usually skip) |

## Common gotchas

- **`{` and `}` mismatch** — the most frequent compile error. Every `{` needs a matching `}`. If you get `Missing } inserted`, count your braces.
- **Empty lines inside `\begin{itemize}` ... `\end{itemize}`** can break the list rendering. Keep list contents tight.
- **Forgetting `\` on a command** — `textbf{X}` (without backslash) outputs the literal text `textbf{X}` instead of bold.
- **Unicode/emoji** can break compilation under the default pdfLaTeX engine. Stick to ASCII or compile with XeLaTeX/LuaLaTeX if you need Unicode.
- **Comments don't nest** — `%` starts a comment that ends at the end of the line. There's no block-comment syntax (use `\iffalse ... \fi` for that).
- **Capitalization matters in commands** — `\Textbf{}` won't work; it's `\textbf{}`.
