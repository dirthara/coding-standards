---
title: 'CS-2: Naming Conventions'
sidebar_position: 3
description: Dirthara Naming Conventions
---

# CS-2: Naming Conventions

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## 1. Code Naming Conventions

### 1.1. Prefixes and suffixes

In general, don't use naming prefixes or suffixes like `Interface`, `Abstract`, `Listener`, `Event`, etc. Instead, make
sure the namespace is correct and use a descriptive name for the class that actually tells you what it does.

### 1.2. Classes

* Class names MUST be declared in `StudlyCaps`.
* Method names MUST be declared in `camelCase`.
* Class properties MUST be declared in `camelCase`
* Class constants MUST be declared in `SNAKE_CASE`.

### 1.3. Enums

* Enum cases MUST be declared in `StudlyCaps`.
* Enum cases SHOULD be backed as much as possible.

### 1.4. Variables and Constants

* Variable names MUST be declared in `camelCase`.
* Constant names MUST be declared in `SNAKE_CASE` with all caps.
* Variable and constant names MUST be meaningful, meaning their purpose SHOULD be obvious by their name.
* Variable names MUST NOT consist of only one letter, such as `$e` or `$x`.
    * Loop variables are the exception to this rule and SHOULD always be one letter, starting at `$i` and following the
      alphabet from there for each nested loop. If the rest of the alphabet is not enough letters for the amount of 
      nested loops you have, something has gone wrong somewhere.

## 2. Package names

When creating new packages use a short and descriptive name for the package. The framework name itself is where the 
cool name lives, but when using the packages it SHOULD be immediately clear what the package does.