---
title: 'CS-6: Documentation'
sidebar_position: 7
description: Dirthara Documentation Conventions
---

# CS-6: Documentation

## README.md

The README covers the repository, not the API. In order:

1. The logo, centred above the title, linked by **relative** path. An absolute `raw.githubusercontent.com` URL names one
   branch and is wrong on every other one, and packages have a branch per version.
2. A short description of the package, linking to the documentation site at
   [dirthara.github.io/docs](https://dirthara.github.io/docs/). Every package points there, so a reader
   who lands on any repository can reach the whole framework.
3. Installation and requirements.
4. The local development environment.
5. How to run the tests.
6. How to run the formatter, linter, analyzer, and coverage gate.
7. Contributing and branching, linking to `CONTRIBUTING.md`.
8. Reporting a vulnerability, linking to `SECURITY.md`.
9. The license, linking to `LICENSE`.

Usage, options, and API documentation do not go in the README. It links to `docs`.

## CONTRIBUTING.md and SECURITY.md

`CONTRIBUTING.md` owns branching, the release process, and what a pull request has to satisfy. `SECURITY.md` states the
supported versions, how to report privately — through GitHub's advisory form, never an email address — and what is in
and out of scope. Both carry a supported-versions table, and the two tables list the same branches; update them
together.

## Usage Documentation

`docs/` holds the usage documentation as markdown files. The `dirthara/docs` repository pulls these files at the ref 
each version pins and builds [the documentation site](https://dirthara.github.io/docs/) from them with Docusaurus, so write them as
if they are already on that site. A package never publishes a site of its own.

The conventions the build depends on:

- **Front matter on every page**, carrying `title`, `sidebar_position`, and `description`. Add `sidebar_label`
  when the sidebar needs something shorter than the title.
- **A `_category_.json` in every subdirectory**, setting `label`, `position`, and a `generated-index` link. Not in the
  root of `docs` — the site generates that one, because a package does not know what it is called next to the others.
- **Relative links that keep the `.md` extension**, so Docusaurus resolves and validates them. A broken link, anchor, or
  image fails the site build rather than warning, and the failure names the file in the package that owns it.
- **A language on every code fence.**
- **Admonitions** (`:::note`, `:::tip`, `:::caution`, `:::danger`) for caveats, rather than bolded prose.
- **MDX-safe prose.** Wrap generics, array shapes, and anything containing `<` or `{` in backticks, or MDX parses it as
  JSX and the build fails.
- **Every package ships `docs/intro.md`** from its first commit. It is what the landing-page card and the footer link
  to.

### Writing it

Document how the package is used, which options exist, and what each one means. Options go in a table with their type,
default, and meaning. Say why a default is what it is when the reason is not obvious, and describe the behaviour that
will surprise someone *before* they hit it.

Keep it truthful against the source. When behaviour changes, the page that describes it changes in the same pull 
request. A pull request that changes behaviour and not its documentation is incomplete, and will be declined.