<!---
{
  "id": "3aed249f-57d0-48f7-88c5-d97714d1caa7",
  "teaches": "Cpp: Associative Containers, Queues and Choosing a Container",
  "depends_on": [
    "f5737440-0818-442e-8330-396eb85df159"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2026-10-29",
  "keywords": [
    "C++",
    "std::map",
    "std::set",
    "std::unordered_map",
    "std::queue",
    "std::stack",
    "complexity",
    "std::chrono",
    "g++"
  ]
}
--->

# Cpp: Associative Containers, Queues and Choosing a Container

> In this exercise you store key-value pairs in `std::map`, remove duplicates with `std::set`, process work in order with `std::queue` and `std::stack`, and measure for yourself which container is the right one for a lookup task. No `sudo` is required.

## Introduction

Sequences like `std::vector` are searched by position. Often you want to search by a **key**: the price of an article, the number of a user. The associative containers do this: `std::map` stores key-value pairs sorted by key and finds an entry in logarithmic time, `std::set` stores unique keys only, and their `unordered_` variants use hashing and find an entry in constant time on average, with no order.

The container adapters `std::queue` (first in, first out) and `std::stack` (last in, first out) restrict an underlying container to a narrow interface, which makes the intent of your code clear.

### 1.1) Further Readings and Other Sources

* [cppreference: std::map](https://en.cppreference.com/w/cpp/container/map)
* [cppreference: std::unordered_map](https://en.cppreference.com/w/cpp/container/unordered_map)
* [cppreference: Containers library](https://en.cppreference.com/w/cpp/container)

## Tasks

### Task 1: A map

```bash
mkdir -p ~/cpp-assoc && cd ~/cpp-assoc
```

```c++
// file: map.cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> stock;
    stock["tesla"] = 3;
    stock["vw"]    = 12;
    stock["benz"]  = 7;

    for (const auto &[name, count] : stock)
        std::cout << name << " = " << count << std::endl;

    auto it = stock.find("vw");
    if (it != stock.end())
        std::cout << "vw: " << it->second << std::endl;
    std::cout << "bmw present: " << stock.count("bmw") << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra map.cpp -o map && ./map
```

The output is sorted by key although you inserted in another order. Each element is a `std::pair` with `first` (key) and `second` (value). `find` returns an iterator, exactly like the algorithms from the iterator exercise.

### Task 2: The trap in `operator[]`

```c++
// file: trap.cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> stock{{"vw", 12}};

    std::cout << "bmw: " << stock["bmw"] << std::endl;
    std::cout << "size: " << stock.size() << std::endl;

    try {
        std::cout << stock.at("audi") << std::endl;
    } catch (const std::out_of_range &) {
        std::cout << "audi not found" << std::endl;
    }
    std::cout << "size: " << stock.size() << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra trap.cpp -o trap && ./trap
```

Reading with `[]` **inserts** a default-constructed value for a missing key. `at()` does not. Use `find` or `count` to ask, `[]` only to write.

### Task 3: A set removes duplicates

```c++
// file: set.cpp
#include <iostream>
#include <set>
#include <vector>

int main() {
    std::vector<int> v{5, 3, 5, 1, 3, 3, 9};
    std::set<int> unique(v.begin(), v.end());

    for (int x : unique) std::cout << x << " ";
    std::cout << std::endl << unique.size() << " unique of " << v.size() << std::endl;
    std::cout << "contains 9: " << unique.count(9) << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra set.cpp -o set && ./set
```

The constructor takes an iterator range, again the pair `begin` and `end`. The set is sorted as a side effect.

### Task 4: Sorted against unordered

```c++
// file: unordered.cpp
#include <iostream>
#include <map>
#include <string>
#include <unordered_map>

int main() {
    std::map<std::string, int>           sorted;
    std::unordered_map<std::string, int> hashed;
    for (std::string s : {"tesla", "vw", "benz", "audi", "bmw"}) {
        sorted[s] = (int)s.size();
        hashed[s] = (int)s.size();
    }
    std::cout << "map:           ";
    for (const auto &p : sorted) std::cout << p.first << " ";
    std::cout << std::endl << "unordered_map: ";
    for (const auto &p : hashed) std::cout << p.first << " ";
    std::cout << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra unordered.cpp -o unordered && ./unordered
```

The `unordered_map` has no meaningful order. Both give you the same operations (`[]`, `find`, `count`, `at`). The map needs `operator<` for its keys, the unordered map a hash function.

### Task 5: Queue and stack

```c++
// file: adapters.cpp
#include <iostream>
#include <queue>
#include <stack>

int main() {
    std::queue<int> q;
    std::stack<int> s;
    for (int i = 1; i <= 3; ++i) { q.push(i); s.push(i); }

    std::cout << "queue (FIFO): ";
    while (!q.empty()) { std::cout << q.front() << " "; q.pop(); }
    std::cout << std::endl << "stack (LIFO): ";
    while (!s.empty()) { std::cout << s.top() << " "; s.pop(); }
    std::cout << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra adapters.cpp -o adapters && ./adapters
```

There is no iterator, no `[]`, no way to look at the middle. The adapters offer only `push`, `pop`, `front`/`top` and `empty`: this is what makes it clear to the reader that you are processing a queue, not an arbitrary list.

### Task 6: Measure instead of guess

```c++
// file: bench.cpp
#include <algorithm>
#include <chrono>
#include <iostream>
#include <list>
#include <map>
#include <unordered_map>
#include <vector>

template <typename F>
long long ms(F f) {
    auto t0 = std::chrono::steady_clock::now();
    f();
    auto t1 = std::chrono::steady_clock::now();
    return std::chrono::duration_cast<std::chrono::milliseconds>(t1 - t0).count();
}

int main() {
    const int N = 20000;
    std::vector<int> v;
    std::list<int> l;
    std::map<int, int> m;
    std::unordered_map<int, int> u;
    for (int i = 0; i < N; ++i) { v.push_back(i); l.push_back(i); m[i] = i; u[i] = i; }

    long long hits = 0;
    std::cout << "vector  find: " << ms([&] { for (int i = 0; i < N; ++i) hits += std::find(v.begin(), v.end(), i) != v.end(); }) << " ms" << std::endl;
    std::cout << "list    find: " << ms([&] { for (int i = 0; i < N; ++i) hits += std::find(l.begin(), l.end(), i) != l.end(); }) << " ms" << std::endl;
    std::cout << "map     find: " << ms([&] { for (int i = 0; i < N; ++i) hits += m.find(i) != m.end(); }) << " ms" << std::endl;
    std::cout << "unord.  find: " << ms([&] { for (int i = 0; i < N; ++i) hits += u.find(i) != u.end(); }) << " ms" << std::endl;
    std::cout << "(hits " << hits << ")" << std::endl;
}
```

```bash
g++ -std=c++17 -O2 -Wall -Wextra bench.cpp -o bench && ./bench
```

The two sequences search linearly, the two maps by key. Increase `N` to 40000 and watch which numbers grow by a factor of four (quadratic) and which hardly change. The lambdas in this program are the subject of a later chapter, read them as "a piece of code handed to `ms`".

## Questions

- How is a `std::map` ordered, and which operation do the keys have to provide?
- Why does `stock["bmw"]` change the size of the map while `at("audi")` does not?
- What is the difference between `std::map` and `std::unordered_map`?
- Which container removes duplicates, and how did Task 3 make use of iterators?
- Why does a `std::queue` have no `operator[]`, and what do you gain from that?
- Which of the four containers was fastest in Task 6, and why does the list lose against the vector for the same linear search?

## Advice

Start with `std::vector`. Choose a map when you search by key, a set when you only need uniqueness, and a queue or a stack when the problem *is* a queue or a stack. Do not trust any rule of thumb blindly, including the ones in this exercise: measure with realistic data.
