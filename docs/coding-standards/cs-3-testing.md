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