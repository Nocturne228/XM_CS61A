# Lab 2: Higher-Order Functions, Lambda Expressions

本实验首先给出了四个topic:

-   Short Circuiting，也就是布尔运算中的短路现象，容易理解，Short-circuiting happens when the operator reaches an operand that allows them to make a conclusion about the expression.
-   Higher-Order Functions，笼统来说高阶函数就是把函数当参数或返回值的函数，包括lambda表达式。
-   Lambda Expressions，单独介绍
-   Environment Diagrams，在CS61A的教材中有详细讲解，推荐使用网站https://tutor.cs61a.org/

## Lambda Expressions

lambda表达式提供了一种定义匿名函数的方式，使得函数不必与名称发生绑定，并且只完成表达式求值的工作，避免副作用。

|                           |                            lambda                            | Def                                                          |
| :-----------------------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|           Type            |            *Expression* that evaluates to a value            | *Statement* that alters the environment                      |
|    Result of execution    | Creates an anonymous lambda function with no intrinsic name. | Creates a function with an intrinsic name and binds it to that name in the current environment. |
| Effect on the environment | Evaluating a `lambda` expression does *not* create or modify any variables. | Executing a `def` statement both creates a new function object *and* binds it to a name in the current environment. |
|           Usage           | A `lambda` expression can be used anywhere that expects an expression, such as in an assignment statement or as the operator or operand to a call expression. | After executing a `def` statement, the created function is bound to a name. You should use this name to refer to the function anywhere that expects an expression. |

https://youtu.be/vCeNq_P3akI?list=PLx38hZJ5RLZcUPWZ1-3HYsRPgZ8OCrvqz

# Questions

由于本lab的问答题比较有趣，所以贴上来了

## Q1

```python
>>> True and 13
13
______
>>> False or 0
0
______
>>> not 10
False
______
>>> not None
True
______
>>> True and 1 / 0
Error
______
>>> True or 1 / 0
True
______
>>> -1 and 1 > 0
True
______
>>> -1 or 5
-1
______
>>> (1 + 1) and 1
1
______
>>> print(3) or ""
3
""
______
>>> def f(x):
...     if x == 0:
...         return "zero"
...     elif x > 0:
...         return "positive"
...     else:
...         return ""
>>> 0 or f(1)
"positive"
______
>>> f(0) or f(-1)
"zero"
______
>>> f(0) and f(-1)
""
______
```

总结这些问题，可以看到python的表达式规范远比C语言宽松，表达式的类型可以很随意，如果出现短路，则以短路位置为主，否则为靠后的子表达式的值。如果是两个非布尔非假（0或None）值的运算也会出现短路。

## Q2

```python
>>> def cake():
...    print('beets')
...    def pie():
...        print('sweets')
...        return 'cake'
...    return pie
>>> chocolate = cake()
beets
______
>>> chocolate
Function
______
>>> chocolate()
sweets
'cake'
______
>>> more_chocolate, more_cake = chocolate(), cake
sweets
______
>>> more_chocolate
'cake'
______
>>> def snake(x, y):
...    if cake == more_cake:
...        return chocolate
...    else:
...        return x + y
>>> snake(10, 20)
Function
______
>>> snake(10, 20)()
sweets
'cake'
______
>>> cake = 'cake'
>>> snake(10, 20)
30
______
```

这里提醒在写带有高阶函数的程序时，要注意什么是程序的行为，什么是返回值，返回的是什么类型，如果返回值是一个函数，就应该作为函数处理。

## Q3

```python
Q: Which of the following statements describes a difference between a def statement and a lambda expression?
Choose the number of the correct choice:
0) A lambda expression cannot return another function.
1) A lambda expression does not automatically bind the function that it returns to a name.
2) A lambda expression cannot have more than two parameters.
3) A def statement can only have one line in its body.
? 1
-- OK! --

Choose the number of the correct choice:
0) When the lambda expression is evaluated.
1) When you assign the lambda expression to a name.
2) When the function returned by the lambda expression is called.
3) When you pass the lambda expression into another function.
? 2
-- OK! --

>>> lambda x: x  # A lambda expression with one parameter x
Function
______
>>> a = lambda x: x  # Assigning the lambda function to the name a
>>> a(5)
5
______
>>> (lambda: 3)()  # Using a lambda expression as an operator in a call exp.
3
______
>>> b = lambda x, y: lambda: x + y  # Lambdas can return other lambdas!
>>> c = b(8, 4)
>>> c
Function
______
>>> c()
12
______
>>> d = lambda f: f(4)  # They can have functions as arguments as well.
>>> def square(x):
...     return x * x
>>> d(square)
16
______
>>> higher_order_lambda = lambda f: lambda x: f(x)
>>> g = lambda x: x * x
>>> higher_order_lambda(2)(g)  # Which argument belongs to which function call?
Error
______
>>> higher_order_lambda(g)(2)
4
______
>>> call_thrice = lambda f: lambda x: f(f(f(x)))
>>> call_thrice(lambda y: y + 1)(0)
3
______
>>> print_lambda = lambda z: print(z)  # When is the return expression of a lambda expression executed?
>>> print_lambda
Function
______
>>> one_thousand = print_lambda(1000)
1000
______
>>> one_thousand # What did the call to print_lambda return?
Nothing
______
```

lambda小妙招~

# Coding

## Q4

这里就需要注意返回的是一个单参数函数，也即`func(x)<-composite_identity(f, g)`，`func`的功能是根据`f`和`g`，针对`x`的运算是否具有交换性。

## Q5

考虑到不同函数具有类似的函数体，可以考虑用一个`count_cond(condition)`使用相同的函数体根据条件来返回不同的函数，这样只需要在外部传入一个定义了循环次数`n`和迭代`i`之间的运算关系的函数即可。

# HOF Diagram Practice

## Q6

使用environment diagram分析程序

```python
n = 7

def f(x):
    n = 8
    return x + 1

def g(x):
    n = 9
    def h():
        return x + 1
    return h

def f(f, x):
    return f(x + n)

f = f(g, n)
g = (lambda y: y())(f)
```

根据程序一步一步来即可，这里只是把很多参数也命名为f来混淆，实际并不复杂。

# Optional

## Q7

实现最小公倍数

```python
def multiple(a, b):
    """Return the smallest number n that is a multiple of both a and b.

    >>> multiple(3, 4)
    12
    >>> multiple(14, 21)
    42
    """
    "*** YOUR CODE HERE ***"
    s = a * b
    while a % b != 0:
        a, b = b, (a % b)
    else:
        return s // b
```

## Q8

写一个3次轮回循环

```python
def cycle(f1, f2, f3):
    """Returns a function that is itself a higher-order function.

    >>> def add1(x):
    ...     return x + 1
    >>> def times2(x):
    ...     return x * 2
    >>> def add3(x):
    ...     return x + 3
    >>> my_cycle = cycle(add1, times2, add3)
    >>> identity = my_cycle(0)
    >>> identity(5)
    5
    >>> add_one_then_double = my_cycle(2)
    >>> add_one_then_double(1)
    4
    >>> do_all_functions = my_cycle(3)
    >>> do_all_functions(2)
    9
    >>> do_more_than_a_cycle = my_cycle(4)
    >>> do_more_than_a_cycle(2)
    10
    >>> do_two_cycles = my_cycle(6)
    >>> do_two_cycles(1)
    19
    """
    "*** YOUR CODE HERE ***"
    def g(n):   # n times
        def h(x):   # x is the number
            i = 0
            while i < n:
                if i % 3 == 0:
                    x = f1(x)
                elif i % 3 == 1:
                    x = f2(x)
                else:
                    x = f3(x)
                i += 1
            return x
        return h
    return g
```

