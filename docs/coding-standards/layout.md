---
id: layout
title: Layout and naming
sidebar_position: 2
description: How a package's directories, namespaces, and class names are organised.
---

Autoloading is [PSR-4](https://www.php-fig.org/psr/psr-4/) and naming is [PSR-1](https://www.php-fig.org/psr/psr-1/);
the formatting that goes with them is [PSR-12](https://www.php-fig.org/psr/psr-12/), covered in
[formatting](formatting.md). PSR-0 is deprecated and is not used, including for the `Tests` namespace.

## One package, two trees

```text
src/     Dirthara\<Namespace>
tests/   Dirthara\<Namespace>\Tests
```

Both are registered as PSR-4 roots in `composer.json`, `src` under `autoload` and `tests` under `autoload-dev`, so the
test namespace never reaches a consumer's autoloader. The directory path below the root is the namespace below the root,
exactly; `src/Metadata/EntityMetadata.php` is `Dirthara\Entity\Metadata\EntityMetadata`.

The test tree mirrors the source tree. A test for `src/Query/EntityQuery.php` is `tests/Query/EntityQueryTest.php`.
Mirroring is what makes an uncovered class obvious.

## The subdirectories a package uses

A package starts flat. A subdirectory appears when a group of classes exists, not in anticipation of one. These names
are reused across packages so that a reader who knows one package can find their way around the next:

| Directory | Holds |
| --- | --- |
| `Contract/` | Interfaces that describe the package's public surface. |
| `Exception/` | The package's exception interface, its context trait, and the exceptions themselves. |
| `Attribute/` | PHP attributes the package defines. |
| `Metadata/` | Value objects describing something the package reflects over. |
| `Type/` | Conversions between PHP values and their stored form. |

Anything else is named after the concept it holds, in the singular: `Persistence`, `Hydration`, `Relation`. A directory
named `Util`, `Helper`, `Common`, `Misc`, or `Support` is a directory nobody can predict the contents of; name the
concept instead.

## Naming

- **Classes, interfaces, enums, and traits** are `StudlyCaps`. **Methods, properties, and variables** are `camelCase`.
  **Constants** are `UPPER_SNAKE_CASE`. This is PSR-1 and PSR-12, and the formatter assumes it.
- **An interface is named for the thing, not for being an interface.** `Collection`, not `ICollection` or
  `CollectionInterface`. The prefix and the suffix both exist to work around a reader not knowing what a file is, which
  the directory already tells them.
- **Where the interface and the class share a name**, the file that needs both aliases the contract on import:

  ```php
  use Dirthara\Collection\Contract\Collection as CollectionContract;
  ```

The concrete class keeps the plain name because it is what callers type. The alias is always `<Name>Contract`, so the
same import reads the same way in every package.
- **An exception ends in `Exception`** and names the failure, not the thrower: `InvalidUriException`, not
  `UriErrorException`. The package-wide exception interface is named after the package — `HttpException` — and lives in
  `Exception/` rather than `Contract/`, because it is the one interface a caller reaches for in a `catch` block rather
  than a type hint. See [exceptions](exceptions.md).
- **A test class ends in `Test`**; a shared base case ends in `TestCase`.
- **A boolean-returning method reads as a question** — `isEmpty()`, `has()`, `canWrite()` — and does not have side
  effects.
- **Abbreviations are words.** `EntityId`, not `EntityID`; `HttpClient`, not `HTTPClient`. One rule beats remembering
  which acronyms are exempt.

## The public surface is the smallest thing that works

Everything is `private` until something outside needs it. `protected` is a commitment to subclasses and belongs only to
a class that is not `final`. A property is `private` or promoted `readonly`; a public mutable property gives up every
invariant the constructor established.

A class a consumer is never meant to name — an internal visitor, a compiler step, a hydration detail — still lives in
`src`, because there is no other place to put it, and is kept out of the documentation. If it is genuinely not part of
the API, say so in its docblock, so the next person knows a change to it is not a breaking change.
