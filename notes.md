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
