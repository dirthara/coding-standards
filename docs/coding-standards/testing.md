---
id: testing
title: Testing
sidebar_position: 6
description: Where tests live, how they are written, and the coverage gate every package has to clear.
---

## The gate

Line coverage of `src` stays at 100%.

```sh
docker compose exec php composer test           # phpunit
docker compose exec php composer test-coverage  # phpunit with clover
docker compose exec php composer coverage       # the gate
```

`composer coverage` fails below 100% and prints the uncovered lines. It runs as part of `composer ci`, so a pull request
that adds a line and not its test does not merge. Cover new code in the pull request that adds it, never in a follow-up.

The number is 100 rather than 90 because any lower number is a negotiation. Once a package can merge at 92%, every
review has to decide whether this particular uncovered branch is the one worth arguing about. At 100 there is nothing to
decide, and the cost is paid where it belongs — at the line that is hard to reach, which is usually hard to reach
because the design is wrong.

:::note
An empty scaffold is the one exception: the test and coverage commands report that they are not applicable while `src`
and `tests` contain no PHP files. As soon as either does, both run normally, and an empty test suite fails.
:::

Coverage is the floor, not the goal. A line executed by a test that asserts nothing is covered and untested. What the
gate catches is the code nobody thought about.

## Layout

Tests live in `tests`, under `Dirthara\<Namespace>\Tests`, in a tree that mirrors `src`. A shared base case per package
— `CollectionTestCase`, `EntityTestCase` — holds the fixtures and helpers the suite reuses.

| Directory | Holds |
| --- | --- |
| `tests/` | Unit tests, mirroring `src`. |
| `tests/Integration/` | Behaviour that needs a real service. |
| `tests/Doubles/` | Fakes and stubs that more than one test uses. |
| `tests/Entities/` | Fixture classes a test maps or reflects over. |

Behaviour that needs a real database belongs in one conformance suite in `tests/Integration` that runs against every
driver, not in a copy per driver. A copy per driver is how three of them end up testing slightly different things.
SQLite runs in memory and always runs; the other drivers skip when their PDO extension is missing, and read their
connection from the `DIRTHARA_<DRIVER>_*` environment variables that `compose.yaml` provides.

## Style

```php
final class ImmutableCollectionTest extends CollectionTestCase
{
    public function testWithReturnsIndependentEntriesAndPreservesOrder(): void
    {
        $original = $this->create(['a' => 1, 7 => null]);
        $replaced = $original->with('a', 2);

        self::assertNotSame($original, $replaced);
        self::assertSame(['a' => 1, 7 => null], $original->toArray());
        self::assertSame(['a' => 2, 7 => null], $replaced->toArray());
    }
}
```

- **A test class is `final`** and named `<Subject>Test`.
- **A test method name is a sentence about behaviour.** `testWithReturnsIndependentEntriesAndPreservesOrder` says what
  broke when it goes red. `testWith` does not.
- **Assert statically.** `self::assertSame(...)`, not `$this->assertSame(...)`.
- **`assertSame`, not `assertEquals`.** `assertEquals` compares loosely and will happily accept `'1'` for `1`, which is
  the bug you were testing for.
- **Test through the public API.** A test that reaches into a private property with reflection is testing the
  implementation, and it is what stops you changing it. The exception is a fixture the package's own reflection needs.
- **A data provider** for the same assertion over many inputs; a separate test for a different behaviour. A provider
  with a `switch` in the test body is two tests.
- **No conditionals in a test.** A test that can take two paths only ever proves one of them, and you cannot tell which
  from the green tick.
- **Test the failures.** Every documented exception has a test that triggers it and asserts its type — and its context,
  when the context is the point.

## PHPUnit is configured to be strict

`phpunit.xml` sets `failOnRisky`, `failOnWarning`, `failOnEmptyTestSuite`, and `beStrictAboutOutputDuringTests`. A risky
test, a deprecation, a test that prints, and a suite that matched nothing are all build failures rather than yellow text
scrolling past in CI. Do not relax these to get a suite green.
