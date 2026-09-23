# Security Policy

## Supported versions

| Branch | Status |
| --- | --- |
| `main` | Active |

This repository has a single branch and no releases. There is no older version to support: the standards describe how
the framework is written now.

## Reporting a vulnerability

This repository contains documentation and no executable code, so a vulnerability found while reading it almost
certainly belongs to a package. Report it privately against the package that has it, using GitHub's **Report a
vulnerability** form on that repository. Do not disclose vulnerabilities in public issues or pull requests.

If the problem is here, a standard that tells a maintainer to do something unsafe, such as a page recommending an
insecure default or a weakened check, report it privately through this repository's [Report a vulnerability](https://github.com/dirthara/coding-standards/security/advisories/new) form. 
A wrong rule reaches every package, so it is treated as a vulnerability rather than a documentation bug.

Include the page or commit, what the rule leads a maintainer to do, and the impact. Maintainers will acknowledge and
assess the report. Confirmed fixes are published with an advisory crediting the reporter unless they prefer otherwise.

## Scope

In scope: the rules in [`docs`](docs), and this repository's own configuration.

Out of scope: the behaviour of any package, which belongs to that package's policy; the tooling that enforces these
rules, which belongs to [`dirthara/package-template`](https://github.com/dirthara/package-template); and the
documentation site, which belongs to [`dirthara/docs`](https://github.com/dirthara/docs).

Bugs in PHP or in third-party dependencies should also be reported upstream.
