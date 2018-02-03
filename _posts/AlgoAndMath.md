---
title: leetcode 算法集锦
date: 2017-07-17 11:27
tags:
	- Algo
	- Math
	- Java
	- C/C++
---

## [leetcode 算法集锦](https://www.nowcoder.com/ta/leetcode)

> 主要是牛客网上 leetcode 的算法题实践. 在 Blog 包含自己的解法和对别人优秀解法的分析.

 序号 | 考点 |               题目               | C/C++   |        Java     
:---:|:---:|:--------------------------------:|:-------:|:---------------:
  01 | 树  | Minimum Depth of Binary Tree     |   NULL  | [题解](#jump_01)  
  02 | 栈  | evaluate-reverse-polish-notation |   NULL  | [题解](#jump_02) 


<!-- more -->

###  <span id="jump_02">[02 : evaluate-reverse-polish-notation](https://www.nowcoder.com/practice/22f9d7dd89374b6c8289e44237c70447?tpId=46&tqId=29031&rp=1&ru=/ta/leetcode&qru=/ta/leetcode/question-ranking)</span>

> Evaluate the value of an arithmetic expression in Reverse Polish Notation.Valid operators are+,-,*,/. Each operand may be an integer or another expression.
> 
> Some examples:
> 
> ["2", "1", "+", "3", "*"] -> ((2 + 1) * 3) -> 9
> 
> ["4", "13", "5", "/", "+"] -> (4 + (13 / 5)) -> 6

- 我的解法

```java
import java.util.Stack;

public class Solution {
    public int evalRPN(String[] tokens) {
        Stack<String> s = new Stack<>();
        int op1 = 0;
        int op2 = 0;
        for (int i = 0; i < tokens.length; i++) {
            switch (tokens[i]) {
                case "+": {
                    op1 = Integer.parseInt(s.pop());
                    op2 = Integer.parseInt(s.pop());
                    s.push(String.valueOf(op2 + op1));
                    break;
                }
                case "-": {
                    op1 = Integer.parseInt(s.pop());
                    op2 = Integer.parseInt(s.pop());
                    s.push(String.valueOf(op2 - op1));
                    break;
                }
                case "*": {
                    op1 = Integer.parseInt(s.pop());
                    op2 = Integer.parseInt(s.pop());
                    s.push(String.valueOf(op2 * op1));
                    break;
                }
                case "/": {
                    op1 = Integer.parseInt(s.pop());
                    op2 = Integer.parseInt(s.pop());
                    s.push(String.valueOf(op2 / op1));
                    break;
                }
                default: {
                    s.push(tokens[i]);
                    break;
                }
            }
        }
        return Integer.parseInt(s.pop());
    }
}
```

- 其他解法

```java
import java.util.Stack;
public class Solution {
    public int evalRPN(String[] tokens) {
        Stack<Integer> stack = new Stack<Integer>();
        for(int i = 0;i<tokens.length;i++){
            try{
                int num = Integer.parseInt(tokens[i]);
                stack.add(num);
            }catch (Exception e) {
                int b = stack.pop();
                int a = stack.pop();
                stack.add(get(a, b, tokens[i]));
            }
        }
        return stack.pop();
    }
    private int get(int a,int b,String operator){
        switch (operator) {
        case "+":
            return a+b;
        case "-":
            return a-b;
        case "*":
            return a*b;
        case "/":
            return a/b;
        default:
            return 0;
        }
    }
}
```

  
### <span id="jump_01">[01 : Minimum Depth of Binary Tree](https://www.nowcoder.com/practice/e08819cfdeb34985a8de9c4e6562e724?tpId=46&tqId=29030&tPage=1&rp=1&ru=/ta/leetcode&qru=/ta/leetcode/question-ranking)</span>

> Given a binary tree, find its minimum depth.The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.

- 我的解法

```java
深度优先遍历所有节点, 直至叶子节点后返回长度. 
每次取当前节点左右子节点的 [最小值+1] 为该节点的最小深度. 
public class Solution {
    public int run(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int length1 = run(root.left);
        int length2 = run(root.right);
        if (length1 == 0 || length2 == 0) {
            return length1 + length2 + 1;
        }
        return Math.min(length1, length2) + 1;
    }
}
```

- 其他思路

```c++
class Solution {
public:
    typedef TreeNode* tree;
    int run(TreeNode *root) {
        //采用广度优先搜索，或者层序遍历，找到的第一个叶节点的深度即是最浅。
      if(! root) return 0;
      queue<tree> qu;
      tree last,now;
      int level,size;
      last = now = root;
      level = 1;qu.push(root);
      while(qu.size()){
        now = qu.front();
        qu.pop();
        size = qu.size();
        if(now->left)qu.push(now->left);
        if(now->right)qu.push(now->right);
        if(qu.size()-size == 0)break;
        if(last == now){
          level++;
          if(qu.size())last = qu.back();
        }
      }
      return level;
    }
};
```