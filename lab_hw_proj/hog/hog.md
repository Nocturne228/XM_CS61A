# PROJ: HOG

本实验需要完成一个与电脑对战掷骰子的游戏，有很多规则需要实现。61A的staff们非常贴心地为每一个Problem都准备了相关的问答来检验理解程度，这在很大程度上帮助了学生，感谢他们的心血付出！

文档地址[https://cs61a.org/proj/hog/](https://cs61a.org/proj/hog/)

>   You will need to use *control statements* and *higher-order functions* together, as described in Sections 1.2 through 1.6 of [Composing Programs](https://www.composingprograms.com/), the online textbook.
>
>   When students in the past have tried to implement the functions without thoroughly reading the problem description, they’ve often run into issues. 😱 **Read each description thoroughly before starting to code.**

项目文件结构：

-   `hog.py`: A starter implementation of Hog
-   `dice.py`: Functions for making and rolling dice
-   `hog_gui.py`: A graphical user interface (GUI) for Hog (updated)
-   `ucb.py`: Utility functions for CS 61A
-   `hog_ui.py`: A text-based user interface (UI) for Hog
-   `ok`: CS 61A autograder
-   `tests`: A directory of tests used by `ok`
-   `gui_files`: A directory of various things used by the web GUI

# Rules

本游戏有三个令人头蒙的规则，描述如下，限于篇幅不再举例：

-   **Sow Sad**. If any of the dice outcomes is a 1, the current player's score for the turn is `1`, regardless of the other values rolled.

-   **Boar Brawl**. A player who chooses to roll zero dice scores three times the absolute difference between the tens digit of the opponent’s score and the ones digit of the current player’s score, or 1, whichever is greater. The ones digit refers to the rightmost digit and the tens digit refers to the second-rightmost digit. If a player's score is a single digit (less than 10), the tens digit of that player's score is 0.
-   **Sus Fuss**. We call a number [*sus*](https://en.wikipedia.org/wiki/Sus_(genus)) if it has exactly 3 or 4 factors, including 1 and the number itself. If, after rolling, the current player's score is a sus number, their score instantly increases to the next prime number.



# Process

## Phase1 Simulator

In the first phase, you will develop a simulator for the game of Hog.

### problem0

骰子在`dice.py`中模拟，文件中实现了一些non-pure的函数，即每次调用函数可能会有不一样的输出，也就是会有side-effect。主要用到的函数调用有下面两个：

-   `def make_fair_dice(sides)`: Return a die that generates values ranging from 1 to SIDES, each with an equal chance.
-   `def make_test_dice(...):`: Return a die that cycles deterministically through OUTCOMES.

完成问答：`python3 ok -q 00 -u`

### problem1

Implement the `roll_dice` function in `hog.py`. It takes two arguments: a positive integer called `num_rolls`, which specifies the number of times to roll a die, and a `dice` function. It returns the number of points scored by rolling the die that number of times in a turn: either the sum of the outcomes or 1 *(Sow Sad)*.

实际上`roll_dice`做的就是通过接收前面骰子的模拟函数选择骰子结果，并接收投掷次数，返回符合规则*Sow Sad*的结果

完成问答`python3 ok -q 01 -u`，代码轻松愉快地就完成了。

### problem2

Implement `boar_brawl`, which takes the player's current score `player_score` and the opponent's current score `opponent_score`, and returns the number of points scored when the player rolls 0 dice and Boar Brawl is invoked.

如果一个palyer选择掷0个骰子，则触发规则*Boar Brawl*，根据自己和对手的分数计算得分。也没有难度。

### problem3

mplement the `take_turn` function, which returns the number of points scored for a turn by rolling the given `dice` `num_rolls` times.

Your implementation of `take_turn` should call both the `roll_dice` and `boar_brawl` functions rather than repeating their implementations.

直接调用即可，完成问答就可以理解题意。

### problem4

简单的计算因数个数

### problem5

实现`play`函数模拟整个hog游戏的运行。函数docstring为（太用心了）：

```python
"""Simulate a game and return the final scores of both players, with
Player 0's score first and Player 1's score second.

E.g., play(always_roll_5, always_roll_5, sus_update) simulates a game in
which both players always choose to roll 5 dice on every turn and the Sus
Fuss rule is in effect.

A strategy function, such as always_roll_5, takes the current player's
score and their opponent's score and returns the number of dice the current
player chooses to roll.

An update function, such as sus_update or simple_update, takes the number
of dice to roll, the current player's score, the opponent's score, and the
dice function used to simulate rolling dice. It returns the updated score
of the current player after they take their turn.

strategy0: The strategy for player0.
strategy1: The strategy for player1.
update:    The update function (used for both players).
score0:    Starting score for Player 0
score1:    Starting score for Player 1
dice:      A function of zero arguments that simulates a dice roll.
goal:      The game ends and someone wins when this score is reached.
"""
```

可以看到，主要组成部分是两个玩家各自的策略（掷几个骰子）和各自的分数，骰子的性质、成绩更新的方式（是否应用前面的规则）以及目标分数（到达目标分数就停止）

这样分析实际上逻辑就很简单了，模拟实际流程就行

## Phase2 Strategies

A *strategy* is a function that takes two arguments: the current player's score and their opponent's score. It returns the number of dice the player will roll, which can be from 0 to 10 (inclusive).

### problem6

根据描述，这里需要返回的是一个接收两个参数的函数，但是输出恒为`always_roll`的参数`n`，很直观。

### problem7

相对较难理解的一题。如果游戏目标是100分，则双方选手有100*100个可能的分数组合。本题需要实现的是take in一个策略，判断此策略是否对任何情况都只会输出相同的答案。简单来说，如果策略永远一直，那么返回True，否则返回False。要做的就是按照传入的策略对于任何起始分数情况都执行一遍策略，直到策略输出发生变化或达到(100, 100) 。

### problem8

Implement `make_averaged`, which is a higher-order function that takes a function `original_function` as an argument.

>   The return value of `make_averaged` is a function that takes in the same arguments as `original_function`. When called with specific arguments, this function should repeatedly call `original_function` on those same arguments, `times_called` times, and return the average of the results. Take a look at the `make_averaged` doctest. Be sure to keep track of what values are being passed into the function!

这里实际上是让固定投掷次数的结果为固定值，由于并不确定原函数`original_function`的参数个数，这里用`(*args)`代表可变参数列表。实际上就是调用`times_called`次`original_function()`，然后计算每次调用得分累计下来的平均值。

### problem9

Implement `max_scoring_num_rolls`, which runs an experiment to determine the number of rolls (from 1 to 10) that gives the maximum average score for a turn. Your implementation should use `make_averaged` and `roll_dice`.

这里实现的`max_scoring_num_rolls`是一个简单的辅助决策函数，他根据`make_averaged`计算最大可能收益，然后返回一个最小风险达到最大平均得分值的策略，也就是掷骰子个数。

### Running Experiments

运行run_experiment查看胜率

```bash
$ python3 hog.py -r

Max scoring num rolls for six-sided dice: 6
always_roll(6) win rate: 0.486
catch_up win rate: 0.5129999999999999
always_roll(3) win rate: 0.3395
always_roll(8) win rate: 0.4555
boar_strategy win rate: 0.501
sus_strategy win rate: 0.506
final_strategy win rate: 0.479
```

我们后续会实现更多策略，到时再对比胜率的变化。

### problem10

A strategy can try to take advantage of the *Boar Brawl* rule by rolling 0 when it is most beneficial to do so.

就是计算掷0个骰子得分的收益。函数接收一个阈值和一个掷骰子个数，如果掷0的收益超过阈值则输出0，否则输出`num_rolls`

You should find that running `python3 hog.py -r` now shows a win rate for `boar_strategy` close to 66-67%.

```bash
$ python3 hog.py -r

Max scoring num rolls for six-sided dice: 6
always_roll(6) win rate: 0.5
catch_up win rate: 0.505
always_roll(3) win rate: 0.355
always_roll(8) win rate: 0.48050000000000004
boar_strategy win rate: 0.6535
sus_strategy win rate: 0.509
final_strategy win rate: 0.491
```

### problem11

实现`sus_strategy`，搞清楚策略的思想，剩下的就只是收益最大化。本题就是考虑boar和sus规则下是否掷0.

### Optional: Problem 12

Implement `final_strategy`, which combines these ideas and any other ideas you have to achieve a high win rate against the baseline strategy. Some suggestions:

-   If you know the goal score (by default it is 100), there's no benefit to scoring more than the goal. Check whether you can win by rolling 0, 1 or 2 dice. If you are in the lead, you might decide to take fewer risks.
-   Instead of using a threshold, roll 0 whenever it would give you more points on average than rolling 6.

```bash
$ python3 hog.py -r

Max scoring num rolls for six-sided dice: 5
always_roll(6) win rate: 0.5095000000000001
catch_up win rate: 0.5045000000000001
always_roll(3) win rate: 0.363
always_roll(8) win rate: 0.4565
boar_strategy win rate: 0.6679999999999999
sus_strategy win rate: 0.6525000000000001
final_strategy win rate: 0.675
```

