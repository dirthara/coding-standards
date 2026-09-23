---
title: 'CS-5: Security'
sidebar_position: 6
description: Dirthara Security Conventions
---

# CS-5: Security

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

Every package in the Dirthara vendor space MUST have a SECURITY.md file detailing how to report security issues 
in a save and protected manner without revealing the security issue publicly. Security issues reported through 
the correct path MUST be resolved as quickly as possible. Keep in mind though that Dirthara is created and maintained 
as a hobby project and maintainers have other, ofter more important responsibilities in their lives.

## Your code

You MUST make sure your code does not contain any security issues, like SQL injection, XSS or other vulnerabilities.

## Dependencies

Dependencies are defined as code from external resources that can be loaded by the application. This can be composer
packages, PEAR packages or downloaded (code) packages.

### Adding dependencies

* You SHOULD only add dependencies when it is needed
    * You MAY add dependencies that add complex code that would be hard to implement yourself, or that requires specific
      knowledge.
    * You MAY add dependencies that add large patches of code that would take a long time to implement yourself.
    * You SHOULD NOT add dependencies for code that could be built by the team itself.
    * You SHOULD NOT rely on a dependency from another package.

### Updating dependencies

* External dependencies MUST be kept up to date.
* Bugfixes (`x.x.1`) and feature updates (`x.1.x`) MAY be added in any working branch if it's needed, but SHOULD be done in a separate branch with a separate pull request
* Updating to a new major version (`2.x.x`) SHOULD be done in a separate branch with a separate pull request, as this often requires changes in other code as well.

