# Style Guide

# Table of Contents
1. [Naming](#naming)
1. [Header Files](#headers)
1. [Formatting](#formatting)
1. [Comments](#comments)
1. [Units](#units)
1. [CMake File](#cmake)
1. [Legacy Code](#legacycode)
1. [Notes](#notes)

## Naming <a name="naming"></a>
### Classes
Classes that are part of the main libary should use our modified UpperCamelCase style of "G4CMPFooBar"*.  

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

You can declare functions in a way that allows the compiler to expand them inline rather than calling them through the usual function call mechanism.

Inlining a function can generate more efficient object code, as long as the inlined function is small. Feel free to inline accessors and mutators, and other short, performance-critical functions.
 Overuse of inlining can actually make programs slower. Depending on a function's size, inlining it can cause the code size to increase or decrease.

A decent rule of thumb is to not inline a function if it is more than 10 lines long. Beware of destructors, which are often longer than they appear because of implicit member- and base-destructor calls!

Another useful rule of thumb: it's typically not cost effective to inline functions with loops or switch statements (unless, in the common case, the loop or switch statement is never executed).

### Names and Order of Includes
Include headers in the following order: G4CMP headers, G4 headers, other libraries' headers (e.g. CLHEP), C system headers, C++ standard library headers, then your project's headers.

Headers should only be included using an angle-bracketed path if the library requires you to do so. In particular, the following headers require angle brackets:

- C and C++ standard library headers (e.g. <stdlib.h> and <string>).
- POSIX, Linux, and Windows system headers (e.g. <unistd.h> and <windows.h>).
- In rare cases, third_party libraries (e.g. <Python.h>).

In MyFoo.cc, whose main purpose is to implement or test the stuff in G4CMPFoo2.hh, order your includes as follows:

1. G4CMPFoo2.hh
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

With the preferred ordering, if the related header G4CMPFoo2.hh omits any necessary includes, the build of MyFoo.cc will break. Thus, this rule ensures that build breaks show up first for the people working on these files, not for innocent people in other packages.

Note that the C headers such as stddef.h are essentially interchangeable with their C++ counterparts (cstddef). Either style is acceptable, but prefer consistency with existing code.

Within each section the includes should be ordered alphabetically. Note that older code might not conform to this rule and should be fixed when convenient.

For the above example, the includes might look like this:

```cpp
#include "G4CMPFoo2.hh"

#include "G4Bar.hh"

#include "CLHEP/Random/DoubConv.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "MyFoo.hh"
```

**Exception**:

Sometimes, system-specific code needs conditional includes. Such code can put conditional includes after other includes. Of course, keep your system-specific code small and localized.  This should be very rare in our case.

Example:

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

- A comment line which is not feasible to split without harming readability, ease of cut and paste or auto-linking -- e.g., if a line contains an example command or a literal URL longer than 80 characters.
- A string literal that cannot easily be wrapped at 80 columns. This may be because it contains URIs or other semantically-critical pieces, or because the literal contains an embedded language, or a multiline literal whose newlines are significant like help messages. In these cases, breaking up the literal would reduce readability, searchability, ability to click links, etc.  This should be very rare for our use case.

- An include statement
- A header guard
- A using-declaration

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

We allow one exception to the above rules: the line breaks inside the curly braces may be omitted if as a result the entire statement appears on a single line.
```cpp
// OK - fits on one line.
if (x == kFoo) { return new Foo(); }
```


### Pointer and Reference Expressions
No spaces around period or arrow. Pointer operators do not have trailing spaces.

The following are examples of correctly-formatted pointer and reference expressions:
```cpp
x = *p;
p = &x;
x = r.y;
x = r->y;
```

When referring to a pointer or reference (variable declarations or definitions, arguments, return types, template parameters, etc), you may place the space before or after the asterisk/ampersand. In the trailing-space style, the space is elided in some cases (template parameters, etc).
```cpp
// These are fine, space preceding.
char *c;
const std::string &str;
int *GetPointer();
std::vector<char *>

// These are fine, space following (or elided).
char* c;
const std::string& str;
int* GetPointer();
std::vector<char*>  // Note no space between '*' and '>'
```
You should do this consistently within a single file. When modifying an existing file, use the style in that file.


### Boolean Expressions
When you have a boolean expression that is longer than the standard line length, be consistent in how you break up the lines.

In this example, the logical AND operator is always at the end of the lines:
```cpp
if (this_one_thing > this_other_thing &&
    a_third_thing == a_fourth_thing &&
    yet_another && last_one) {
  ...
}
```
Note that when the code wraps in this example, both of the && logical AND operators are at the end of the line. This is more common, though wrapping all operators at the beginning of the line is also allowed (This decision should be consistent within the file). 

Feel free to insert extra parentheses judiciously because they can be very helpful in increasing readability when used appropriately, but be careful about overuse. Also note that you should always use the punctuation operators, such as && and ~, rather than the word operators, such as and and compl.

### Preprocessor Directives

The hash mark that starts a preprocessor directive should always be at the beginning of the line.

Even when preprocessor directives are within the body of indented code, the directives should start at the beginning of the line.  

```cpp
// Good - directives at beginning of line
  if (lopsided_score) {
#if DISASTER_PENDING      // Correct -- Starts at beginning of line
    DropEverything();
# if NOTIFY               // OK but not required -- Spaces after #
    NotifyClient();
# endif
#endif
    BackToNormal();
  }
```

### Class Format

Sections in ```public```, ```protected``` and ```private``` are not indented.

The basic format for a class definition is:
```cpp
class MyClass : public OtherClass {
public:      
  MyClass();  // 2 space indent.
  explicit MyClass(int var);
  ~MyClass() {}

  void SomeFunction();
  void SomeFunctionThatDoesNothing() {
  }

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

private:
  bool SomeInternalFunction();

  int some_var_;
  int some_other_var_;
};
```
Things to note:

- Any base class name should be on the same line as the subclass name, subject to the 80-column limit.
- The ```public:```, ```protected:```, and ```private:``` keywords should not be indented.
- Except for the first instance, these keywords should be preceded by a blank line. This rule is optional in small classes.
- Do not leave a blank line after these keywords.
- The public section should be first, followed by the protected and finally the private section.

### Namespace Formatting
Namespaces do not add an extra level of indentation. For example, use:
```cpp
namespace {

void foo() {  // Correct.  No extra indentation within namespace.
  ...
}

}  // namespace
```

### Horizontal Whitespace

#### general
```cpp
int i = 0;  // Two spaces before end-of-line comments.

void f(bool b) {  // Open braces should always have a space before them.
  ...
int i = 0;  // Semicolons usually have no space before them.
// Spaces inside braces for braced-init-list are optional.  If you use them,
// put them on both sides!
int x[] = { 0 };
int x[] = {0};

// Spaces around the colon in inheritance and initializer lists.
class Foo : public Bar {
public:
  // For inline function implementations, put spaces between the braces
  // and the implementation itself.
  Foo(int b) : Bar(), baz_(b) {}  // No spaces inside empty braces.
  void Reset() { baz_ = 0; }  // Spaces separating braces from implementation.
  ...
```
#### loops and conditionals
```cpp
if (b) {          // Space after the keyword in conditions and loops.
} else {          // Spaces around else.
}
while (test) {}   // There is usually no space inside parentheses.
switch (i) {
for (int i = 0; i < 5; ++i) {
// Loops and conditions may have spaces inside parentheses, but this
// is rare.  Be consistent.
switch ( i ) {
if ( test ) {
for ( int i = 0; i < 5; ++i ) {
// For loops always have a space after the semicolon.  They may have a space
// before the semicolon, but this is rare.
for ( ; i < 5 ; ++i) {
  ...

// Range-based for loops always have a space before and after the colon.
for (auto x : counts) {
  ...
}
switch (i) {
  case 1:         // No space before colon in a switch case.
    ...
  case 2: break;  // Use a space after a colon if there's code after it.
```
#### operators
```cpp
// Assignment operators always have spaces around them.
x = 0;

// Other binary operators usually have spaces around them, but it's
// OK to remove spaces around factors.  Parentheses should have no
// internal padding.
v = w * x + y / z;
v = w*x + y/z;
v = w * (x + z);

// No spaces separating unary operators and their arguments.
x = -5;
++x;
if (x && !y)
  ...
```
#### templates and casts 
```cpp
// No spaces inside the angle brackets (< and >), before
// <, or between >( in a cast
std::vector<std::string> x;
y = static_cast<char*>(x);

// Spaces between type and pointer are OK, but be consistent.
std::vector<char *> x;
```

### Vertical Whitespace

The basic principle is: The more code that fits on one screen, the easier it is to follow and understand the control flow of the program. Use whitespace purposefully to provide separation in that flow.

Don't use blank lines when you don't have to. In particular, don't put more than one or two blank lines between functions, resist starting functions with a blank line, don't end functions with a blank line, and be sparing with your use of blank lines. A blank line within a block of code serves like a paragraph break in prose: visually separating two thoughts.

Some rules of thumb to help when blank lines may be useful:

- Blank lines at the beginning or end of a function do not help readability.
- Blank lines inside a chain of if-else blocks may well help readability.
- A blank line before a comment line usually helps readability — the introduction of a new comment suggests the start of a new thought, and the blank line makes it clear that the comment goes with the following thing instead of the preceding.
- Blank lines immediately inside a declaration of a namespace or block of namespaces may help readability by visually separating the load-bearing content from the (largely non-semantic) organizational wrapper. Especially when the first declaration inside the namespace(s) is preceded by a comment, this becomes a special case of the previous rule, helping the comment to "attach" to the subsequent declaration.

## Comments <a name="comments"></a>
### Change tracking
We have moved to git to track changes in the code.  It is no longer need to track changes at the top of every file like we did in the bad old days.  

It is the responcibility of the developer to provide meaningful git commit messages and PR descriptions. 

### Doxogen/Sphynix 
Look you doxogen hooks/format

## Units  <a name="units"></a>
Need to says something here about CGS and CLHEP units.  The native unit of Geant4 are cm, g, and s.  Often times these are less than convient for our development work.  It is strongly encouraged to use CLHEP units to explicitly define your units.

```cpp
#include "CLHEP/Units/SystemOfUnits.h"

// unit examples
double foo_microns = 100 * CLHEP::um;
double foo_mm = 200 * CLHEP::mm;
...
```


## Cmake <a name="cmake"></a>
Probably don't need this.

## Legacy Code (non-conformant code) <a name="legacycode"></a>

G4CMP was inially developed without the above guide.  While there was there was some effort to include the style decision of the orginal developers in this guide it is expected that portions of this code base are non-conformant.  

While maintaining this legacy code, judicious use of reformating may be appropriate for easy of readability and maintance in the future...

...but, **when in doubt leave it alone!** 

## Notes <a name="notes"></a>
This guide was cherry-picked from the [Google C++](https://google.github.io/styleguide/cppguide.html) and [LLVM](https://llvm.org/docs/CodingStandards.html) style guides and much of it is copied directly so credit is given to them.  Effort was made to merge with the choices of the orginal developers where appropriate. 