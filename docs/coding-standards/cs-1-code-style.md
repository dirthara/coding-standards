---
title: 'CS-1: Code Style'
sidebar_position: 2
description: Dirthara code style
---

# CS-1: PHP Code Style Standards

Dirthara is framework primarily written in modern PHP, therefore these code style standards cover mostly PHP. If at any 
point in the future it becomes necessary to add code style for other programming languages we will do so.

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

## 1. PSR-12

This guide extends and expands on [PSR-12](https://www.php-fig.org/psr/psr-12/),
the extended coding style guide, which in turn expands and replaces PSR-1 and PSR-2.

* Files MUST use only `<?php` tags.
* The opening php statement MUST be followed by a `decalre(strict_types=1);`
* Files MUST use only UTF-8 without BOM for PHP code.
* Files SHOULD either declare symbols (classes, functions, constants, etc.) or cause side effects (e.g. generate output,
  change .ini settings, etc.) but SHOULD NOT do both.
* Namespaces and classes MUST follow the autoloading PSR: [PSR-4](https://www.php-fig.org/psr/psr-4/).
* Class names MUST be declared in `StudlyCaps`.
* Class constants MUST be declared in all upper case with underscore separators.
* Method names MUST be declared in `camelCase`.
* Enum cases MUST be declared in `StudlyCaps`.
* Code MUST use 4 spaces for indenting, not tabs.
* There MUST NOT be a hard limit on line-length; the soft limit MUST be 120 characters.
* There MUST be one blank line after the `namespace` declaration, and there MUST be one blank line after the block of
  `use` declarations.
* Opening braces for classes MUST go on the next line, and closing braces MUST go on the next line after the body.
* Opening braces for methods MUST go on the next line, and closing braces MUST go on the next line after the body.
* Visibility MUST be declared on all properties and methods; `abstract` and `final` MUST be declared before the
  visibility; `static` and `readonly` MUST be declared after the visibility.
* Control structure keywords MUST have one space after them; method and function calls MUST NOT.
* Opening braces for control structures MUST go on the same line, and closing braces MUST go on the next line after the
  body.
* Opening parentheses for control structures MUST NOT have a space after them, and closing parentheses for control
  structures MUST NOT have a space before.

## 2. DocBlocks

This guide extends and expands on [PSR-5](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc.md),
the PHPDoc standard.

* DocBlocks are mandatory if the signature can't be used for type information.
* There SHOULD not be blank lines between DocBlocks and the documented element.
* All items of the `@param`, `@throws`, `@return`, `@var`, and `@type` phpdoc tags MUST be aligned vertically.
* Annotations in DocBlocks SHOULD be ordered so that param annotations come first, then throws annotations, then return
  annotations.
* Scalar types MUST always be written in the same form. `int`, not `integer`; `bool`, not `boolean`; `float`, not
  `real` or `double`.
* DocBlocks SHOULD start and end with content, excluding the very first and last line of the DocBlocks.
* If a DocBlock needs to be added to f.i. a method to document an `@throws`, only the elements that can't be inferred 
  from the signature SHOULD be documented.

```php
/**
 * This is an example DocBlock description.
 * 
 * @param string|null $param3 

 * @throws InvalidArgumentException
 */
public function example(int $param1, Class $param2, $param3 = null): string
{
    if (!is_null($param3) {
        throw new InvalidArgumentException();
    }

    return 'string';
}

public function example2(string $param1, string $param2, ?string $param3 = null): string
{
    if (!is_null($param3) {
        return $param1;
    }

    return $param2;
}
```

The last example in the above code example does not need a DocBlock, because al information can be deferred from the
method signature.

## 3. Inline comments

Try to prevent inline comments as much as possible. The code SHOULD be self-explanatory by adhering to the Single
Responsibility Principle and having descriptive class and method names. Only add inline comments when it's actually
necessary.

## 4. Blank line after open tag

Ensure there is no code on the same line as the PHP open tag, and it is followed by a blank line.

```php
<?php

declare(strict_types=1)

class D
{
}
```

## 5. Class imports

* Unused class imports SHOULD be removed.
* There SHOULD be no empty lines between use statements.
* Use statements MUST be ordered by length.
* All classes, interfaces and traits SHOULD be imported via a `use` statement.
* All root-level PHP functions and classes SHOULD be imported via a `use` statement.
* `use` declarations MUST be grouped in `use` declarations that import classes, functions and constants, with an
  empty line between each block.

```php
<?php

use A;
use B;
use C;

use function x;
use function y;
use function longer;

use const first;
use const second;

class D
{
}
```

## 6. Arrays and parameters

* Multiline arrays MUST always contain a trailing comma.
* PHP short array syntax MUST be used in favour of the traditional syntax.
* If a method signature extends the line limit, each parameter SHOULD be placed on its own line.
* When spacing parameters across multiple lines, a trailing comma SHOULD be used.
* When calling a method with multiple parameters that extends the line limit, each parameter SHOULD be placed on its
  own line.
* When spacing parameters in a method call across multiple lines, a trailing comma SHOULD be used.

```php
$array = [
    [
        1,
        2,
        3,
    ],
];

class A
{
    public function veryLongFunctionWithParameters(
        LongClassName $firstParameter, 
        AnotherLongClassName $anotherParameter, 
        ThirdClassName $thirdParameter, 
    ): void
}

$a = new A();
$a->veryLongFunctionWithParameters(
    new LongClassName(),
    new AnotherLongClassName(),
    new ThirdClassName(), 
);
```

## 7. Class properties and constants

* Class properties MUST adhere to the same naming convention as `variables`. See CS-2.
* Class properties SHOULD be `readonly` as much as possible, only use mutable properties when actually needed.
* Class properties MUST have a visibility set to either `public`, `protected` or `private`, or a combination.
    * `protected` or `private` are preferred over `public`.
        * Use the correct visibility when your class is extended:
            * `protected` can be accessed by the class itself and its inherited classes.
            * `private` can be accessed only by the class itself.
    * Only use `public` for _DTOs_ (_Data Transfer Objects_) or `readonly` properties.
    * For public *read* access to a property you SHOULD use a `get` property hook or a `readonly` property.
    * For public *write* access to a property you SHOULD use a `set` property hook.
* An empty line MUST be placed between each class property.
* Promoted properties SHOULD be used as much as possible.
* When a class uses promoted properties, every property MUST be placed on its own line.
* Class constants MUST adhere to the same naming convention as `constants`. See CS-2.
* Class constants MUST have a visibility set to either `public`, `protected` or `private`.
* Class constants SHOULD have a type declared.

```php
<?php

class PropertyDemonstration
{
    private array $thisIsAnArray = [];
    
    private string $name = 'Default value' {
        get => $this->value;
        set(string $value) {
            if (empty($value)) {
                throw new \InvalidArgumentException();
            }
            
            $this->name = $value;
        }
    }
    
    private const int CLASS_CONSTANT = 1;

    public function __construct(
        private readonly Repository $repository, 
        private readonly OtherRepository $otherRepository, 
    ) {}
}
```

## 8. Automated tools

* The Mago formatter SHOULD be used to fix problems with code style.
* The developer SHOULD not rely only on this fixer, but MUST actively implement these code style standards.
* Errors in code style WILL fail the merge request pipeline and MUST be fixed before continuing.

## 9. Unary operators

Place unary operators (!, --, ++, ...) adjacent to the affected variable.

```php
$i++;
$i--;

--$i;
++$i;

function foo(&$variable)

if (!$expression)
if (!isset($variable))

return !$expression;
```

## 10. Return types and PHP8 language constructs

Return type should be separated from `:` with a space. `:` should be connected to the closing `)` of the method or 
function.

```php
public function foo(): Bar
{
}
```

Use constructor promotion as much as possible. Whenever constructor promotion is used, all properties MUST be placed
on their own line, with a trailing comma.

```php
<?php

class PropertyDemonstration
{
    private array $thisIsAnArray = [];

    /**
     * @var int
     */
    const CLASS_CONSTANT = 1;

    public function __construct(
        private readonly Repository $repository, 
        private readonly OtherRepository $otherRepository, 
    ) {
    }
}
```

## 11. Traits

Each individual Trait that is imported into a class MUST be included one-per-line, and each inclusion MUST have its
own `use` import statement. Trait imports MUST be ordered by length.

```php
<?php

class A
{
    use X;
    use Y;
    use Longer;
}
```

## 12. Variable declaration vs. return directly

**Directly return** the value when there is only one operation,

```php
public function foo(string $bar): string
{
    return ucwords($bar);
}
```

**Declare variables** in case there are multiple small operations. That way you can set a breakpoint to debug, and it
will be easier to understand what the operation does (without the need of adding inline-comments).

```php
public function foo(string $bar): void
{
    $ucWordsBar = ucwords($bar);
    $output     = str_replace("world", "user", $ucWordsBar);
    
    echo $output;
}
```

**Split the code into several methods** when there are larger or more complex operation (Single Responsibility
Principle).

```php
class UserInfo
{
    public function foo(string $bar): void
    {
        $ucWordsBar = ucwords($bar);
        $replaced   = str_replace("world", "user", $ucWordsBar);
        $stats      = $this->userStats($replaced);
        
        echo $replaced . $stats;
    }
    
    public function userStats(): string
    {
        $ip    = $_SERVER['REMOTE_ADDR'];
        $count = $this->getCountForIP($ip);
        
        return sprintf('You visited this site %d times from ip address %s', $count, $ip);
    }
    
    public function getCountForIP(string $ip): int
    {
        // Some operation
        $count = 0;
        
        return $count;
    }
}
```

## 13. Method order

* If a class has a `__construct()` method defined, this SHOULD be the first method.
* If a class has a `__destruct()` method defined, this SHOULD come directly after the constructor, or if there is no
  constructor, this SHOULD be the first method.
* Methods and properties SHOULD be grouped by their visibility.
* The order of other methods in classes SHOULD be `public` methods first, then `protected` methods, and `private`
  methods last.
* Magic methods SHOULD NOT be used, but if they are used, they MUST adhere to the visibility rules, and they SHOULD be
  the first methods within their visibility except `__construct()` and `__destruct()`.
* The order of properties in classes SHOULD be based on the same visibility rules as that of methods.
* `static` methods and properties SHOULD be placed before other methods and properties within their visibility.
* `abstract` methods and properties SHOULD be placed after `static` methods and properties, but before other methods and
  properties within their visibility.

## 14. Magic methods

* Magic methods SHOULD NOT be used, unless there is a valid reason to.
* The magic method `__construct()` MUST be used to declare a class constructor, as this is the only way to do so.
* The magic method `__destruct()` MUST be used to declare a class destructor, as this is the only way to do so.
* The magic methods `__sleep()`, `__wakeup()`, `__serialize()` and `__unserialize()` MAY be used to define serialization
  behaviour. The `Serializable` interface is PREFERRED over magic methods.
* The magic method `__toString()` MAY be used to define stringify behaviour. If you use this method, the `Stringable`
  interface SHOULD also be used.
* The magic methods `__call()`, `__callStatic()`, `__get()`, `__set()`, `__isset()` and `__unset()` SHOULD NOT be used.
* The magic method `__invoke()` MAY be used to promote SRP.
* The magic methods `__set_state()` and `__debugInfo()` MAY be used for debugging purposes, but MUST NOT be commited
  into production code.
* The magic method `__clone` MAY be used when cloning an object has specific requirements, though the `clone with` 
  syntax is PREFERRED.

## 15. Final and readonly classes

Classes MUST be declared `final` and `readonly` wherever possible. This is the default; deviate only when the language
or framework prevents it.

```php
<?php

final readonly class FileService implements FileServiceContract
{
    public function __construct(
        private FilesystemManager $filesystemManager,
        private string $diskName,
    ) {
    }
}
```

## 16. File endings

* There should be no newline at the end of the file.
