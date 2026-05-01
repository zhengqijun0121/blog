# CERT C++ 规范英文版


# CERT C++ 规范英文版

> [https://ww2.mathworks.cn/help/bugfinder/cert-c-rules.html?lang=en](https://ww2.mathworks.cn/help/bugfinder/cert-c-rules.html?lang=en)

**[中文版本](./015_CertCpp规范.md)**

----

## Declare and Initialization

[DCL30-C] Declare objects with appropriate storage durations.

[DCL39-C] Avoid information leakage in structure padding.

[DCL40-C] Do not create incompatible declarations of the same function or object.

[DCL50-CPP] Do not define a C-style variadic function.

[DCL51-CPP] Do not declare or define a reserved identifier.

[DCL52-CPP] Never qualify a reference type with const or volatile.

[DCL53-CPP] Do not write syntactically ambiguous declarations.

[DCL54-CPP] Overload allocation and deallocation functions as a pair in the same scope.

[DCL55-CPP] Avoid information leakage when passing a class object across a trust boundary.

[DCL56-CPP] Avoid cycles during initialization of static objects.

[DCL57-CPP] Do not let exceptions escape from destructors or deallocation functions.

[DCL58-CPP] Do not modify the standard namespaces.

[DCL59-CPP] Do not define an unnamed namespace in a header file.

[DCL60-CPP] Obey the one-definition rule.

----

## Expressions

[EXP34-C] Do not dereference null pointers.

[EXP35-C] Do not modify objects with temporary lifetime.

[EXP36-C] Do not cast pointers into more strictly aligned pointer types.

[EXP37-C] Call functions with the correct number and type of arguments.

[EXP39-C] Do not access a variable through a pointer of an incompatible type.

[EXP42-C] Do not compare padding data.

[EXP45-C] Do not perform assignments in selection statements.

[EXP46-C] Do not use a bitwise operator with a Boolean-like operand.

[EXP47-C] Do not call va_arg with an argument of the incorrect type.

[EXP50-CPP] Do not depend on the order of evaluation for side effects.

[EXP51-CPP] Do not delete an array through a pointer of the incorrect type.

[EXP52-CPP] Do not rely on side effects in unevaluated operands.

[EXP53-CPP] Do not read uninitialized memory.

[EXP54-CPP] Do not access an object outside of its lifetime.

[EXP55-CPP] Do not access a cv-qualified object through a cv-unqualified type.

[EXP56-CPP] Do not call a function with a mismatched language linkage.

[EXP57-CPP] Do not cast or delete pointers to incomplete classes.

[EXP58-CPP] Pass an object of the correct type to va_start.

[EXP59-CPP] Use offsetof() on valid types and members.

[EXP60-CPP] Do not pass a nonstandard-layout type object across execution boundaries.

[EXP61-CPP] A lambda object must not outlive any of its reference captured objects.

[EXP62-CPP] Do not access the bits of an object representation that are not part of the object's value representation.

[EXP63-CPP] Do not rely on the value of a moved-from object.

----

## Integers

[INT30-C] Ensure that unsigned integer operations do not wrap.

[INT31-C] Ensure that integer conversions do not result in lost or misinterpreted data.

[INT32-C] Ensure that operations on signed integers do not result in overflow.

[INT33-C] Ensure that division and remainder operations do not result in divide-by-zero errors.

[INT34-C] Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand.

[INT35-C] Use correct integer precisions.

[INT36-C] Converting a pointer to integer or integer to pointer.

[INT50-CPP] Do not cast to an out-of-range enumeration value.

----

## Containers

[ARR30-C] Do not form or use out-of-bounds pointers or array subscripts.

[ARR37-C] Do not add or subtract an integer to a pointer to a non-array object.

[ARR38-C] Guarantee that library functions do not form invalid pointers.

[ARR39-C] Do not add or subtract a scaled integer to a pointer.

[CTR50-CPP] Guarantee that container indices and iterators are within the valid range.

[CTR51-CPP] Use valid references, pointers, and iterators to reference elements of a container.

[CTR52-CPP] Guarantee that library functions do not overflow.

[CTR53-CPP] Use valid iterator ranges.

[CTR54-CPP] Do not subtract iterators that do not refer to the same container.

[CTR55-CPP] Do not use an additive operator on an iterator if the result would overflow.

[CTR56-CPP] Do not use pointer arithmetic on polymorphic objects.

[CTR57-CPP] Provide a valid ordering predicate.

[CTR58-CPP] Predicate function objects should not be mutable.

----

## Characters and strings

[STR30-C] Do not attempt to modify string literals.

[STR31-C] Guarantee that storage for strings has sufficient space for character data and the null terminator.

[STR32-C] Do not pass a non-null-terminated character sequence to a library function that expects a string.

[STR34-C] Cast characters to unsigned char before converting to larger integer sizes.

[STR37-C] Arguments to character-handling functions must be representable as an unsigned char.

[STR38-C] Do not confuse narrow and wide character strings and functions.

[STR50-CPP] Guarantee that storage for strings has sufficient space for character data and the null terminator.

[STR51-CPP] Do not attempt to create a std::string from a null pointer.

[STR52-CPP] Use valid references, pointers, and iterators to reference elements of a basic_string.

[STR53-CPP] Range check element access.

----

## Memory Management

[MEM30-C] Do not access freed memory.

[MEM31-C] Free dynamically allocated memory when no longer needed.

[MEM34-C] Only free memory allocated dynamically.

[MEM35-C] Allocate sufficient memory for an object.

[MEM36-C] Do not modify the alignment of objects by calling realloc().

[MEM50-CPP] Do not access freed memory.

[MEM51-CPP] Properly deallocate dynamically allocated resources.

[MEM52-CPP] Detect and handle memory allocation errors.

[MEM53-CPP] Explicitly construct and destruct objects when manually managing object lifetime.

[MEM54-CPP] Provide placement new with properly aligned pointers to sufficient storage capacity.

[MEM55-CPP] Honor replacement dynamic storage management requirements.

[MEM56-CPP] Do not store an already-owned pointer value in an unrelated smart pointer.

[MEM57-CPP] Avoid using default operator new for over-aligned types.

----

## Input and output

[FIO30-C] Exclude user input from format strings.

[FIO32-C] Do not perform operations on devices that are only appropriate for files.

[FIO34-C] Distinguish between characters read from a file and EOF or WEOF.

[FIO37-C] Do not assume that fgets() or fgetws() returns a nonempty string when successful.

[FIO38-C] Do not copy a FILE object.

[FIO39-C] Do not alternately input and output from a stream without an intervening flush or positioning call.

[FIO40-C] Reset strings on fgets() or fgetws() failure.

[FIO41-C] Do not call getc(), putc(), getwc(), or putwc() with a stream argument that has side effects.

[FIO42-C] Close files when they are no longer needed.

[FIO44-C] Only use values for fsetpos() that are returned from fgetpos().

[FIO45-C] Avoid TOCTOU race conditions while accessing files.

[FIO46-C] Do not access a closed file.

[FIO47-C] Use valid format strings.

[FIO50-CPP] Do not alternately input and output from a file stream without an intervening positioning call.

[FIO51-CPP] Close files when they are no longer needed.

----

## Exceptions and error handling

[ERR30-C] Set errno to zero before calling a library function known to set errno, and check errno only after the function returns a value indicating failure.

[ERR32-C] Do not rely on indeterminate values of errno.

[ERR33-C] Detect and handle standard library errors.

[ERR34-C] Detect errors when converting a string to a number.

[ERR50-CPP] Do not abruptly terminate the program.

[ERR51-CPP] Handle all exceptions.

[ERR52-CPP] Do not use setjmp() or longjmp().

[ERR53-CPP] Do not reference base classes or class data members in a constructor or destructor function-try-block handler.

[ERR54-CPP] Catch handlers should order their parameter types from most derived to least derived.

[ERR55-CPP] Honor exception specifications.

[ERR56-CPP] Guarantee exception safety.

[ERR57-CPP] Do not leak resources when handling exceptions.

[ERR58-CPP] Handle all exceptions thrown before main() begins executing.

[ERR59-CPP] Do not throw an exception across execution boundaries.

[ERR60-CPP] Exception objects must be nothrow copy constructible.

[ERR61-CPP] Catch exceptions by lvalue reference.

[ERR62-CPP] Detect errors when converting a string to a number.

----

## Object-oriented programming

[OOP50-CPP] Do not invoke virtual functions from constructors or destructors.

[OOP51-CPP] Do not slice derived objects.

[OOP52-CPP] Do not delete a polymorphic object without a virtual destructor.

[OOP53-CPP] Write constructor member initializers in the canonical order.

[OOP54-CPP] Gracefully handle self-copy assignment.

[OOP55-CPP] Do not use pointer-to-member operators to access nonexistent members.

[OOP56-CPP] Honor replacement handler requirements.

[OOP57-CPP	Prefer special member functions and overloaded operators to C] Standard Library functions.

[OOP58-CPP] Copy operations must not mutate the source object.

----

## Concurrency

[CON33-C] Avoid race conditions when using library functions.

[CON37-C] Do not call signal() in a multithreaded program.

[CON40-C] Do not refer to an atomic variable twice in an expression.

[CON41-C] Wrap functions that can fail spuriously in a loop.

[CON43-C] Do not allow data races in multithreaded code.

[CON50-CPP] Do not destroy a mutex while it is locked.

[CON51-CPP] Ensure actively held locks are released on exceptional conditions.

[CON52-CPP] Prevent data races when accessing bit-fields from multiple threads.

[CON53-CPP] Avoid deadlock by locking in a predefined order.

[CON54-CPP] Wrap functions that can spuriously wake up in a loop.

[CON55-CPP] Preserve thread safety and liveness when using condition variables.

[CON56-CPP] Do not speculatively lock a non-recursive mutex that is already owned by the calling thread.

----

## Miscellaneous

[ENV30-C] Do not modify the object referenced by the return value of certain functions.

[ENV31-C] Do not rely on an environment pointer following an operation that may invalidate it.

[ENV32-C] All exit handlers must return normally.

[ENV33-C] Do not call system().

[ENV34-C] Do not store pointers returned by certain functions.

[FLP30-C] Do not use floating-point variables as loop counters.

[FLP32-C] Prevent or detect domain and range errors in math functions.

[FLP34-C] Ensure that floating-point conversions are within range of the new type.

[FLP36-C] Preserve precision when converting integral values to floating-point type.

[FLP37-C] Do not use object representations to compare floating-point values.

[MSC30-C] Do not use the rand() function for generating pseudorandom numbers.

[MSC32-C] Properly seed pseudorandom number generators.

[MSC33-C] Do not pass invalid data to the asctime() function.

[MSC37-C] Ensure that control never reaches the end of a non-void function.

[MSC38-C] Do not treat a predefined identifier as an object if it might only be implemented as a macro.

[MSC39-C] Do not call va_arg() on a va_list that has an indeterminate value.

[MSC40-C] Do not violate constraints.

[MSC41-C] Never hard code sensitive information.

[MSC50-CPP] Do not use std::rand() for generating pseudorandom numbers.

[MSC51-CPP] Ensure your random number generator is properly seeded.

[MSC52-CPP] Value-returning functions must return a value from all exit paths.

[MSC53-CPP] Do not return from a function declared [[noreturn]].

[MSC54-CPP] A signal handler must be a plain old function.

[PRE30-C] Do not create a universal character name through concatenation.

[PRE31-C] Avoid side effects in arguments to unsafe macros.

[PRE32-C] Do not use preprocessor directives in invocations of function-like macros.

[SIG31-C] Do not access shared objects in signal handlers.

[SIG34-C] Do not call signal() from within interruptible signal handlers.

[SIG35-C] Do not return from a computational exception signal handler.

----

**[中文版本](./015_CertCpp规范.md)**

----

