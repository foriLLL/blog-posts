---
description: 记录 Python 和 C++ 运算符优先级以及他们的一些区别，方便以后查阅。
time: 2023-11-06T10:26:09+08:00
tags: 
heroImage: 
---

在 Python 和 C++ 中，运算符优先级是不同的，有一些常用的语句返回的结果由于优先级的不同会有区别，可能会造成一些使用时的错误，比如：

```py
# python
1 & 3 == 1  # True
```

```cpp
// c++
bool result = 1 & 3 == 1;
cout << result << endl;     // 0
```

在 Python 中，`&` 的优先级比 `==` 高，所以先计算 `1 & 3`，结果为 `1`，然后再计算 `1 == 1`，结果为 `True`。  
而在 C++ 中，`==` 的优先级比 `&` 高，所以先计算 `3 == 1`，结果为 `False`，然后再计算 `1 & False`，结果为 `0`。  
在编译这段代码时， clang++ 也会给出警告：
```sh
test.cpp:6:21: warning: & has lower precedence than ==; == will be evaluated first [-Wparentheses]
    bool result = 1 & 3 == 1;
                    ^~~~~~~~
test.cpp:6:21: note: place parentheses around the '==' expression to silence this warning
    bool result = 1 & 3 == 1;
                    ^
                      (     )
test.cpp:6:21: note: place parentheses around the & expression to evaluate it first
    bool result = 1 & 3 == 1;
                    ^
                  (    )
1 warning generated.
```

接下来附上 Python 和 C++ 的运算符优先级表格，方便以后查阅。

## Python 运算符优先级

以下表格总结了Python中的运算符优先级，从最高优先级（最具约束力）到最低优先级（最少约束力）。同一框中的运算符具有相同的优先级。除非明确给出语法，否则运算符都是二元的。同一框中的运算符从左到右分组（除了指数和条件表达式，它们从右到左分组）。

请注意，comparisons、membership tests 以及 identity tests 都具有相同的优先级，并具有从左到右的链接特性。

| Operator                                                                      | Description                                                                        |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `(expressions...)`, `[expressions...]`, `{key: value...}`, `{expressions...}` | Binding or parenthesized expression, list display, dictionary display, set display |
| `x[index]`, `x[index:index]`, `x(arguments...)`, `x.attribute`                | Subscription, slicing, call, attribute reference                                   |
| `await x`                                                                     | Await expression                                                                   |
| `**`                                                                          | Exponentiation[^5]                                                                 |
| `+x`, `-x`, `~x`                                                              | Positive, negative, bitwise NOT                                                    |
| `*`, `@`, `/`, `//`, `%`                                                      | Multiplication, matrix multiplication, division, floor division, remainder[^6]     |
| `+`, `-`                                                                      | Addition and subtraction                                                           |
| `<<`, `>>`                                                                    | Shifts                                                                             |
| `&`                                                                           | Bitwise AND                                                                        |
| `^`                                                                           | Bitwise XOR                                                                        |
| `                                                                             | `                                                                                  | Bitwise OR |
| `in`, `not in`, `is`, `is not`, `<`, `<=`, `>`, `>=`, `!=`, `==`              | Comparisons, including membership tests and identity tests                         |
| `not x`                                                                       | Boolean NOT                                                                        |
| `and`                                                                         | Boolean AND                                                                        |
| `or`                                                                          | Boolean OR                                                                         |
| `if` – `else`                                                                 | Conditional expression                                                             |
| `lambda`                                                                      | Lambda expression                                                                  |
| `:=`                                                                          | Assignment expression                                                              |


## C++ 运算符优先级

| Precedence | Operator              | Description                                                                                                                    | Associativity                                                                                                      |
| ---------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| 1          | `::`                  | [Scope resolution](/w/cpp/language/identifiers#Qualified_identifiers)                                                          | Left-to-right →                                                                                                    |
| 2          | `a++` `a--`           | Suffix/postfix [increment and decrement](/w/cpp/language/operator_incdec)                                                      | Left-to-right →                                                                                                    |
| 2          | `<type>()` `<type>{}` | [Functional cast](/w/cpp/language/explicit_cast)                                                                               |                                                                                                                    |
| 2          | `a()`                 | [Function call](/w/cpp/language/operator_other#Built-in_function_call_operator)                                                |                                                                                                                    |
| 2          | `a[]`                 | [Subscript](/w/cpp/language/operator_member_access#Built-in_subscript_operator)                                                |                                                                                                                    |
| 2          | `.` `->`              | [Member access](/w/cpp/language/operator_member_access#Built-in_member_access_operators)                                       |                                                                                                                    |
| 3          | `++a` `--a`           | Prefix [increment and decrement](/w/cpp/language/operator_incdec)                                                              | Right-to-left ←                                                                                                    |
| 3          | `+a` `-a`             | Unary [plus and minus](/w/cpp/language/operator_arithmetic#Unary_arithmetic_operators)                                         |                                                                                                                    |
| 3          | `!` `~`               | [Logical NOT](/w/cpp/language/operator_logical) and [bitwise NOT](/w/cpp/language/operator_arithmetic#Bitwise_logic_operators) |                                                                                                                    |
| 3          | `(<type>)`            | [C-style cast](/w/cpp/language/explicit_cast)                                                                                  |                                                                                                                    |
| 3          | `*a`                  | [Indirection](/w/cpp/language/operator_member_access#Built-in_indirection_operator) (dereference)                              |                                                                                                                    |
| 3          | `&a`                  | [Address-of](/w/cpp/language/operator_member_access#Built-in_address-of_operator)                                              |                                                                                                                    |
| 3          | `sizeof`              | [Size-of](/w/cpp/language/sizeof) [^1]                                                                                         |                                                                                                                    |
| 3          | `co_await`            | [await-expression](/w/cpp/language/coroutines) (C++20)                                                                         |                                                                                                                    |
| 3          | `new` `new[]`         | [Dynamic memory allocation](/w/cpp/language/new)                                                                               |                                                                                                                    |
| 3          | `delete` `delete[]`   | [Dynamic memory deallocation](/w/cpp/language/delete)                                                                          |                                                                                                                    |
| 4          | `.*` `->*`            | [Pointer-to-member](/w/cpp/language/operator_member_access#Built-in_pointer-to-member_access_operators)                        | Left-to-right →                                                                                                    |
| 5          | `a*b` `a/b` `a%b`     | [Multiplication, division, and remainder](/w/cpp/language/operator_arithmetic#Multiplicative_operators)                        |                                                                                                                    |
| 6          | `a+b` `a-b`           | [Addition and subtraction](/w/cpp/language/operator_arithmetic#Additive_operators)                                             |                                                                                                                    |
| 7          | `<<` `>>`             | Bitwise [left shift and right shift](/w/cpp/language/operator_arithmetic#Bitwise_shift_operators)                              |                                                                                                                    |
| 8          | `<=>`                 | [Three-way comparison operator](/w/cpp/language/operator_comparison#Three-way_comparison) (since C++20)                        |                                                                                                                    |
| 9          | `<` `<=` `>` `>=`     | For [relational operators](/w/cpp/language/operator_comparison) `<` and `≤` and `>` and `≥` respectively                       |                                                                                                                    |
| 10         | `==` `!=`             | For [equality operators](/w/cpp/language/operator_comparison) `=` and `≠` respectively                                         |                                                                                                                    |
| 11         | `a&b`                 | [Bitwise AND](/w/cpp/language/operator_arithmetic#Bitwise_logic_operators)                                                     |                                                                                                                    |
| 12         | `^`                   | [Bitwise XOR](/w/cpp/language/operator_arithmetic#Bitwise_logic_operators) (exclusive or)                                      |                                                                                                                    |
| 13         | `                     | `                                                                                                                              | [Bitwise OR](/w/cpp/language/operator_arithmetic#Bitwise_logic_operators) (inclusive or)                           |                                                |
| 14         | `&&`                  | [Logical AND](/w/cpp/language/operator_logical)                                                                                |                                                                                                                    |
| 15         | `                     |                                                                                                                                | `                                                                                                                  | [Logical OR](/w/cpp/language/operator_logical) |  |
| 16         | `a?b:c`               | [Ternary conditional](/w/cpp/language/operator_other#Conditional_operator) [^2]                                                | Right-to-left ←                                                                                                    |
| 16         | `throw`               | [throw operator](/w/cpp/language/throw)                                                                                        |                                                                                                                    |
| 16         | `co_yield`            | [yield-expression](/w/cpp/language/coroutines) (C++20)                                                                         |                                                                                                                    |
| 16         | `=`                   | [Direct assignment](/w/cpp/language/operator_assignment#Builtin_direct_assignment) (provided by default for C++ classes)       |                                                                                                                    |
| 16         | `+=` `-=`             | [Compound assignment](/w/cpp/language/operator_assignment#Builtin_compound_assignment) by sum and difference                   |                                                                                                                    |
| 16         | `*=` `/=` `%=`        | [Compound assignment](/w/cpp/language/operator_assignment#Builtin_compound_assignment) by product, quotient, and remainder     |                                                                                                                    |
| 16         | `<<=` `>>=`           | [Compound assignment](/w/cpp/language/operator_assignment#Builtin_compound_assignment) by bitwise left shift and right shift   |                                                                                                                    |
| 16         | `&=` `^=` `           | =`                                                                                                                             | [Compound assignment](/w/cpp/language/operator_assignment#Builtin_compound_assignment) by bitwise AND, XOR, and OR |                                                |
| 17         | `,`                   | [Comma](/w/cpp/language/operator_other#Built-in_comma_operator)                                                                | Left-to-right →                                                                                                    |

## 参考

* [cppreference: C++ Operator Precedence](https://en.cppreference.com/w/cpp/language/operator_precedence)
* [doc.python: Operator precedence](https://docs.python.org/3/reference/expressions.html#operator-precedence)