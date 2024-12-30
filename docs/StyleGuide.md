# Style Guide

# Table of Contents
1. [Naming](#naming)
2. [Header Files](#headers)
3. [Formatting](#formatting)
4. [Units](#units)
4. [Doxogen/Sphyinx/Auto Docs](#auto-docs)
5. [CMake File](#cmake)
6. [Notes](#notes)

## Naming <a name="naming"></a>
### Classes
Classes that are part of the main libary should use our modified UpperCamelCase style of "G4CMPXxxYyy"*.  

Classes that are a not a part of the main libary, i.e. examples, tests, and validation, should use UpperCamelCase but not the "G4CMP" prefix as to not confuse new users.    

*_Very early utility classes, in particular the code that handles the config.txt files (like G4LatticeLogical, etc.), and the phonon particle types, have the plain "G4" prefix.  In fact, there are versions of those classes in the G4 library.  If we change the names in G4CMP, then the G4 version will be "exposed" and confuse builds._

### Files
There should usually be one class per .cc file that follows the naming convention above.  See also [Header Files](#headers).  

## Header Files <a name="headers"></a>
In general, every .cc file should have an associated .hh file. There are some common exceptions, such as unit tests or macros and small .cc files containing just a main() function.

Correct use of header files can make a huge difference to the readability, size and performance of your code. 

The following rules will guide you through the various aspects of using header files.

### Self-contained Headers
Header files should be self-contained (compile on their own) and end in .hh. Non-header files that are meant for inclusion should end in .icc and be used sparingly.

There are rare cases where a file designed to be included is not self-contained. These are typically intended to be included at unusual locations, such as the middle of another file. They might not use header guards, and might not include their prerequisites. Name such files with the .icc extension. Use sparingly, and prefer self-contained headers when possible.

### The #define Guard
All header files should have #define guards to prevent multiple inclusion.

```cpp
#ifndef FOO_BAR_BAZ_HH
#define FOO_BAR_BAZ_HH 1

...

#endif  // FOO_BAR_BAZ_HH
```

### Include Headers Judiciously
#include hurts compile time performance. Don’t do it unless you have to, especially in header files.

[If a source or header file refers to a symbol defined elsewhere, the file should directly include a header file which properly intends to provide a declaration or definition of that symbol. It should not include header files for any other reason.]: (comment)

However, you must include all of the header files that you are using. It is recommended not to rely on transitive inclusions. This allows people to remove no-longer-needed #include statements from their headers without breaking clients.

[### Forward Declarations
Avoid using forward declarations where possible. Instead, include the headers you need.]: (comment)


### Inline Functions
Define functions inline only when they are small, say, 10 lines or fewer.

You can declare functions in a way that allows the compiler to expand them inline rather than calling them through the usual function call mechanism.

Inlining a function can generate more efficient object code, as long as the inlined function is small. Feel free to inline accessors and mutators, and other short, performance-critical functions.

Overuse of inlining can actually make programs slower. Depending on a function's size, inlining it can cause the code size to increase or decrease. Inlining a very small accessor function will usually decrease code size while inlining a very large function can dramatically increase code size. On modern processors smaller code usually runs faster due to better use of the instruction cache.

A decent rule of thumb is to not inline a function if it is more than 10 lines long. Beware of destructors, which are often longer than they appear because of implicit member- and base-destructor calls!

Another useful rule of thumb: it's typically not cost effective to inline functions with loops or switch statements (unless, in the common case, the loop or switch statement is never executed).

It is important to know that functions are not always inlined even if they are declared as such; for example, virtual and recursive functions are not normally inlined. Usually recursive functions should not be inline. The main reason for making a virtual function inline is to place its definition in the class, either for convenience or to document its behavior, e.g., for accessors and mutators.

### Names and Order of Includes
Include headers in the following order: G4CMP headers, G4 headers, other libraries' headers (e.g. CLHEP), C system headers, C++ standard library headers, then your project's headers.


Headers should only be included using an angle-bracketed path if the library requires you to do so. In particular, the following headers require angle brackets:

- C and C++ standard library headers (e.g. <stdlib.h> and <string>).
- POSIX, Linux, and Windows system headers (e.g. <unistd.h> and <windows.h>).
- In rare cases, third_party libraries (e.g. <Python.h>).

In mydetector.cc order your includes as follows:

1. G4CMPFoo.hh
1. A blank line
1. G4Bar.hh
1. A blank line
1. Other libraries' .hh files
1. A blank line
1. C system headers, and any other headers in angle brackets with the .h extension, e.g., <unistd.h>, <stdlib.h>, <Python.h>
1. A blank line
1. C++ standard library headers (without file extension), e.g., <algorithm>, <cstddef>
1. A blank line
1. Your project's .hh files

Separate each non-empty group with one blank line.

With the preferred ordering, if the related header dir2/foo2.h omits any necessary includes, the build of dir/foo.cc or dir/foo_test.cc will break. Thus, this rule ensures that build breaks show up first for the people working on these files, not for innocent people in other packages.

dir/foo.cc and dir2/foo2.h are usually in the same directory (e.g., base/basictypes_test.cc and base/basictypes.h), but may sometimes be in different directories too.

Note that the C headers such as stddef.h are essentially interchangeable with their C++ counterparts (cstddef). Either style is acceptable, but prefer consistency with existing code.

Within each section the includes should be ordered alphabetically. Note that older code might not conform to this rule and should be fixed when convenient.

For example, the includes in google-awesome-project/src/foo/internal/fooserver.cc might look like this:

```cpp
#include "G4CMPFoo.hh"

#include "G4Bar.hh"

#include "CLHEP/Random/DoubConv.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "mydetector.hh"
```

**Exception**:

Sometimes, system-specific code needs conditional includes. Such code can put conditional includes after other includes. Of course, keep your system-specific code small and localized. Example:

```cpp
#include "foo/public/fooserver.h"

#include "base/port.h"  // For LANG_CXX11.

#ifdef LANG_CXX11
#include <initializer_list>
#endif  // LANG_CXX11
```


## Formatting <a name="formatting"></a>

### Line Length
80 characters is the maximum.

A line may exceed 80 characters if it is:

- a comment line which is not feasible to split without harming readability, ease of cut and paste or auto-linking -- e.g., if a line contains an example command or a literal URL longer than 80 characters.
- a string literal that cannot easily be wrapped at 80 columns. This may be because it contains URIs or other semantically-critical pieces, or because the literal contains an embedded language, or a multiline literal whose newlines are significant like help messages. In these cases, breaking up the literal would reduce readability, searchability, ability to click links, etc. Except for test code, such literals should appear at namespace scope near the top of a file. If a tool like Clang-Format doesn't recognize the unsplittable content, disable the tool around the content as necessary.

- an include statement.
- a header guard
- a using-declaration

### Spaces vs. Tabs
Use only spaces, and indent 2 spaces at a time.

We use spaces for indentation. Do not use tabs in your code. You should set your editor to emit spaces when you hit the tab key.

### Function Declarations and Definitions

Functions look like this:
```cpp
ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
  DoSomething();
  ...
}
```

If you have too much text to fit on one line:
```cpp
ReturnType ClassName::ReallyLongFunctionName(Type par_name1, Type par_name2,
                                             Type par_name3) {
  DoSomething();
  ...
}
```

or if you cannot fit even the first parameter:
```cpp
ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
    Type par_name1,  // 4 space indent
    Type par_name2,
    Type par_name3) {
  DoSomething();  // 2 space indent
  ...
}
```
Some points to note:

- Choose good parameter names.
- A parameter name may be omitted only if the parameter is not used in the function's definition.
- If you cannot fit the return type and the function name on a single line, break between them.
- If you break after the return type of a function declaration or definition, do not indent.
- The open parenthesis is always on the same line as the function name.
- There is never a space between the function name and the open parenthesis.
- There is never a space between the parentheses and the parameters.
- The open curly brace is always on the end of the last line of the function declaration, not the start of the next line.
The close curly brace is either on the last line by itself or on the same line as the open curly brace.
- There should be a space between the close parenthesis and the open curly brace.
- All parameters should be aligned if possible.
- Default indentation is 2 spaces.
- Wrapped parameters have a 4 space indent.

### Floating-point Literals
Floating-point literals should always have a radix point, with digits on both sides, even if they use exponential notation. Readability is improved if all floating-point literals take this familiar form, as this helps ensure that they are not mistaken for integer literals, and that the E/e of the exponential notation is not mistaken for a hexadecimal digit. It is fine to initialize a floating-point variable with an integer literal (assuming the variable type can exactly represent that integer), but note that a number in exponential notation is never an integer literal.
```cpp
float f = 1.0f;
float f2 = 1.0;  // Also OK
float f3 = 1;    // Also OK
long double ld = -0.5L;
double d = 1248.0e6;
```

```cpp
// BAD EXAMPLES DON'T DO THIS!!!
float f = 1.f;
long double ld = -.5L;
double d = 1248e6;
```

### Function Calls
Either write the call all on a single line, wrap the arguments at the parenthesis, or start the arguments on a new line indented by four spaces and continue at that 4 space indent. In the absence of other considerations, use the minimum number of lines, including placing multiple arguments on each line where appropriate.

Function calls have the following format:
```cpp
bool result = DoSomething(argument1, argument2, argument3);
```

```cpp
bool result = DoSomething(averyveryveryverylongargument1,
                          argument2, argument3);
```

```cpp
if (...) {
  ...
  ...
  if (...) {
    bool result = DoSomething(
        argument1, argument2,  // 4 space indent
        argument3, argument4);
    ...
  }
```
Put multiple arguments on a single line to reduce the number of lines necessary for calling a function unless there is a specific readability problem. Some find that formatting with strictly one argument on each line is more readable and simplifies editing of the arguments. However, we prioritize for the reader over the ease of editing arguments, and most readability problems are better addressed with the following techniques.

### Looping and branching statements
At a high level, looping or branching statements consist of the following components:

- One or more statement keywords (e.g. if, else, switch, while, do, or for).
- One condition or iteration specifier, inside parentheses.
- One or more controlled statements, or blocks of controlled statements.

For these statements:
- The components of the statement should be separated by single spaces (not line breaks).
- Inside the condition or iteration specifier, put one space (or a line break) between each semicolon and the next token, except if the token is a closing parenthesis or another semicolon.
- Inside the condition or iteration specifier, do not put a space after the opening parenthesis or before the closing parenthesis.
- Put any controlled statements inside blocks (i.e. use curly braces).
- Inside the controlled blocks, put one line break immediately after the opening brace, and one line break immediately before the closing brace.
```cpp
if (condition) {                   // Good - no spaces inside parentheses, space before brace.
  DoOneThing();                    // Good - two-space indent.
  DoAnotherThing();
} else if (int a = f(); a != 3) {  // Good - closing brace on new line, else on same line.
  DoAThirdThing(a);
} else {
  DoNothing();
}

// Good - the same rules apply to loops.
while (condition) {
  RepeatAThing();
}

// Good - the same rules apply to loops.
do {
  RepeatAThing();
} while (condition);

// Good - the same rules apply to loops.
for (int i = 0; i < 10; ++i) {
  RepeatAThing();
}
```

### Pointer and Reference Expressions

### Change tracking
  We have moved to git to track changes in the code.  It is no longer need to track this changes a the top of every file like we did in the bad old days.  However it is up the the developer to provide meaningful git commit messages. 

## Doxogen/Sphynix <a name="auto-docs"></a>
Look you doxogen hooks/format

## Cmake <a name="cmake"></a>
Probably don't need this.


## Notes <a name="notes"></a>
This guide was cherry-picked from the [Google C++](https://google.github.io/styleguide/cppguide.html) and [LLVM](https://llvm.org/docs/CodingStandards.html) style guides or taken from the choices of the orginal developers.