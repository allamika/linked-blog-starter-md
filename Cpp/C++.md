Properties:
- _Configuration_ is set to “All Configurations”
- click _C/C++ > Language tab_, and set _Conformance mode_ to _Yes (/permissive-)_
- _C/C++ > General tab_ and set _Warning level_ to _Level4 (/W4)_
- From _C/C++ > External Includes tab_, set _External Header Warning Level_ to _Level3 (/external:W3)_
- Configuration Properties > C/C++ > Language > Language Standard > ISO C++ Latest 


### Initialisation

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
-  