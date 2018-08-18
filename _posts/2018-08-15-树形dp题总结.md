---
layout: post
title: "树形dp题总结"
description: "树形dp题的总结"
categories: [Code]
tags: [python, code]
redirect_from:
  - /2018/08/15/
---
* Kramdown table of contents
{:toc .toc}

## **树形dp题总结**

**此类题特征：需要遍历一遍二叉树节点，判断是否满足条件。且一般情况可以由下往上，在下层构建好所需信息之后，传到上层，然后在上层根据条件做进一步的信息整合，直到遍历完成之后返回整合完毕的信息。**

### **1. 求二叉树中的最长距离**

> 假设二叉树一个节点可以向上也可以向下，那么二叉树上的一个节点就能到达二叉树上其他任意节点，一个节点到另一个节点的距离定义为到达该节点的最短路径（也就是不绕路)，求二叉树中的最长距离。

**分析：**

这个题明显的我们需要遍历二叉树，找到每一个节点到其他节点的最长距离然后比较返回最长的那一个。如果我们就这么想，实现起来就挺麻烦。如果换一种思考方式，我们找以每一个节点为头的子树上的最长距离，然后返回最大的那一个就行了。以每一个节点为头就给够了启发——递归啊兄弟。下面分析情况：
1. 最长距离在该节点左子树上, 此时需要知道左子树上的最长距离；
2. 最长距离在该节点右子树上, 此时需要知道右子树上的最长距离；
3. 最长距离包括该节点, 此时需要知道左子树高度，右子树高度；

所以，综上，我们需要子节点每次给我们返回自己的最长距离以及高度，下面就可以构造返回类型了：
`hight, max_dis`，代码如下：

#### **代码：**

```python
def get_max_dis(root):
    """得到二叉树上的最远距离"""
    if root is None:
        return 0, 0

    left_hight, left_max_dis = get_max_dis(root.left)
    right_hight, right_max_dis = get_max_dis(root.right)

    hight = max(left_hight, right_hight) + 1
    # case 1
    p1 = left_max_dis
    # case 2
    p2 = right_max_dis
    # case 3
    p3 = left_hight + right_hight + 1

    return hight, max(p1, p2, p3)
```


### **2. 验证是否是二叉平衡树**

> 明显我们可以判断以每个节点为头结点的子树是否是二叉平衡树，每个子节点需要给我们返回自己是不是平衡树，以及自己的高度，构造返回类型：`is_sbt, height`。

#### **代码：**

```python
def is_sbt(root):
    """验证是否是二叉平衡树"""
    if root is None:
        return True, 0

    isbst_left, height_left = is_sbt(root.left)
    isbst_right, height_right = is_sbt(root.right)

    if not isbst_left or not isbst_right:
        return False, -1

    if isbst_left and isbst_right and abs(height_left - height_right) > 1:
        return False, -1

    height = max(height_left, height_right) + 1

    return True, height
```

### **3. 得到二叉树中最大的二叉搜索树的大小**

> 老套路，以每个子节点给头结点，得到需要的信息。

**分析情况：**
1. 最大bst在该节点左子树上，此时需要左子树上最大bst大小；
2. 最大bst在该节点右子树上，此时需要在右子树上最大bst大小；
3. 最大bst是以该节点为头的二叉树，此时需要左子树是不是bst，右子树是不是bst，左子树的最大值，右子树的最小值四个条件辅助判断；

综上，子节点需要提供给我们的返回类型包括：
* `is_bst`：是否为二叉搜索树；
* `max_subbst`：包含的最大二叉搜索树大小；`
* `max_max`：包含的最大值；
* `min_min`：包含的最小值；

#### **代码：**

```python
def get_max_subbst(root):
    """得到二叉树中最大的二叉搜索树大小"""
    if root is None:
        return True, 0,  -sys.maxsize, sys.maxsize

    is_bst_left, max_subbst_left, max_left, min_left = get_max_subbst(
        root.left)
    is_bst_right, max_subbst_right, max_right, min_right = get_max_subbst(
        root.right)
    
    # case 1
    p1 = max_subbst_left
    # case 2 
    p2 = max_subbst_right
    # case 3
    p3 = -sys.maxsize
    if is_bst_left and is_bst_right and root.val > max_left and root.val < min_right:
        p3 = max_subbst_left + 1 + max_subbst_right
    max_subbst = max(p1, p2, p3)
    is_bst = max_subbst == p3
    max_max = max(root.val, max_left, max_right)
    min_min = min(root.val, min_left, min_right)

    return is_bst, max_subbst, max_max, min_min
```

### **总结：**

从上面三个例子可以看出，此类问题最复杂的地方算是分析情况了，然后就是构建base case，尤其是第三题，在遍历到的节点为空的时候，怎么构建base case给上级节点提供正确的不影响后序判断的信息是很重要的。

