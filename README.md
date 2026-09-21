<p align="center">
  <img src="logo-no-bg.png" alt="Dirthara" width="480">
</p>

# Dirthara Coding Standards

The coding standards every Dirthara package is written to, published on the Dirthara documentation site at
<https://dirthara.github.io/docs/>, which documents the whole framework.

This repository holds the rules as markdown in [`docs`](docs/intro.md): the PHP baseline, the directory and naming
layout, the formatting and import order, how types are expressed, how failures are reported, what a test has to prove,
what the documentation covers, which tools run, and what a release is allowed to contain.

## What is here, and what is not

The standards are text. The things that *enforce* them are not here — they live in
[`dirthara/package-template`](https://github.com/dirthara/package-template), which every package is copied from:

| Enforced by | Lives in |
| --- | --- |
| Formatting, linting, analysis, architecture rules | `mago.toml` |
| Import order | `scripts/sort-imports.php` |
| Test strictness | `phpunit.xml` |
| 100% line coverage of `src` | `scripts/coverage.php` |
| Everything above, on every push | `.github/workflows/ci.yml` |
| What a release archive contains | `.gitattributes` |

So a rule described here is a rule something fails on, and each page says which command catches it. Where a rule is a
convention no tool can check, the page says that too, and review is the enforcement.

There is no PHP source, no `composer.json`, and no release. Nothing here is installed with Composer.

## Reading the standards

Read them on the [documentation site](https://dirthara.github.io/docs/), where they are built together with the packages
they describe. The markdown in [`docs`](docs/intro.md) is the source of those pages; the site pulls this repository at
build time rather than keeping a copy.

## Changing a standard

A standard and its enforcement change together: the page here, the template that checks it, and the packages already
copied from that template. A page edited on its own leaves a rule nothing checks; a template edited on its own leaves a
check nobody can explain.

See [CONTRIBUTING.md](CONTRIBUTING.md) for how that lands, and [AGENTS.md](AGENTS.md) for agent instructions.

## Branching

One long-lived branch, `main`. Packages have a branch per supported version and no `main`; this repository does not,
because a standard is not versioned per release line. A package that predates a rule is brought up to it rather than
exempted from it.

## Security

This repository contains no code. Report a vulnerability in the package that has it, through that package's GitHub
advisory form. See [SECURITY.md](SECURITY.md).

## License

Copyright (c) 2026 Dirthara. Released under the [MIT License](LICENSE).
