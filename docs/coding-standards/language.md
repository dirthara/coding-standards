---
id: language
title: Language
sidebar_position: 1
description: The PHP baseline a Dirthara package targets, and which language features it is expected to use.
---

## PHP 8.5, widened deliberately

```json
"require": {
    "php": "^8.5"
}
```

A package requires `^8.5`. The constraint states the versions the package has actually been tested against, and it is
widened by hand once a new one has been.

Support is only ever **added**. A version that has been supported stays supported; see below.

:::note
`^8.5` already means `>=8.5.0 <9.0.0`, so it covers 8.6, 8.7, and every later release in the 8 series. A new **minor**
therefore needs a matrix entry and no constraint change. The constraint changes at a new **major**, and it changes by
union rather than by replacement:

```json
"php": "^8.5 || ^9.0"
```
:::

Not `>=8.5`. An open-ended constraint installs the package on a PHP that nobody has run the suite against, and the first
person to find out is a user whose application broke at runtime. A caret says only what has been verified, and a refused
install on an unsupported major is a clear message at the moment it can still be acted on.

There is no compatibility layer for anything older. 8.5 is the floor because it was the current stable release when the
framework started, and the code uses what the recent releases added — property hooks and asymmetric visibility show up
in the [exception context trait](exceptions.md) alone. Back-porting around them would cost more than it buys.

### Adding a PHP release

When a new PHP version is released, the package adds it rather than waiting for a reason to:

1. **Add it to the `php` matrix** in `.github/workflows/ci.yml`.
2. **Fix whatever it breaks**, in the same pull request where possible.
3. **Widen the constraint if it is a new major** — `^8.5` becomes `^8.5 || ^9.0`. A new minor is already covered.
4. **Update the supported-versions tables** in `CONTRIBUTING.md` and `SECURITY.md`, which have to list the same thing.

The matrix is what earns the constraint, so the order matters: the version is tested, then it is promised. Never widen
the constraint for a version that is not in the matrix.

### A version is never dropped by one package

Narrowing the constraint — removing `^8.5` once `^9.0` is there — is a framework-wide decision, taken for every package
at once. One package requiring PHP 9 while the rest still allow 8.5 means an application using two of them cannot
satisfy both, which is a problem it did not ask for and cannot solve.

So: never narrow the constraint in a pull request that is about something else, never narrow it because a new syntax
would be convenient, and never narrow it in one package to see how it goes. Until a package-wide update says otherwise,
8.5 stays supported.

## Every file declares strict types

```php
<?php

declare(strict_types=1);

namespace Dirthara\Collection;
```

`declare(strict_types=1)` is the first statement in every PHP file, source and test alike, with no exceptions. Mago's
`strict-types` rule is configured with `allow-disabling = false`, so the suppression comment that would normally let a
file opt out is itself rejected.

Weak mode silently coerces `"5"` to `5` and `1.5` to `1` at every boundary, which turns a caller's bug into this
package's wrong answer. Strict mode makes it a `TypeError` at the call that caused it.

## Files declare, they do not do

A file contains one class, interface, trait, or enum, named the same as the file, and has no side effects: no output, no
`ini_set`, no function calls at the top level. Autoloading a class must be free of consequences, because Composer
decides when it happens.

## Classes are final and closed by default

```php
final readonly class BelongsToOneMetadata { /* … */ }
```

- **`final` unless extension is designed.** Most classes in a package are `final`. A class that is not final is a class
  whose protected surface, method visibility, and constructor signature are part of the public API, and every one of
  those becomes a breaking change later. Drop `final` when you have decided to support a subclass, and then treat the
  protected surface as public.
- **`readonly` when the state is fixed at construction.** Value objects, metadata, configuration, and results are
  `final readonly`. Immutability is what makes it safe to hand the same instance to two callers.
- **`abstract` for shared behaviour, an interface for a contract.** An abstract class exists when subclasses share
  implementation, as `Collection` does for its mutable and immutable forms. When only the signature is shared, it is an
  interface.

## Reach for the language before writing the code

- **Constructor property promotion** for anything the constructor only stores. A promoted property with a trailing comma
  per line stays readable at ten parameters.
- **Enums** for a closed set of values, never a group of class constants. An enum gives the parameter a type; a `string`
  constant gives it a comment.
- **Readonly promoted properties** rather than a getter over a private field. A getter that only returns the field adds
  a method to maintain and nothing else.
- **`match`** rather than `switch` for a value-producing branch: it is an expression, it compares strictly, and it
  throws on an unhandled input instead of falling through.
- **First-class callable syntax** (`$this->handle(...)`) rather than a string or array callable, which the analyzer
  cannot follow.
- **Named arguments** at a call whose meaning is otherwise unreadable — `new Column(nullable: true)` rather than
  `new Column(null, true, false)`. Named arguments make a parameter name part of the API; rename one only in a release
  that may break callers.

## What a package does not do

- **No error suppression.** `@` hides the diagnostic and leaves the failure. Test the condition or catch the exception.
- **No `eval`, no variable variables, no dynamic property creation.** Each one is a hole the analyzer cannot see
  through, and dynamic properties are deprecated besides.
- **No globals and no static mutable state.** A package is a library; two instances in one process must not be able to
  affect each other. Static is for named constructors and pure helpers.
- **No `exit`, `die`, `echo`, or `print` in `src`.** A library returns or throws. Writing to output or ending the
  process is the application's decision.
- **No silent `catch`.** An empty catch block, or one that only returns a default, discards the reason something failed.
  See [exceptions](exceptions.md).
- **No suppression of an analyzer or linter diagnostic** to make a check pass. Fix the code, or change the rule in
  `mago.toml` deliberately and say why in the pull request.

:::tip
`composer lint` reports most of this, and `composer lint-fix` applies the safe fixes. The rules that are turned off in
`mago.toml` — method count, parameter count, cyclomatic complexity — are off because a metric is a bad proxy for the
design questions above, not because the design does not matter.
:::
