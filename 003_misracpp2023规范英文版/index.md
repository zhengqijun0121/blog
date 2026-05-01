# Misra C++ 2023 规范英文版


# MISRA C++ 2023 规范英文版

> [https://ww2.mathworks.cn/help/bugfinder/misra-cpp-2023-rules-and-directives.html?lang=en](https://ww2.mathworks.cn/help/bugfinder/misra-cpp-2023-rules-and-directives.html?lang=en)

**[中文版本](./013_MisraCpp2023规范.md)**

----

## Language independent issues

[Rule 0.0.1 Required] A function shall not contain unreachable statements.

[Rule 0.0.2 Advisory] Controlling expressions should not be invariant.

[Rule 0.1.1 Advisory] A value should not be unnecessarily written to a local object.

[Rule 0.1.2 Required] The value returned by a function shall be used.

[Rule 0.2.1 Advisory] Variables with limited visibility should be used at least once.

[Rule 0.2.2 Required] A named function parameter shall be used at least once.

[Rule 0.2.3 Advisory] Types with limited visibility should be used at least once.

[Rule 0.2.4 Advisory] Functions with limited visibility should be used at least once.

[Dir 0.3.1 Advisory] Floating-point arithmetic should be used appropriately.

[Dir 0.3.2 Required] A function call shall not violate the function's preconditions.

----

## General principles

[Rule 4.1.1 Advisory] A program shall conform to `ISO/IEC 14882:2017` (C++17).

[Rule 4.1.2 Required] Deprecated features should not be used.

[Rule 4.1.3 Required] There shall be no occurrence of undefined or critical unspecified behaviour.

[Rule 4.6.1 Required] Operations on a memory location shall be sequenced appropriately.

----

## Lexical conventions

[Rule 5.0.1 Advisory] Trigraph-like sequences should not be used.

[Rule 5.7.1 Required] The character sequence `/*` shall not be used within a `C-style` comment.

[Dir 5.7.2 Advisory] Sections of code should not be "commented out".

[Rule 5.7.3 Required] Line-splicing shall not be used in `//` comments.

[Rule 5.10.1 Required] User-defined identifiers shall have an appropriate form.

[Rule 5.13.1 Required] Within character literals and non raw-string literals, `\` shall only be used to form a defined escape sequence or universal character name.

[Rule 5.13.2 Required] Octal escape sequences, hexadecimal escape sequences and universal character names shall be terminated.

[Rule 5.13.3 Required] Octal constants shall not be used.

[Rule 5.13.4 Required] Unsigned integer literals shall be appropriately suffixed.

[Rule 5.13.5 Required] The lowercase form of `L` shall not be used as the first character in a literal suffix.

[Rule 5.13.6 Required] An integer-literal of type `long long` shall not use a single `L` or `l` in any suffix.

[Rule 5.13.7 Required] String literals with different encoding prefixes shall not be concatenated.

----

## Basic concepts

[Rule 6.0.1 Required] Block scope declarations shall not be visually ambiguous.

[Rule 6.0.2 Advisory] When an array with external linkage is declared, its size should be explicitly specified.

[Rule 6.0.3 Advisory] The only declarations in the global namespace should be `main`, namespace declarations and `extern "C"` declarations.

[Rule 6.0.4 Required] The identifier `main` shall not be used for a function other than the global function `main`.

[Rule 6.2.1 Required] The one-definition rule shall not be violated.

[Rule 6.2.2 Required] All declarations of a variable or function shall have the same type.

[Rule 6.2.3 Required] The source code used to implement an entity shall appear only once.

[Rule 6.2.4 Required] A header file shall not contain definitions of functions or objects that are non-inline and have external linkage.

[Rule 6.4.1 Required] A variable declared in an inner scope shall not hide a variable declared in an outer scope.

[Rule 6.4.2 Required] Derived classes shall not conceal functions that are inherited from their bases.

[Rule 6.4.3 Required] A name that is present in a dependent base shall not be resolved by unqualified lookup.

[Rule 6.5.1 Advisory] A function or object with external linkage should be introduced in a header file.

[Rule 6.5.2 Advisory] Internal linkage should be specified appropriately.

[Rule 6.7.1 Required] Local variables shall not have static storage duration.

[Rule 6.7.2 Required] Global variables shall not be used.

[Rule 6.8.1 Required] An object shall not be accessed outside of its lifetime.

[Rule 6.8.2 Mandatory] A function must not return a reference or a pointer to a local variable with automatic storage duration.

[Rule 6.8.3 Required] An assignment operator shall not assign the address of an object with automatic storage duration to an object with a greater lifetime.

[Rule 6.8.4 Advisory] Member functions returning references to their object should be ref-qualified appropriately.

[Rule 6.9.1 Required] The same type aliases shall be used in all declarations of the same entity.

[Rule 6.9.2 Advisory] The names of the standard signed integer types and standard unsigned integer types should not be used.

----

## Standard conventions

[Rule 7.0.1 Required] There shall be no conversion from type `bool`.

[Rule 7.0.2 Required] There shall be no conversion to type `bool`.

[Rule 7.0.3 Required] The numerical value of a character shall not be used.

[Rule 7.0.4 Required] The operands of bitwise operators and shift operators shall be appropriate.

[Rule 7.0.5 Required] Integral promotion and the usual arithmetic conversions shall not change the signedness or the type category of an operand.

[Rule 7.0.6 Required] Assignment between numeric types shall be appropriate.

[Rule 7.11.1 Required] `nullptr` shall be the only form of the null-pointer-constant.

[Rule 7.11.2 Required] An array passed as a function argument shall not decay to a pointer.

[Rule 7.11.3 Required] A conversion from function type to pointer-to-function type shall only occur in appropriate contexts.

----

## Expressions

[Rule 8.0.1 Advisory] Parentheses should be used to make the meaning of an expression appropriately explicit.

[Rule 8.1.1 Required] A non-transient `lambda` shall not implicitly capture `this`.

[Rule 8.1.2 Advisory] Variables should be captured explicitly in a non-transient `lambda`.

[Rule 8.2.1 Required] A virtual base class shall only be cast to a derived class by means of `dynamic_cast`.

[Rule 8.2.2 Required] `C-style` casts and functional notation casts shall not be used.

[Rule 8.2.3 Required] A cast shall not remove any `const` or `volatile` qualification from the type accessed via a pointer or by reference.

[Rule 8.2.4 Required] Casts shall not be performed between a pointer to function and any other type.

[Rule 8.2.5 Required] `reinterpret_cast` shall not be used.

[Rule 8.2.6 Required] An object with integral, enumerated, or pointer to `void` type shall not be cast to a pointer type.

[Rule 8.2.7 Advisory] A cast should not convert a pointer type to an integral type.

[Rule 8.2.8 Required] An object pointer type shall not be cast to an integral type other than `std::uintptr_t` or `std::intptr_t`.

[Rule 8.2.9 Required] The operand to `typeid` shall not be an expression of polymorphic class type.

[Rule 8.2.10 Required] Functions shall not call themselves, either directly or indirectly.

[Rule 8.2.11 Required] An argument passed via ellipsis shall have an appropriate type.

[Rule 8.3.1 Advisory] The built-in unary `-` operator should not be applied to an expression of unsigned type.

[Rule 8.3.2 Advisory] The built-in unary `+` operator should not be used.

[Rule 8.7.1 Required] Pointer arithmetic shall not form an invalid pointer.

[Rule 8.7.2 Required] Subtraction between pointers shall only be applied to pointers that address elements of the same array.

[Rule 8.9.1 Required] The built-in relational operators `>`, `>=`, `<` and `<=` shall not be applied to objects of pointer type, except where they point to elements of the same array.

[Rule 8.14.1 Advisory] The right-hand operand of a logical `&&` or `||` operator should not contain persistent side effects.

[Rule 8.18.1 Mandatory] An object or subobject must not be copied to an overlapping object.

[Rule 8.18.2 Advisory] The result of an assignment operator should not be used.

[Rule 8.19.1 Advisory] The comma operator should not be used.

[Rule 8.20.1 Advisory] An unsigned arithmetic operation with constant operands should not wrap.

----

## Statements

[Rule 9.2.1 Required] An explicit type conversion shall not be an expression statement.

[Rule 9.3.1 Required] The body of an iteration-statement or a selection-statement shall be a compound-statement.

[Rule 9.4.1 Required] All `if ... else if` constructs shall be terminated with an `else` statement.

[Rule 9.4.2 Required] The structure of a `switch` statement shall be appropriate.

[Rule 9.5.1 Required] Legacy `for` statements should be simple.

[Rule 9.5.2 Required] A `for-range-initializer` shall contain at most one function call.

[Rule 9.6.1 Advisory] The `goto` statement should not be used.

[Rule 9.6.2 Required] A `goto` statement shall reference a label in a surrounding block.

[Rule 9.6.3 Required] The `goto` statement shall jump to a label declared later in the function body.

[Rule 9.6.4 Required] A function declared with the `[[noreturn]]` attribute shall not return.

[Rule 9.6.5 Required] A function with `non-void` return type shall return a value on all paths.

----

## Declarations

[Rule 10.0.1 Advisory] A declaration should not declare more than one variable or member variable.

[Rule 10.1.1 Advisory] The target type of a pointer or lvalue reference parameter should be `const-qualified` appropriately.

[Rule 10.1.2 Required] The `volatile` qualifier shall be used appropriately.

[Rule 10.2.1 Required] An enumeration shall be defined with an explicit underlying type.

[Rule 10.2.2 Advisory] Unscoped enumerations should not be declared.

[Rule 10.2.3 Required] The numeric value of an unscoped enumeration with no fixed underlying type shall not be used.

[Rule 10.3.1 Advisory] There should be no unnamed namespaces in header files.

[Rule 10.4.1 Required] The `asm` declaration shall not be used.

----

## Declarators

[Rule 11.3.1 Advisory] Variables of array type should not be declared.

[Rule 11.3.2 Advisory] The declaration of an object should contain no more than two levels of pointer indirection.

[Rule 11.6.1 Advisory] All variables should be initialized.

[Rule 11.6.2 Mandatory] The value of an object must not be read before it has been set.

[Rule 11.6.3 Required] Within an enumerator list, the value of an implicitly-specified enumeration constant shall be unique.

----

## Classes

[Rule 12.2.1 Advisory] Bit-fields should not be declared.

[Rule 12.2.2 Required] A bit-field shall have an appropriate type.

[Rule 12.2.3 Required] A named bit-field with signed integer type shall not have a length of one bit.

[Rule 12.3.1 Required] The `union` keyword shall not be used.

----

## Derived classes

[Rule 13.1.1 Advisory] Classes should not be inherited virtually.

[Rule 13.1.2 Required] An accessible base class shall not be both virtual and non-virtual in the same hierarchy.

[Rule 13.3.1 Required] User-declared member functions shall use the `virtual`, `override` and `final` specifiers appropriately.

[Rule 13.3.2 Required] Parameters in an overriding virtual function shall not specify different default arguments.

[Rule 13.3.3 Required] The parameters in all declarations or overrides of a function shall either be unnamed or have identical names.

[Rule 13.3.4 Required] A comparison of a potentially virtual pointer to member function shall only be with `nullptr`.

----

## Member access control

[Rule 14.1.1 Advisory] Non-static data members should be either all private or all public.

----

## Special member functions

[Rule 15.0.1 Required] Special member functions shall be provided appropriately.

[Rule 15.0.2 Advisory] User-provided copy and move member functions of a class should have appropriate signatures.

[Rule 15.1.1 Required] An object's dynamic type shall not be used from within its constructor or destructor.

[Rule 15.1.2 Advisory] All constructors of a class should explicitly initialize all of its virtual base classes and immediate base classes.

[Rule 15.1.3 Required] Conversion operators and constructors that are callable with a single argument shall be explicit.

[Rule 15.1.4 Advisory] All direct, non-static data members of a class should be initialized before the class object is accessible.

[Rule 15.1.5 Required] A class shall only define an initializer-list constructor when it is the only constructor.

[Dir 15.8.1 Advisory] User-provided copy assignment operators and move assignment operators shall handle self-assignment.

----

## Overriding

[Rule 16.5.1 Required] The logical `AND` and logical `OR` operators shall not be overloaded.

[Rule 16.5.2 Required] The `address-of` operator shall not be overloaded.

[Rule 16.6.1 Advisory] Symmetrical operators should only be implemented as non-member functions.

----

## Templates

[Rule 17.8.1 Required] Function templates shall not be explicitly specialized.

----

## Exception handling

[Rule 18.1.1 Required] An exception object shall not have pointer type.

[Rule 18.1.2 Required] An empty `throw` shall only occur within the compound-statement of a `catch` handler.

[Rule 18.3.1 Advisory] There should be at least one exception handler to catch all otherwise unhandled exceptions.

[Rule 18.3.2 Required] An exception of class type shall be caught by `const` reference or reference.

[Rule 18.3.3 Required] Handlers for a `function-try-block` of a constructor or destructor shall not refer to non-static members from their class or its bases.

[Rule 18.4.1 Required] Exception-unfriendly functions shall be `noexcept`.

[Rule 18.5.1 Advisory] A `noexcept` function should not attempt to propagate an exception to the calling function.

[Rule 18.5.2 Advisory] Program-terminating functions should not be used.

----

## Preprocessing directives

[Rule 19.0.1 Required] A line whose first token is `#` shall be a valid preprocessing directive.

[Rule 19.0.2 Required] Function-like macros shall not be defined.

[Rule 19.0.3 Advisory] `#include` directives should only be preceded by preprocessor directives or comments.

[Rule 19.0.4 Advisory] `#undef` should only be used for macros defined previously in the same file.

[Rule 19.1.1 Required] The `defined` preprocessor operator shall be used appropriately.

[Rule 19.1.2 Required] All `#else`, `#elif` and `#endif` preprocessor directives shall reside in the same file as the `#if`, `#ifdef` or `#ifndef` directive to which they are related.

[Rule 19.1.3 Required] All identifiers used in the controlling expression of `#if` or `#elif` preprocessing directives shall be defined prior to evaluation.

[Rule 19.2.1 Required] Precautions shall be taken in order to prevent the contents of a header file being included more than once.

[Rule 19.2.2 Required] The `#include` directive shall be followed by either a `<filename>` or `"filename"` sequence.

[Rule 19.2.3 Required] The `'` or `"` or `\` characters and the `/*` or `//` character sequences shall not occur in a header file name.

[Rule 19.3.1 Advisory] The `#` and `##` preprocessor operators should not be used.

[Rule 19.3.2 Required] A macro parameter immediately following a `#` operator shall not be immediately followed by a `##` operator.

[Rule 19.3.3 Required] The argument to a mixed-use macro parameter shall not be subject to further expansion.

[Rule 19.3.4 Required] Parentheses shall be used to ensure macro arguments are expanded appropriately.

[Rule 19.3.5 Required] Tokens that look like a preprocessing directive shall not occur within a macro argument.

[Rule 19.6.1 Advisory] The `#pragma` directive and the `_Pragma` operator should not be used.

----

## Language support library

[Rule 21.2.1 Required] The library functions `atof`, `atoi`, `atol` and `atell` from `<cstdlib>` shall not be used.

[Rule 21.2.2 Required] The string handling functions from `<cstring>`, `<cstdlib>`, `<cwchar>` and `<cinttypes>` shall not be used.

[Rule 21.2.3 Required] The library function `system` from `<cstdlib>` shall not be used.

[Rule 21.2.4 Required] The macro `offsetof` shall not be used.

[Rule 21.6.1 Advisory] Dynamic memory should not be used.

[Rule 21.6.2 Required] Dynamic memory shall be managed automatically.

[Rule 21.6.3 Required] Advanced memory management shall not be used.

[Rule 21.6.4 Required] If a project defines either a sized or unsized version of a global operator `delete`, then both shall be defined.

[Rule 21.6.5 Required] A pointer to an incomplete class type shall not be deleted.

[Rule 21.10.1 Required] The features of `<cstdarg>` shall not be used.

[Rule 21.10.2 Required] The standard header file `<csetjmp>` shall not be used.

[Rule 21.10.3 Required] The facilities provided by the standard header file `<csignal>` shall not be used.

----

## Diagnostics library

[Rule 22.3.1 Required] The `assert` macro shall not be used with a constant-expression.

[Rule 22.4.1 Required] The literal value zero shall be the only value assigned to `errno`.

----

## General utilities library

[Rule 23.11.1 Required] The raw pointer constructors of `std::shared_ptr` and `std::unique_ptr` should not be used.

----

## Strings library

[Rule 24.5.1 Required] The character handling functions from `<cctype>` and `<cwctype>` shall not be used.

[Rule 24.5.2 Required] The `C++` Standard Library functions `memcpy`, `memmove`, `memcmp` from `<cstring>` shall not be used.

----

## Localization library

[Rule 25.5.1 Required] The `setlocale` and `std::locale::global` functions shall not be called.

[Rule 25.5.2 Mandatory] The pointers returned by the `C++` Standard Library functions `localeconv`, `getenv`, `setlocale` or `strerror` must only be used as if they have pointer to `const-qualified` type.

[Rule 25.5.3 Mandatory] The pointer returned by the `C++` Standard Library functions `asctime`, `ctime`, `gmtime`, `localtime`, `localeconv`, `getenv`, `setlocale` or `strerror` must not be used following a subsequent call to the same function.

----

## Containers library

[Rule 26.3.1 Advisory] `std::vector` should not be specialized with `bool`.

----

## Algorithms library

[Rule 28.3.1 Required] Predicates shall not have persistent side effects.

[Rule 28.6.1 Required] The argument to `std::move` shall be a `non-const` lvalue.

[Rule 28.6.2 Required] Forwarding references and `std::forward` shall be used together.

[Rule 28.6.3 Required] An object shall not be used while in a potentially moved-from state.

[Rule 28.6.4 Required] The result of `std::remove`, `std::remove_if`, `std::unique` and `empty` shall be used.

----

## Input/output library

[Rule 30.0.1 Required] The `C` Library input/output functions shall not be used.

[Rule 30.0.2 Required] Reads and writes on the same file stream shall be separated by a positioning operation.

----

**[中文版本](./013_MisraCpp2023规范.md)**

----

