Properties:
- _Configuration_ is set to “All Configurations”
- click _C/C++ > Language tab_, and set _Conformance mode_ to _Yes (/permissive-)_
- _C/C++ > General tab_ and set _Warning level_ to _Level4 (/W4)_
- From _C/C++ > External Includes tab_, set _External Header Warning Level_ to _Level3 (/external:W3)_
- Configuration Properties > C/C++ > Language > Language Standard > ISO C++ Latest 


#### Initialisation

```cpp
int a;         // default-initialization (no initializer)

// Traditional initialization forms:
int b = 5;     // copy-initialization (initial value after equals sign)
int c ( 6 );   // direct-initialization (initial value in parenthesis)

// Modern initialization forms (preferred):
int d { 7 };   // direct-list-initialization (initial value in braces)
int e {};      // value-initialization (empty braces)

- Default-initialization => garbage value
- In most cases, value-initialization will implicitly initialize the variable to zero (or whatever value is closest to zero for a given type)
- List-initialization is the preferred form of initialization in modern C++
- Q: When should I initialize with { 0 } vs {}?

Use direct-list-initialization when you’re actually using the initial value:

```cpp
int x { 0 };    // direct-list-initialization with initial value 0
std::cout << x; // we're using that 0 value here
```

- Default-initialization => garbage value
- In most cases, value-initialization will implicitly initialize the variable to zero (or whatever value is closest to zero for a given type)
- List-initialization is the preferred form of initialization in modern C++

#### OIStream

##### `std::cout` is buffered
Consider a rollercoaster ride at your favorite amusement park. Passengers show up (at some variable rate) and get in line. Periodically, a train arrives and boards passengers (up to the maximum capacity of the train). When the train is full, or when enough time has passed, the train departs with a batch of passengers, and the ride commences. Any passengers unable to board the current train wait for the next one.
This analogy is similar to how output sent to `std::cout` is typically processed in C++. Statements in our program request that output be sent to the console. However, that output is typically not sent to the console immediately. Instead, the requested output “gets in line”, and is stored in a region of memory set aside to collect such requests (called a **buffer**). Periodically, the buffer is **flushed**, meaning all of the data collected in the buffer is transferred to its destination (in this case, the console).

##### `std::endl` vs `\n`:
- `std::endl`: move cursor to a new line + flush the output buffer (slow)
- `\n` : move cursor to a new line
- Best practice: Prefer `\n` over `std::endl` when outputting text to the console.

##### `std::cin` is buffered
Similarly, inputting data is also a two stage process:
- The individual characters you enter as input are added to the end of an input buffer (inside `std::cin`). The enter key (pressed to submit the data) is also stored as a `'\n'` character.
- The extraction operator ‘>>’ removes characters from the front of the input buffer and converts them into a value that is assigned (via copy-assignment) to the associated variable. This variable can then be used in subsequent statements

##### The basic extraction process [](https://www.learncpp.com/cpp-tutorial/introduction-to-iostream-cout-cin-and-endl/#extraction)
Here’s a simplified view of how operator `>>` works for input.
1. If `std::cin` is not in a good state (e.g. the prior extraction failed and `std::cin` has not yet been cleared), no extraction is attempted, and the extraction process aborts immediately.
2. Leading whitespace characters (spaces, tabs, and newlines at the front of the buffer) are discarded from the input buffer. This will discard an unextracted newline character remaining from a prior line of input.
3. If the input buffer is now empty, operator `>>` will wait for the user to enter more data. Any leading whitespace is discarded from the entered data.
4. operator `>>` then extracts as many consecutive characters as it can, until it encounters either a newline character (representing the end of the line of input) or a character that is not valid for the variable being extracted to.
The result of the extraction process is as follows:
- If the extraction aborted in step 1, then no extraction attempt occurred. Nothing else happens.
- If any characters were extracted in step 4 above, extraction is a success. The extracted characters are converted into a value that is then copy-assigned to the variable.
- If no characters could be extracted in step 4 above, extraction has failed. The object being extracted to is copy-assigned the value `0` (as of C++11), and any future extractions will immediately fail (until `std::cin` is cleared).
Any non-extracted characters (including newlines) remain available for the next extraction attempt.

#### Uninitialized variables and undefined behavior

Recap:
- Initialized = The object is given a known value at the point of definition.
- Assignment = The object is given a known value beyond the point of definition.
- Uninitialized = The object has not been given a known value yet.

#### Base Concept
A **statement** is a type of instruction that causes the program to perform some action. Statements are often terminated by a semicolon.

A **function** is a collection of statements that execute sequentially. Every C++ program must include a special function named _main_. When you run your program, execution starts at the top of the _main_ function.

In programming, the name of a function (or object, type, template, etc…) is called its **identifier**.

The rules that govern how elements of the C++ language are constructed is called **syntax**. A **syntax error** occurs when you violate the grammatical rules of the language.

**Comments** allow the programmer to leave notes in the code. C++ supports two types of comments. Line comments start with a `//` and run to the end of the line. Block comments start with a `/*` and go to the paired `*/` symbol. Don’t nest block comments.

You can use comments to temporarily disable lines or sections of code. This is called commenting out your code.

**Data** is any information that can be moved, processed, or stored by a computer. A single piece of data is called a **value**. Common examples of values include letters (e.g. `a`), numbers (e.g. `5`), and text (e.g. `Hello`).

A variable is a named piece of memory that we can use to store values. In order to create a variable, we use a statement called a **definition statement**. When the program is run, each defined variable is **instantiated**, which means it is assigned a memory address.

A **data type** tells the compiler how to interpret a piece of data into a meaningful value. An **integer** is a number that can be written without a fractional component, such as 4, 27, 0, -2, or -12.

**Copy assignment** (via operator=) can be used to assign an already created variable a value.

The process of specifying an initial value for an object is called **initialization**, and the syntax used to initialize an object is called an **initializer**.

#### “Out of scope” vs “going out of scope”

The terms “out of scope” and “going out of scope” can be confusing to new programmers.

An identifier is out of scope anywhere it cannot be accessed within the code. In the example above, the identifier `x` is in scope from its point of definition to the end of the `main` function. The identifier `x` is out of scope outside of that code region.

The term “going out of scope” is typically applied to objects rather than identifiers. We say an object goes out of scope at the end of the scope (the end curly brace) in which the object was instantiated. In the example above, the object named `x` goes out of scope at the end of the function `main`.

A local variable’s lifetime ends at the point where it goes out of scope, so local variables are destroyed at this point.

Note that not all types of variables are destroyed when they go out of scope. We’ll see examples of these in future lessons.

Remember, lifetime is a runtime property, and scope is a compile-time property.

#### Forward declaration

We can also fix this by using a forward declaration.

A **forward declaration** allows us to tell the compiler about the existence of an identifier _before_ actually defining the identifier.

In the case of functions, this allows us to tell the compiler about the existence of a function before we define the function’s body. This way, when the compiler encounters a call to the function, it’ll understand that we’re making a function call, and can check to ensure we’re calling the function correctly, even if it doesn’t yet know how or where the function is defined.

To write a forward declaration for a function, we use a **function declaration** statement (also called a **function prototype**). The function declaration consists of the function’s return type, name, and parameter types, terminated with a semicolon. The names of the parameters can be optionally included. The function body is not included in the declaration.

#### The global namespace

In C++, any name that is not defined inside a class, function, or a namespace is considered to be part of an implicitly-defined namespace called the **global namespace** (sometimes also called **the global scope**).

