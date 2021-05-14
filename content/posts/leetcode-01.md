---
title: "Leetcode记录"
date: 2021-4-28T00:11:23+08:00
draft: false
tags:
    - leetcode
keywords:
    - leetcode
---

# Leetcode记录
本文记录一下自己刷题的过程，从最简单的开始，语言使用python3

### 二进制求和
使用了`int()`把给定的字符串以二进制的base来识别给定的string，然后进行相加，再把相加后的数转成二进制，再把结果转成string，替换前边的`0b`

#### 知识点
`int()`根据文档，第二个参数是base进制数，以什么样的进制来识别给定的字符串。
`bin()`负责把给定的int来转化成二进制数，在前边会有一个prefix '0b'

### 各位相加
```python
class Solution:
    def addDigits(self, num: int) -> int:
        def cal(n):
            return (n//10) + (n % 10)
        while True:
            if num < 10:
                return num
            else:
                num = cal(num)
```

#### 知识点
使用了递归的写法，每次递归获取个位和前边所有位的和，如果结果不小于10，那就继续递归，直到结果小于10返回