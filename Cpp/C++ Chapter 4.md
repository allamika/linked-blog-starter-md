 
#### Introduction to fundamental data types

|Types|Category|Meaning|Example|
|---|---|---|---|
|float  <br>double  <br>long double|Floating Point|a number with a fractional part|3.14159|
|bool|Integral (Boolean)|true or false|true|
|char  <br>wchar_t  <br>char8_t (C++20)  <br>char16_t (C++11)  <br>char32_t (C++11)|Integral (Character)|a single character of text|‘c’|
|short int  <br>int  <br>long int  <br>long long int (C++11)|Integral (Integer)|positive and negative whole numbers, including 0|64|
|std::nullptr_t (C++11)|Null Pointer|a null pointer|nullptr|
|void|Void|no type|n/a|
C++ contains three sets of types.

The first two are built-in to the language itself (and do not require the inclusion of a header to use):

- The “fundamental data types” provide the most the basic and essential data types.
- The “compound data types” provide more complex data types and allow for the creation of custom (user-defined) types
- The third (and largest) set of types is provided by the C++ standard library


#### Chars

| Name            | Symbol     | Meaning                                                                                |
| --------------- | ---------- | -------------------------------------------------------------------------------------- |
| Alert           | \a         | Makes an alert, such as a beep                                                         |
| Backspace       | \b         | Moves the cursor back one space                                                        |
| Formfeed        | \f         | Moves the cursor to next logical page                                                  |
| Newline         | \n         | Moves cursor to next line                                                              |
| Carriage return | \r         | Moves cursor to beginning of line                                                      |
| Horizontal tab  | \t         | Prints a horizontal tab                                                                |
| Vertical tab    | \v         | Prints a vertical tab                                                                  |
| Single quote    | \’         | Prints a single quote                                                                  |
| Double quote    | \”         | Prints a double quote                                                                  |
| Backslash       | \\         | Prints a backslash.                                                                    |
| Question mark   | \?         | Prints a question mark.  <br>No longer relevant. You can use question marks unescaped. |
| Octal number    | \(number)  | Translates into char represented by octal                                              |
| Hex number      | \x(number) | Translates into char represented by hex number                                         |

Best practice: Prefer `\n` over `std::endl` when outputting text to the console.

#### Summary

Chapter Review

The smallest unit of memory is a **binary digit**, also called a **bit**. The smallest unit amount of memory that can be addressed (accessed) directly is a **byte**. The modern standard is that a byte equals 8 bits.

A **data type** tells the compiler how to interpret the contents of memory in some meaningful way.

C++ comes with support for many fundamental data types, including floating point numbers, integers, boolean, chars, null pointers, and void.

**Void** is used to indicate no type. It is primarily used to indicate that a function does not return a value.

Different types take different amounts of memory, and the amount of memory used may vary by machine.

The **sizeof** operator can be used to return the size of a type in bytes.

**Signed integers** are used for holding positive and negative whole numbers, including 0. The set of values that a specific data type can hold is called its **range**. When using integers, keep an eye out for overflow and integer division problems.

**Unsigned integers** only hold positive numbers (and 0), and should generally be avoided unless you’re doing bit-level manipulation.

**Fixed-width integers** are integers with guaranteed sizes, but they may not exist on all architectures. The fast and least integers are the fastest and smallest integers that are at least some size. `std::int8_t` and `std::uint8_t` should generally be avoided, as they tend to behave like chars instead of integers.

**size_t** is an unsigned integral type that is used to represent the size or length of objects.

**Scientific notation** is a shorthand way of writing lengthy numbers. C++ supports scientific notation in conjunction with floating point numbers. The digits in the significand (the part before the e) are called the **significant digits**.

**Floating point** is a set of types designed to hold real numbers (including those with a fractional component). The **precision** of a number defines how many significant digits it can represent without information loss. A **rounding error** can occur when too many significant digits are stored in a floating point number that can’t hold that much precision. Rounding errors happen all the time, even with simple numbers such as 0.1. Because of this, you shouldn’t compare floating point numbers directly.

The **Boolean** type is used to store a `true` or `false` value.

**If statements** allow us to execute one or more lines of code if some condition is true. The conditional expression of an if-statement is interpreted as a boolean value. An **else statement** can be used to execute a statement when a prior if-statement condition evaluates to false.

**Char** is used to store values that are interpreted as an ASCII character. When using chars, be careful not to mix up ASCII code values and numbers. Printing a char as an integer value requires use of `static_cast.`

Angled brackets are typically used in C++ to represent something that needs a parameterizable type. This is used with `static_cast` to determine what data type the argument should be converted to (e.g. `static_cast<int>(x)` will return the value of `x` as an `int`).