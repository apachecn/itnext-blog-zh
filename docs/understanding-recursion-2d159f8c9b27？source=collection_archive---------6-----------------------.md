# 理解递归

> 原文：<https://itnext.io/understanding-recursion-2d159f8c9b27?source=collection_archive---------6----------------------->

![](img/3058740061a2d6b5576e49a1998aa761.png)

简单来说，递归就是调用函数本身。它可以用来将复杂的问题分解成更小的可管理的类似单元，这些单元可以由同一个功能来处理。

# 递归与迭代

迭代函数是循环重复代码的某个部分，递归函数是**再次调用自己**来重复代码。

例如计算一组数字的总和

```
function iterativeSum(arr) {
    let sum = 0;
    for (let i of arr) {
        sum += i;
    }
    return sum;
} function recursiveSum(arr) {
    if (arr.length === 0) {
        return 0;
    }
    return arr[0] + recursiveSum(arr.slice(1));
}/**
 *
 * iterativeSum([1,2,3]) => 1 + 2 + 3 => 6
 *
 * recursiveSum([1,2,3])
 *     => 1 + recursiveSum([2,3])
 *     =>       2 + recursiveSum([3])
 *     =>           3 + recursiveSum([])
 *     =>               0
 *     => 0 + 3 + 2 + 1 => 6
 */console.log(iterativeSum([1,2,3])); //6
console.log(recursiveSum([1,2,3])); //6
```

# 如何使用递归

# 基本条件是必须的

使用没有基条件的递归会导致无限递归。当递归调用自身的函数时，基本条件指示何时终止进程。

```
function infiniteRecursion() {
    // keeps calling itself without termination
    return infiniteRecursion();
}
```

# 将问题分解成功能本身可以处理的更小的单元。

例如

*一个数组的*之和=第一个元素的值+ *数组其余部分的*之和。

这就是我们在上面的 *recursiveSum* 例子中递归实现的方式。

# 熟能生巧

# 雌三醇环戊醚

## 问题:

从数字 1 开始，重复加 5 或乘 3，可以产生无限多的新数字。写一个函数，给定一个数，试图找出产生这个数的一系列这样的加法和乘法。

## 想法:

为了找到一个数的解(姑且称之为 *A* )，我们尝试将当前数加 5 或乘 3(姑且称之为 *B* )，并使用递归来检查结果是否有解(即 *B + 5* 或 *B \* 3 *)。基础条件是当前数大于(boom！)或等于(找到解！)*一个*。

## 解决方案:

```
function findSolution(num) {
    function find(start, history) {
        if(start > num) {
            return null; // boom!
        } else if (start === num) {
            return history; //solution found
        }
        return find(start + 5, `(${history} + 5)`) || find(start * 3, `(${history} * 3)`);
    } return find(1, '1');
}console.log(findSolution(13)); //(((1 * 3) + 5) + 5)
console.log(findSolution(20)); //null
```

# Q2

*   问题:[按](https://www.geeksforgeeks.org/tree-traversals-inorder-preorder-and-postorder/)(左、右、上)顺序遍历二叉树
*   解决办法

```
class Node {
    constructor(value, left=null, right=null) {
        this.value = value;
        this.left = left;
        this.right = right;
    }
}function inorder(node, fn) {
    if(node == null) {
        return;
    }
    inorder(node.left, fn);
    fn(node);
    inorder(node.right, fn);
}function test() {
    /**
     *        A
     *      /   \
     *    B       C
     *   / \       \
     *  E   F       H 
     */
    let E = new Node('E'),
        F = new Node('F'),
        H = new Node('H'),
        B = new Node('B', E, F),
        C = new Node('C', null, H),
        A = new Node('A', B, C);
    inorder(A, node => console.log(node.value)); // E B F A C H
}
test();
```

# 参考

*   [雄辩的 JavaScript](https://www.amazon.com/Eloquent-JavaScript-2nd-Ed-Introduction/dp/1593275846)

# 通知；注意

*   读书笔记系列如果想关注最新的新闻/文章，请[【观看】](https://github.com/n0ruSh/the-art-of-reading)订阅。