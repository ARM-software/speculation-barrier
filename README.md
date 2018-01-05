# Speculation Barrier

This header file implements a set of wrapper macros for the
`__builtin_load_no_speculate` builtin function detailed at
[https://www.arm.com/security-update](https://www.arm.com/security-update).
This builtin function defines a speculation barrier, which can be used to
limit the conditions under which a value which has been loaded can be used
under speculative execution.

The header file provided here allows a migration path to using the builtin
function for users who are unable to immediately upgrade to a compiler which
supports the builtin. Arm recommends using an upgraded compiler where possible
to ensure the most comprehensive support for the mitigation provided by the
builtin function.

## Use

This header file can be included in a project as any other C header would be.
For full details on usage of the builtin function please see
[https://www.arm.com/security-update](https://www.arm.com/security-update).

This header provides three wrapper macros, which correspond to using the
`__builtin_load_no_speculate` builtin function with three, four and five
arguments respectively.

```
  load_no_speculate (__ptr, __low, __high)
  load_no_speculate_fail (__ptr, __low, __high, __failval)
  load_no_speculate_cmp (__ptr, __low, __high, __failval, __cmpptr)
```

As an example, consider this function, also given as an example at
[https://www.arm.com/security-update](https://www.arm.com/security-update).

```
int array[N]; 
int foo (unsigned n) 
{ 
  int tmp; 
  if (n < N) 
    tmp = array[n] 
  else 
    tmp = FAIL; 

  return tmp; 
}
```

This can result in a speculative return of the value at `array[n]`, even
if `n >= N`. To mitigate against this, we can use the wrapper macros defined
in this header:

```
#include "speculation_barrier.h"

int foo (unsigned n) 
{ 
  int *lower = array;
  int *ptr = array + n; 
  int *upper = array + N; 
  return load_no_speculate_fail (ptr, lower, upper, FAIL);
}
```

This will ensure that speculative execution can only continue using a value
stored within the array or with `FAIL`.

## Supported Environments

This header provides a migration path to using the builtin function for
the AArch64 execution state of Armv8-A, and for the A32 (Arm) and T32 (Thumb)
execution states of Armv7-A and Armv8-A.

Support for other architectures is currently only provided when using a
compiler which provides the predefine `__HAVE_LOAD_NO_SPECULATE`,
indicating compiler support for the `__builtin_load_no_speculate` builtin
function.

## Compatibility

This header has been tested with Arm Compiler 6 versions 6.5 and above,
GCC versions 4.8 and above, including GCC 7 with prototype support for the
compiler builtin function, and with an LLVM/Clang development toolchain
(version 6.0.0).

## Testing

A set of testcases are provided in `tests.c`. These test the abstract machine
semantics of the provided wrapper macros. The expected behaviour of running the
test program is for no output to be printed. Any output indicates a failure to
implement the required abstract machine semantics of the builtin.

Note that the testcases provided do not check whether speculative execution
has been inhibited.

## License

This project is licensed under the Boost Software License 1.0
(SPDX-License-Identifier: BSL-1.0). See the [LICENSE.md](LICENSE.md) file
for details.

## Contributing

Contributions to this project are welcome under the Boost Software License
1.0 (SPDX-License-Identifier: BSL-1.0).

Arm does not foresee this project requiring a high volume of contributions,
but welcomes contributions adding support for other architectures, further
testcases, and bug-fixes.

## Further details

For further details on use of the `__builtin_load_no_speculate` compiler
builtin, please refer to
[https://www.arm.com/security-update](https://www.arm.com/security-update).

