[![Build Status](https://travis-ci.org/abo-abo/lispy.svg?branch=master)](https://travis-ci.org/abo-abo/lispy)
[![MELPA](http://melpa.org/packages/lispy-badge.svg)](http://melpa.org/#/lispy)
[![MELPA Stable](http://stable.melpa.org/packages/lispy-badge.svg)](http://stable.melpa.org/#/lispy)

<p align="center">
  <img src="https://raw.githubusercontent.com/abo-abo/lispy/readme/doc/lispy-logo.png"
   alt="lispy logo"/>
</p>

> short and sweet LISP editing

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc/generate-toc again -->
**Table of Contents**

- [Introduction](#introduction)
    - [Relation to vi](#relation-to-vi)
    - [Features](#features)
    - [Function reference](#function-reference)
- [Getting Started](#getting-started)
    - [Installation instructions](#installation-instructions)
        - [via MELPA](#via-melpa)
        - [via el-get](#via-el-get)
    - [Configuration instructions](#configuration-instructions)
    - [Customization instructions](#customization-instructions)
- [Operating on lists](#operating-on-lists)
    - [How to get into list-editing mode (special)](#how-to-get-into-list-editing-mode-special)
    - [Digit keys in special](#digit-keys-in-special)
    - [How to get out of special](#how-to-get-out-of-special)
    - [List commands overview](#list-commands-overview)
        - [Inserting pairs](#inserting-pairs)
        - [Reversible commands](#reversible-commands)
        - [Keys that modify whitespace](#keys-that-modify-whitespace)
        - [List of IDE-like commands](#list-of-ide-like-commands)
        - [Command chaining](#command-chaining)
        - [Navigating with `ace-jump-mode`-related commands](#navigating-with-ace-jump-mode-related-commands)
- [Operating on regions](#operating-on-regions)
    - [Ways to activate region](#ways-to-activate-region)
    - [Move region around](#move-region-around)
    - [Switch to the other side of the region](#switch-to-the-other-side-of-the-region)
    - [Grow/shrink region](#growshrink-region)
    - [Commands that operate on region](#commands-that-operate-on-region)
- [IDE-like features](#ide-like-features)
    - [`lispy-describe-inline`](#lispy-describe-inline)
    - [`lispy-arglist-inline`](#lispy-arglist-inline)
    - [`lispy-eval`](#lispy-eval)
    - [`lispy-eval-and-insert`](#lispy-eval-and-insert)
    - [`lispy-follow`](#lispy-follow)
    - [`lispy-goto`](#lispy-goto)
    - [`lispy-goto-local`](#lispy-goto-local)
- [Demos](#demos)
    - [[Demo 1: Practice generating code](http://abo-abo.github.io/lispy/demo-1)](#demo-1-practice-generating-codehttpabo-abogithubiolispydemo-1)
    - [[Demo 2: The substitution model for procedure application](http://abo-abo.github.io/lispy/demo-2)](#demo-2-the-substitution-model-for-procedure-applicationhttpabo-abogithubiolispydemo-2)
    - [[Demo 3: Down the rabbit hole](http://abo-abo.github.io/lispy/demo-3)](#demo-3-down-the-rabbit-holehttpabo-abogithubiolispydemo-3)
    - [[Demo 4: Project Euler p100 and Clojure](http://abo-abo.github.io/lispy/demo-4)](#demo-4-project-euler-p100-and-clojurehttpabo-abogithubiolispydemo-4)
- [Screencasts](#screencasts)

<!-- markdown-toc end -->

# Introduction

This package implements a special key binding method and various commands that are
useful for navigating and editing LISP code.
The key binding method is influenced by vi, although this isn't modal editing *per se*.
The list manipulation commands are influenced by and are a superset of Paredit.

## Relation to vi

Here's a quote from Wikipedia on how vi works, in case you don't know:

> vi is a modal editor: it operates in either insert mode (where typed
> text becomes part of the document) or normal mode (where keystrokes
> are interpreted as commands that control the edit session). For
> example, typing i while in normal mode switches the editor to insert
> mode, but typing i again at this point places an "i" character in
> the document. From insert mode, pressing ESC switches the editor
> back to normal mode.

Here's an illustration of Emacs, vi and lispy bindings for inserting a
char and calling a command:

                  | insert "j"     | forward-list
------------------|----------------|--------------
Emacs             | <kbd>j</kbd>   | <kbd>C-M-n</kbd>
vi in insert mode | <kbd>j</kbd>   | impossible
vi in normal mode | impossible     | <kbd>j</kbd>
lispy             | <kbd>j</kbd>   | <kbd>j</kbd>

Advantages/disadvantages:

- Emacs can both insert and call commands without switching modes (since it has none),
  but the command bindings are long
- vi has short command bindings, but you have to switch modes between inserting and calling commands
- lispy has short command bindings and doesn't need to switch modes

Of course it's not magic, lispy needs to have normal/insert mode to
perform both functions with <kbd>j</kbd>.  The difference from vi is that the
mode is *explicit* instead of *implicit* - it's determined by the point
position or the region state:

- you're in normal mode when the point is before/after paren or the
  region is active
- otherwise you're in insert mode

But if you ask:

> What if I want to insert when the point is before/after paren or the region is active?

The answer is that because of the LISP syntax you don't want to write this:

```cl
j(progn
   (forward-char 1))k
```

Also, Emacs does nothing special by default when the region is active
and you press a normal key, so new commands can be called in that situation.

## Features

- Basic navigation by-list and by-region:
    - <kbd>h</kbd> moves left
    - <kbd>j</kbd> moves down
    - <kbd>k</kbd> moves up
    - <kbd>l</kbd> moves right
- Paredit transformations, callable by plain letters:
    - <kbd>></kbd> slurps
    - <kbd><</kbd> barfs
    - <kbd>r</kbd> raises
    - <kbd>C</kbd> convolutes
- [IDE-like features](#ide-like-features) for Elisp, Clojure, Scheme and Common Lisp:
    - <kbd>e</kbd> evals
    - <kbd>E</kbd> evals and inserts
    - <kbd>g</kbd> jumps to tag with semantic
    - <kbd>M-.</kbd> jumps to symbol, <kbd>M-,</kbd> jumps back
    - <kbd>C-1</kbd> shows documentation in an overlay
    - <kbd>C-2</kbd> shows arguments in an overlay
    - [<kbd>Z</kbd>](http://abo-abo.github.io/lispy/#lispy-edebug-stop) breaks
      out of `edebug`, while storing current function's arguments

- Code manipulation:
    - <kbd>i</kbd> prettifies code (remove extra space, hanging parens ...)
    - <kbd>xi</kbd> transforms `cond` expression to equivalent `if` expressions
    - <kbd>xc</kbd> transforms `if` expressions to an equivalent `cond` expression
    - <kbd>xf</kbd> flattens function or macro call (extract body and substitute arguments)
    - <kbd>xr</kbd> evals and replaces
    - <kbd>xl</kbd> turns current `defun` into a `lambda`
    - <kbd>xd</kbd> turns current `lambda` into a `defun`
    - <kbd>O</kbd> formats the code into one line
    - <kbd>M</kbd> formats the code into multiple lines

- Misc. bindings:
    - outlines navigation/folding (<kbd>J</kbd>, <kbd>K</kbd>, <kbd>I</kbd>, <kbd>i</kbd>)
    - narrow/widen (<kbd>N</kbd>, <kbd>W</kbd>)
    - `ediff` (<kbd>b</kbd>, <kbd>B</kbd>)
    - `ert` (<kbd>T</kbd>)
    - `edebug` (<kbd>xe</kbd>)

## Function reference
Most functions are cataloged and described at http://abo-abo.github.io/lispy/.

# Getting Started
## Installation instructions
### via MELPA

It's easiest/recommended to install from [MELPA](http://melpa.org/).
Here's a minimal MELPA configuration for your `~/.emacs`:

```cl
(package-initialize)
(add-to-list 'package-archives '("melpa" . "http://melpa.org/packages/"))
```

Afterwards, <kbd>M-x package-install RET lispy RET</kbd> (you might
want to <kbd>M-x package-refresh-contents RET</kbd> beforehand if
you haven't done so recently).

### via el-get

[el-get](https://github.com/dimitri/el-get) also features a lispy recipe.
Use <kbd>M-x el-get-install RET lispy RET</kbd> to install.

## Configuration instructions
**Enable lispy automatically for certain modes**

After installing, you can call <kbd>M-x lispy-mode</kbd> for any
buffer with a LISP dialect source.  To have `lispy-mode` activated
automatically, use something like this:


```cl
(add-hook 'emacs-lisp-mode-hook (lambda () (lispy-mode 1)))
```

**Enable lispy for `eval-expression`**

Although I prefer to eval things in `*scratch*`, sometimes
<kbd>M-:</kbd> - `eval-expression` is handy.  Here's how to use lispy
in the minibuffer during `eval-expression`:

```cl
(defun conditionally-enable-lispy ()
  (when (eq this-command 'eval-expression)
    (lispy-mode 1)
    (local-set-key "β" 'helm-lisp-completion-at-point)))
(add-hook 'minibuffer-setup-hook 'conditionally-enable-lispy)
```

## Customization instructions

If you want to replace some of the `lispy-mode`'s bindings you can do
it like this:

```cl
(eval-after-load "lispy"
  `(progn
     ;; replace with own function
     (define-key lispy-mode-map (kbd "C-e") 'my-custom-eol)
     ;; replace with major-mode's default
     (define-key lispy-mode-map (kbd "C-j") nil)))
```

# Operating on lists

## How to get into list-editing mode (special)

The plain keys will call commands when:
- the point is positioned before paren
- the point is positioned after paren
- the region is active

When one of the first two conditions is true, I say that the point is
special. When the point is special, it's very clear to which sexp the
list-manipulating command will be applied to, what the result be and
where the point should end up afterwards.  You can enhance this effect
with `show-paren-mode` or similar.

Here's an illustration to this effect, with `lispy-clone` (here, `|`
represents the point):

before              | key          | after
--------------------|--------------|------------------------
`(looking-at "(")|` | <kbd>c</kbd> |  `(looking-at "(")`
                    |              |  `(looking-at "(")|`

before              | key          | after
--------------------|--------------|------------------------
`|(looking-at "(")` | <kbd>c</kbd> |  `|(looking-at "(")`
                    |              |   ` (looking-at "(")`

You can use plain Emacs navigation commands to get into special, or you can use
some of the dedicated commands:

Key Binding     | Description
----------------|-----------------------------------------------------------
<kbd>]</kbd>    | `lispy-forward` - move to the end of the closest list, analogous to <kbd>C-M-n</kbd> (`forward-list`)
`［`            | `lispy-backward` - move to the start of the closest list, analogous to <kbd>C-M-p</kbd> (`backward-list`)
<kbd>C-3</kbd>  | `lispy-out-forward` - exit current list forwards, analogous to `up-list`
<kbd>)</kbd>   | `lispy-out-forward-nostring` exit current list forwards, but self-insert in strings and comments

These are the few lispy commands that don't care whether the point is
special or not. Other such bindings are <kbd>DEL</kbd>, <kbd>C-d</kbd>, <kbd>C-k</kbd>.

Special is useful for manipulating/navigating lists.  If you want to
manipulate symbols, use [region selection](#operating-on-regions)
instead.

## Digit keys in special

When special, the digit keys call `digit-argument` which is very
useful since most lispy commands accept a numeric argument.
For instance, <kbd>3c</kbd> is equivalent to <kbd>ccc</kbd> (clone sexp 3 times), and
<kbd>4j</kbd> is equivalent to <kbd>jjjj</kbd> (move point 4 sexps down).

Some useful applications are <kbd>9l</kbd> and <kbd>9h</kbd> - they exit list forwards
and backwards respectively at most 9 times which makes them
effectively equivalent to `end-of-defun` and `beginning-of-defun`.  Or
you can move to the last sexp of the file with <kbd>999j</kbd>.

## How to get out of special

To get out of the special position, you can use any of the good-old
navigational commands such as <kbd>C-f</kbd> or <kbd>C-n</kbd>.
Additionally <kbd>SPC</kbd> will break out of special to get around the
situation when you have the point between the open parens like this

    (|(

and want to start inserting; <kbd>SPC</kbd> will change the code to
this:

    (| (

## List commands overview
The full reference is [here](http://abo-abo.github.io/lispy/).

### Inserting pairs

Here's a list of commands for inserting [pairs](http://abo-abo.github.io/lispy/#lispy-pair):

key               | command
------------------|-------------------------------------------------------------------
  <kbd>(</kbd>    | [`lispy-parens`](http://abo-abo.github.io/lispy/#lispy-parens)
  <kbd>{</kbd>    | [`lispy-braces`](http://abo-abo.github.io/lispy/#lispy-braces)
  <kbd>}</kbd>    | [`lispy-brackets`](http://abo-abo.github.io/lispy/#lispy-brackets)
  <kbd>"</kbd>    | [`lispy-quotes`](http://abo-abo.github.io/lispy/#lispy-quotes)

### Reversible commands

A lot of Lispy commands come in pairs - one reverses the other:

 key            | command                  | key                              | command
----------------|--------------------------|----------------------------------|----------------------
 <kbd>j</kbd>   | `lispy-down`             | <kbd>k</kbd>                     | `lispy-up`
 <kbd>s</kbd>   | `lispy-move-down`        | <kbd>w</kbd>                     | `lispy-move-up`
 <kbd>></kbd>   | `lispy-slurp`            | <kbd><</kbd>                     | `lispy-barf`
 <kbd>c</kbd>   | `lispy-clone`            | <kbd>C-d</kbd> or <kbd>DEL</kbd> |
 <kbd>C</kbd>   | `lispy-convolute`        | <kbd>C</kbd>                     | reverses itself
 <kbd>d</kbd>   | `lispy-different`        | <kbd>d</kbd>                     | reverses itself
 <kbd>M-j</kbd> | `lispy-split`            | <kbd>+</kbd>                     | `lispy-join`
 <kbd>O</kbd>   | `lispy-oneline`          | <kbd>M</kbd>                     | `lispy-multiline`
 <kbd>S</kbd>   | `lispy-stringify`        | <kbd>C-u "</kbd>                 | `lispy-quotes`
 <kbd>;</kbd>   | `lispy-comment`          | <kbd>C-u ;</kbd>                 | `lispy-comment`
 <kbd>xi</kbd>  | `lispy-to-ifs`           | <kbd>xc</kbd>                    | `lispy-to-cond`

### Keys that modify whitespace

These commands handle whitespace in addition to inserting the expected
thing.

 key            | command
----------------|---------------------------
 <kbd>SPC</kbd> | `lispy-space`
 <kbd>:</kbd>   | `lispy-colon`
 <kbd>^</kbd>   | `lispy-hat`
 <kbd>C-m</kbd> | `lispy-newline-and-indent`

### List of IDE-like commands

 key            | command
----------------|--------------------------
 <kbd>C-1</kbd> | `lispy-describe-inline`
 <kbd>C-2</kbd> | `lispy-arglist-inline`
 <kbd>e</kbd>   | `lispy-eval`
 <kbd>E</kbd>   | `lispy-eval-and-insert`
 <kbd>D</kbd>   | `lispy-describe`
 <kbd>g</kbd>   | `lispy-goto`
 <kbd>G</kbd>   | `lispy-goto-local`
 <kbd>F</kbd>   | `lispy-follow`

Details [here](#ide-like-features).

### Command chaining

Most special commands will leave the point special after they're
done.  This allows to chain them as well as apply them
continuously by holding the key.  Some useful hold-able keys are
<kbd>jkf<>cws;</kbd>.
Not so useful, but fun is <kbd>/</kbd>: start it from `|(` position and hold
until all your Lisp code is turned into Python :).

### Navigating with `ace-jump-mode`-related commands

 key            | command
----------------|--------------------------
 <kbd>q</kbd>   | `lispy-ace-paren`
 <kbd>Q</kbd>   | `lispy-ace-char`
 <kbd>a</kbd>   | `lispy-ace-symbol`
 <kbd>H</kbd>   | `lispy-ace-symbol-replace`
 <kbd>-</kbd>   | `lispy-ace-subword`

<kbd>q</kbd> - `lispy-ace-paren` jumps to a "(" character within current
top-level form (e.g. `defun`). It's much faster than typing in the
`ace-jump-mode` binding + selecting "(", and there's less candidates,
since they're limited to the current top-level form.

<kbd>a</kbd> - `lispy-ace-symbol` will let you select which symbol to
mark within current form. This can be followed up with e.g. eval,
describe, follow, raise etc. Or you can simply <kbd>m</kbd> to
deactivate the mark and edit from there.

<kbd>-</kbd> - `lispy-ace-subword` is a niche command for a neat combo. Start with:

    (buffer-substring-no-properties
     (region-beginning)|)

Type <kbd>c</kbd>, <kbd>-</kbd>, <kbd>b</kbd> and <kbd>C-d</kbd> to get:

    (buffer-substring-no-properties
     (region-beginning)
     (region-|))

Fill `end` to finish the statement.

# Operating on regions
Sometimes the expression that you want to operate on isn't bounded by parens.
In that case you can mark it with a region and operate on that.

## Ways to activate region
While in special:
- Mark a sexp with <kbd>m</kbd> - `lispy-mark-list`
- Mark a symbol within sexp <kbd>a</kbd> - `lispy-ace-symbol`.

While not in special:
- <kbd>C-SPC</kbd> - `set-mark-command`
- mark a symbol at point with <kbd>M-m</kbd> - `lispy-mark-symbol`
- mark containing expression (list or string or comment) with <kbd>C-M-,</kbd> - `lispy-mark`

## Move region around

The arrow keys <kbd>j</kbd>/<kbd>k</kbd> will move the region up/down within the current
list.  The actual code will not be changed.

## Switch to the other side of the region

Use <kbd>d</kbd> - `lispy-different` to switch between different sides
of the region. The side is important since the grow/shrink operations
apply to current side of the region.

## Grow/shrink region

Use a combination of:
- <kbd>></kbd> - `lispy-slurp` - extend by one sexp from the current side. Use digit
  argument to extend by several sexps.
- <kbd><</kbd> - `lispy-barf` - shrink by one sexp from the current side. Use digit
  argument to shrink by several sexps.

The other two arrow keys will mark the parent list of the current region:

- <kbd>h</kbd> - `lispy-out-backward` - mark the parent list with the point on the left
- <kbd>l</kbd> - `lispy-out-forward` - mark the parent list with the point on the right

To do the reverse of the previous operation, i.e. to mark the first
child of marked list, use <kbd>i</kbd> - `lispy-tab`.

## Commands that operate on region
- <kbd>m</kbd> - `lispy-mark-list` - deactivate region
- <kbd>c</kbd> - `lispy-clone` - clone region and keep it active
- <kbd>s</kbd> - `lispy-move-down` - move region one sexp down
- <kbd>w</kbd> - `lispy-move-up` - move region one sexp up
- <kbd>u</kbd> - `lispy-undo` - deactivate region and undo
- <kbd>t</kbd> - `lispy-teleport` - move region inside the sexp you select with
  `lispy-ace-paren`.
- <kbd>C</kbd> - `lispy-convolute` - exchange the order of application of two
  sexps that contain point.
- <kbd>n</kbd> - `lispy-new-copy` - copy region as kill without deactivating
the region.  Useful to search for currently marked symbol with <kbd>n g C-y</kbd>.

# IDE-like features

These features are specific to the Lisp dialect used.  Currently Elisp
and Clojure (via `cider`) are supported.  There's also basic
evaluation support for Scheme (via `geiser`) and Common lisp (via
`slime`).

## `lispy-describe-inline`

Bound to <kbd>C-1</kbd>. Show the doc for the current function inline.

<kbd>C-h f</kbd> is fine, but the extra buffer, and having to navigate to a symbol
is tiresome. <kbd>C-1</kbd> toggles on/off the inline doc for current function.
No extra buffer necessary:

![screenshot](https://raw.github.com/abo-abo/lispy/master/doc/doc-1.png)

Here's how it looks for Clojure:

![screenshot](https://raw.github.com/abo-abo/lispy/master/doc/doc-2.png)

## `lispy-arglist-inline`
Bound to <kbd>C-2</kbd>. Show arguments for current function inline.

`eldoc-mode` is cool, but it shows you arguments *over there* and
you're writing *over here*!. No problem, <kbd>C-2</kbd> fixes that:

![screenshot](https://raw.github.com/abo-abo/lispy/master/doc/arglist-2.png)

As you see, normal, &optional and &rest arguments have each a
different face. Here's how it looks for Clojure:

![screenshot](https://raw.github.com/abo-abo/lispy/master/doc/arglist-3.png)

## `lispy-eval`

Bound to <kbd>e</kbd>.

Eval current expression. Works from the beginning of the list as well.

## `lispy-eval-and-insert`
Bound to <kbd>E</kbd>.

Eval and insert current expression. Works from the beginning of list
as well.

## `lispy-follow`
Bound to <kbd>F</kbd>.

Follow to definition of current function.  While in Clojure, if can't
resolve the symbol in current namespace, searches for it in all loaded
namespaces.

## `lispy-goto`

Bound to <kbd>g</kbd>.

Use `helm` to select a symbol to jump to from all top-level symbols in
the in current directory.

Works out of the box for Elisp, Scheme and Common Lisp.
[clojure-semantic](https://github.com/kototama/clojure-semantic)
is required for Clojure.

## `lispy-goto-local`

Bound to <kbd>G</kbd>.

Like `lispy-goto`, but offers only symbols from the current file.

# Demos

## [Demo 1: Practice generating code](http://abo-abo.github.io/lispy/demo-1)
## [Demo 2: The substitution model for procedure application](http://abo-abo.github.io/lispy/demo-2)
## [Demo 3: Down the rabbit hole](http://abo-abo.github.io/lispy/demo-3)
## [Demo 4: Project Euler p100 and Clojure](http://abo-abo.github.io/lispy/demo-4)

# Screencasts

I've recorded a few gifs that show some features:

- [1-gif](https://raw.github.com/abo-abo/lispy/master/doc/screencast-1.gif)
- [2-vimeo](https://vimeo.com/85831418),
  [2-ogv](https://raw.github.com/abo-abo/lispy/master/doc/screencast-2.ogv),
  [2-gif](https://raw.github.com/abo-abo/lispy/master/doc/screencast-2.gif)
- [3-gif](https://raw.github.com/abo-abo/lispy/master/doc/screencast-3.gif)
- [4-gif](https://raw.github.com/abo-abo/lispy/master/doc/oneline-multiline.gif)
- [5-gif](https://raw.github.com/abo-abo/lispy/master/doc/screencast-4.gif)

The older stuff can be found on [vimeo](http://vimeo.com/user24828177/videos).

The new stuff is on https://www.youtube.com/user/abo5abo/videos.
