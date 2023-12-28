# Inline QA

[TOC]

### Q：内联函数中的静态变量链接性？

摘自[cppreference.com](https://en.cppreference.com/w/cpp/language/inline)：

> 1. An *inline function* or *inline variable*(since C++17) has the following properties:
>    1. The definition of an inline function or variable(since C++17) must be reachable in the translation unit where it is accessed (not necessarily before the point of access).
>    2. An inline function or variable(since C++17) with [external linkage](https://en.cppreference.com/w/cpp/language/storage_duration#external_linkage) (e.g. not declared static) has the following additional properties:
>       1. **There may be [more than one definition](https://en.cppreference.com/w/cpp/language/definition#One_Definition_Rule) of an inline function or variable(since C++17) in the program as long as each definition appears in a different translation unit and (for non-static inline functions and variables(since C++17)) all definitions are identical. For example, an inline function or an inline variable(since C++17) may be defined in a header file that is included in multiple source files.**
>       2. It must be declared inline in every translation unit.
>       3. It has the same address in every translation unit.
>
> In an inline function,
>
> - **Function-local static objects in all function definitions are shared across all translation units (they all refer to the same object defined in one translation unit).**
> - Types defined in all function definitions are also the same in all translation units.

