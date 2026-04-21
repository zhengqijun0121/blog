# CERT C 规范


# CERT C 规范

----

## English Version

### Preprocessor

CERT C: Rule PRE30-C	Do not create a universal character name through concatenation

CERT C: Rule PRE31-C	Avoid side effects in arguments to unsafe macros

CERT C: Rule PRE32-C	Do not use preprocessor directives in invocations of function-like macros

----

### Declarations and initialization

CERT C: Rule DCL30-C	Declare objects with appropriate storage durations

CERT C: Rule DCL31-C	Declare identifiers before using them

CERT C: Rule DCL36-C	Do not declare an identifier with conflicting linkage classifications

CERT C: Rule DCL37-C	Do not declare or define a reserved identifier

CERT C: Rule DCL38-C	Use the correct syntax when declaring a flexible array member

CERT C: Rule DCL39-C	Avoid information leakage in structure padding

CERT C: Rule DCL40-C	Do not create incompatible declarations of the same function or object

CERT C: Rule DCL41-C	Do not declare variables inside a switch statement before the first case label

----

### Expressions

CERT C: Rule EXP30-C	Do not depend on the order of evaluation for side effects

CERT C: Rule EXP32-C	Do not access a volatile object through a nonvolatile reference

CERT C: Rule EXP33-C	Do not read uninitialized memory

CERT C: Rule EXP34-C	Do not dereference null pointers

CERT C: Rule EXP35-C	Do not modify objects with temporary lifetime

CERT C: Rule EXP36-C	Do not cast pointers into more strictly aligned pointer types

CERT C: Rule EXP37-C	Call functions with the correct number and type of arguments

CERT C: Rule EXP39-C	Do not access a variable through a pointer of an incompatible type

CERT C: Rule EXP40-C	Do not modify constant objects

CERT C: Rule EXP42-C	Do not compare padding data

CERT C: Rule EXP43-C	Avoid undefined behavior when using restrict-qualified pointers

CERT C: Rule EXP44-C	Do not rely on side effects in operands to sizeof, _Alignof, or _Generic

CERT C: Rule EXP45-C	Do not perform assignments in selection statements

CERT C: Rule EXP46-C	Do not use a bitwise operator with a Boolean-like operand

CERT C: Rule EXP47-C	Do not call va_arg with an argument of the incorrect type

----

### Integers

CERT C: Rule INT30-C	Ensure that unsigned integer operations do not wrap

CERT C: Rule INT31-C	Ensure that integer conversions do not result in lost or misinterpreted data

CERT C: Rule INT32-C	Ensure that operations on signed integers do not result in overflow

CERT C: Rule INT33-C	Ensure that division and remainder operations do not result in divide-by-zero errors

CERT C: Rule INT34-C	Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand

CERT C: Rule INT35-C	Use correct integer precisions

CERT C: Rule INT36-C	Converting a pointer to integer or integer to pointer

----

### Floating point

CERT C: Rule FLP30-C	Do not use floating-point variables as loop counters

CERT C: Rule FLP32-C	Prevent or detect domain and range errors in math functions

CERT C: Rule FLP34-C	Ensure that floating-point conversions are within range of the new type

CERT C: Rule FLP36-C	Preserve precision when converting integral values to floating-point type

CERT C: Rule FLP37-C	Do not use object representations to compare floating-point values

----

### Arrays

CERT C: Rule ARR30-C	Do not form or use out-of-bounds pointers or array subscripts

CERT C: Rule ARR32-C	Ensure size arguments for variable length arrays are in a valid range

CERT C: Rule ARR36-C	Do not subtract or compare two pointers that do not refer to the same array

CERT C: Rule ARR37-C	Do not add or subtract an integer to a pointer to a non-array object

CERT C: Rule ARR38-C	Guarantee that library functions do not form invalid pointers

CERT C: Rule ARR39-C	Do not add or subtract a scaled integer to a pointer

----

### Characters and strings

CERT C: Rule STR30-C	Do not attempt to modify string literals

CERT C: Rule STR31-C	Guarantee that storage for strings has sufficient space for character data and the null terminator

CERT C: Rule STR32-C	Do not pass a non-null-terminated character sequence to a library function that expects a string

CERT C: Rule STR34-C	Cast characters to unsigned char before converting to larger integer sizes

CERT C: Rule STR37-C	Arguments to character-handling functions must be representable as an unsigned char

CERT C: Rule STR38-C	Do not confuse narrow and wide character strings and functions

----

### Memory Management

CERT C: Rule MEM30-C	Do not access freed memory

CERT C: Rule MEM31-C	Free dynamically allocated memory when no longer needed

CERT C: Rule MEM33-C	Allocate and copy structures containing a flexible array member dynamically

CERT C: Rule MEM34-C	Only free memory allocated dynamically

CERT C: Rule MEM35-C	Allocate sufficient memory for an object

CERT C: Rule MEM36-C	Do not modify the alignment of objects by calling realloc()

----

### Input and output

CERT C: Rule FIO30-C	Exclude user input from format strings

CERT C: Rule FIO32-C	Do not perform operations on devices that are only appropriate for files

CERT C: Rule FIO34-C	Distinguish between characters read from a file and EOF or WEOF

CERT C: Rule FIO37-C	Do not assume that fgets() or fgetws() returns a nonempty string when successful

CERT C: Rule FIO38-C	Do not copy a FILE object

CERT C: Rule FIO39-C	Do not alternately input and output from a stream without an intervening flush or positioning call

CERT C: Rule FIO40-C	Reset strings on fgets() or fgetws() failure

CERT C: Rule FIO41-C	Do not call getc(), putc(), getwc(), or putwc() with a stream argument that has side effects

CERT C: Rule FIO42-C	Close files when they are no longer needed

CERT C: Rule FIO44-C	Only use values for fsetpos() that are returned from fgetpos()

CERT C: Rule FIO45-C	Avoid TOCTOU race conditions while accessing files

CERT C: Rule FIO46-C	Do not access a closed file

CERT C: Rule FIO47-C	Use valid format strings

----

### Environment

CERT C: Rule ENV30-C	Do not modify the object referenced by the return value of certain functions

CERT C: Rule ENV31-C	Do not rely on an environment pointer following an operation that may invalidate it

CERT C: Rule ENV32-C	All exit handlers must return normally

CERT C: Rule ENV33-C	Do not call system()

CERT C: Rule ENV34-C	Do not store pointers returned by certain functions

----

### Signals

CERT C: Rule SIG30-C	Call only asynchronous-safe functions within signal handlers

CERT C: Rule SIG31-C	Do not access shared objects in signal handlers

CERT C: Rule SIG34-C	Do not call signal() from within interruptible signal handlers

CERT C: Rule SIG35-C	Do not return from a computational exception signal handler

----

### Error handling

CERT C: Rule ERR30-C	Set errno to zero before calling a library function known to set errno, and check errno only after the function returns a value indicating failure

CERT C: Rule ERR32-C	Do not rely on indeterminate values of errno

CERT C: Rule ERR33-C	Detect and handle standard library errors

CERT C: Rule ERR34-C	Detect errors when converting a string to a number

----

### Concurrency

CERT C: Rule CON30-C	Clean up thread-specific storage

CERT C: Rule CON31-C	Do not destroy a mutex while it is locked

CERT C: Rule CON32-C	Prevent data races when accessing bit fields from multiple threads

CERT C: Rule CON33-C	Avoid race conditions when using library functions

CERT C: Rule CON34-C	Declare objects shared between threads with appropriate storage durations

CERT C: Rule CON35-C	Avoid deadlock by locking in a predefined order

CERT C: Rule CON36-C	Wrap functions that can spuriously wake up in a loop

CERT C: Rule CON37-C	Do not call signal() in a multithreaded program

CERT C: Rule CON38-C	Preserve thread safety and liveness when using condition variables

CERT C: Rule CON39-C	Do not join or detach a thread that was previously joined or detached

CERT C: Rule CON40-C	Do not refer to an atomic variable twice in an expression

CERT C: Rule CON41-C	Wrap functions that can fail spuriously in a loop

CERT C: Rule CON43-C	Do not allow data races in multithreaded code

----

### Miscellaneous

CERT C: Rule MSC30-C	Do not use the rand() function for generating pseudorandom numbers

CERT C: Rule MSC32-C	Properly seed pseudorandom number generators

CERT C: Rule MSC33-C	Do not pass invalid data to the asctime() function

CERT C: Rule MSC37-C	Ensure that control never reaches the end of a non-void function

CERT C: Rule MSC38-C	Do not treat a predefined identifier as an object if it might only be implemented as a macro

CERT C: Rule MSC39-C	Do not call va_arg() on a va_list that has an indeterminate value

CERT C: Rule MSC40-C	Do not violate constraints

CERT C: Rule MSC41-C	Never hard code sensitive information

----

### POSIX

CERT C: Rule POS30-C	Use the readlink() function properly

CERT C: Rule POS34-C	Do not call putenv() with a pointer to an automatic variable as the argument

CERT C: Rule POS35-C	Avoid race conditions while checking for the existence of a symbolic link

CERT C: Rule POS36-C	Observe correct revocation order while relinquishing privileges

CERT C: Rule POS37-C	Ensure that privilege relinquishment is successful

CERT C: Rule POS38-C	Beware of race conditions when using fork and file descriptors

CERT C: Rule POS39-C	Use the correct byte ordering when transferring data between systems

CERT C: Rule POS44-C	Do not use signals to terminate threads

CERT C: Rule POS47-C	Do not use threads that can be canceled asynchronously

CERT C: Rule POS48-C	Do not unlock or destroy another POSIX thread's mutex

CERT C: Rule POS49-C	When data must be accessed by multiple threads, provide a mutex and guarantee no adjacent data is also accessed

CERT C: Rule POS50-C	Declare objects shared between POSIX threads with appropriate storage durations

CERT C: Rule POS51-C	Avoid deadlock with POSIX threads by locking in predefined order

CERT C: Rule POS52-C	Do not perform operations that can block while holding a POSIX lock

CERT C: Rule POS53-C	Do not use more than one mutex for concurrent waiting operations on a condition variable

CERT C: Rule POS54-C	Detect and handle POSIX library errors

----

### Microsoft windows

CERT C: Rule WIN30-C	Properly pair allocation and deallocation functions

----

### Recommendations

#### Preprocessor

CERT C: Rec. PRE00-C	Prefer inline or static functions to function-like macros

CERT C: Rec. PRE01-C	Use parentheses within macros around parameter names

CERT C: Rec. PRE02-C	Macro replacement lists should be parenthesized (Since R2026a)

CERT C: Rec. PRE03-C	Prefer typedefs to defines for encoding non-pointer type (Since R2024a)

CERT C: Rec. PRE04-C	Do not reuse a standard header file name (Since R2025a)

CERT C: Rec. PRE05-C	Understand macro replacement when concatenating tokens or performing stringification (Since R2024b)

CERT C: Rec. PRE06-C	Enclose header files in an inclusion guard

CERT C: Rec. PRE07-C	Avoid using repeated question marks

CERT C: Rec. PRE08-C	Guarantee that header file names are unique (Since R2024a)

CERT C: Rec. PRE09-C	Do not replace secure functions with deprecated or obsolescent functions

CERT C: Rec. PRE10-C	Wrap multistatement macros in a do-while loop

CERT C: Rec. PRE11-C	Do not conclude macro definitions with a semicolon

CERT C: Rec. PRE12-C	Do not define unsafe macros (Since R2024a)

----


