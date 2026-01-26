# leetcode76. 最小覆盖子串：给定两个字符串 `s` 和 `t`，长度分别是 `m` 和 `n`，返回 `s` 中的 `最短窗口` 子串，使得该子串包含 `t` 中的每一个字符（包括重复字符）。如果没有这样的子串，返回空字符串 `""`。测试用例保证答案唯一。

子字符串 是字符串中连续的 非空 字符序列。

---

# LeetCode 76：最小覆盖子串（Minimum Window Substring）

---

## （0）【纯人话、不写代码】完整解题思路（零基础版）

### 这道题在干什么？

有两个字符串：

* **s**：一条很长的字符串（比如 `"ADOBECODEBANC"`）
* **t**：一条较短的字符串（比如 `"ABC"`）

目标是：

> 在 **s** 里面，找一段 **最短的连续子串**，
> 使得这段子串 **包含 t 里的所有字符**，
> 而且 **t 里有几个，该子串里也要有几个**（不能少）。

如果找不到，就返回空字符串 `""`。

---

### 用生活语言理解

可以把问题理解成：

* **s** 是一条长货架
* **t** 是一张购物清单
* 要在货架上找一段**最短的连续区间**，
  这个区间里要能买齐购物清单上的所有东西

---

### 为什么不能暴力？

如果用“枚举所有子串”的方式：

* s 最长是 10⁵
* 子串数量是 O(n²)
* 每个子串再检查是否满足条件

➡️ 会直接超时

---

### 正确思路：**滑动窗口（双指针）**

核心思想只有一句话：

> **右边界不断扩大，直到“够了”；
> 左边界不断收缩，直到“不够”；
> 在“刚刚好够”的时候记录答案。**

---

### 滑动窗口是啥？

想象一个**可伸缩的窗口**，在字符串 s 上滑动：

```
s = A D O B E C O D E B A N C
      ↑           ↑
     left        right
```

窗口做两件事：

1. **右指针向右移动**：把字符“装进窗口”
2. **左指针向右移动**：把字符“踢出窗口”

---

### 如何判断“够不够”？

关键问题：
👉 怎么知道当前窗口已经包含了 t 的所有字符？

解决方案：

1. 用一个表，记录 **t 中每个字符需要几个**
2. 再用一个表，记录 **窗口中当前有几个**
3. 只要每个字符都 **满足 ≥ 需要的数量**，就说明“够了”

---

### 什么时候更新答案？

* **窗口第一次满足条件时**
* **每次左边界收缩但仍满足条件时**

此时窗口是：

> 当前右边界固定下，**能做到的最短窗口**

---

### 整体流程一句话总结

> 右指针负责“扩张直到合法”，
> 左指针负责“收缩直到刚好不合法”，
> 在合法区间中取最短。

---

---

## （1）Python3 标准答案（逐行中文注释）

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        from collections import Counter  # 导入计数工具，用来统计字符出现次数

        need = Counter(t)                 # 统计字符串 t 中每个字符需要的次数
        window = {}                       # 记录当前窗口中每个字符的出现次数

        left = 0                          # 窗口左边界下标
        right = 0                         # 窗口右边界下标
        valid = 0                         # 当前窗口中满足 need 要求的字符种类数

        start = 0                         # 记录最小窗口的起始位置
        length = float('inf')             # 记录最小窗口长度，初始设为无限大

        while right < len(s):             # 右边界不断向右移动
            c = s[right]                  # 取右边界字符
            right += 1                    # 右边界右移一格

            if c in need:                 # 如果该字符是需要的
                window[c] = window.get(c, 0) + 1   # 窗口中该字符次数 +1
                if window[c] == need[c]:           # 如果刚好满足需要次数
                    valid += 1                      # 满足条件的字符种类数 +1

            while valid == len(need):     # 当窗口已经包含所有需要字符
                if right - left < length: # 如果当前窗口更短
                    start = left          # 更新最短窗口起点
                    length = right - left # 更新最短窗口长度

                d = s[left]               # 准备移除左边界字符
                left += 1                 # 左边界右移一格

                if d in need:             # 如果移除的是需要的字符
                    if window[d] == need[d]:  # 如果移除前刚好满足
                        valid -= 1            # 满足条件的字符种类数 -1
                    window[d] -= 1            # 窗口中该字符次数 -1

        if length == float('inf'):         # 如果从未找到合法窗口
            return ""                      # 返回空字符串
        else:
            return s[start:start + length]# 返回最短覆盖子串
```

---

---

## （2）代码逐字逐句、零基础级解释

> 本部分极其详细，请耐心阅读。

---

### 第一行

```python
class Solution:
```

* `class`：定义一个“盒子”
* `Solution`：盒子的名字
* LeetCode 要求把代码写在这个盒子里

---

### 第二行

```python
def minWindow(self, s: str, t: str) -> str:
```

* `def`：定义一个函数
* `minWindow`：函数名字
* `s`：第一个输入字符串
* `t`：第二个输入字符串
* `-> str`：表示返回结果是字符串

---

### 第三行

```python
from collections import Counter
```

* `collections`：Python 自带工具箱
* `Counter`：专门用来“数东西”的工具

例如：

```python
Counter("AABC") → {'A': 2, 'B': 1, 'C': 1}
```

---

### 第四行

```python
need = Counter(t)
```

* 统计字符串 `t` 中每个字符需要的数量
* 这是“购物清单”

---

### 第五行

```python
window = {}
```

* 创建一个空字典
* 用来记录“当前窗口里有什么”

---

### 第六、七行

```python
left = 0
right = 0
```

* `left`：窗口左边界
* `right`：窗口右边界
* 窗口范围是：`[left, right)`

---

### 第八行

```python
valid = 0
```

* 表示：有多少种字符已经满足 `need` 的要求
* **不是字符数量，是字符种类数量**

---

### 第九、十行

```python
start = 0
length = float('inf')
```

* `start`：最优窗口的起点
* `length`：最优窗口的长度
* `float('inf')`：无限大，用来表示“还没找到答案”

---

### 主循环开始

```python
while right < len(s):
```

* 只要右边界没越界，就继续向右滑动

---

### 取字符并右移

```python
c = s[right]
right += 1
```

* `c`：当前加入窗口的字符
* `right += 1`：右边界向右移动

---

### 如果是需要的字符

```python
if c in need:
```

* 只关心 t 中出现过的字符
* 其他字符直接忽略

---

### 更新窗口计数

```python
window[c] = window.get(c, 0) + 1
```

* `window.get(c, 0)`：如果没有，就当作 0
* 当前字符数量 +1

---

### 判断是否刚好满足

```python
if window[c] == need[c]:
    valid += 1
```

* 当某个字符数量 **第一次刚好达到需要数量**
* `valid` 加 1

---

### 判断窗口是否已经“合法”

```python
while valid == len(need):
```

* `len(need)`：需要的字符种类总数
* 两者相等 → 窗口已经包含所有需要字符

---

### 更新最短答案

```python
if right - left < length:
    start = left
    length = right - left
```

* 当前窗口更短 → 记录下来

---

### 左边界收缩

```python
d = s[left]
left += 1
```

* `d`：被移出窗口的字符

---

### 如果移除的是关键字符

```python
if d in need:
```

---

### 判断是否破坏了条件

```python
if window[d] == need[d]:
    valid -= 1
window[d] -= 1
```

* 如果移除前刚好满足 → 移除后不满足
* `valid` 需要减 1

---

### 循环结束后的返回

```python
if length == float('inf'):
    return ""
else:
    return s[start:start + length]
```

* 如果没找到合法窗口 → 返回空
* 否则返回记录的最短子串

---

---

## （3）完整数值算例 + 全过程表格

### 示例

```text
s = "ADOBECODEBANC"
t = "ABC"
```

### need 表

| 字符 | 需要数量 |
| -- | ---- |
| A  | 1    |
| B  | 1    |
| C  | 1    |

---

### 关键过程表（精简但不跳步）

| 步骤  | left | right | 新字符 | window(A,B,C) | valid | 是否更新答案      |
| --- | ---- | ----- | --- | ------------- | ----- | ----------- |
| 1   | 0    | 1     | A   | A=1           | 1     | 否           |
| 2   | 0    | 4     | B   | A=1,B=1       | 2     | 否           |
| 3   | 0    | 6     | C   | A=1,B=1,C=1   | 3     | 是（"ADOBEC"） |
| 4   | 1    | 6     | 移A  | A=0           | 2     | 否           |
| ... | ...  | ...   | ... | ...           | ...   | ...         |
| 最终  | 9    | 13    | C   | A=1,B=1,C=1   | 3     | 是（"BANC"）   |

---

---

## （4）关键公式与表达式来源解释

### `right - left`

* 窗口长度公式
* 因为 `right` 指向的是 **下一个字符**
* 所以长度 = `right - left`

---

### `valid == len(need)`

* `valid`：当前满足条件的字符种类数
* `len(need)`：总共需要的字符种类数
* 相等 → 所有字符都满足
---

## （6）【背诵法则】——考试/面试秒写模板

### 一句话口诀

> 右扩张收集字符，
> 左收缩压缩窗口，
> 用 need + window + valid 控制合法性。

---

### 背代码骨架

```python
need = Counter(t)
window = {}
left = right = valid = 0

while right < len(s):
    加字符
    更新 valid

    while valid == len(need):
        更新答案
        移字符
        更新 valid
```

---

### 背三件关键事

1. **valid 表示“满足条件的字符种类数”**
2. **只在 valid == len(need) 时更新答案**
3. **右指针找合法，左指针找最短**

---
