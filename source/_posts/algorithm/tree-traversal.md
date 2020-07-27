---
title: 二叉树遍历
comments: true
date: 2020-07-27 09:53:37
tags: algorithm
---

二叉树遍历
<!--more-->

## 树

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None
```

## 深度优先遍历

```python
def dfs(root):
    result = []
    pre_order_traversal(root) # 前序遍历
    # in_order_traversal(root) # 中序遍历
    # post_order_traversal(root) # 后续遍历
    return result


def pre_order_traversal(root):
    if not root:
        return
    result += [root.val]
    if root.left:
        pre_order_traversal(root.left)
    if root.right:
        pre_order_traversal(root.right)


def in_order_traversal(root):
    if not root:
        return
    if root.left:
        in_order_traversal(root.left)
    result += [root.val]
    if root.right:
        in_order_traversal(root.right)


def post_order_traversal(root):
    if not root:
        return
    if root.left:
        post_order_traversal(root.left)
    if root.right:
        post_order_traversal(root.right)
    result += [root.val]
```


## 广度优先遍历

```python
from collections import deque

def bfs(root):
    if not root:
        return []
    queue = deque()
    result = []
    queue.append(root)
    while queue:
        node = queue.popleft()
        result += [node.val]
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
    return result
```
