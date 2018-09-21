---
layout: post
tille: 二叉树遍历方式总结：递归、非递归、morris
description: 二叉树遍历方式总结
categories: [Code]
tags:
    - python
    - code
    - binary tree
---
* Kramdown table of contents
{:toc .toc}

# **二叉树遍历方式总结：递归、非递归、morris**

## **二叉树节点定义：**

```python
# _*_ coding: utf-8 _*_
class Node():
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None
```

## **morris遍历的普遍过程：**

> 遍历到节点cur的时候：
* 如果cur节点没有左子树，cur指向cur的右孩子；
* 如果cur节点有左子树:
    1. 如果左子树的最右节点的右孩子是指向空的（第一次到达cur节点），那么将这个最右节点的右孩子设置为cur节点，然后cur指向cur的左孩子；
    2. 如果左子树的最右节点的右孩子是指向cur节点的（第二次到达cur节点），那么将这个最右节点的右孩子还原为空，然后cur指向cur的右孩子；
    
### **代码：**

```python
class MorrisTraverse():
    """morris神级遍历普通过程"""
    def morris_traverse(self, root):
        if root is None:
            return None
        
        cur = root
        most_right = None
        while cur:
            most_right = cur.left
            # 如果cur节点有左孩子
            if most_right:
                # 找到cur节点左孩子的最右节点
                while most_right.right and most_right.right != cur:
                    most_right = most_right.right
                # 如果是第一次到达节点cur，将该节点左孩子的最后节点的右孩子指向cur
                if most_right.right is None:
                    most_right.right = cur
                    cur = cur.left
                    continue
                # 如果是第二次到达节点cur，第一次到达时建立的连接断开
                else:
                    most_right.right = None
            cur = cur.right

```
## **遍历方法集合类**
### **递归实现前中后序遍历：**
```python
class Traverse():
    """二叉树遍历方法类"""
    # 前序
    def preorder_traverse(self, root):
        if root is None:
            return None
        # 第一次遍历到就打印
        print(root.val, end=' ')
        self.preorder_traverse(root.left)
        self.preorder_traverse(root.right)

    # 中序
    def inorder_traverse(self, root):
        if root is None:
            return None
        
        self.inorder_traverse(root.left)
        # 第二次遍历到就打印
        print(root.val, end=' ')
        self.inorder_traverse(root.right)

    # 后序
    def postorder_traverse(self, root):
        if root is None:
            return None
        self.postorder_traverse(root.left)
        self.postorder_traverse(root.right)
        # 第三次遍历到打印
        print(root.val, end=' ')
```

### **非递归实现前中后序遍历：**
#### **前序：**
> 前序非递归也是最简单的，因为第一次就打印所以只需要按序遍历就好了。过程可总结如下：
* 压栈过程：首先压入根节点, 对于`pop`出的节点，如果有左孩子就入栈，如果有右孩子也入栈；；
* 出栈过程：`pop`一个节点，然后执行压栈过程；

```python
    def preorder_traverse_with_stack(self, root):
        if root is None:
            return None

        stack = []
        stack.append(root)
        while stack:
            node = stack.pop()
            print(node.val, end=' ')
            # 先压入右孩子
            if node.right:
                stack.append(node.right)
            # 再压入左孩子
            if node.left:
                stack.append(node.left)
        print('')
```
#### **中序：**
> 中序非递归，因为中序是第二次访问到该节点的时候才打印，所以过程可以总结如下：
* 压栈过程：第一次访问到该节点压栈，然后再去找该节点的左孩子压栈；
* 出栈过程：当前无节点可压栈（已经到了最左的叶子节点), `pop`一个节点并打印，如果`pop`的节点有右孩子，那么重复压栈过程。

```python
    def inorder_traverse_with_stack(self, root):
        """
        非递归中序遍历与前序遍历相比其实就是多了一个给每个节点找左孩子然后压栈的过程。
        """
        if root is None:
            return None

        stack = []
        while stack or root:
            if root:
                stack.append(root)
                root = root.left
            else:
                root = stack.pop()
                print(root.val, end=' ')
                root = root.right
        print('')
```

#### **后序：**
> 后序非递归有两种经典方法：
1. 使用两个栈：
    由于我们知道前序遍历顺序是中左右，所以如果我们改写前序成中右左，然后把前序序列压栈，再弹出的顺序就是左右中，也就是后序遍历；
2. 使用一个栈：
    使用一个栈要注意弹出的节点的右节点是否在栈内，如果在栈内的话，要把右节点也弹出并且打印，然后再把第一次弹出的节点压栈，去找弹出的右节点有没有右节点，有的话就压栈。

```python
    def post_traverse_with_two_stack(self, root):
        """
        这里用了一个取巧的方法，因为前序遍历是中左右，所以如果我们可以实现中右左的遍历，然后把遍历过程压入备用栈，然后从备用栈弹出的顺序就变成了左右中，也就是后序遍历。
        """
        if root is None:
            return None

        stack = []
        stack_temp = []
        stack.append(root)
        while stack:
            node = stack.pop()
            stack_temp.append(node)
            if node.left:
                stack.append(node.left)
            if node.right:
                stack.append(node.right)
        while stack_temp:
            print(stack_temp.pop().val, end=' ')
        print('')

    def post_traverse_with_one_stack(self, root):
        """
        只用一个栈的后序遍历。
        1. 要判断弹出的节点，有没有右节点，有的话，压入右节点，再压入该节点
        2. 要注意pop两次时的条件
        **其实后序遍历就是逆序打印各子树右边界的过程，这个思想就用在morris逆序遍历上**
        """
        if root is None:
            return None
        stack = []
        while stack or root:
            while root:
                if root.right:
                    stack.append(root.right)
                stack.append(root)
                root = root.left

            node = stack.pop()
            if stack and node.right and node.right == stack[-1]:
                root = stack.pop()
                stack.append(node)
            else:
                print(node.val, end=' ')
                root = None
        print('')
```
后来对单栈后序遍历做了一些小总结：
![post traversal part1](http://oq782gkz3.bkt.clouddn.com/post_traversal1.jpeg)
![post traversal part2](http://oq782gkz3.bkt.clouddn.com/post_traversal2.jpeg)

代码：

```python
# -*- coding=utf-8 -*-


class TreeNode():
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None

class PostTraversal():
    def post_traversal_1(self, root):
        # 先加根节点 后加右子节点
        if root is None:
            return None
        stack = []
        help_dict = {}
        while stack or root:
            while root:
                stack.append(root)
                if root.right:
                    stack.append(root.right)
                root = root.left

            node = stack.pop()
            has_pass = help_dict.get(node, False)
            # 如果弹出节点为叶子节点，那么打印
            if not node.left and not node.right:
                print(node.val, end=' ')
            # 如果弹出节点不是叶子节点，且弹出节点是父节点的右子节点
            # 且没有进行过该节点的压栈过程
            # 那么继续while中的压栈过程
            elif stack and stack[-1].right == node and not has_pass:
                root = node
                help_dict[node] = True
            # 其他不应该重复压栈的节点
            else:
                print(node.val, end=' ')

        print('')

    def post_traversal_2(self, root):
        if root is None:
            return None
        
        stack = []
        while stack or root:
            while root:
                if root.right:
                    stack.append(root.right)
                stack.append(root)
                root = root.left

            node = stack.pop()
            # 下面这个交换顺序的过程其实解决了上面那个解决方法中无法处理多次重复到达一个节点
            # 并重复进行该节点的压栈过程的问题
            if stack and node.right == stack[-1]:
                root = stack.pop()
                stack.append(node)
            else:
                print(node.val, end=' ')
        print('')

if __name__ == "__main__":
    # 完全二叉树: The answer is 4 5 2 3 1.
    root = TreeNode(1)
    root.left = TreeNode(2)
    root.right = TreeNode(3)
    root.left.left = TreeNode(4)
    root.left.right = TreeNode(5)
    # 只有左子树: The answer is 3 2 1.
    root1 = TreeNode(1)
    root1.left = TreeNode(2)
    root1.left.left = TreeNode(3)
    # 只有右子树: The answer is 3 2 1.
    root2 = TreeNode(1)
    root2.right = TreeNode(2)
    root2.right.right = TreeNode(3)
    # 普通二叉树: The answer is 6 3 2 5 4 1.
    root3 = TreeNode(1)
    root3.left = TreeNode(2)
    root3.left.left = TreeNode(3)
    root3.left.left.right = TreeNode(6)
    root3.right = TreeNode(4)
    root3.right.right = TreeNode(5)

    tree = [root, root1, root2, root3]
    ex = PostTraversal()
    for root in tree:
        ex.post_traversal_1(root)
        ex.post_traversal_2(root)
```
### **借用morris遍历实现前中后序遍历**
> morris序中，如果一个节点有左子树，那么这个节点会访问两次，如果一个节点没有左子树，那么这个节点只会访问到一次；

#### **前序：**
> 借用morris序，在第一次访问该节点的时候就打印，就是前序序列；

```python
    def morris_pre_traverse(self, root):
        if root is None:
            return None
        
        cur = root
        most_right = None
        while cur:
            most_right = cur.left
            if most_right:
                while most_right.right and most_right.right != cur:
                    most_right = most_right.right
                if most_right.right is None:
                    most_right.right = cur
                    # 第一次到达即打印
                    print(cur.val, end=' ')
                    cur = cur.left
                    continue
                else:
                    most_right.right = None
            else:
                
                # 没有左子树的只会到达一次
                print(cur.val, end=' ')
            cur = cur.right
        print('')
```

#### **中序：** 
> 借用morris序，如果一个节点有左子树，第二次访问到时打印，如果一个节点没有左子树，那么第一次访问到时就打印。

```python
    def morris_in_traverse(self, root):
        if root is None:
            return None
        
        cur = root
        most_right = None
        while cur:
            most_right = cur.left
            if most_right:
                while most_right.right and most_right.right != cur:
                    most_right = most_right.right
                if most_right.right is None:
                    most_right.right = cur
                    cur = cur.left
                    continue
                else:     
                    most_right.right = None
            # 没有左子树的第一次到达或者有左子树的第二次到达才打印
            print(cur.val, end=' ')
            cur = cur.right
        print(' ')
```

#### **后序：** 
> morris序最多只能到达一个节点两次，而后序遍历是第三次访问到一个节点时才打印，所以需要借助于一点儿小技巧。

```python
    def morris_post_traverse(self, root):
        if root is None:
            return None
        
        cur = root
        most_right = None
        while cur:
            most_right = cur.left
            if most_right:
                while most_right.right and most_right.right != cur:
                    most_right = most_right.right
                if most_right.right is None:
                    most_right.right = cur
                    cur = cur.left
                    continue
                else:
                    most_right.right = None
                    # 在第二次到达时逆序打印cur节点的左子树的右边界
                    self.print_edge(cur.left)
            cur = cur.right
        # 逆序打印整个二叉树的右边界
        self.print_edge(root)
        print('')
    
    def print_edge(self, node):
        tail = self.reverse_edge(node)
        temp = tail
        while temp:
            print(temp.val, end=' ')
            temp = temp.right
        self.reverse_edge(tail)


    def reverse_edge(self, node):
        pre = None
        while node:
            next_node = node.right
            node.right = pre
            pre = node
            node = next_node
        return pre
```

### **测试前中后序遍历正确性**

```python
def main():
    # 完全二叉树
    n1 = Node(1)
    n1.left = Node(2)
    n1.right = Node(3)
    n1.left.left = Node(4)
    n1.left.right = Node(5)
    n1.right.left = Node(6)
    n1.right.right = Node(7)

    # 只有左子树
    n2 = Node(1)
    n2.left = Node(2)
    n2.left.left = Node(3)
    n2.left.left.left = Node(4)

    # 只有右子树
    n3 = Node(1)
    n3.right = Node(2)
    n3.right.right = Node(3)
    n3.right.right.right = Node(4)

    ex = Traverse()
    for item in [n1, n2, n3]:
        ex.preorder_traverse(item)
        print('')
        ex.preorder_traverse_with_stack(item)
        ex.morris_pre_traverse(item)
        print('*' * 20)

        ex.inorder_traverse(item)
        print('')
        ex.inorder_traverse_with_stack(item)
        ex.morris_in_traverse(item)
        print('*' * 20)

        ex.postorder_traverse(item)
        print('')
        ex.post_traverse_with_one_stack(item)
        ex.post_traverse_with_two_stack(item)
        ex.morris_post_traverse(item)
        print('_' * 20)

if __name__ == '__main__':
    main()
```

### **层次遍历：**

```python
def level_trans(root):
    if root is None:
        return None

    de = deque()
    de.appendleft(root)
    while de:
        loop = len(de)
        while loop > 0:
            root = de.pop()
            print(root.val, end=' ')
            if root.left:
                de.appendleft(root.left)
            if root.right:
                de.appendleft(root.right)
            loop -= 1
        print('')
```

变种层次遍历，打印从一颗二叉树左边看过去能看见的节点:

```python
def print_mostleft_node(root):
    if root is None:
        return None

    de = deque()
    de.appendleft(root)
    flag = True
    while de:
        loop = len(de)
        while loop > 0:
            root = de.pop()
            if flag:
                print(root.val, end=' ')
                flag = False
            if root.left:
                de.appendleft(root.left)
            if root.right:
                de.appendleft(root.right)
            loop -= 1
        print('')
        flag = True
```

其实就是打印每层第一个节点。
