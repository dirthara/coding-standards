---
id: types
title: Types
sidebar_position: 4
description: Native types, the generics that only a docblock can express, and the analyzer that checks both.
---

## Everything the language can type, the language types

Every parameter, every return, and every property carries a native type. A method with no return type is not allowed to
exist; a method that returns nothing is `: void`.

```php
public function has(int|string $key): bool
public function clear(): void
```

Prefer the narrowest type that is honest. `iterable` where the method only iterates, `array` where it indexes. A union
is fine when the domain is a union; `mixed` is for a value the package genuinely does not constrain, such as the value
stored in a collection.

Nullable is `?T` for a single type and `T|null` inside a union, which is what the formatter produces. A parameter
defaulting to `null` is still explicitly nullable — implicit nullability is removed from the language.

## Docblocks carry what the type system cannot

A docblock exists to add information, never to repeat it:

```php
/**
 * @template TKey of array-key
 * @template TValue
 *
 * @implements CollectionContract<TKey, TValue>
 */
abstract class Collection implements CollectionContract
{
    /**
     * @var array<TKey, TValue>
     */
    protected array $items = [];

    /**
     * @param TKey $key
     * @param TValue $value
     */
    public function set(int|string $key, mixed $value): void
```

Write a docblock when, and only when, one of these is true:

| Situation | Tag |
| --- | --- |
| The class is generic | `@template`, `@extends`, `@implements` |
| An `array` has a known key and value type | `@param array<string, mixed>`, `@var` |
| A `string` is a class name | `@param class-string<T>` |
| An `array` is known to be non-empty | `@param non-empty-array<string, string>` |
| A return is narrower than the native type | `@return self<TKey, TValue>` |
| The method throws a package exception | `@throws MappingException` |

A `@param string $name` next to `string $name` is deleted on sight. So is a `@return` that repeats the native type, and
a `@throws` for a `TypeError` — that is a bug, not a documented outcome.

:::caution
Generics in prose need backticks. The documentation site is MDX, and an unfenced `array<string, mixed>` parses as a JSX
tag and fails the build. This applies to `docs`, not to docblocks.
:::

## `@throws` is part of the signature

Every method that can throw one of the package's exceptions lists it. A caller cannot catch what it cannot see, and the
analyzer uses the tag to check that a documented failure is a real one. Tag the exception the method throws, not the
whole hierarchy: `@throws MappingException`, not `@throws EntityException`, unless both can genuinely come out.

## The analyzer runs clean, with no baseline

```sh
docker compose exec php composer analyze  # mago analyze
docker compose exec php composer guard    # mago guard, the architecture rules
```

Both run inside `composer mago`, which `composer ci` runs, so a pull request cannot merge with a finding.

There is no baseline file and no per-line suppression. A baseline makes the existing violations invisible and the new
ones indistinguishable from them; the package is small enough that the honest option — fixing it — is available. If the
analyzer is wrong, narrow the type until it is right, or state the case in the pull request and change the rule for
every package at once.
