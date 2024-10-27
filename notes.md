# 1. Introduction

## Command Line Compilation
1. Preprocessor - Deal with #include, #define, etc directives
2. Compiler - Converts C++ source code into assembly (.s files)
3. Assembler - Turns assembled code into object code (.o files)
4. Linker - Object files are linked together to make an executable program

We will use g++ as our compiler. Basic usage:

```g++ main.cpp otherFile.cpp -o execFileName```

- ```-std=c++14```: Enable C++14 support
- ```-g```: Add debugging information to the output
- ```-Wall```: Turn on most compiler warnings

# 2. Structures

## When and why to use `auto`

When?
- You don't care about the exact type (iterators).
- When its type is clear from context (templates).
- When you can't figure out the type (lambdas).
- Avoid using auto for return values (exception: generic programming)

Why?
- Correctness: no implicit conversions, uninitialized variables.
- Flexibility: code easily modifiable if type changes need to be made.
- Powerful: very important when we get to templates!
- Modern IDE’s (eg. Qt Creator) can infer a type simply by hovering your cursor
over any auto, so readability not an issue!

## Unpack aggregate structures using structured binding
``` cpp
std::pair<std::pair<double, double>, bool> solve_quadratic(int a, int b, int c) {
    int D = b*b - 4*a*c;
    if (D >= 0) {
        double root1 = (-b + sqrt(D))/(2*a);
        double root2 = (-b - sqrt(D))/(2*a);
        return {{root1, root2}, true};
    } else {
        return {{0, 0}, false};
    }
}

int main() {
    int a = read_int("Type in a: ");
    int b = read_int("Type in b: ");
    int c = read_int("Type in c: ");
    auto [roots, found] = solve_quadratic(a, b, c);
    auto [root1, root2] = roots;
    if (found) {
        // if it's a double root (root1 == root2), this is fine as well
        std::cout << "The roots are: " << root1 << " and " << root2 << std::endl;
    } else {
        std::cout << "No roots found." << std::endl;
    }
    
}
```

# 3. References

## Challenge quiz
- In `"auto[i, s] = make_pair(3, "hi")"` the compiler will deduce that `s` is a `std::string`.

FALSE: A string literal is a C-string (`const char*`), and `auto` will deduce `s` is a C-string.

- Structured binding can unpack individual elements of a Stanford `Vector` (or `std::vector`).

FALSE:You can only know the number of elements of a vector at run-time.

- The line "auto i;" compiles.

FALSE: auto does not allow uninitialized variables (can't figure out the type)

## Recap auto and structures
Careful: `auto` discards `const` and `references`!

auto drops const/reference unless explicitly specified

## You cannot reassign a reference after construction
A reference remains an alias to whatever variable it was bound to.
``` cpp
    vector<int> original{1, 2};
    vector<int> copy = original;
    vector<int>& lref = original;
    original.push_back(3);
    copy.push_back(4);
    lref.push_back(5);
    // original (lref) = {1, 2, 3, 5}
    // copy = {1, 2, 4}
    lref = copy;
    copy.push_back(6);
    lref.push_back(7);
    // original = {1, 2, 4, 7}
    // copy = {1, 2, 4, 6}
```

Never return a reference to a local (automatic) variable, as they will go out of scope when you try to read/write to the reference.
```cpp
int& front(const std::string& file) {
    std::vector<int> vec = readFile(file);
    return vec[0];
}

int main() {
    front("text.txt") = 4; // undefined behavior
}
```

# 4. Streams

```cpp
#include <sstream> // for stringstream
#include <iostream> // for cout

using namespace std;

int main() {
    ostringstream oss("Ito En Green Tea ");
    // ostringstream oss("Ito En Green Tea ", stringstream::ate);
    cout << oss.str() << endl;
    // Ito En Green Tea
    
    oss << 16.9 << " Ounce ";
    cout << oss.str() << endl;
    // 16.9 Ounce n Tea

    oss << "(Pack of " << 12 << ")";
    cout << oss.str() << endl;
    // 16.9 Ounce (Pack of 12)

    return 0;
}
```

# 6. Iterators

Some operations on some containers may invalidate iterators!!!

# 7. Templates

## Variadic Templates

Varadic Templates can use compile-time recursion.

`f(h(args...) + args...);`

expands to

`f(h(E1,E2,E3) + E1, h(E1,E2,E3) + E2, h(E1,E2,E3) + E3)`

```cpp
template <typename T, typename... Ts>
auto my_min(T num, Ts... args) {
  auto min = my_min(args...);
  if (num < min) min = num;
  return min;
}

template <typename T>
auto my_min(T num) {
    return num;
}
```

## Function objects("Functors")

```cpp
class GreaterThan{
    public:
        GreaterThan(int limit) : limit(limit) {}
        bool operator() {int val} {return val >= limit};
    private:
        int limit;
}
```

Key idea: create an object which can act like a function since it has an () operator.

## Lambda functions

```cpp
int main() {
    vector<int> vec{1, 3, 5, 7, 9};
    int limit = 5;
    auto is_less_than_limit = [limit](auto val) {
        return val < limit;
    }
    count_occurences(vec.begin(), vec.end(), is_less_than_limit);
    return 0;
}
```

- 1st `auto`: don't know the type, ask the compiler
- `limit`: capture clause, gives access to outside variables
- 2nd `auto`: parameter list, can use `auto`
- `-> bool`: return type, optional

Accessible variables inside lambda limited to capture clause and parameter list.
```cpp
auto is_less_than_limit = [limit](auto val) -> bool {
    return val < limit;
}
```

You can also capture by reference.
```cpp
set<string> teas{“black”, “green”, “oolong”};
string banned = “boba”; // pls … this is not a tea
auto liked_by_Avery = [&teas, banned](auto type) {
    return teas.count(type) && type != banned;
};
```

You can also capture everything by value or reference.
```cpp
// capture all by value, except teas is by reference
auto func1 = [=, &teas](parameters) -> return_value {
    // body
};

// capture all by reference, except banned is by value
auto func2 = [&, banned](parameters) -> return_value {
    // body
};
```

- Lambdas are function objects that can capture variables that are not parameter.
- Lambdas can be passed into template functions as a predicate.

## SFINAE
- Substitution Failure Is Not An Error
- When substituting the deduced types fails (in the immediate
context) because the type doesn't satisfy implicit interfaces, this
does not result in a compile error.
- Instead, this candidate function is not part of the viable set.
The other candidates will still be processed.

Very common SFINAE template: use this overload if [expression] compiles
```cpp
template <typename T>
auto function(const T& a)
        -> decltype((void) [expression], [return type]()) {
    // function implementation
}
```

This template is valid if a.size() compiles.
```cpp
template <typename T>
auto print_size(const T& a)
        -> decltype((void) a.size(), size_t()) {
    cout << “printing with size member function: ”;
    cout << a.size() << endl;
    return a.size();
}
// T = int (fail)
// T = vector<int> (success)
// T = vector<int>* (fail)
```

# 8. Functions and Algorithms

- We need a special iterator `back_inserter()` which extends the container.
```cpp
string dep = ”CS”;
auto isDep = [dep](const auto& course) {
    return course.name.size() >= dep.size &&
            course.substr(0, dep.size()) == dep;
};

std::copy_if(csCourses.begin(), csCourses.end(),
        back_inserter(csCourses), isDep);
```

- Challenge Problem: Implement the logic of remove from before!
```cpp
template <typename ForwardIt, typename T>
ForwardIt remove(ForwardIt first, ForwardIt last,
                    const T& value) {
    first = std::find(first, last, value);
    if (first != last)
        for(ForwardIt i = first; ++i != last; )
            if (!(*i == value))
                *first++ = std::move(*i);
    return first;
}
```

- but `std::remove` does not change the size of the container!

erase-remove idiom
```cpp
v.erase(
    std::remove_if(v.begin(), v.end(), pred),
    v.end()
);
```

- Stream iterators read from istreams or write to ostreams!
```cpp
std::cout << "odd numbers in to_vector are: ";
std::copy_if(to_vector.begin(), to_vector.end(),
            std::ostream_iterator<int>(std::cout, " "),
            [](int x) { return x % 2 != 0; });
std::cout << '\n';
```

- [`std::bind`](https://en.cppreference.com/w/cpp/utility/functional/bind)

# 9. STL Summary

# 10. Classes and Const Correctness

- const reference = a reference that cannot be used to
modify the object that is being referenced
- const method = a method of a class that can't change any
class variables and can only call other const methods
- const object = an object declared as const that can only call
its const methods

## Challenge

`const int* const myClassMethod(const int* const &param) const;`

(my solution...)
- myClassMethod is a const method
- it takes a const reference `param` to a pointer which points to a const int
- it returns a const pointer which points to a const int

# 11. Operators

## General rule of thumb: member vs. non-member

1. Some operators must be implemented as members ( eg. `[]`, `()`, `->`, `=` ) due to C++ semantics.
2. Some must be implemented as non-members ( eg. `<<`, if you are writing class for rhs, not lhs).
3. If unary operator ( eg. `++` ), implement as member.
4. If binary operator and treats both operands equally (eg. both unchanged) implement as non-member (maybe `friend`). Examples: `+`, `<`.
5. If binary operator and not both equally (changes lhs), implement as member (allows easy access to lhs private members). Examples: `+=`.

## Principle of Least Astonishment (POLA)

1. Design operators primarily to mimic conventional usage.
2. Use nonmember functions for symmetric operators.
3. Always provide all out of a set of related operators.

# 12. Special Member Functions

## Prefix vs. Postfix

```cpp
iterator& iterator::operator++(); // prefix

iterator iterator::operator++(int); // postfix
```

## What are the things you have to do in an assignment operator?

1. Check for self-assignment.
2. Make sure you to when to free existing members.
3. Copy assign each automatically assignable member.
4. Manually copy all other members.
5. Return a reference to itself (that was just reassigned).

## You can prevent copies from being made by explicitly deleting these operations.

```cpp
class PasswordManager {
public:
    PasswordManager();
    ~PasswordManager();
    // other methods
    PasswordManager(const PasswordManager& rhs) = delete;
    PasswordManager& operator=(const PasswordManager& rhs) = delete;
private:
    // other stuff
}
```

## rule of zero / rule of three

If the default copy constructor, assignment, and destructor work, then use the default ones and don't declare your own.

If you explicitly define (or delete) a copy constructor, copy assignment, or destructor, you should define (or delete) all three.

## copy elision and return value optimization (RVO)

Copy elision is an optimization that the compiler makes to skip unnecessary copies.

Copy elision is guaranteed by compiler in C++17.

# 13. Move Semantics

