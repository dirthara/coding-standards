---
id: packaging
title: Packaging
sidebar_position: 9
description: What a release archive contains, how its inputs are pinned, and the branch-per-version strategy.
---

## A release contains only what an install loads

`.gitattributes` marks every development path `export-ignore`, so the archive Composer downloads is `src`,
`composer.json`, `LICENSE`, `README.md`, `SECURITY.md`, and `docs`. Tests, CI configuration, the Docker environment, the
Mago and PHPUnit configuration, the agent instructions, the branding, and `composer.lock` stay in the repository and out
of the archive.

```sh
git archive HEAD | tar -t
```

Weight is the smaller reason. The larger one is that a consumer's tooling reads what ships: a security scanner walking
`vendor/` finds any lockfile there and reports the versions pinned inside it, which are not the versions the consuming
project resolved. Every downstream user gets a false alarm. So the lockfile stays committed — CI installs from it, and
`composer validate --strict` keeps it honest — and is excluded from the archive rather than deleted.

Two things to remember:

- **Add the path when you add the file.** A new top-level development file that is not listed in `.gitattributes` ships
  to every consumer. The list is explicit rather than pattern-based so that adding something is deliberate.
- **It only takes effect at the next tag.** A host builds the archive from the tagged tree and applies `export-ignore`
  as that tree declares it.

## Third-party actions are pinned to a commit SHA

Every `uses:` reference to another repository names a full 40-character commit SHA, with the version as a trailing
comment:

```yaml
uses: actions/checkout@fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09 # v5
```

A tag or a branch is a moving target: whoever controls that repository can point it at different code at any time, and
the next run executes that code with access to this repository's secrets. A commit SHA cannot be moved. The trailing
comment is not decoration — Dependabot reads it to know which version the pin represents.

Resolve the reference before pinning it, because two things are easy to get wrong:

```sh
git ls-remote https://github.com/<owner>/<repo> 'refs/tags/<tag>' 'refs/tags/<tag>^{}'
```

An **annotated** tag resolves to a tag object rather than a commit; use the `refs/tags/<tag>^{}` line when there is one.
And a major version such as `v3` is sometimes a **branch**, in which case nothing is returned — pin the commit it
currently points at and comment it with the matching release.

## Dependency updates are automated with a cooldown

`.github/dependabot.yml` carries one entry per ecosystem in use: `composer` while a lockfile is committed,
`github-actions` while there are workflows. An updater that skips an ecosystem leaves that part of the repository
unpatched.

Each entry sets a cooldown of at least three days and groups its ecosystem into a single pull request. A malicious or
broken release is usually discovered and withdrawn within days, so waiting costs nothing and avoids the early-adopter
window the 2024 xz-utils backdoor targeted. Grouping matters because what gets judged — by a reviewer and by a scorer —
is how long the *oldest* open update has been waiting.

## A branch per supported version

Packages have no `main`. Every supported version has its own long-lived branch named for its major and minor (`0.1`,
`0.2`, `1.0`), the newest is the default branch, and a patch is a tag on the branch it belongs to rather than a branch
of its own.

- **A feature** targets the newest release branch.
- **A fix** targets the earliest supported branch that has the bug, and is then **merged forward** in ascending order.
  Merge rather than cherry-pick: a merge records that the fix arrived, so the same conflict is not resolved twice.
- **Never merge a newer branch into an older one.** That is how a patch release ends up containing a feature.

Each branch declares the PHP versions it supports in the `php` constraint of `composer.json` and in the `php` matrix of
`.github/workflows/ci.yml`, and the two have to agree — a constraint that allows a version CI never runs is a promise
nothing checks. Because the workflow lives on the branch, adding a PHP version to `1.0` does not change what `0.1`
tests.

What a branch does not get to do is narrow the constraint. Support is added per branch and removed only by a
framework-wide decision. See [language](language.md).

:::note
This repository is the exception: coding standards describe how the framework is written now, so they have a single
`main`. The documentation site is the other exception, for the same kind of reason.
:::

## A release is gated on a perfect score

Every package scores 100 on [Plumb](https://plumbphp.dev) before it is tagged. The free API needs no key:

```sh
curl https://plumbphp.dev/api/v1/packages/dirthara/<package>
curl -X POST https://plumbphp.dev/api/v1/packages/dirthara/<package>
```

Read the result knowing which half of it you are looking at. The checks for the workflow pins, the updater
configuration, and the security policy read the **repository**, and change as soon as a commit is pushed. The checks for
the lockfile and the lean archive read the **released archive**, and cannot change until a version is tagged. A package
sitting below 100 right after a packaging change is usually waiting for a tag rather than misconfigured.
