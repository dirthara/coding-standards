---
title: 'CS-4: Git & Branching'
sidebar_position: 5
description: Dirthara Git & Branching Conventions
---

# CS-4: Git & Branching strategies

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## 1. Main and working branches

Dirthara packages MUST NOT use the default naming strategy of a `main` and a `develop` branch. Instead, Dirthara
packages create a long-lived branch per supported version.

```
0.1   ← the first release line
0.2   ← the next minor, branched from 0.1
1.0   ← the next major, branched from 0.2
```

The newest release branch is the repository's default branch. A release branch lives as long as the version it carries 
is supported, and is deleted only when that version reaches end of life.

Patch versions do not get a branch. They are tags on the release branch they belong to, so `0.1.3` is a tag on `0.1`.

## 2. Working branches

Work happens on a short-lived branch and arrives through a pull request. Name it after what it does:

- `feature/`
- `bugfix/`
- `hotfix/`
- `task/`

The prefix MUST be followed by the relevant ticket number, followed by a short description of the work in the branch:

- `feature/66-name-of-the-feature`
- `bugfix/2-short-bug-description`

If no ticket number is available, a ticket SHOULD be created before creating the branch.

What you target depends on what you are doing:

- **A feature** targets the newest release branch. Features never go into an older line, because that line is already 
  released.
- **A fix** targets the *earliest supported branch that has the bug*, not the newest. A bug present since `0.1` is fixed 
  on `0.1` and merged or cherry-picked forwards until it reaches the latest version

Branch off the branch you intend to target, so the pull request contains your commits and nothing else.

### Forward merging

A fix that lands on an older branch has to reach every newer one. After the pull request merges, merge the release 
branches upward in ascending order:

```sh
git switch 0.1
git pull

git switch 0.2
git merge 0.1
git push

git switch 1.0
git merge 0.2
git push
```

Merge, rather than cherry-pick. A merge records that the fix reached each branch, so the next forward merge does not 
offer it again and the same conflict is never resolved twice.

:::warning 
Never merge a newer branch into an older one. That drags unreleased work into a released line, which is how a patch ]
release ends up containing a feature.
:::

Expect conflicts in `composer.json` and `.github/workflows/ci.yml` when the branches support different PHP versions. 
Resolve them in favour of the branch you are merging into; the newer branch keeps its own constraint and its own
matrix.

## 3. Commits

* Commit messages MUST contain the relevant ticket number.
* Commit messages SHOULD be short and descriptive, no longer than a single line of 120 characters.
* You SHOULD split your work into multiple smaller commits, to keep the commit messages short.
* You MAY use temporary commits like `WIP`, for instance to commit your work at the end of the day, but these commits
  MUST be removed before the branch is merged into the target branch.
* You SHOULD NOT squash your work into a single commit, keep as much as possible of the history complete.

## 4. Conflicts

* If your bugfix, hotfix, or task branch has conflicts with the target branch, you SHOULD use `git rebase` to resolve these conflicts.
* You SHOULD NOT merge the target branch into your working branch

## 5. Merging

* You MUST delete your working branch after merging it into the target branch.
* Commits SHOULD NOT be squashed when merging a working branch into the target branch.

## 6. Releases

A release is a tag on a release branch:

```sh
git switch 0.1
git tag 0.1.3
git push origin 0.1.3
```

A release is gated on a perfect [Plumb](https://plumbphp.dev) score. Every package SHOULD score 100 before it is tagged.

```sh
curl -X POST https://plumbphp.dev/api/v1/packages/dirthara/database
```

Score the repository first and fix what it reports, because the checks split by what they read. The workflow pins, the 
updater configuration, and the security policy read the repository and change as soon as a commit is pushed. The
lockfile and the lean archive read the released archive, so they cannot pass before a tag exists. A first release 
therefore scores the repository to 100, tags, and rescores to confirm the archive.

Opening a new minor or major means branching from the newest release branch, pointing the repository's default branch 
at it, and applying branch protection to it:

```sh
git switch -c 0.2 0.1
git push -u origin 0.2
```

Then update the supported versions table below and in the SECURITY.md file.
