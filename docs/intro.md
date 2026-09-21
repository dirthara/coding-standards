---
id: intro
title: Coding Standards
sidebar_position: 1
description: The coding standards every Dirthara package is written to, and how they are enforced.
---

Dirthara packages are written by different hands at different times and are meant to read as one library. These are the
rules that make that true: how a file is laid out, what the type system is expected to carry, how failures are reported,
what a test has to prove, and what a release is allowed to contain.

Two things make this repository different from the packages it describes.

It has no code and no release. Nothing here is installed with Composer; the standards are text, and the things that
enforce them — `mago.toml`, `phpunit.xml`, the import sorter, the CI workflow, the coverage gate — live in
[`dirthara/package-template`](https://github.com/dirthara/package-template), which every package is copied from.

It has a single `main` branch, where a package has one branch per supported version. A standard is not versioned per
release line: it describes how the framework is written now, and a package that predates a rule is brought up to it
rather than exempted from it.

:::note
A rule here is only real if something fails when it is broken. Each page says which command catches the violation. Where
a rule is a convention that no tool can check, it says so, and review is the enforcement.
:::

## The rules

| Page | Covers |
| --- | --- |
| [Language](coding-standards/language.md) | The PHP baseline and which language features to reach for. |
| [Layout and naming](coding-standards/layout.md) | Directories, namespaces, and what things are called. |
| [Formatting and imports](coding-standards/formatting.md) | The formatter, and the import order it does not own. |
| [Types](coding-standards/types.md) | Native types, generics in docblocks, and the analyzer. |
| [Exceptions](coding-standards/exceptions.md) | The exception interface, its context trait, and wrapping. |
| [Testing](coding-standards/testing.md) | Test layout, style, and the coverage gate. |
| [Documentation](coding-standards/documentation.md) | The README, and the `docs` directory this site is built from. |
| [Tooling](coding-standards/tooling.md) | The development environment and the commands CI runs. |
| [Packaging](coding-standards/packaging.md) | What a release contains, and how its inputs are pinned. |

## Changing a standard

A standard and its enforcement change together. Editing a page here without changing the template leaves a rule nothing
checks, and changing the template without editing a page here leaves a check nobody can explain. Both belong in the same
pull request, along with the packages already copied from the template. See
[CONTRIBUTING.md](https://github.com/dirthara/coding-standards/blob/main/CONTRIBUTING.md).
