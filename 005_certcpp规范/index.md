# CERT C++ 规范


# CERT Cpp 规范

----

## English Version

### Declare and Initialization


CERT C++: DCL30-C	Declare objects with appropriate storage durations

CERT C++: DCL39-C	Avoid information leakage in structure padding

CERT C++: DCL40-C	Do not create incompatible declarations of the same function or object

CERT C++: DCL50-CPP	Do not define a C-style variadic function

CERT C++: DCL51-CPP	Do not declare or define a reserved identifier

CERT C++: DCL52-CPP	Never qualify a reference type with const or volatile

CERT C++: DCL53-CPP	Do not write syntactically ambiguous declarations

CERT C++: DCL54-CPP	Overload allocation and deallocation functions as a pair in the same scope

CERT C++: DCL55-CPP	Avoid information leakage when passing a class object across a trust boundary

CERT C++: DCL56-CPP	Avoid cycles during initialization of static objects

CERT C++: DCL57-CPP	Do not let exceptions escape from destructors or deallocation functions

CERT C++: DCL58-CPP	Do not modify the standard namespaces

CERT C++: DCL59-CPP	Do not define an unnamed namespace in a header file

CERT C++: DCL60-CPP	Obey the one-definition rule

----

### Expressions


CERT C++: EXP34-C	Do not dereference null pointers

CERT C++: EXP35-C	Do not modify objects with temporary lifetime

CERT C++: EXP36-C	Do not cast pointers into more strictly aligned pointer types

CERT C++: EXP37-C	Call functions with the correct number and type of arguments

CERT C++: EXP39-C	Do not access a variable through a pointer of an incompatible type

CERT C++: EXP42-C	Do not compare padding data

CERT C++: EXP45-C	Do not perform assignments in selection statements

CERT C++: EXP46-C	Do not use a bitwise operator with a Boolean-like operand

CERT C++: EXP47-C	Do not call va_arg with an argument of the incorrect type

CERT C++: EXP50-CPP	Do not depend on the order of evaluation for side effects

CERT C++: EXP51-CPP	Do not delete an array through a pointer of the incorrect type

CERT C++: EXP52-CPP	Do not rely on side effects in unevaluated operands

CERT C++: EXP53-CPP	Do not read uninitialized memory

CERT C++: EXP54-CPP	Do not access an object outside of its lifetime

CERT C++: EXP55-CPP	Do not access a cv-qualified object through a cv-unqualified type

CERT C++: EXP56-CPP	Do not call a function with a mismatched language linkage

CERT C++: EXP57-CPP	Do not cast or delete pointers to incomplete classes

CERT C++: EXP58-CPP	Pass an object of the correct type to va_start

CERT C++: EXP59-CPP	Use offsetof() on valid types and members

CERT C++: EXP60-CPP	Do not pass a nonstandard-layout type object across execution boundaries

CERT C++: EXP61-CPP	A lambda object must not outlive any of its reference captured objects

CERT C++: EXP62-CPP	Do not access the bits of an object representation that are not part of the object's value representation

CERT C++: EXP63-CPP	Do not rely on the value of a moved-from object

----

### Integers


CERT C++: INT30-C	Ensure that unsigned integer operations do not wrap

CERT C++: INT31-C	Ensure that integer conversions do not result in lost or misinterpreted data

CERT C++: INT32-C	Ensure that operations on signed integers do not result in overflow

CERT C++: INT33-C	Ensure that division and remainder operations do not result in divide-by-zero errors

CERT C++: INT34-C	Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand

CERT C++: INT35-C	Use correct integer precisions

CERT C++: INT36-C	Converting a pointer to integer or integer to pointer

CERT C++: INT50-CPP	Do not cast to an out-of-range enumeration value

----

### Containers


CERT C++: ARR30-C	Do not form or use out-of-bounds pointers or array subscripts

CERT C++: ARR37-C	Do not add or subtract an integer to a pointer to a non-array object

CERT C++: ARR38-C	Guarantee that library functions do not form invalid pointers

CERT C++: ARR39-C	Do not add or subtract a scaled integer to a pointer

CERT C++: CTR50-CPP	Guarantee that container indices and iterators are within the valid range

CERT C++: CTR51-CPP	Use valid references, pointers, and iterators to reference elements of a container

CERT C++: CTR52-CPP	Guarantee that library functions do not overflow

CERT C++: CTR53-CPP	Use valid iterator ranges

CERT C++: CTR54-CPP	Do not subtract iterators that do not refer to the same container

CERT C++: CTR55-CPP	Do not use an additive operator on an iterator if the result would overflow

CERT C++: CTR56-CPP	Do not use pointer arithmetic on polymorphic objects

CERT C++: CTR57-CPP	Provide a valid ordering predicate

CERT C++: CTR58-CPP	Predicate function objects should not be mutable

----

### Characters and strings

CERT C++: STR30-C	Do not attempt to modify string literals

CERT C++: STR31-C	Guarantee that storage for strings has sufficient space for character data and the null terminator

CERT C++: STR32-C	Do not pass a non-null-terminated character sequence to a library function that expects a string

CERT C++: STR34-C	Cast characters to unsigned char before converting to larger integer sizes

CERT C++: STR37-C	Arguments to character-handling functions must be representable as an unsigned char

CERT C++: STR38-C	Do not confuse narrow and wide character strings and functions

CERT C++: STR50-CPP	Guarantee that storage for strings has sufficient space for character data and the null terminator

CERT C++: STR51-CPP	Do not attempt to create a std::string from a null pointer

CERT C++: STR52-CPP	Use valid references, pointers, and iterators to reference elements of a basic_string

CERT C++: STR53-CPP	Range check element access

----

### Memory Management

CERT C++: MEM30-C	Do not access freed memory

CERT C++: MEM31-C	Free dynamically allocated memory when no longer needed

CERT C++: MEM34-C	Only free memory allocated dynamically

CERT C++: MEM35-C	Allocate sufficient memory for an object

CERT C++: MEM36-C	Do not modify the alignment of objects by calling realloc()

CERT C++: MEM50-CPP	Do not access freed memory

CERT C++: MEM51-CPP	Properly deallocate dynamically allocated resources

CERT C++: MEM52-CPP	Detect and handle memory allocation errors

CERT C++: MEM53-CPP	Explicitly construct and destruct objects when manually managing object lifetime

CERT C++: MEM54-CPP	Provide placement new with properly aligned pointers to sufficient storage capacity

CERT C++: MEM55-CPP	Honor replacement dynamic storage management requirements

CERT C++: MEM56-CPP	Do not store an already-owned pointer value in an unrelated smart pointer

CERT C++: MEM57-CPP	Avoid using default operator new for over-aligned types

----

### Input and output

CERT C++: FIO30-C	Exclude user input from format strings

CERT C++: FIO32-C	Do not perform operations on devices that are only appropriate for files

CERT C++: FIO34-C	Distinguish between characters read from a file and EOF or WEOF

CERT C++: FIO37-C	Do not assume that fgets() or fgetws() returns a nonempty string when successful

CERT C++: FIO38-C	Do not copy a FILE object

CERT C++: FIO39-C	Do not alternately input and output from a stream without an intervening flush or positioning call

CERT C++: FIO40-C	Reset strings on fgets() or fgetws() failure

CERT C++: FIO41-C	Do not call getc(), putc(), getwc(), or putwc() with a stream argument that has side effects

CERT C++: FIO42-C	Close files when they are no longer needed

CERT C++: FIO44-C	Only use values for fsetpos() that are returned from fgetpos()

CERT C++: FIO45-C	Avoid TOCTOU race conditions while accessing files

CERT C++: FIO46-C	Do not access a closed file

CERT C++: FIO47-C	Use valid format strings

CERT C++: FIO50-CPP	Do not alternately input and output from a file stream without an intervening positioning call

CERT C++: FIO51-CPP	Close files when they are no longer needed

----

### Exceptions and error handling

CERT C++: ERR30-C	Set errno to zero before calling a library function known to set errno, and check errno only after the function returns a value indicating failure

CERT C++: ERR32-C	Do not rely on indeterminate values of errno

CERT C++: ERR33-C	Detect and handle standard library errors

CERT C++: ERR34-C	Detect errors when converting a string to a number

CERT C++: ERR50-CPP	Do not abruptly terminate the program

CERT C++: ERR51-CPP	Handle all exceptions

CERT C++: ERR52-CPP	Do not use setjmp() or longjmp()

CERT C++: ERR53-CPP	Do not reference base classes or class data members in a constructor or destructor function-try-block handler

CERT C++: ERR54-CPP	Catch handlers should order their parameter types from most derived to least derived

CERT C++: ERR55-CPP	Honor exception specifications

CERT C++: ERR56-CPP	Guarantee exception safety

CERT C++: ERR57-CPP	Do not leak resources when handling exceptions

CERT C++: ERR58-CPP	Handle all exceptions thrown before main() begins executing

CERT C++: ERR59-CPP	Do not throw an exception across execution boundaries

CERT C++: ERR60-CPP	Exception objects must be nothrow copy constructible

CERT C++: ERR61-CPP	Catch exceptions by lvalue reference

CERT C++: ERR62-CPP	Detect errors when converting a string to a number

----

### Object-oriented programming

CERT C++: OOP50-CPP	Do not invoke virtual functions from constructors or destructors

CERT C++: OOP51-CPP	Do not slice derived objects

CERT C++: OOP52-CPP	Do not delete a polymorphic object without a virtual destructor

CERT C++: OOP53-CPP	Write constructor member initializers in the canonical order

CERT C++: OOP54-CPP	Gracefully handle self-copy assignment

CERT C++: OOP55-CPP	Do not use pointer-to-member operators to access nonexistent members

CERT C++: OOP56-CPP	Honor replacement handler requirements

CERT C++: OOP57-CPP	Prefer special member functions and overloaded operators to C Standard Library functions

CERT C++: OOP58-CPP	Copy operations must not mutate the source object

----

### Concurrency

CERT C++: CON33-C	Avoid race conditions when using library functions

CERT C++: CON37-C	Do not call signal() in a multithreaded program

CERT C++: CON40-C	Do not refer to an atomic variable twice in an expression

CERT C++: CON41-C	Wrap functions that can fail spuriously in a loop

CERT C++: CON43-C	Do not allow data races in multithreaded code

CERT C++: CON50-CPP	Do not destroy a mutex while it is locked

CERT C++: CON51-CPP	Ensure actively held locks are released on exceptional conditions

CERT C++: CON52-CPP	Prevent data races when accessing bit-fields from multiple threads

CERT C++: CON53-CPP	Avoid deadlock by locking in a predefined order

CERT C++: CON54-CPP	Wrap functions that can spuriously wake up in a loop

CERT C++: CON55-CPP	Preserve thread safety and liveness when using condition variables

CERT C++: CON56-CPP	Do not speculatively lock a non-recursive mutex that is already owned by the calling thread

----

### Miscellaneous

CERT C++: ENV30-C	Do not modify the object referenced by the return value of certain functions

CERT C++: ENV31-C	Do not rely on an environment pointer following an operation that may invalidate it

CERT C++: ENV32-C	All exit handlers must return normally

CERT C++: ENV33-C	Do not call system()

CERT C++: ENV34-C	Do not store pointers returned by certain functions

CERT C++: FLP30-C	Do not use floating-point variables as loop counters

CERT C++: FLP32-C	Prevent or detect domain and range errors in math functions

CERT C++: FLP34-C	Ensure that floating-point conversions are within range of the new type

CERT C++: FLP36-C	Preserve precision when converting integral values to floating-point type

CERT C++: FLP37-C	Do not use object representations to compare floating-point values

CERT C++: MSC30-C	Do not use the rand() function for generating pseudorandom numbers

CERT C++: MSC32-C	Properly seed pseudorandom number generators

CERT C++: MSC33-C	Do not pass invalid data to the asctime() function

CERT C++: MSC37-C	Ensure that control never reaches the end of a non-void function

CERT C++: MSC38-C	Do not treat a predefined identifier as an object if it might only be implemented as a macro

CERT C++: MSC39-C	Do not call va_arg() on a va_list that has an indeterminate value

CERT C++: MSC40-C	Do not violate constraints

CERT C++: MSC41-C	Never hard code sensitive information

CERT C++: MSC50-CPP	Do not use std::rand() for generating pseudorandom numbers

CERT C++: MSC51-CPP	Ensure your random number generator is properly seeded

CERT C++: MSC52-CPP	Value-returning functions must return a value from all exit paths

CERT C++: MSC53-CPP	Do not return from a function declared [[noreturn]]

CERT C++: MSC54-CPP	A signal handler must be a plain old function

CERT C++: PRE30-C	Do not create a universal character name through concatenation

CERT C++: PRE31-C	Avoid side effects in arguments to unsafe macros

CERT C++: PRE32-C	Do not use preprocessor directives in invocations of function-like macros

CERT C++: SIG31-C	Do not access shared objects in signal handlers

CERT C++: SIG34-C	Do not call signal() from within interruptible signal handlers

CERT C++: SIG35-C	Do not return from a computational exception signal handler

----

### Member access control

AUTOSAR C++14 Rule A11-0-1	A non-POD type should be defined as class
AUTOSAR C++14 Rule A11-0-2	A type defined as struct shall: (1) provide only public data members, (2) not provide any special member functions or methods, (3) not be a base of another struct or class, (4) not inherit from another struct or class
AUTOSAR C++14 Rule A11-3-1	Friend declarations shall not be used
AUTOSAR C++14 Rule M11-0-1	Member data in non-POD class types shall be private

----

### Special member functions

AUTOSAR C++14 Rule A12-0-1	If a class declares a copy or move operation, or a destructor, either via "=default", "=delete", or via a user-provided declaration, then all others of these five special member functions shall be declared as well
AUTOSAR C++14 Rule A12-0-2	Bitwise operations and operations that assume data representation in memory shall not be performed on objects
AUTOSAR C++14 Rule A12-1-1	Constructors shall explicitly initialize all virtual base classes, all direct non-virtual base classes and all non-static data members
AUTOSAR C++14 Rule A12-1-2	Both NSDMI and a non-static member initializer in a constructor shall not be used in the same type
AUTOSAR C++14 Rule A12-1-3	If all user-defined constructors of a class initialize data members with constant values that are the same across all constructors, then data members shall be initialized using NSDMI instead
AUTOSAR C++14 Rule A12-1-4	All constructors that are callable with a single argument of fundamental type shall be declared explicit
AUTOSAR C++14 Rule A12-1-5	Common class initialization for non-constant members shall be done by a delegating constructor
AUTOSAR C++14 Rule A12-1-6	Derived classes that do not need further explicit initialization and require all the constructors from the base class shall use inheriting constructors
AUTOSAR C++14 Rule A12-4-1	Destructor of a base class shall be public virtual, public override or protected non-virtual
AUTOSAR C++14 Rule A12-4-2	If a public destructor of a class is non-virtual, then the class should be declared final
AUTOSAR C++14 Rule A12-6-1	All class data members that are initialized by the constructor shall be initialized using member initializers
AUTOSAR C++14 Rule A12-7-1	If the behavior of a user-defined special member function is identical to implicitly defined special member function, then it shall be defined "=default" or be left undefined
AUTOSAR C++14 Rule A12-8-1	Move and copy constructors shall move and respectively copy base classes and data members of a class, without any side effects
AUTOSAR C++14 Rule A12-8-2	User-defined copy and move assignment operators should use user-defined no-throw swap function
AUTOSAR C++14 Rule A12-8-3	Moved-from object shall not be read-accessed
AUTOSAR C++14 Rule A12-8-4	Move constructor shall not initialize its class members and base classes using copy semantics
AUTOSAR C++14 Rule A12-8-5	A copy assignment and a move assignment operators shall handle self-assignment
AUTOSAR C++14 Rule A12-8-6	Copy and move constructors and copy assignment and move assignment operators shall be declared protected or defined "=delete" in base class
AUTOSAR C++14 Rule A12-8-7	Assignment operators should be declared with the ref-qualifier &
AUTOSAR C++14 Rule M12-1-1	An object's dynamic type shall not be used from the body of its constructor or destructor

----

### Overriding

AUTOSAR C++14 Rule A13-1-2	User defined suffixes of the user defined literal operators shall start with underscore followed by one or more letters
AUTOSAR C++14 Rule A13-1-3	User defined literals operators shall only perform conversion of passed parameters
AUTOSAR C++14 Rule A13-2-1	An assignment operator shall return a reference to "this"
AUTOSAR C++14 Rule A13-2-2	A binary arithmetic operator and a bitwise operator shall return a "prvalue"
AUTOSAR C++14 Rule A13-2-3	A relational operator shall return a boolean value
AUTOSAR C++14 Rule A13-3-1	A function that contains "forwarding reference" as its argument shall not be overloaded
AUTOSAR C++14 Rule A13-5-1	If "operator[]" is to be overloaded with a non-const version, const version shall also be implemented
AUTOSAR C++14 Rule A13-5-2	All user-defined conversion operators shall be defined explicit
AUTOSAR C++14 Rule A13-5-3	User-defined conversion operators should not be used
AUTOSAR C++14 Rule A13-5-4	If two opposite operators are defined, one shall be defined in terms of the other
AUTOSAR C++14 Rule A13-5-5	Comparison operators shall be non-member functions with identical parameter types and noexcept
AUTOSAR C++14 Rule A13-6-1	Digit sequences separators ' shall only be used as follows: (1) for decimal, every 3 digits, (2) for hexadecimal, every 2 digits, (3) for binary, every 4 digits

----

### Templates

AUTOSAR C++14 Rule A14-1-1	A template should check if a specific template argument is suitable for this template
AUTOSAR C++14 Rule A14-5-1	A template constructor shall not participate in overload resolution for a single argument of the enclosing class type
AUTOSAR C++14 Rule A14-5-2	Class members that are not dependent on template class parameters should be defined in a separate base class
AUTOSAR C++14 Rule A14-5-3	A non-member generic operator shall only be declared in a namespace that does not contain class (struct) type, enum type or union type declarations
AUTOSAR C++14 Rule A14-7-1	A type used as a template argument shall provide all members that are used by the template
AUTOSAR C++14 Rule A14-7-2	Template specialization shall be declared in the same file (1) as the primary template (2) as a user-defined type, for which the specialization is declared
AUTOSAR C++14 Rule A14-8-2	Explicit specializations of function templates shall not be used
AUTOSAR C++14 Rule M14-5-3	A copy assignment operator shall be declared when there is a template assignment operator with a parameter that is a generic parameter
AUTOSAR C++14 Rule M14-6-1	In a class template with a dependent base, any name that may be found in that dependent base shall be referred to using a qualified-id or this->

----

### Exception handling

AUTOSAR C++14 Rule A15-0-2	At least the basic guarantee for exception safety shall be provided for all operations. In addition, each function may offer either the strong guarantee or the nothrow guarantee
AUTOSAR C++14 Rule A15-0-3	Exception safety guarantee of a called function shall be considered
AUTOSAR C++14 Rule A15-0-7	Exception handling mechanism shall guarantee a deterministic worst-case time execution time
AUTOSAR C++14 Rule A15-1-1	Only instances of types derived from std::exception should be thrown
AUTOSAR C++14 Rule A15-1-2	An exception object shall not be a pointer
AUTOSAR C++14 Rule A15-1-3	All thrown exceptions should be unique
AUTOSAR C++14 Rule A15-1-4	If a function exits with an exception, then before a throw, the function shall place all objects/resources that the function constructed in valid states or it shall delete them.
AUTOSAR C++14 Rule A15-1-5	Exceptions shall not be thrown across execution boundaries
AUTOSAR C++14 Rule A15-2-1	Constructors that are not noexcept shall not be invoked before program startup
AUTOSAR C++14 Rule A15-2-2	If a constructor is not noexcept and the constructor cannot finish object initialization, then it shall deallocate the object's resources and it shall throw an exception
AUTOSAR C++14 Rule A15-3-3	Main function and a task main function shall catch at least: base class exceptions from all third-party libraries used, std::exception and all otherwise unhandled exceptions
AUTOSAR C++14 Rule A15-3-4	Catch-all (ellipsis and std::exception) handlers shall be used only in (a) main, (b) task main functions, (c) in functions that are supposed to isolate independent components and (d) when calling third-party code that uses exceptions not according to AUTOSAR C++14 guidelines
AUTOSAR C++14 Rule A15-3-5	A class type exception shall be caught by reference or const reference
AUTOSAR C++14 Rule A15-4-1	Dynamic exception-specification shall not be used
AUTOSAR C++14 Rule A15-4-2	If a function is declared to be noexcept, noexcept(true) or noexcept(<true condition>), then it shall not exit with an exception
AUTOSAR C++14 Rule A15-4-3	The noexcept specification of a function shall either be identical across all translation units, or identical or more restrictive between a virtual member function and an overrider
AUTOSAR C++14 Rule A15-4-4	A declaration of non-throwing function shall contain noexcept specification
AUTOSAR C++14 Rule A15-4-5	Checked exceptions that could be thrown from a function shall be specified together with the function declaration and they shall be identical in all function declarations and for all its overriders
AUTOSAR C++14 Rule A15-5-1	All user-provided class destructors, deallocation functions, move constructors, move assignment operators and swap functions shall not exit with an exception. A noexcept exception specification shall be added to these functions as appropriate
AUTOSAR C++14 Rule A15-5-2	Program shall not be abruptly terminated. In particular, an implicit or explicit invocation of std::abort(), std::quick_exit(), std::_Exit(), std::terminate() shall not be done
AUTOSAR C++14 Rule A15-5-3	The std::terminate() function shall not be called implicitly
AUTOSAR C++14 Rule M15-0-3	Control shall not be transferred into a try or catch block using a goto or a switch statement
AUTOSAR C++14 Rule M15-1-1	The assignment-expression of a throw statement shall not itself cause an exception to be thrown
AUTOSAR C++14 Rule M15-1-2	NULL shall not be thrown explicitly
AUTOSAR C++14 Rule M15-1-3	An empty throw (throw;) shall only be used in the compound statement of a catch handler
AUTOSAR C++14 Rule M15-3-1	Exceptions shall be raised only after startup and before termination
AUTOSAR C++14 Rule M15-3-3	Handlers of a function-try-block implementation of a class constructor or destructor shall not reference non-static members from this class or its bases
AUTOSAR C++14 Rule M15-3-4	Each exception explicitly thrown in the code shall have a handler of a compatible type in all call paths that could lead to that point
AUTOSAR C++14 Rule M15-3-6	Where multiple handlers are provided in a single try-catch statement or function-try-block for a derived class and some or all of its bases, the handlers shall be ordered most-derived to base class
AUTOSAR C++14 Rule M15-3-7	Where multiple handlers are provided in a single try-catch statement or function-try-block, any ellipsis (catch-all) handler shall occur last

----

### Preprocessing directives

AUTOSAR C++14 Rule A16-0-1	The preprocessor shall only be used for unconditional and conditional file inclusion and include guards, and using specific directives
AUTOSAR C++14 Rule A16-2-1	The ', ", /*, //, \ characters shall not occur in a header file name or in #include directive
AUTOSAR C++14 Rule A16-2-2	There shall be no unused include directives
AUTOSAR C++14 Rule A16-2-3	An include directive shall be added explicitly for every symbol used in a file
AUTOSAR C++14 Rule A16-6-1	#error directive shall not be used
AUTOSAR C++14 Rule A16-7-1	The #pragma directive shall not be used
AUTOSAR C++14 Rule M16-0-1	#include directives in a file shall only be preceded by other preprocessor directives or comments
AUTOSAR C++14 Rule M16-0-2	Macros shall only be #define'd or #undef'd in the global namespace
AUTOSAR C++14 Rule M16-0-5	Arguments to a function-like macro shall not contain tokens that look like pre-processing directives
AUTOSAR C++14 Rule M16-0-6	In the definition of a function-like macro, each instance of a parameter shall be enclosed in parentheses, unless it is used as the operand of # or ##
AUTOSAR C++14 Rule M16-0-7	Undefined macro identifiers shall not be used in #if or #elif pre-processor directives, except as operands to the defined operator
AUTOSAR C++14 Rule M16-0-8	If the # token appears as the first token on a line, then it shall be immediately followed by a preprocessing token
AUTOSAR C++14 Rule M16-1-1	The defined pre-processor operator shall only be used in one of the two standard forms
AUTOSAR C++14 Rule M16-1-2	All #else, #elif and #endif pre-processor directives shall reside in the same file as the #if or #ifdef directive to which they are related
AUTOSAR C++14 Rule M16-2-3	Include guards shall be provided
AUTOSAR C++14 Rule M16-3-1	There shall be at most one occurrence of the # or ## operators in a single macro definition
AUTOSAR C++14 Rule M16-3-2	The # and ## operators should not be used

----

### Library Introduction

AUTOSAR C++14 Rule A17-0-1	Reserved identifiers, macros and functions in the C++ standard library shall not be defined, redefined or undefined
AUTOSAR C++14 Rule A17-1-1	Use of the C Standard Library shall be encapsulated and isolated
AUTOSAR C++14 Rule A17-6-1	Non-standard entities shall not be added to standard namespaces
AUTOSAR C++14 Rule M17-0-2	The names of standard library macros and objects shall not be reused
AUTOSAR C++14 Rule M17-0-3	The names of standard library functions shall not be overridden
AUTOSAR C++14 Rule M17-0-5	The setjmp macro and the longjmp function shall not be used

----

### Language support library

AUTOSAR C++14 Rule A18-0-1	The C library facilities shall only be accessed through C++ library headers
AUTOSAR C++14 Rule A18-0-2	The error state of a conversion from string to a numeric value shall be checked
AUTOSAR C++14 Rule A18-0-3	The library <clocale> (locale.h) and the setlocale function shall not be used
AUTOSAR C++14 Rule A18-1-1	C-style arrays shall not be used
AUTOSAR C++14 Rule A18-1-2	The std::vector<bool> specialization shall not be used
AUTOSAR C++14 Rule A18-1-3	The std::auto_ptr shall not be used
AUTOSAR C++14 Rule A18-1-4	A pointer pointing to an element of an array of objects shall not be passed to a smart pointer of single object type
AUTOSAR C++14 Rule A18-1-6	All std::hash specializations for user-defined types shall have a noexcept function call operator
AUTOSAR C++14 Rule A18-5-1	Functions malloc, calloc, realloc and free shall not be used
AUTOSAR C++14 Rule A18-5-2	Non-placement new or delete expressions shall not be used
AUTOSAR C++14 Rule A18-5-3	The form of delete operator shall match the form of new operator used to allocate the memory
AUTOSAR C++14 Rule A18-5-4	If a project has sized or unsized version of operator 'delete' globally defined, then both sized and unsized versions shall be defined
AUTOSAR C++14 Rule A18-5-5	Memory management functions shall ensure the following: (a) deterministic behavior resulting with the existence of worst-case execution time, (b) avoiding memory fragmentation, (c) avoid running out of memory, (d) avoiding mismatched allocations or deallocations, (e) no dependence on non-deterministic calls to kernel
AUTOSAR C++14 Rule A18-5-7	If non-real-time implementation of dynamic memory management functions is used in the project, then memory shall only be allocated and deallocated during non-real-time program phases
AUTOSAR C++14 Rule A18-5-8	Objects that do not outlive a function shall have automatic storage duration
AUTOSAR C++14 Rule A18-5-9	Custom implementations of dynamic memory allocation and deallocation functions shall meet the semantic requirements specified in the corresponding "Required behaviour" clause from the C++ Standard
AUTOSAR C++14 Rule A18-5-10	Placement new shall be used only with properly aligned pointers to sufficient storage capacity
AUTOSAR C++14 Rule A18-5-11	"operator new" and "operator delete" shall be defined together
AUTOSAR C++14 Rule A18-9-1	The std::bind shall not be used
AUTOSAR C++14 Rule A18-9-2	Forwarding values to other functions shall be done via: (1) std::move if the value is an rvalue reference, (2) std::forward if the value is forwarding reference
AUTOSAR C++14 Rule A18-9-3	The std::move shall not be used on objects declared const or const&
AUTOSAR C++14 Rule A18-9-4	An argument to std::forward shall not be subsequently used
AUTOSAR C++14 Rule M18-0-3	The library functions abort, exit, getenv and system from library <cstdlib> shall not be used
AUTOSAR C++14 Rule M18-0-4	The time handling functions of library <ctime> shall not be used
AUTOSAR C++14 Rule M18-0-5	The unbounded functions of library <cstring> shall not be used
AUTOSAR C++14 Rule M18-2-1	The macro offsetof shall not be used
AUTOSAR C++14 Rule M18-7-1	The signal handling facilities of <csignal> shall not be used

----

### Diagnostics library

AUTOSAR C++14 Rule M19-3-1	The error indicator errno shall not be used

----

### Genaral utilities library

AUTOSAR C++14 Rule A20-8-1	An already-owned pointer value shall not be stored in an unrelated smart pointer
AUTOSAR C++14 Rule A20-8-2	A std::unique_ptr shall be used to represent exclusive ownership
AUTOSAR C++14 Rule A20-8-3	A std::shared_ptr shall be used to represent shared ownership
AUTOSAR C++14 Rule A20-8-4	A std::unique_ptr shall be used over std::shared_ptr if ownership sharing is not required
AUTOSAR C++14 Rule A20-8-5	std::make_unique shall be used to construct objects owned by std::unique_ptr
AUTOSAR C++14 Rule A20-8-6	std::make_shared shall be used to construct objects owned by std::shared_ptr
AUTOSAR C++14 Rule A20-8-7	A std::weak_ptr shall be used to represent temporary shared ownership.

----

### Strings library

AUTOSAR C++14 Rule A21-8-1	Arguments to character-handling functions shall be representable as an unsigned char

----

### Containers library

AUTOSAR C++14 Rule A23-0-1	An iterator shall not be implicitly converted to const_iterator
AUTOSAR C++14 Rule A23-0-2	Elements of a container shall only be accessed via valid references, iterators, and pointers

----

### Algorithms library

AUTOSAR C++14 Rule A25-1-1	Non-static data members or captured values of predicate function objects that are state related to this object's identity shall not be copied
AUTOSAR C++14 Rule A25-4-1	Ordering predicates used with associative containers and STL sorting and related algorithms shall adhere to a strict weak ordering relation

----

### Ramdom number generation

AUTOSAR C++14 Rule A26-5-1	Pseudorandom numbers shall not be generated using std::rand()
AUTOSAR C++14 Rule A26-5-2	Random number engines shall not be default-initialized

----

### Input/output library

AUTOSAR C++14 Rule A27-0-1	Inputs from independent components shall be validated.
AUTOSAR C++14 Rule A27-0-2	A C-style string shall guarantee sufficient space for data and the null terminator
AUTOSAR C++14 Rule A27-0-3	Alternate input and output operations on a file stream shall not be used without an intervening flush or positioning call
AUTOSAR C++14 Rule A27-0-4	C-style strings shall not be used
AUTOSAR C++14 Rule M27-0-1	The stream input/output library <cstdio> shall not be used

----


