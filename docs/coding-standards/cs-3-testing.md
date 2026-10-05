---
title: 'CS-3: Testing'
sidebar_position: 4
description: Dirthara Testing Conventions
---

# CS-3: Testing

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## 1. Folder structure

* The folder structure for tests MUST reflect the folder structure of the source of the package it is testing.

## 2. Naming conventions

* Test classes MUST be suffixed with `Test`.
* Test class names MUST be written in `StudlyCaps`.
* Test methods SHOULD NOT be prefixed with `test_`. Test methods SHOULD be marked as test by using the `#[Test]` 
  attribute.
* Test methods MUST be written in `snake_case` with lower letters.
* Attributes MAY be used to add metadata to test methods.

## 3. Code coverage

* All new code MUST be covered by tests.
* Code coverage SHOULD NOT decrease after a new commit.
* Code coverage MUST be 100% for every pull request before it is allowed to be merged.

## 4. Coverage metadata

Every test class MUST declare what it covers and what it uses, with PHPUnit's attributes on the class.

* A test MUST declare the code it covers with `#[CoversClass]`, `#[CoversFunction]`, `#[CoversMethod]`, `#[CoversTrait]`,
  or another `Covers*` attribute. A test that deliberately covers nothing MUST declare `#[CoversNothing]`.
* A test MUST declare every other piece of package code it touches with `#[UsesClass]`, `#[UsesFunction]`,
  `#[UsesMethod]`, `#[UsesTrait]`, or another `Uses*` attribute.

```php
<?php

declare(strict_types=1);

namespace Dirthara\Http\Tests;

use Dirthara\Http\Authority;
use Dirthara\Http\PercentEncoding;
use PHPUnit\Framework\Attributes\CoversClass;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\Attributes\UsesClass;
use PHPUnit\Framework\TestCase;

#[CoversClass(Authority::class)]
#[UsesClass(PercentEncoding::class)]
final class AuthorityTest extends TestCase
{
    #[Test]
    public function it_parses_a_host(): void
    {
        // ...
    }
}
```

Code that a test only uses does not count towards coverage. A class therefore reaches the 100% required by section 3
only through tests that cover it, not through tests that happen to run it.

PHPUnit enforces both rules, and `failOnRisky` turns each violation into a failure:

* `requireCoverageMetadata` makes a test without a `Covers*` attribute risky. `composer test` fails.
* `beStrictAboutCoverageMetadata` makes a test that runs code it does not list in a `Covers*` or `Uses*` attribute
  risky. This check needs coverage data, so `composer test-coverage` and CI fail, but `composer test` does not.

:::note
These settings are in the `phpunit.xml` of the package template. Packages created before this rule do not carry them
yet, and their tests do not declare the attributes. A package turns the settings on once its tests do.
:::