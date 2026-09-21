# Contributing

## Branching

This repository has a single long-lived branch, `main`.

That is deliberately unlike the packages, which have no `main` and give every supported version its own branch. A
package needs a branch per version because `0.1` and `1.0` are different code, maintained and released independently. A
coding standard is not: it describes how the framework is written *now*. Two branches would mean two answers to the same
question, and a package sitting on the older one would be conforming to a rule nobody else follows.

A package that predates a rule is therefore brought up to it, not exempted from it. If bringing it up is too large for
one pull request, say so on the page — an honest "not yet true of every package" is worth more than a second branch.

Work happens on a short-lived branch and arrives through a pull request. Name it after what it does:

| Branch | For |
| --- | --- |
| `feature/<issue>-short-slug` | A new or changed standard. |
| `bugfix/<issue>-short-slug` | A page that is wrong about what the tooling does. |

## What belongs here, and what does not

This repository holds the rules. It does not hold the code that enforces them, and it does not hold usage documentation
for any package.

- **A new rule, or a change to one**, is a pull request here — *and* a pull request against
  [`dirthara/package-template`](https://github.com/dirthara/package-template), and against the packages already copied
  from it. See below.
- **A page that describes the tooling wrongly** is a pull request here. This is the common case: the template changed
  and the page did not.
- **A change to `mago.toml`, `phpunit.xml`, the CI workflow, the import sorter, or the coverage gate** is a pull request
  against the template. The page here describes it; it does not configure it.
- **How to use a package** belongs in that package's `docs` directory, so that a change in behaviour and the paragraph
  describing it are reviewed together.
- **How the documentation site is built** belongs in [`dirthara/docs`](https://github.com/dirthara/docs).

## A standard and its enforcement change together

This is the rule that matters most here, because the failure is quiet.

A page edited on its own leaves a rule that nothing checks: it is advice, and it is violated within two packages. A
template edited on its own leaves a check that nobody can explain, which is worse — the next person to hit it removes
it.

So a pull request that adds or changes a rule states, in its description:

1. **What fails** when the rule is broken, and which command reports it. If the answer is "nothing", say that explicitly
   and explain why review is enough.
2. **The template change** that enforces it, linked.
3. **The packages that do not conform yet**, and whether they are being brought up in this change or tracked in an
   issue.

The scaffold is copied, not inherited. Nothing propagates on its own.

## Before you open a pull request

There is no build to run — the repository is markdown. What there is to check, you check by reading:

- **The rule is true of the packages.** A standard nobody follows is a wish. Before writing "every package does X", grep
  for X in `collection`, `database`, `entity`, `schema`, and `http`.
- **The page names its enforcement.** See above.
- **The markdown is MDX-safe.** Every page in `docs` is built by Docusaurus. Generics, array shapes, and anything
  containing `<` or `{` go in backticks, or the site build fails. So do broken relative links, broken anchors, and
  broken images.
- **The front matter is complete.** `id`, `title`, `sidebar_position`, and `description` on every page; a
  `_category_.json` in every subdirectory, but never in the root of `docs` — the site generates that one.
- **The conventions the pages describe are the ones they follow.** These pages are the example.

To see a change rendered before it is merged, build the site against your working copy:

```sh
docker compose exec -e DOCS_SOURCE_CODING_STANDARDS=../coding-standards docusaurus npm run pull
```

from a checkout of [`dirthara/docs`](https://github.com/dirthara/docs).

## Publishing

Merging to `main` is the release. There is no tag and no version: the site pulls this repository at build time, and a
nightly build picks up a merge here without a commit anywhere else.
