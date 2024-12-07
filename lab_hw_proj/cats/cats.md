# Proj: Cats

https://cs61a.org/proj/cats/



## Phase1: Typing

### problem1

Implement `pick`. This function selects which paragraph the user will type for the typing test. It takes three parameters:

-   `paragraphs`: a list of potential paragraphs (strings)
-   `select`: a function that evaluates a paragraph and returns `True` if it meets certain criteria, and `False` otherwise
-   `k`: a non-negative integer representing the index of the desired paragraph among those that meet the criteria

The `pick` function returns the `k`th paragraph that satisfies the `select` function. If no such paragraph exists (because `k` is greater than or equal to the number of qualifying paragraphs), then `pick` returns an empty string.

一个list comprehension加一个condition list即可

### problem2

Implement the `about` function, which takes a list of `subject` words. It returns a function that, when given a paragraph, checks whether the paragraph contains any of the words from the subject list

实现一个`about`函数，他返回可以检查一个词是否在`about`函数接收的`subject`单词列表中，也就是可以作为`pick`的`select`函数，简单的去标点、分割、小写化、判断是否存在即可

### problem3

Implement `accuracy`, which takes both a `typed` paragraph and a `source` paragraph. It returns the percentage of words in `typed` that exactly match the corresponding words in `source`. 

按照题目给出的算法即可，只需要注意一方为0或者双方都为0的情况。

### problem4

按照题目给出的算法即可。

## Phase2: Autocorrect

### problem5

Implement `autocorrect`, which takes a `typed_word`, a `word_list`, a `diff_function`, and a `limit`. The goal of `autocorrect` is to return the word in `word_list` that is closest to the provided `typed_word`, as determined by `diff_function`.

根据传入的`diff_function`，找到一个与`typed_word`最相近的单词进行纠正。这里可以利用`min`函数可以自定义`key`来自定义比较最小值的方式，这个函数还用到了decorator装饰器，会在后面解释：

```python
@memo
def autocorrect(typed_word, word_list, diff_function, limit):
    if typed_word in word_list:
        return typed_word
    closest_word = min(word_list, key=lambda word: diff_function(typed_word, word, limit))
    if diff_function(typed_word, closest_word, limit) <= limit:
        return closest_word
    else:
        return typed_word
```

### Problem6

计算类似编辑距离，不过这里只是针对两个词的对应位置判断是否需要修改，所以很多情况下的值远大于编辑距离。编辑距离会在下问中计算。这里的函数体也和编辑距离的计算几乎差不多，只是不需要考虑编辑操作的三种情况：

```python
def furry_fixes(typed, source, limit):
    if typed == "" or source == "":
        return max(len(typed), len(source))
    if limit == 0:
        if typed == source:
            return 0
        else:
            return 1
    return int(typed[0] != source[0]) + furry_fixes(typed[1:], source[1:], limit - int(typed[0] != source[0]))
```

### problem7

写力扣一定会写过的编辑距离题目，有三种操作：添加、删除、替换。分别考虑处理即可

```python
@memo_diff
def minimum_mewtations(typed, source, limit):
    if typed == "" or source == "": # Base cases should go here, you may add more base cases as needed.
        return max(len(typed), len(source))
    # Recursive cases should go below here
    elif limit == 0: # Feel free to remove or add additional cases
        return int(typed != source)
    elif typed == source:
        return 0
    elif typed[0] == source[0]:
        return minimum_mewtations(typed[1:], source[1:], limit)
    else:
        add = 1 + minimum_mewtations(typed, source[1:], limit-1)# Fill in these lines
        remove = 1 + minimum_mewtations(typed[1:], source, limit-1)
        substitute = 1 + minimum_mewtations(typed[1:], source[1:], limit-1)
        return min(add, min(remove, substitute))
```

## Phase3: Multiplayer

Problem8, 9, 10都是为了多人游戏实现的一些函数，难度不算高，略。

## Phase 4: Efficiency

### Decorator

A Python **decorator** allows you to modify a pre-existing function without changing the function's structure.

Specifically, a decorator function is a higher-order function that...

-   Takes the original function as an input
-   Returns a new function with modified functionality
-   This new function **must** contain the same arguments as the original function

### Memoization

记忆化搜索是为了避免计算机在诸如递归过程中进行重复的计算浪费时间，将计算结果存储下来达到空间换时间的效果。以斐波那契数的递归程序为例

![fib](https://cs61a.org/proj/cats/images/fib_tree.png)



我们要做的是实现`memo_diff`，它会将已经计算出的单词距离保存在cache中，从而避免过多的递归调用：

```python
def memo_diff(diff_function):
    """A memoization function."""
    cache = {}

    def memoized(typed, source, limit):
        key = (typed, source)
        if key in cache:
            cache_value, cache_limit = cache[key]
            if limit <= cache_limit:
                return cache_value
        value = diff_function(typed, source, limit)
        cache[key] = (value, limit)
        return value
        # END PROBLEM EC

    return memoized
```

