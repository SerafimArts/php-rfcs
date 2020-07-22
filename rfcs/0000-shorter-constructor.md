 * Name: `Shorter Constructor`
 * Date: 2020-07-22
 * Author: Kirill Nesmeyanov <nesk@xakep.ru>
 * Proposed Version: PHP 8.1
 * RFC PR: [php/php-rfcs#0004](https://github.com/SerafimArts/php-rfcs/blob/shorter-constructor/rfcs/0000-shorter-constructor.md)

## Introduction

Current [Constructor Property Promotion](https://wiki.php.net/rfc/constructor_promotion) 
syntax assumes redundant `{}` suffix which in the case of following the PSR 
assumes an extra line break.

### Inline Example

```php
final class UserValueObject
{
    public function __construct(public string $name, public string $email)
    {
    }
}
```

### Multiline Example

```php
final class UserValueObject
{
    public function __construct(
        public string $name, 
        public string $email,
        public \DateTimeInterface $created,
        public ?\DateTimeInterface $updated = null
    ) {
    }
}
```

## Proposal

It is proposed to provide an alternative constructor syntax (trailing `;`) that 
allows declaration without a mandatory body.

Such changes allow you to get rid of visual noise and are syntactically 
consistent with the already existing declarations of interfaces and abstract 
methods where the method body is also missing.

### Inline Example

```php
final class UserValueObject
{
    public function __construct(public string $name, public string $email);
}
```

### Multiline Example

```php
final class UserValueObject
{
    public function __construct(
        public string $name, 
        public string $email,
        public \DateTimeInterface $created,
        public ?\DateTimeInterface $updated = null
    );
}
```

### Inheritance

Regular arguments don't make sense if the constructor body is empty, so it 
makes sense to pass them to the parent class (if they exist) or throw an 
error if they don't exist.

Below is a simple example of a similar vector based case that uses the 
existing syntax:

```php
class Vector2
{
    public function __construct(
        public float $x = 0.0,
        public float $y = 0.0
    ) {
    }
}

class Vector3 extends Vector2
{
    public function __construct(
        float $x = 0.0,
        float $y = 0.0,
        public float $z = 0.0
    ) {
        parent::__construct(x: $x, y: $y);
    }
}
```

In the case of the new syntax, when the body is dropped, nothing should change.

```php
class Vector2
{
    public function __construct(
        public float $x = 0.0,
        public float $y = 0.0
    );
}

class Vector3 extends Vector2
{
    public function __construct(
        float $x = 0.0, // passed to parent argument "x"
        float $y = 0.0, // passed to parent argument "y"
        public float $z = 0.0
    );
}
```

## Possible Errors

### Lack Of Constructor Promotion

```php
class Example
{
    public function __construct($value);
}

// Fatal error: Uncaught TypeError: 
// Example::__construct(): Argument #1 ($value) must be promoted or Example class must contain a parent
```

### Lack Of Parent's Argument

```php
class ParentClass
{
    public function __construct(public string $value);
}

class ChildClass extends ParentClass
{
    public function __construct(string $some, public string $any);
}

// Fatal error: Uncaught TypeError: 
// ChildClass::__construct(): Argument #1 ($some) not defined in ParentClass::__construct() or must be promoted
```

## Backwards Incompatible Changes

None

## Proposed Voting Choices

- Simple yes/no vote.
