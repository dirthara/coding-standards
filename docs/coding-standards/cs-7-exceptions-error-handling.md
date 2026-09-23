---
title: 'CS-7: Exceptions & Error Handling'
sidebar_position: 8
description: Dirthara Exceptions & Error Handling
---

# CS-7: Exceptions & Error Handling

## One exception interface per package

Each package defines a single package-wide exception **interface** extending `Throwable`, and every exception the
package throws implements it. A caller that wants to handle anything from the package catches one type; a caller that
wants one failure catches the specific one.

The main exception interface MUST define two things: a publicly readable array with the context, and a public method 
to add context to the exception. How this works internally is up to the package itself, but the main interface MUST 
declare these options at the very least:

```php
<?php

declare(strict_types=1);

namespace Dirthara\PackageName\Exception;

use Throwable;

interface PackageNameException extends Throwable
{
    /**
     * @var array<string, mixed>
     */
    public array $context { get; }

    /**
     * @param array<string, mixed> $context
     */
    public function addContext(array $context): static;
}
```

When creating custom exceptions in a package, choose an SPL base exception from PHP to extend that fits your exception.

| SPL parent                 | For                                                           |
|----------------------------|---------------------------------------------------------------|
| `InvalidArgumentException` | A caller passed something the package cannot accept.          |
| `RuntimeException`         | The operation failed while running.                           |
| `LogicException`           | Reserved for bugs — a package does not throw one at a caller. |

Always throw a custom exception that implements the main exception interface of the package, never throw SPL exceptions 
yourself. For instance, if you receive an invalid argument, do no throw `\InvalidArgumentException`. Instead, create 
a custom `InvalidArgument` exception in the package that extends `\InvalidArgumentException` and implements the 
`PackageNameException`. 

You SHOULD make custom exceptions specific, so a custom `InvalidArgumentException` as described above is probably too 
generic. You MAY group related exceptions together in one exception.

## Named factories

An exception SHOULD not be constructed with `new` at the throw site. Each failure SHOULD get a named static factory on 
the exception class, so the message and the context for that failure are written in exactly one place and cannot drift
between two call sites. You SHOULD also provide as much context as possible.

```php
public static function unableToMove(string $targetPath): self
{
    return new self(message: sprintf('Unable to move uploaded file to "%s".', $targetPath), context: [
        'targetPath' => $targetPath,
    ]);
}

public static function invalidPort(int $port): self
{
    return new self(message: sprintf('The given port "%d" is invalid.', $port), context: ['port' => $port]);
}
```

- **The factory name is the failure**, read from the throw site: `throw UploadedFileException::alreadyMoved();`.
- **A factory that wraps takes the original**, as `forInvalidUri(string $uri, ?Throwable $previous = null)` does, and
  passes it as `previous:`.
- Error messages are written in English and SHOULD contain a detailed error message. 
- A package never logs and never depends on a logger.

:::danger
Never put credentials or sensitive values in context, and never in the message. A connection string, a password, a
token, or a row of user data in an exception ends up in a log file, an error tracker, and a bug report. A dependency's
message can contain any of these, so do not copy one verbatim into yours without knowing what it can hold.
:::

## Wrapping a dependency's failure

When a dependency throws, catch the types it documents and rethrow a specialised exception of the current package, 
passing the original as `previous` and attaching the operation's context. A caller should never need a dependency in 
a `catch` block to handle a failure this package caused.

- **Catch the specific types, never `Throwable`.** A `TypeError` or a `LogicException` is a bug in this package rather
  than a failure of the operation. Turning one into a domain exception hides it, and the test suite stops being able to
  find it.
- **Do not rewrap an exception that already implements this package's interface.** Add what you know with `addContext()`
  and rethrow, so the specific type survives for the caller.
- **Do wrap an exception from another Dirthara package.** Which package this one is built on is not something its
  callers should have to catch.

```php
try {
    $statement = $this->connection->prepare($sql);
} catch (DatabaseException $exception) {
    throw EntityDatabaseException::prepareFailed($metadata->entity, previous: $exception);
}
```