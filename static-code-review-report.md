# Static Code Review Report

Repository: `M-FalakSher/Review-Checklist`  
Project: `SVV static code verification`  
Language: `C++`  

This report reviews the provided C++ source for static code issues. The line numbers below are based on the snippet provided in the task description.

## 1) Review checklist by category

| Category | Result | Notes |
|---|---|---|
| Initialization | Issue | Uninitialized variables and missing initialization checks. |
| Pointers | Issue | Raw pointers are dereferenced without null checks and freed incorrectly. |
| Arrays/Buffers | Issue | Fixed-size buffer overflow risk and out-of-bounds writes. |
| Loops | Issue | Off-by-one loop bug in `totalMonsterHealth()`. |
| Arithmetic | Issue | Integer truncation, uninitialized sum, and signed/unsigned conversion problems. |
| Control flow | Issue | Assignment instead of comparison, missing default return value, and invalid dereference paths. |
| Functional calls | Issue | Returning dangling references, incorrect `delete` usage, and memory leaks. |
| Classes | Issue | Rule of Three violation and unsafe ownership semantics. |

## 2) Code review report

| Line number | Defect type | Why it is a problem | Proposed fix | Severity |
|---|---|---|---|---|
| 47 | Logic / comparison bug | `if (monsters[i]->id = id)` assigns the value instead of comparing it. This changes the object's state and may return the wrong monster. | Replace with `if (monsters[i]->id == id)`. | High |
| 54–58 | Null pointer / arithmetic bug | `findMonster()` may return `nullptr`; dereferencing `m` without a null check is unsafe. `unsigned int dmg = damage;` also converts a negative `damage` into a huge positive number. | Check `if (m == nullptr) return 0;` and use `int dmg = damage;` or validate input first. | Critical |
| 64–66 | Array bounds / loop bug | The loop condition `i <= monsterCount` reads `monsters[monsterCount]` even though valid indices are only `0` to `monsterCount - 1`. | Use `for (int i = 0; i < monsterCount; i++)`. | High |
| 70–73 | Dangling reference | `getMonsterLabel()` returns a reference to a local variable `label`, which is destroyed when the function exits. | Return a value instead: `std::string getMonsterLabel(int id)`. | Critical |
| 77–80 | Initialization bug | `double sum;` is uninitialized before use in `sum += monsters[i]->attack;`. | Initialize it as `double sum = 0.0;`. | High |
| 83–86 | Use-after-free | `delete m;` is followed by `m->name`, which accesses freed memory. | Do not access the pointer after deletion, or remove it from the collection before free. | Critical |
| 90–91 | Arithmetic / precision bug | `attack * armorPercent / 100` uses integer division, which truncates decimal results. | Use floating-point division: `static_cast<double>(attack) * armorPercent / 100.0`. | Medium |
| 94–99 | Missing return path | `playerRank()` only returns for some conditions and has no default result when `score <= 100`. This leads to undefined behavior. | Add a default return such as `return 0;` or a final `else` branch. | Medium |
| 102–109 | Buffer overflow / memory management bug | `char* name = new char[10];` is too small for many input strings, and `std::cin >> name` can overflow the buffer. `new[]` also needs `delete[]`, not `delete`. | Use `std::string name; std::cin >> name;` and let the compiler manage memory. | Critical |
| 122 | Memory leak | `bonus` is allocated with `new` and never deleted. | Delete it before exit, or better, use a smart pointer. | Medium |
| 134 | Shallow copy / Rule of Three violation | `Game copy = game;` copies raw pointers, so both objects will free the same heap memory, causing double free or invalid access. | Define a proper copy constructor and copy-assignment operator, or use `std::unique_ptr`/`std::shared_ptr`. | Critical |

## Summary

The code has multiple static verification issues in several categories. The most severe issues are:

- pointer misuse and null dereference,
- dangling references,
- incorrect `delete` handling,
- buffer overflow risk,
- out-of-bounds access,
- shallow-copy problems in `Game`.

These issues should be fixed before the program is considered safe for production or reliable for static verification.

## Recommended fix direction

1. Replace raw owning pointers with `std::unique_ptr<Monster>` or `std::vector<std::unique_ptr<Monster>>`.
2. Use `std::string` instead of `char*` for user input.
3. Fix all comparison, loop, and arithmetic mistakes.
4. Add null checks and bounds checks throughout the logic.
5. Implement correct copy semantics for `Game` (Rule of Three / Rule of Five).

---

This report was prepared as part of the static code verification task for the provided C++ program.
