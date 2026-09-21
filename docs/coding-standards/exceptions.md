---
id: exceptions
title: Exceptions
sidebar_position: 5
description: The exception interface every package defines, the context trait behind it, and when a failure is wrapped.
---

## One exception interface per package

Each package defines a single package-wide exception **interface** extending `Throwable`, and every exception the
package throws implements it. A caller that wants to handle anything from the package catches one type; a caller that
wants one failure catches the specific one.

```php
<?php

declare(strict_types=1);

namespace Dirthara\Http\Exception;

use Throwable;

interface HttpException extends Throwable
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

There is no base exception class. A concrete exception extends the SPL class that describes the failure and implements
the package interface alongside it:

| SPL parent | For |
| --- | --- |
| `InvalidArgumentException` | A caller passed something the package cannot accept. |
| `RuntimeException` | The operation failed while running. |
| `LogicException` | Reserved for bugs — a package does not throw one at a caller. |

A class has one parent, and that slot decides what the exception *means*.

Spend it on a package base class and every exception the package throws inherits its meaning from which package threw
it, which is the least interesting thing about a failure. Spend it on the SPL hierarchy and the parent says what kind of
failure it is. `InvalidUriException extends InvalidArgumentException` states that the caller passed something the
package cannot accept; `UploadedFileException extends RuntimeException` states that the operation failed while running.
That distinction holds no matter which package noticed it, and it is the one a caller actually branches on.

The interface then adds the other fact — who raised it — for free, because a class can implement any number of them.
Nothing is given up to get both.

The practical consequence is that `catch (InvalidArgumentException)`, which a consumer writes without knowing this
package exists, keeps matching. A base class would have silently stopped it from doing so.

Concrete exceptions are `final`. The interface is the extension point.

## Context comes from a trait

The shared behaviour is a `HasExceptionContext` trait, so every exception has it without inheriting from anything:

```php
<?php

declare(strict_types=1);

namespace Dirthara\Http\Exception;

use function array_merge;

trait HasExceptionContext
{
    /**
     * @var array<string, mixed>
     */
    public protected(set) array $context = [] {
        get {
            return $this->context;
        }
    }

    /**
     * @param array<string, mixed> $context
     */
    public function addContext(array $context): static
    {
        $this->context = array_merge($this->context, $context);

        return $this;
    }
}
```

Context is a **property**, not a `getContext()` method. `public protected(set)` makes it readable by anyone and writable
only from inside the hierarchy, which is the whole of what a getter was ever for. `addContext()` is the one supported
way to add to it, and returns `static` so it can be chained onto a rethrow.

Every exception uses the trait and accepts an optional `array<string, mixed> $context` as the fourth constructor
argument, after message, code, and previous, assigning it after `parent::__construct()`:

```php
final class UploadedFileException extends RuntimeException implements HttpException
{
    use HasExceptionContext;

    /**
     * @param array<string, mixed> $context
     */
    public function __construct(string $message = '', int $code = 0, ?Throwable $previous = null, array $context = [])
    {
        parent::__construct($message, $code, $previous);

        $this->context = $context;
    }
}
```

## Exceptions are built by named factories

An exception is never constructed with `new` at the throw site. Each failure gets a named static factory on the
exception class, so the message and the context for that failure are written in exactly one place and cannot drift
between two call sites:

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
- **Arguments are passed by name** — `message:`, `code:`, `previous:`, `context:` — because the constructor's positional
  order is an SPL convention, not something a reader should have to recall.
- **Context goes through the constructor**, not a chained `addContext()`. Reserve `addContext()` for adding what a later
  frame knows to an exception that already exists.
- **A factory that wraps takes the original**, as `forInvalidUri(string $uri, ?Throwable $previous = null)` does, and
  passes it as `previous:`.

## Context, not logging

Context carries what the thrower knew: the operation, what it acted on, the identifier involved.

A package never logs and never depends on a logger. An application's handler reads `$context` and passes it to its PSR-3
logger, adding the caught exception under the `exception` key — that key holds the exception even if context already has
an entry by that name.

:::danger
Never put credentials or sensitive values in context, and never in the message. A connection string, a password, a
token, or a row of user data in an exception ends up in a log file, an error tracker, and a bug report. A dependency's
message can contain any of these, so do not copy one verbatim into yours without knowing what it can hold.
:::

Name in each package's own documentation which identifiers its exceptions should carry and which values they must never
carry, so the rule is concrete rather than a principle.

## Wrapping a dependency's failure

When a dependency throws, catch the types it documents and rethrow a specialised exception of this package, passing the
original as `previous` and attaching the operation's context. A caller should never need a dependency in a `catch` block
to handle a failure this package caused.

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

## Choosing what to throw

- **An invalid argument is an exception, not a return value.** A method that cannot do what it was asked throws; it does
  not return `null`, `false`, or an empty result to mean failure. `null` is reserved for a genuine absence that the
  caller is expected to handle, and that is documented as such.
- **A message is a sentence.** Capitalised, ending in a period, naming the thing that failed and the value that caused
  it, quoted. `The given port "70000" is invalid.` beats `Invalid port`.
- **An error code is `0`** unless the package defines a scheme for them. An unexplained integer is worse than none.
- **Every documented failure has a test** that triggers it and asserts the type, and asserts the context when the
  context is the point.

:::note
`http` is the package this page is written from. `collection` and `entity` still define a base exception *class* with a
`getContext()` method, from the convention that preceded this one, and are brought up to the interface when they are
next touched.
:::
