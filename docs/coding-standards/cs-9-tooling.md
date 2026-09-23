---
title: 'CS-9: Tooling'
sidebar_position: 10
description: Dirthara Tooling
---

# CS-9: Tooling

## Nothing is installed on the host

Every package ships a Docker Compose environment, and every PHP and Composer command runs inside it. The image provides
the PHP version the branch supports, Composer, Mago, Xdebug, and a PDO driver for each database the package supports.

```sh
LOCAL_UID=$(id -u) LOCAL_GID=$(id -g) docker compose up -d --build php
docker compose exec php composer install
```

`LOCAL_UID` and `LOCAL_GID` build the container's user with your own ids, so files it writes into the working directory
stay editable. `docker compose up -d php` also starts the database services and waits until each reports healthy, so the
suite never races a database that is still booting.

The point of the container is not convenience. It is that CI runs the same image, built the same way, so a green run
locally means a green run there. A toolchain installed per machine drifts, and the drift only ever surfaces in CI.

## The Composer scripts

| Script | Does |
| --- | --- |
| `composer ci` | Everything CI runs: Mago, the tooling tests, the suite with coverage, the gate. |
| `composer mago` | Formatter check, import check, lint, analyze, guard — all of them, even after one fails. |
| `composer fmt` | Format and sort imports. |
| `composer fmt-check` | Both, as a check. |
| `composer lint` / `lint-fix` | Lint; apply fixes, including potentially unsafe ones. |
| `composer analyze` | Static analysis. |
| `composer guard` | The configured architecture rules. |
| `composer cs` | `lint-fix` then `fmt` — the "make it right" command. |
| `composer test` | The test suite. |
| `composer test-coverage` | The suite with Xdebug on, writing `build/coverage/clover.xml`. |
| `composer coverage` | The 100% line-coverage gate. |
| `composer imports` / `imports-check` | The import sorter. |

Two habits: run `composer cs` before you read your own diff, and `composer ci` before you open a pull request.
`composer cs` applies potentially unsafe lint fixes, so read what it changed.

:::note
Xdebug is inactive by default and enabled only for the coverage run, because a loaded debugger costs most of the suite's
speed.
:::

## What CI enforces

The workflow has two jobs and one gate:

- **Mago** runs the formatter check, the import-sorter's own tests, the import check, the linter, the analyzer, and the
  architecture rules. Every step runs even when an earlier one failed, so one run reports everything wrong rather than
  the first thing.
- **Tests** runs once per PHP version in the matrix, which lists every released version the `php` constraint in
  `composer.json` allows. The two are kept in step; the matrix entry comes first. It validates `composer.json` with
  `--strict`, installs from the committed lockfile, runs the suite with coverage, annotates the failures, and applies
  the coverage gate.
- **CI** is a job that succeeds only when both succeeded. It is the single required check in branch protection, so
  adding a PHP version to the matrix never means editing branch protection.

## The configuration is shared, not per package

`mago.toml`, `phpunit.xml`, the workflow, the Dockerfile, `compose.yaml`, the import sorter, and the coverage script all
come from [`dirthara/package-template`](https://github.com/dirthara/package-template). A package does not invent its own.

The scaffold is **copied, not inherited**. That is the one thing to remember: improving the tooling in one package
leaves every other package behind unless the change is carried back into the template, and from the template into the
packages already copied from it. A standard that only holds in the newest package is not a standard.