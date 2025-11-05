
<img src="https://raw.githubusercontent.com/libinjection/libinjection/main/misc/libinjection.svg" width="70%">

![CI](https://github.com/libinjection/libinjection/workflows/CI/badge.svg)
[![license](https://img.shields.io/badge/license-BSD_3--Clause-blue.svg?style=flat)](https://raw.githubusercontent.com/client9/libinjection/master/COPYING)


SQL / SQLI tokenizer parser analyzer. For

* C and C++
* [PHP](https://libinjection.client9.com/doc-sqli-php)
* [Python](https://libinjection.client9.com/doc-sqli-python)
* [Lua](/lua)
* [Java](https://github.com/jeonglee/Libinjection) (external port)
* [LuaJIT/FFI](https://github.com/p0pr0ck5/lua-ffi-libinjection) (external port)

See [https://www.client9.com/](https://www.client9.com/)
for details and presentations.

Simple example:

```c
#include <stdio.h>
#include <strings.h>
#include <errno.h>
#include "libinjection.h"
#include "libinjection_sqli.h"

int main(int argc, const char* argv[])
{
    struct libinjection_sqli_state state;
    injection_result_t result;

    const char* input = argv[1];
    size_t slen = strlen(input);

    /* in real-world, you would url-decode the input, etc */

    libinjection_sqli_init(&state, input, slen, FLAG_NONE);
    result = libinjection_is_sqli(&state);
    
    if (result == LIBINJECTION_RESULT_ERROR) {
        fprintf(stderr, "error: parser encountered an error\n");
        return 2;
    } else if (result == LIBINJECTION_RESULT_TRUE) {
        fprintf(stderr, "sqli detected with fingerprint of '%s'\n", state.fingerprint);
        return 1;
    }
    
    /* LIBINJECTION_RESULT_FALSE - no SQLi detected */
    return 0;
}
```

```
$ gcc -Wall -Wextra examples.c libinjection_sqli.c
$ ./a.out "-1' and 1=1 union/* foo */select load_file('/etc/passwd')--"
sqli detected with fingerprint of 's&1UE'
```

More advanced samples:

* [sqli_cli.c](/src/sqli_cli.c)
* [reader.c](/src/reader.c)
* [fptool](/src/fptool.c)

VERSION INFORMATION
===================

See [CHANGELOG](CHANGELOG.md) for details.

Versions are listed as "major.minor.point"

Major are significant changes to the API and/or fingerprint format.
Applications will need recompiling and/or refactoring.

Minor are C code changes.  These may include
 * logical change to detect or suppress
 * optimization changes
 * code refactoring

Point releases are purely data changes.  These may be safely applied.

ERROR HANDLING
==============

As of version 4.0.0, libinjection uses an `injection_result_t` enum for return values instead of `int`:

```c
typedef enum injection_result_t {
    LIBINJECTION_RESULT_FALSE = 0,   // No injection detected (benign input)
    LIBINJECTION_RESULT_TRUE = 1,    // Injection detected
    LIBINJECTION_RESULT_ERROR = -1   // Parser error (invalid state)
} injection_result_t;
```

**Important:** Prior to v4.0.0, libinjection would call `abort()` and terminate the process when encountering parser errors. Now it returns `LIBINJECTION_RESULT_ERROR` instead, allowing your application to handle errors gracefully.

**Backward Compatibility:** The enum values `LIBINJECTION_RESULT_FALSE` (0) and `LIBINJECTION_RESULT_TRUE` (1) maintain backward compatibility with code that checks for true/false values. However, applications should be updated to handle `LIBINJECTION_RESULT_ERROR` (-1) to prevent treating parser errors as benign input.

**Migration:** See [MIGRATION.md](MIGRATION.md) for guidance on updating existing code.

QUALITY AND DIAGNOSITICS
========================

The continuous integration results at GitHub tests the following:

- [x] build and unit-tests under GCC
- [x] build and unit-tests under Clang
- [x] static analysis using [clang static analyzer](http://clang-analyzer.llvm.org)
- [x] static analysis using [cppcheck](https://github.com/danmar/cppcheck)
- [x] checks for memory errors using [valgrind](http://valgrind.org/)

LICENSE
=============

Copyright (c) 2012-2016 Nick Galbreath

Licensed under the standard [BSD 3-Clause](http://opensource.org/licenses/BSD-3-Clause) open source
license.  See [COPYING](/COPYING) for details.

## BUILD TARGETS

Some of the previous help runners have been merged into the Makefile. E.g.:

* run-clang-asan.sh -> `make clan-asan`
* make-ci.sh -> `make ci`

If you run `make cppcheck` you will see this warning printed:
```
nofile:0 information missingIncludeSystem Cppcheck cannot find all the include files (use --check-config for details)
```
You can safely ignore it as it is just saying that standard include files are being ignored (which is the recommended option):
```
example1.c:1:0: information: Include file: <stdio.h> not found. Please note: Cppcheck does not need standard library headers to get proper results. [missingIncludeSystem]
```

EMBEDDING
=============

The [src](/src)
directory contains everything, but you only need to copy the following
into your source tree:

* [src/libinjection.h](/src/libinjection.h)
* [src/libinjection_error.h](/src/libinjection_error.h)
* [src/libinjection_sqli.c](/src/libinjection_sqli.c)
* [src/libinjection_sqli_data.h](/src/libinjection_sqli_data.h)
* [COPYING](/COPYING)

Usually the new autoconf build system takes care of the `LIBINJECTION_VERSION` definition.
But that might now be available when you are embedding the above files.

This is solved by manually defining the version you are embedding to your `CFLAGS`.

E.g.: `CFLAGS="-DLIBINJECTION_VERSION=\"3.9.2.65-dfe6-dirty\""`

An easy way to get the version tag is to execute `git describe` in this directory.
