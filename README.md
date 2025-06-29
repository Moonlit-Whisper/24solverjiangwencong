# 24点游戏求解器
[24点游戏](https://baike.baidu.com/item/24%E7%82%B9) 求解器：为你给出所有不等价解法。[试试吧！](https://moonlit-whisper.github.io/24solverjiangwencong/)

蒋文聪的课程作业之一，与[广义24点概率](https://github.com/Moonlit-Whisper/Generalized-24-points-probability)相辅相成

## 相似解法
例如，对于数字为 2, 3, 6, Q(12) 的牌，可能的解法有：

    2 + 4 + 6 + 12 = 24
    4 × 6 ÷ 2 + 12 = 24
    12 ÷ 4 × (6 + 2) = 24
    ...

这里会过滤掉相似的解法，例如：

    2 + 4 + 6 + 12 = 24
    (12 + 6) + (4 + 2) = 24   // 结合律
    4 × 6 ÷ 2 + 12 = 24
    6 ÷ 2 × 4 + 12 = 24       // 交换律

还有其他类似情况，例如：

    4 × 6 + 5 - 5 = 24
    4 × 6 × 5 / 5 = 24        // 无意义的重复数字

    12 × (13 × 1 - 11) = 24
    12 × (13 ÷ 1 - 11) = 24   // 无意义的1

    (2 + 2) × (3 + 3) = 24
    2 × 2 × (3 + 3) = 24      // 2 + 2 = 2 × 2

    (2 × 6 - 6) × 4 = 24
    (6 + 6) ÷ 2 × 4 = 24      // 2 × a - a = (a + a) ÷ 2

## 算法
我们需要所有4个操作数的不等价表达式的集合。例如 `a × b ÷ c + d = d + a ÷ c × b`，所以我们只保留 `a × b ÷ c + d`，而不保留 `d + a ÷ c × b`。

我们通过遍历括号结构、操作数位置和运算符的组合列举了所有表达式。然后用4个随机数测试每个表达式（并多次运行以避免碰撞）。等价的表达式总是有相同的结果，反之亦然。最终我们找到了1170个（和数列 [A140606](http://oeis.org/A140606) 匹配）不等价表达式。

对于4个不同数字（`a, b, c, d`），我们可以测试每个表达式是否等于24。例如 `2, 3, 6, 12`，有 `a + b + c + d`、`b × c ÷ a + d` 以及其他8个表达式匹配到24。

但这对于含有两个相同数字 `(a, b, c, c)`，或含有 `1` (`1, a, b, c`)，或含有 `2` 等情况并不适用。所以我们还需要其它不等价表达式集合：

    a, a, b, c // 两个a可以交换，或可能无用
    a, a, b, b
    a, a, a, b
    a, a, a, a
    1, a, b, c // 1可能无用
    1, a, a, b
    1, a, a, a
    1, 1, a, b
    1, 1, a, a
    1, 1, 1, a
    2, 2, a, b // 2 + 2 = 2 × 2
    2, 2, a, a
    2, 2, 2, a
    1, 2, 2, a
    2, a, a, b // 2 × a - a = (a + a) ÷ 2

在我们的 JavaScript 程序中，[pregen.js](https://github.com/Moonlit-Whisper/24solverjiangwencong/blob/main/pregen.js) 生成了这些不等价表达式的集合并保存为 [24-expressions.js](https://github.com/Moonlit-Whisper/24solverjiangwencong/blob/main/24-expressions.js)。[24.js](https://github.com/Moonlit-Whisper/24solverjiangwencong/blob/main/24.js) 则决定使用哪一组，并找出所有匹配的表达式。

对于 `a, a', b, c`（其中 `a' = a + 1`），我们还要考虑 `a' - a` 可能无用。因此需要过滤掉所有包含 `(a' - a) ×`、`× (a' - a)` 和 `÷ (a' - a)` 的表达式，然后尝试 `b + c` 和 `b × c` 作为补充。

## 已知缺陷

`1, 1, 5, 5` 只有一种不等价解法，因为我们认为 `(a + 1) × (a - 1)` 和 `a × a - 1 × 1` 是相同的。

`2, 4, 6, 6` 有一对看似相似的解法：`(6 - (4 - 2)) × 6`（A）和 `(6 - 4 ÷ 2) × 6`（B），但我们没有把 `4 - 2` 和 `4 ÷ 2` 视为相似。

`2, 4, 6, 6` 还有另一对看似相似的解法：`(6 - 4 + 2) × 6`（C）和 `(6 - 4) × 2 × 6`（D），但我们没有把 `(a" - a) + 2` 和 `(a" - a) × 2`（其中 `a" = a + 2`）视为相似。

要实现上述两个考虑很难，因为A和C显然类似，但B和D完全不相似。

## 解最多的组合
`2, 4, 8, 10` 有11种解法：

    10 + 8 + 4 + 2 = 24
    (10 - 4) × 8 ÷ 2 = 24
    (10 × 4 + 8) ÷ 2 = 24
    ((10 + 2) × 8 ÷ 4 = 24
    10 × 2 + 8 - 4 = 24
    (10 - 2) × 4 - 8 = 24
    8 × 4 - 10 + 2 = 24
    (8 ÷ 4 + 10) × 2 = 24
    (8 × 2 - 10) × 4 = 24
    (10 - 8 ÷ 2)) × 4 = 24
    10 × 4 - 8 × 2 = 24

## 仅有分数解的组合

    1, 3, 4, 6
    1, 4, 5, 6（2个解）
    1, 5, 5, 5
    1, 6, 6, 8
    1, 8, 12, 12
    2, 2, 11, 11
    2, 2, 13, 13
    2, 3, 5, 12
    2, 4, 10, 10
    2, 5, 5, 10
    2, 7, 7, 10
    3, 3, 7, 7
    3, 3, 8, 8
    4, 4, 7, 7
    5, 5, 7, 11
    5, 7, 7, 11

## 有中间大数的解

    1, 7, 13, 13
    6, 12, 12, 13
    1, 6, 11, 13
    6, 11, 12, 12
    5, 10, 10, 13
    1, 5, 11, 11
    5, 10, 10, 11
    4, 8, 8, 13
    4, 4, 10, 10
    4, 8, 8, 11
    6, 9, 9, 10
    3, 8, 8, 10
    3, 5, 7, 13
    3, 6, 6, 11
    1, 2, 7, 7
    5, 8, 9, 13
    5, 9, 10, 11
    4, 7, 11, 13
    4, 9, 11, 11
    4, 10, 10, 11
    6, 7, 7, 11
    3, 5, 8, 13
    5, 5, 8, 11
    2, 3, 13, 13

像 `a × b - a × c = 24` 这种类型的答案未列出，例如 `10, 12, 12, 12`

## 其它较难的解

`3, 7, 9, 13` 有2个解：

    (7 - 13 ÷ 3) × 9 = 24 // 分数解
    9 × 7 - 13 × 3 = 24   // 中间大数
