---
id: formatting
title: Formatting and imports
sidebar_position: 3
description: The formatter that owns whitespace, and the import order it deliberately does not own.
---

## The style is PSR-12

The baseline is the PHP-FIG standards, not a house invention:

| Standard | Covers |
| --- | --- |
| [PSR-1](https://www.php-fig.org/psr/psr-1/) | Basic coding standard: one declaration per file, no side effects, `StudlyCaps` classes, `camelCase` methods, `UPPER_SNAKE_CASE` constants. |
| [PSR-12](https://www.php-fig.org/psr/psr-12/) | Extended coding style: four-space indentation, brace placement, the order of `declare`, `namespace`, and `use`, keyword casing, visibility on every property and method, the 120-column soft limit. |

PSR-12 supersedes PSR-2, which PHP-FIG deprecated in 2019 and which predates most of the syntax a package uses. Cite
PSR-12; there is nothing in PSR-2 that PSR-12 does not carry forward.

Nothing here contradicts either one. Where this page adds a rule, it is a rule PSR-12 leaves open — import order is the
main one, since PSR-12 fixes where the `use` block goes and says nothing about how it is sorted.

:::note
The 120 columns are a soft limit in PSR-12 and a target for the formatter. A line it cannot break — a long string
literal, a URL in a comment — stays long rather than being mangled, and a handful in `database`, `entity`, and `schema`
are. Do not hand-wrap one to get under the limit.
:::

## The formatter is the authority

Formatting is not a matter of taste in a Dirthara package, because it is not a matter of judgement:
[Mago](https://mago.carthage.software/) formats every file in `src` and `tests`, and `composer fmt-check` fails the
build when a file differs from its output.

```sh
docker compose exec php composer fmt        # format, then sort imports
docker compose exec php composer fmt-check  # what CI runs
```

Never hand-format around the formatter, and never argue a diff it produced into a pull request description. If the
output is wrong often enough to matter, change the setting in `mago.toml` — once, for every package.

The settings that are not defaults are these:

```toml
[formatter]
sort-uses = "preserve"
separate-use-types = true
expand-use-groups = true
```

`expand-use-groups` turns `use A\{B, C};` into one statement per symbol, so a diff touching one import touches one line.
`separate-use-types` puts class, function, and constant imports in separate blocks. `sort-uses = "preserve"` hands the
order to the script below.

## Imports are sorted by length

A separate sorter, `composer imports`, orders each block of `use` statements by the character length of the whole
statement, ascending, with ties broken case-insensitively and then byte-wise:

```php
use Traversable;
use ArrayIterator;
use Dirthara\Collection\Exception\KeyNotFoundException;
use Dirthara\Collection\Contract\Collection as CollectionContract;

use function count;
use function in_array;
use function array_keys;
use function array_key_exists;
```

Alphabetical order scatters the short, familiar imports through a wall of long namespaces. Length order puts the shape
of the list on the left margin, so the one import that is unusually long — the one worth noticing — is the one that
sticks out.

Things to know about the sorter:

- It sorts **each contiguous block** separately. A blank line or a change of import type starts a new block, which is
  why the class and function blocks above are ordered independently.
- It **skips a block it cannot safely rewrite**: one with a comment attached to a statement, a grouped import, or
  anything spanning more than a line. Let the formatter expand it first.
- `composer imports-check` is the CI form and fails with the file name.

## Import symbols, do not qualify them

Functions and constants are imported, not written fully qualified at the call site:

```php
use function count;
use function array_key_exists;

// …
return array_key_exists($key, $this->items);
```

An imported function is resolved once, at the top of the file, where a reader can see whether it is PHP's or the
package's. The alternative — a leading backslash at every call — is noise at the point where the code is doing
something.

A class is always imported rather than written out inline, with two exceptions that stay fully qualified because
importing them would mislead: a `class-string` inside a string literal, and a dynamic class name.

:::note
The half of this rule that every package already follows is the second one: nothing in Dirthara writes `\sprintf()` at a
call site. The imports themselves are not there yet — `database` and `schema` import consistently, `entity` and `http`
barely do. New code imports; a file already open for another reason is brought up as it is touched.
:::

## The rest of the house style

These are not formatter settings; they are what the code does.

- **Guard clauses over nesting.** Return, `continue`, or throw at the top, so the body of a method is the case it is
  actually about.
- **A blank line before `return`** when the method did more than one thing.
- **Strict comparison.** `===` and `!==` always; `==` is not used. Compare an array to `[]` rather than calling
  `empty()`, which is true for `0` and `''` as well.
- **One statement per line**, and no assignment inside a condition.
- **Comments explain why.** A comment that restates the code is deleted at the next change and wrong before then. The
  exception is a docblock carrying type information the language cannot — see [types](types.md).
