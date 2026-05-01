# CERT C 规范英文版


# CERT C 规范英文版

> [https://ww2.mathworks.cn/help/bugfinder/cert-c-rules-and-recommendations.html?lang=en](https://ww2.mathworks.cn/help/bugfinder/cert-c-rules-and-recommendations.html?lang=en)

----

## Rules

### Preprocessor

[Rule PRE30-C] Do not create a universal character name through concatenation.

[Rule PRE31-C] Avoid side effects in arguments to unsafe macros.

[Rule PRE32-C] Do not use preprocessor directives in invocations of function-like macros.

----

### Declarations and initialization

[Rule DCL30-C] Declare objects with appropriate storage durations.

[Rule DCL31-C] Declare identifiers before using them.

[Rule DCL36-C] Do not declare an identifier with conflicting linkage classifications.

[Rule DCL37-C] Do not declare or define a reserved identifier.

[Rule DCL38-C] Use the correct syntax when declaring a flexible array member.

[Rule DCL39-C] Avoid information leakage in structure padding.

[Rule DCL40-C] Do not create incompatible declarations of the same function or object.

[Rule DCL41-C] Do not declare variables inside a switch statement before the first case label.

----

### Expressions

[Rule EXP30-C] Do not depend on the order of evaluation for side effects.

[Rule EXP32-C] Do not access a volatile object through a nonvolatile reference.

[Rule EXP33-C] Do not read uninitialized memory.

[Rule EXP34-C] Do not dereference null pointers.

[Rule EXP35-C] Do not modify objects with temporary lifetime.

[Rule EXP36-C] Do not cast pointers into more strictly aligned pointer types.

[Rule EXP37-C] Call functions with the correct number and type of arguments.

[Rule EXP39-C] Do not access a variable through a pointer of an incompatible type.

[Rule EXP40-C] Do not modify constant objects.

[Rule EXP42-C] Do not compare padding data.

[Rule EXP43-C] Avoid undefined behavior when using restrict-qualified pointers.

[Rule EXP44-C] Do not rely on side effects in operands to sizeof, _Alignof, or _Generic.

[Rule EXP45-C] Do not perform assignments in selection statements.

[Rule EXP46-C] Do not use a bitwise operator with a Boolean-like operand.

[Rule EXP47-C] Do not call va_arg with an argument of the incorrect type.

----

### Integers

[Rule INT30-C] Ensure that unsigned integer operations do not wrap.

[Rule INT31-C] Ensure that integer conversions do not result in lost or misinterpreted data.

[Rule INT32-C] Ensure that operations on signed integers do not result in overflow.

[Rule INT33-C] Ensure that division and remainder operations do not result in divide-by-zero errors.

[Rule INT34-C] Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand.

[Rule INT35-C] Use correct integer precisions.

[Rule INT36-C] Converting a pointer to integer or integer to pointer.

----

### Floating point

[Rule FLP30-C] Do not use floating-point variables as loop counters.

[Rule FLP32-C] Prevent or detect domain and range errors in math functions.

[Rule FLP34-C] Ensure that floating-point conversions are within range of the new type.

[Rule FLP36-C] Preserve precision when converting integral values to floating-point type.

[Rule FLP37-C] Do not use object representations to compare floating-point values.

----

### Arrays

[Rule ARR30-C] Do not form or use out-of-bounds pointers or array subscripts.

[Rule ARR32-C] Ensure size arguments for variable length arrays are in a valid range.

[Rule ARR36-C] Do not subtract or compare two pointers that do not refer to the same array.

[Rule ARR37-C] Do not add or subtract an integer to a pointer to a non-array object.

[Rule ARR38-C] Guarantee that library functions do not form invalid pointers.

[Rule ARR39-C] Do not add or subtract a scaled integer to a pointer.

----

### Characters and strings

[Rule STR30-C] Do not attempt to modify string literals.

[Rule STR31-C] Guarantee that storage for strings has sufficient space for character data and the null terminator.

[Rule STR32-C] Do not pass a non-null-terminated character sequence to a library function that expects a string.

[Rule STR34-C] Cast characters to unsigned char before converting to larger integer sizes.

[Rule STR37-C] Arguments to character-handling functions must be representable as an unsigned char.

[Rule STR38-C] Do not confuse narrow and wide character strings and functions.

----

### Memory Management

[Rule MEM30-C] Do not access freed memory.

[Rule MEM31-C] Free dynamically allocated memory when no longer needed.

[Rule MEM33-C] Allocate and copy structures containing a flexible array member dynamically.

[Rule MEM34-C] Only free memory allocated dynamically.

[Rule MEM35-C] Allocate sufficient memory for an object.

[Rule MEM36-C] Do not modify the alignment of objects by calling realloc().

----

### Input and output

[Rule FIO30-C] Exclude user input from format strings.

[Rule FIO32-C] Do not perform operations on devices that are only appropriate for files.

[Rule FIO34-C] Distinguish between characters read from a file and EOF or WEOF.

[Rule FIO37-C] Do not assume that fgets() or fgetws() returns a nonempty string when successful.

[Rule FIO38-C] Do not copy a FILE object.

[Rule FIO39-C] Do not alternately input and output from a stream without an intervening flush or positioning call.

[Rule FIO40-C] Reset strings on fgets() or fgetws() failure.

[Rule FIO41-C] Do not call getc(), putc(), getwc(), or putwc() with a stream argument that has side effects.

[Rule FIO42-C] Close files when they are no longer needed.

[Rule FIO44-C] Only use values for fsetpos() that are returned from fgetpos().

[Rule FIO45-C] Avoid TOCTOU race conditions while accessing files.

[Rule FIO46-C] Do not access a closed file.

[Rule FIO47-C] Use valid format strings.

----

### Environment

[Rule ENV30-C] Do not modify the object referenced by the return value of certain functions.

[Rule ENV31-C] Do not rely on an environment pointer following an operation that may invalidate it.

[Rule ENV32-C] All exit handlers must return normally.

[Rule ENV33-C] Do not call system().

[Rule ENV34-C] Do not store pointers returned by certain functions.

----

### Signals

[Rule SIG30-C] Call only asynchronous-safe functions within signal handlers.

[Rule SIG31-C] Do not access shared objects in signal handlers.

[Rule SIG34-C] Do not call signal() from within interruptible signal handlers.

[Rule SIG35-C] Do not return from a computational exception signal handler.

----

### Error handling

[Rule ERR30-C] Set errno to zero before calling a library function known to set errno, and check errno only after the function returns a value indicating failure.

[Rule ERR32-C] Do not rely on indeterminate values of errno.

[Rule ERR33-C] Detect and handle standard library errors.

[Rule ERR34-C] Detect errors when converting a string to a number.

----

### Concurrency

[Rule CON30-C] Clean up thread-specific storage.

[Rule CON31-C] Do not destroy a mutex while it is locked.

[Rule CON32-C] Prevent data races when accessing bit fields from multiple threads.

[Rule CON33-C] Avoid race conditions when using library functions.

[Rule CON34-C] Declare objects shared between threads with appropriate storage durations.

[Rule CON35-C] Avoid deadlock by locking in a predefined order.

[Rule CON36-C] Wrap functions that can spuriously wake up in a loop.

[Rule CON37-C] Do not call signal() in a multithreaded program.

[Rule CON38-C] Preserve thread safety and liveness when using condition variables.

[Rule CON39-C] Do not join or detach a thread that was previously joined or detached.

[Rule CON40-C] Do not refer to an atomic variable twice in an expression.

[Rule CON41-C] Wrap functions that can fail spuriously in a loop.

[Rule CON43-C] Do not allow data races in multithreaded code.

----

### Miscellaneous

[Rule MSC30-C] Do not use the rand() function for generating pseudorandom numbers.

[Rule MSC32-C] Properly seed pseudorandom number generators.

[Rule MSC33-C] Do not pass invalid data to the asctime() function.

[Rule MSC37-C] Ensure that control never reaches the end of a non-void function.

[Rule MSC38-C] Do not treat a predefined identifier as an object if it might only be implemented as a macro.

[Rule MSC39-C] Do not call va_arg() on a va_list that has an indeterminate value.

[Rule MSC40-C] Do not violate constraints.

[Rule MSC41-C] Never hard code sensitive information.

----

### POSIX

[Rule POS30-C] Use the readlink() function properly.

[Rule POS34-C] Do not call putenv() with a pointer to an automatic variable as the argument.

[Rule POS35-C] Avoid race conditions while checking for the existence of a symbolic link.

[Rule POS36-C] Observe correct revocation order while relinquishing privileges.

[Rule POS37-C] Ensure that privilege relinquishment is successful.

[Rule POS38-C] Beware of race conditions when using fork and file descriptors.

[Rule POS39-C] Use the correct byte ordering when transferring data between systems.

[Rule POS44-C] Do not use signals to terminate threads.

[Rule POS47-C] Do not use threads that can be canceled asynchronously.

[Rule POS48-C] Do not unlock or destroy another POSIX thread's mutex.

[Rule POS49-C] When data must be accessed by multiple threads, provide a mutex and guarantee no adjacent data is also accessed.

[Rule POS50-C] Declare objects shared between POSIX threads with appropriate storage durations.

[Rule POS51-C] Avoid deadlock with POSIX threads by locking in predefined order.

[Rule POS52-C] Do not perform operations that can block while holding a POSIX lock.

[Rule POS53-C] Do not use more than one mutex for concurrent waiting operations on a condition variable.

[Rule POS54-C] Detect and handle POSIX library errors.

----

### Microsoft windows

[Rule WIN30-C] Properly pair allocation and deallocation functions.

----

## Recommendations

### Preprocessor

[Rec. PRE00-C] Prefer inline or static functions to function-like macros.

[Rec. PRE01-C] Use parentheses within macros around parameter names.

[Rec. PRE02-C] Macro replacement lists should be parenthesized.

[Rec. PRE03-C] Prefer typedefs to defines for encoding non-pointer type.

[Rec. PRE04-C] Do not reuse a standard header file name.

[Rec. PRE05-C] Understand macro replacement when concatenating tokens or performing stringification.

[Rec. PRE06-C] Enclose header files in an inclusion guard.

[Rec. PRE07-C] Avoid using repeated question marks.

[Rec. PRE08-C] Guarantee that header file names are unique.

[Rec. PRE09-C] Do not replace secure functions with deprecated or obsolescent functions.

[Rec. PRE10-C] Wrap multistatement macros in a do-while loop.

[Rec. PRE11-C] Do not conclude macro definitions with a semicolon.

[Rec. PRE12-C] Do not define unsafe macros.

----

### Declarations and initialization

[Rec. DCL00-C] Const-qualify immutable objects.

[Rec. DCL01-C] Do not reuse variable names in subscopes.

[Rec. DCL02-C] Use visually distinct identifiers.

[Rec. DCL03-C] Use a static assertion to test the value of a constant expression.

[Rec. DCL04-C] Do not declare more than one variable per declaration.

[Rec. DCL05-C] Use typedefs of non-pointer types only.

[Rec. DCL06-C] Use meaningful symbolic constants to represent literal values.

[Rec. DCL07-C] Include the appropriate type information in function declarators.

[Rec. DCL09-C] Declare functions that return errno with a return type of errno_t.

[Rec. DCL20-C] Explicitly specify void when a function accepts no arguments.

[Rec. DCL10-C] Maintain the contract between the writer and caller of variadic functions.

[Rec. DCL11-C] Understand the type issues associated with variadic functions.

[Rec. DCL12-C] Implement abstract data types using opaque types.

[Rec. DCL13-C] Declare function parameters that are pointers to values not changed by the function as const.

[Rec. DCL15-C] Declare file-scope objects or functions that do not need external linkage as static.

[Rec. DCL16-C] Use 'L,' not 'l,' to indicate a long value.

[Rec. DCL18-C] Do not begin integer constants with 0 when specifying a decimal value.

[Rec. DCL19-C] Minimize the scope of variables and functions.

[Rec. DCL21-C] Understand the storage of compound literals.

[Rec. DCL22-C] Use volatile for data that cannot be cached.

[Rec. DCL23-C] Guarantee that mutually visible identifiers are unique.

----

### Expressions

[Rec. EXP00-C] Use parentheses for precedence of operation.

[Rec. EXP02-C] Be aware of the short-circuit behavior of the logical AND and OR operators.

[Rec. EXP03-C] Do not assume the size of a structure is the sum of the sizes of its members.

[Rec. EXP05-C] Do not cast away a const qualification.

[Rec. EXP07-C] Do not diminish the benefits of constants by assuming their values in expressions.

[Rec. EXP08-C] Ensure pointer arithmetic is used correctly.

[Rec. EXP09-C] Use sizeof to determine the size of a type or variable.

[Rec. EXP10-C] Do not depend on the order of evaluation of subexpressions or the order in which side effects take place.

[Rec. EXP11-C] Do not make assumptions regarding the layout of structures with bit-fields.

[Rec. EXP12-C] Do not ignore values returned by functions.

[Rec. EXP13-C] Treat relational and equality operators as if they were nonassociative.

[Rec. EXP15-C] Do not place a semicolon on the same line as an if, for, or while statement.

[Rec. EXP16-C] Do not compare function pointers to constant values.

[Rec. EXP19-C] Use braces for the body of an if, for, or while statement.

[Rec. EXP20-C] Perform explicit tests to determine success, true and false, and equality.

----

### Integers

[Rec. INT00-C] Understand the data model used by your implementation(s).

[Rec. INT02-C] Understand integer conversion rules.

[Rec. INT04-C] Enforce limits on integer values originating from tainted sources.

[Rec. INT07-C] Use only explicitly signed or unsigned char type for numeric values.

[Rec. INT08-C] Verify that all integer values are in range.

[Rec. INT09-C] Ensure enumeration constants map to unique values.

[Rec. INT10-C] Do not assume a positive remainder when using the % operator.

[Rec. INT12-C] Do not make assumptions about the type of a plain int bit-field when used in an expression.

[Rec. INT13-C] Use bitwise operators only on unsigned operands.

[Rec. INT14-C] Avoid performing bitwise and arithmetic operations on the same data.

[Rec. INT16-C] Do not make assumptions about representation of signed integers.

[Rec. INT17-C] Define integer constants in an implementation-independent manner.

[Rec. INT18-C] Evaluate integer expressions in a larger size before comparing or assigning to that size.

----

### Floating point

[Rec. FLP00-C] Understand the limitations of floating-point numbers.

[Rec. FLP02-C] Avoid using floating-point numbers when precise computation is needed.

[Rec. FLP03-C] Detect and handle floating-point errors.

[Rec. FLP06-C] Convert integers to floating point for floating-point operations.

----

### Arrays

[Rec. ARR01-C] Do not apply the sizeof operator to a pointer when taking the size of an array.

[Rec. ARR02-C] Explicitly specify array bounds, even if implicitly defined by an initializer.

----

### Characters and strings

[Rec. STR00-C] Represent characters using an appropriate type.

[Rec. STR02-C] Sanitize data passed to complex subsystems.

[Rec. STR03-C] Do not inadvertently truncate a string.

[Rec. STR06-C] Do not assume that strtok() leaves the parse string unchanged.

[Rec. STR07-C] Use the bounds-checking interfaces for string manipulation.

[Rec. STR10-C] Do not concatenate different type of string literals.

[Rec. STR11-C] Do not specify the bound of a character array initialized with a string literal.

----

### Memory management

[Rec. MEM00-C] Allocate and free memory in the same module, at the same level of abstraction.

[Rec. MEM01-C] Store a new value in pointers immediately after free().

[Rec. MEM02-C] Immediately cast the result of a memory allocation function call into a pointer to the allocated type.

[Rec. MEM03-C] Clear sensitive information stored in reusable resources.

[Rec. MEM04-C] Beware of zero-length allocations.

[Rec. MEM05-C] Avoid large stack allocations.

[Rec. MEM06-C] Ensure that sensitive data is not written out to disk.

[Rec. MEM11-C] Do not assume infinite heap space.

[Rec. MEM12-C] Consider using a goto chain when leaving a function on error when using and releasing resources.

----

### Input and output

[Rec. FIO02-C] Canonicalize path names originating from tainted sources.

[Rec. FIO03-C] Do not make assumptions about fopen() and file creation.

[Rec. FIO06-C] Create files with appropriate access permissions.

[Rec. FIO08-C] Take care when calling remove() on an open file.

[Rec. FIO10-C] Take care when using the rename() function.

[Rec. FIO11-C] Take care when specifying the mode parameter of fopen().

[Rec. FIO21-C] Do not create temporary files in shared directories.

[Rec. FIO24-C] Do not open a file that is already open.

----

### Environment

[Rec. ENV01-C] Do not make assumptions about the size of an environment variable.

----

### Signals

[Rec. SIG02-C] Avoid using signals to implement normal functionality.

----

### Error handling

[Rec. ERR00-C] Adopt and implement a consistent and comprehensive error-handling policy.

[Rec. ERR01-C] Use ferror() rather than errno to check for FILE stream errors.

[Rec. ERR06-C] Understand the termination behavior of assert() and abort().

[Rec. ERR07-C] Prefer functions that support error checking over equivalent functions that don't.

----

### Application programming interfaces

[Rec. API01-C] Avoid laying out strings in memory directly before sensitive data.

[Rec. API02-C] Functions that read or write to or from an array should take an argument to specify the source or target size.

[Rec. API04-C] Provide a consistent and usable error-checking mechanism.

[Rec. API05-C] Use conformant array parameters.

----

### Concurrency

[Rec. CON01-C] Acquire and release synchronization primitives in the same module, at the same level of abstraction.

[Rec. CON05-C] Do not perform operations that can block while holding a lock.

----

### Miscellaneous

[Rec. MSC00-C] Compile cleanly at high warning levels.

[Rec. MSC01-C] Strive for logical completeness.

[Rec. MSC04-C] Use comments consistently and in a readable fashion.

[Rec. MSC05-C] Do not manipulate time_t typed values directly.

[Rec. MSC06-C] Beware of compiler optimizations.

[Rec. MSC12-C] Detect and remove code that has no effect or is never executed.

[Rec. MSC13-C] Detect and remove unused values.

[Rec. MSC15-C] Do not depend on undefined behavior.

[Rec. MSC17-C] Finish every set of statements associated with a case label with a break statement.

[Rec. MSC18-C] Be careful while handling sensitive data, such as passwords, in program code.

[Rec. MSC20-C] Do not use a switch statement to transfer control into a complex block.

[Rec. MSC21-C] Use robust loop termination conditions.

[Rec. MSC22-C] Use the setjmp(), longjmp() facility securely.

[Rec. MSC24-C] Do not use deprecated or obsolescent functions.

----

### POSIX

[Rec. POS04-C] Avoid using PTHREAD_MUTEX_NORMAL type mutex locks.

[Rec. POS05-C] Limit access to files by creating a jail.

----

### Microsoft windows

[Rec. WIN00-C] Be specific when dynamically loading libraries.

[Rec. WIN01-C] Do not forcibly terminate execution.

[Rec. WIN02-C] Restrict privileges when spawning child processes.

[Rec. WIN03-C] Understand HANDLE inheritance.

[Rec. WIN04-C] Consider encrypting function pointers.

----

