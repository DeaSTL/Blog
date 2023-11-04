# Embracing C++20: Debunking the Unwarranted Hate

In the vast realm of programming languages, C++ has often been met with mixed emotions. Some developers swear by its power and performance, while others bemoan its complexity and steep learning curve. However, the hate towards C++ is often unwarranted, especially with the advent of C++20, which brings a slew of modern features and improvements to the table. In this article, we'll explore why you should consider using C++20 for your next project and how managing dependencies, once a pain point, has become more accessible.

### The New Face of C++: Standard 20
While Standard 23 is in the works, it may be a while before we see tooling that works around c++23 unless you're a *visual studio enjoyer*.

the latest iteration of the language, has breathed new life into C++. It introduces features that simplify code, enhance productivity, and improve readability. Let's delve into a few examples to highlight the benefits of modern C++20.

### Simplified Range-Based Loops

```cpp
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // Old way
    for (auto it = numbers.begin(); it != numbers.end(); ++it) {
        std::cout << *it << " ";
    }
    
    // New way (C++20)
    for (const auto& num : numbers) {
        std::cout << num << " ";
    }
    
    return 0;
}
```

C++20 introduces concise range-based loops, making code more readable and eliminating the need for iterator boilerplate.

### Concepts for Type Safety

```cpp

template <typename T>
concept Numeric = std::is_arithmetic_v<T>;

template <Numeric T>
T add(T a, T b) {
    return a + b;
}
```

### Dependency Management Made Easy
One of the concerns that newer programmers may have is managing dependencies in C++. However, this challenge has significantly diminished in recent times. Tools like Conan and Vcpkg have emerged to simplify dependency management.

### Conan: The C/C++ Package Manager
Conan is a powerful C/C++ package manager that allows you to easily manage and share libraries. With a simple conan install command, you can fetch and integrate dependencies into your project.

```bash
$ conan install boost/1.75.0
```

### Vcpkg: The C++ Library Manager
Vcpkg is another fantastic option for managing C++ libraries. It offers a vast collection of pre-built packages that can be installed with a single command.

```bash
$ vcpkg install openssl
```
These tools take the headache out of dependency management, making it a breeze for C++ developers.

### Setting Up GCC 13 for C++20
While C++20 brings many benefits, setting up the latest compiler, like GCC 13, can be a bit challenging, especially if it's not available in your package manager. However, rest assured that as C++20 gains popularity, it's likely to become more readily available in mainstream package managers. Until then, compiling it from scratch is a viable option for those eager to embrace the latest features.

### Conclusion
In conclusion, the hate towards C++ is often misdirected towards the language itself when, in reality, it's more about the tooling surrounding it. With C++20 and improved dependency management tools like Conan and Vcpkg, C++ development is becoming more accessible and enjoyable. The modern features of C++20, combined with simplified dependency management, make C++ a strong contender for your next project. So, don't dismiss C++ without giving it a chance, as it may just be the right tool for the job. Happy coding!

