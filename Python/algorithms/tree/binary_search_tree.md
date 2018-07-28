# 二叉搜索树

A binary search tree is a binary tree in which, for each node:

The node's value is greater than all values in the left subtree.
The node's value is less than all values in the right subtree.
BSTs are useful for quick lookups. If the tree is balanced, we can search for a given value in the tree in O(lg(n))O(lg(n)) time.

- 寻找二叉搜索树中第二大的元素

```python
class BinaryTreeNode(object):

def __init__(self, value):
    self.value = value
    self.left  = None
    self.right = None

def insert_left(self, value):
    self.left = BinaryTreeNode(value)
    return self.left

def insert_right(self, value):
    self.right = BinaryTreeNode(value)
    return self.right
```