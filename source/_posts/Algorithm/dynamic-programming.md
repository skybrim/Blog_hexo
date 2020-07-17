---
title: 动态规划
date: 2019/7/13
tags: algorithm
comments: true
---

暴力法 -> 记忆化递归 -> 动态规划
<!--more-->

## 三道例题

先给出两道例题，按照 暴力法 -> 记忆化递归 -> 动态规划 的顺序，给出结题方法。

### 爬楼梯

* 题目：  
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。  
每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？  
注意：给定 n 是一个正整数。

* 暴力递归解  
关键思路：斐波那契数列  
第 n 阶的方法数 = 第 n-1 阶的方法数 + 第 n-2 阶的方法数  
最终拆分到，第 1 个台阶方法数 = 1，第 2 个台阶方法数 = 2。

```Swift
    func climbStairs(_ n: Int) -> Int {
        guard n > 2 else {
            return n
        }
        return climbStairs(n - 1) + climbStairs(n - 2)
    }
```

* 记忆化递归  
结题思路和暴力递归一样，但是考虑到暴力法中，有很多重复计算，此时记录初次计算的值，以便后续使用。  
用空间换取时间。  

```Swift
    func climbStairs(_ n: Int) -> Int {
        guard n > 2 else {
            return n
        }
        var store = Array(repeating: 0, count: n + 1)
        store[1] = 1
        store[2] = 2
        func help(_ step: Int) -> Int {
            if store[step] != 0 { return store[step] }
            store[step] = help(step - 1) + help(step - 2)
            return store[step]
        }
        return help(n)
    }
```

* 动态规划  
记忆化递归，是自顶向下的方法，即将较大规模的问题，向较小规模的问题拆分。  
动态规划，是自底向上的方法，从最小规模的方法，逐级向上推算。

```Swift
//记忆化递归 -> 记忆化动态规划，此时记录了所有的解
    func climbStairs(_ n: Int) -> Int {
        guard n > 2 else {
            return n
        }
        var res = Array(repeating: 0, count: n + 1)
        res[1] = 1
        res[2] = 2
        for i in 3 ... n {
            res[i] = res[i - 1] + res[i - 2]
        }
        return res[n]
    }
//优化空间，只记录前两个解
    func climbStairs(_ n: Int) -> Int {
        guard n > 2 else {
            return n
        }
        var res = 0
        var pre_1 = 1
        var pre_2 = 2
        for _ in 3 ... n {
            res = pre_1 + pre_2
            pre_1 = pre_2
            pre_2 = res
        }
        return res
    }
```

### 零钱兑换

* 题目：  
给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。

* 暴力解

```Swift
    func coinChange(_ coins: [Int], _ amount: Int) -> Int {
        if amount == 0 { return 0 }
        var res = Int.max
        for coin in coins {
            if amount - coin < 0 { continue }
            let tmp = coinChange(coins, amount - coin)
            if tmp == -1 { continue }
            res = min(res, tmp + 1)
        }
        return res == Int.max ? -1 : Int(res)
    }
```

* 记忆化递归

```Swift
    func coinChange(_ coins: [Int], _ amount: Int) -> Int {
        func help(_ coins: [Int], _ amount: Int, _ memory: inout [Int]) -> Int {
            if amount == 0 { return 0 }
            if memory[amount] != -2 { return memory[amount] }
            var res = Int.max
            for coin in coins {
                if amount - coin < 0 { continue }
                let tmp = help(coins, amount - coin, &memory)
                if tmp == -1 { continue }
                res = min(res, tmp + 1)
            }
            memory[amount] = res == Int.max ? -1 : Int(res)
            return memory[amount]
        }
        var memory = Array(repeating: -2, count: amount + 1)
        return help(coins, amount, &memory)
    }
```

* 动态规划

```Swift
    func coinChange(_ coins: [Int], _ amount: Int) -> Int {
        if amount == 0 { return 0 }
        var memory = Array(repeating: amount + 1, count: amount + 1)
        memory[0] = 0
        for i in 1 ... amount {
            for coin in coins {
                if i - coin >= 0 {
                    memory[i] = min(memory[i], memory[i - coin] + 1)
                }
            }
        }
        return memory[amount] > amount ? -1 : memory[amount]
    }
```

## 总结

通过例题，我们可以总结思路：

* 状态转移方程  
动态规划类题目，第一步就是得到状态转移方程。状态转移方程，就是 f(n) = f(n-1) + f(n-2)。，即，状态 n 是入户通过状态 n-1 和状态 n-2 相加获取。  
这一步，通常是大家不想看的暴力解法，但是，这一步是最重要的。  
后续的解法，都是围绕如何优化这一步来的。

* 记忆化递归  
通过记录已知的解，使用空间来优化时间。

* 动态规划  
自底向上。

## 最后

一开始，很难直接一步写出动态规划，但是通过 状态转移方程 -> 记忆化递归 -> 动态规划，这个思路，可以逐步推导出解法。  
通过练习强化，当练习到一定程度，相信一定可以一步写出动态规划解法。
