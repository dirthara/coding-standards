# Project instructions

## Ownership
Dirthara owns this repository. Attribute copyright, licensing, and authorship to `Dirthara` rather than to an individual
maintainer. The MIT `LICENSE` reads `Copyright (c) <year> Dirthara`, and new files or documents that name an owner use
the same name.

## What this repository is
It holds the coding standards every Dirthara package is written to, as markdown under [`docs`](docs). It holds no PHP,
no `composer.json`, and no release.

It also holds no enforcement. `mago.toml`, `phpunit.xml`, `scripts/`, the CI workflow, and `.gitattributes` live in
[`dirthara/package-template`](https://github.com/dirthara/package-template), which every package is copied from. A page here describes a check; it does not 
configure one.

## A rule and its enforcement change together
Never add or change a rule here alone. A page edited on its own is advice that nothing checks; a template edited on its
own is a check nobody can explain. The change spans this repository, the template, and the packages already copied from
the template — the scaffold is copied, not inherited, so nothing propagates on its own.

State in the change what fails when the rule is broken and which command reports it. If nothing fails, say so explicitly
rather than implying a check that does not exist.

## Do not state what is not true
Before writing that every package does something, verify it against `collection`, `database`, `entity`, `schema`, and
`http` in the sibling directories. A standard nobody follows is a wish, and this site is where it would be read as fact.

## Branching
One long-lived branch, `main`. This is unlike the packages, which have a branch per supported version and no `main`, and
it is deliberate: a standard is not versioned per release line. Read [CONTRIBUTING.md](CONTRIBUTING.md) before
branching.

## Committing
Never run `git commit`, `git push`, `git tag`, or anything else that writes to history or to the remote. Stage nothing
and commit nothing: the maintainer commits and pushes every change themselves. Leave the work in the working tree and
say what is ready.

## Documentation
The pages in `docs` are built into the documentation site by [`dirthara/docs`](https://github.com/dirthara/docs), so
they follow the same conventions they describe: front matter carrying `id`, `title`, `sidebar_position`, and
`description`; a `_category_.json` in every subdirectory but never in the root of `docs`; relative links that keep their
`.md` extension; a language on every code fence; admonitions instead of bolded prose; and MDX-safe text, which means
generics, array shapes, and anything containing `<` or `{` go in backticks or the build fails.

`README.md` covers the repository. `CONTRIBUTING.md` owns branching and what belongs here. `SECURITY.md` points reports
at the package that owns the code.
