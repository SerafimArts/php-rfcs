 * Name: `ffi-improvements`
 * Date: 2020-07-01
 * Author: Kirill Nesmeyanov <nesk@xakep.ru>
 * Proposed Version: PHP 8.0
 * RFC PR: [php/php-rfcs#0003](https://github.com/php/php-rfcs/pull/3)

## Introduction

Currently, the FFI API contains a number of problems. This RFC offers their 
solution.

### Working Directory

The first problem is the inability to specify the working directory from which 
libraries are loaded via FFI in the PHP ZTS working environment.

- https://bugs.php.net/bug.php?id=79439
- https://github.com/php/doc-en/pull/121

### API Inconsistency

Two static methods are used to load libraries:
- `FFI::cdef(string $code = '', string $lib = null);`
- `FFI::load(string $filename);`

Technically they are similar: The first loads the C header files from a PHP 
string, and second of the physical file. But both approaches impose a number 
of limitations.

In the case of `FFI::cdef()`:
- Can specify the library file (`*.so`, `*.dll`, etc) from the PHP code.
- Cannot specify "scope" from PHP code.
- Cannot declare `FFI_LIB` define to specify the library file from the header C code.
- Cannot declare `FFI_SCOPE` define from C header code.

In the case of `FFI::load()`:
- Cannot specify library file from PHP code.
- Cannot specify "scope" from PHP code.
- Can declare `FFI_LIB` define to specify the library file from the C header code.
- Can declare `FFI_SCOPE` define to specify the library file from the C header code.

## Proposal

It is proposed to solve these problems.

### Allow C-Defines In Any Cases

- Add support for `FFI_LIB` and` FFI_SCOPE` defines in each of the methods 
(i.e. `FFI::load()` and `FFI::cdef ()`).

- Add support for `FFI_LD` define to specify the libraries working directory.

The follow header code should work, as in the case of loading from using 
`FFI::load()`, and using `FFI::cdef()` methods:

```c
#define FFI_SCOPE "example"
#define FFI_LIB   "../bin/example.so"
#define FFI_LD    "../bin"

// c-headers code
// ...
```

### Loading Options

Allow passing an array as the second argument of each of the methods
options to control library loading. These options passed from PHP code must 
take precedence over those defined as constants inside header files.

After accepting these changes, method signatures should look as follows
way:

```php
FFI::load(string $filename, array $options = []);

FFI::cdef(string $code = '', array $options = []);
FFI::cdef(string $code = '', string $lib = null);
```

> Please note that for backward compatibility, allow option with passing a 
> string as the second argument to the `FFI::cdef()` method.

Options may contain the following keys to specify libraries loading parameters:

```php
FFI::cdef('<...code...>', [
    'library'   => 'path/to/library.so',
    'scope'     => 'example',
    'libdir'    => '/path/to/working_directory'
]);
```

Where
- `library` - Library file name
- `scope` - Scope (identifier) of the library to load using `FFI::scope()`
- `libdir` - Library working directory

If these options are specified, the corresponding definition in the header file 
must to be ignored.

## Miscellaneous

### Library Directory And Multithreading Environment (ZTS)

Specifying the working directory of libraries, is a global operation 
that affects all threads. 

For example:

```c
// Windows OS (from kernel32.dll)
BOOL SetDllDirectoryA(LPCSTR lpPathName);
// SetDllDirectoryA('path/to/libdir');

// Linux OS
int setenv(const char *name, const char *value, int overwrite);
// setenv('LD_LIBRARY_PATH', 'path/to/libdir', 1);
```

However, using mutexes for downloads in a ZTS 
environment should solve this problem.

Since libraries in the working environment are mostly loaded through 
preloading, after loading PHP extensions, waiting for loading in another thread 
should not affect performance in real projects. Since in runtime everything 
will be loaded at the preloading stage.

In other cases, well, delays in waiting for downloads cannot be avoided.

## Backwards Incompatible Changes

None

## Future Scope

The features discussed in the following are not part of this proposal.

## Proposed Voting Choices

Three votes expected:

- Add support for `FFI_LD` define: Simple yes/no vote.
- Add support for defines in all methods of loading defines: Simple yes/no vote.
- Add options support: Simple yes/no vote.
