---
title: Binary Tree
date: 2014-04-16 15:53:17
tags:
 - Interview
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

## 二叉树概念

* 前序遍历（ DLR）也叫做先根遍历，可记做根左右。 前序遍历首先访问根结点然后遍历左子树，最后遍历右子树。在遍历左、右子树时，仍然先访问根结点，然后遍历左子树，最后遍历右子树。  
* 中序遍历，也叫中根遍历，遍历顺序为 左子树 -根 -右子树。  
* 后序遍历，也叫后根遍历，遍历顺序为 左子树 -右子树 -根。  

## 同构

同构指两棵树的结构，包括子树的结构相似，节点的值可以不同。

```java
	class BinaryTree{
	    char value
	    BinaryTree left
	    BinaryTree right
	}

	def g = new BinaryTree(value:'G')
	def f = new BinaryTree(value:'F')
	def e = new BinaryTree(value:'E')
	def d = new BinaryTree(value:'D')
	def c = new BinaryTree(value:'C',left: f,right: g)
	def b = new BinaryTree(value:'B',left: d,right: e)
	def a = new BinaryTree(value:'A',left: b,right: c)

	def g1 = new BinaryTree(value:'g')
	def f1 = new BinaryTree(value:'f')
	def e1 = new BinaryTree(value:'e')
	def d1 = new BinaryTree(value:'d')
	def c1 = new BinaryTree(value:'c',left: f1)
	def b1 = new BinaryTree(value:'b',left: d1,right: e1)
	def a1 = new BinaryTree(value:'a',left: b1,right: c1)


	def boolean like(BinaryTree a, BinaryTree b){
	    if(a==null && b==null){
	        return true
	    }else{
	        if(a!=null && b!=null){
	            return like(a.left,b.left) & like(a.right,b.right)
	        }else{
	            return false
	        }
	    }
	}
	println preorder(a)
	println postorder(a)
	println middleorder(a)
	println like(a,a1)
```

## 遍历
**Groovy 版**
```java
def String preorder(BinaryTree tree){
		String s = ''
		if(tree != null){
				s += tree.value
				s += preorder(tree.left)
				s += preorder(tree.right)
		}
		return s
}

def String postorder(BinaryTree tree){
		String s = ''
		if(tree != null){
				s += postorder(tree.left)
				s += postorder(tree.right)
				s += tree.value
		}
		return s
}

def String middleorder(BinaryTree tree){
		String s = ''
		if(tree != null){
				s += middleorder(tree.left)
				s += tree.value
				s += middleorder(tree.right)
		}
		return s
}
```
**Java 版**
```java
package com.rolex.algorithm;

/**
 * 二叉树遍历
 * <p/>
 * User: rolex
 * Date: 2015/4/12
 * version: 1.0
 */
public class Algo0412 {

    public static void main(String[] args) {
        Algo0412 algo = new Algo0412();
        BinaryTree d = new BinaryTree('D');
        BinaryTree e = new BinaryTree('E');
        BinaryTree f = new BinaryTree('F');
        BinaryTree g = new BinaryTree('G');
        BinaryTree c = new BinaryTree('C', f, g);
        BinaryTree b = new BinaryTree('B', d, e);
        BinaryTree a = new BinaryTree('A', b, c);

        System.out.println("preorder : " + algo.preorder(a));
        System.out.println("inorder : " + algo.inorder(a));
        System.out.println("postorder : " + algo.postorder(a));
    }

    public String preorder(BinaryTree t) {
        if (t == null) {
            return "";
        }
        String s = "";
        s += t.getValue();
        s += preorder(t.getLeft());
        s += preorder(t.getRight());
        return s;
    }

    public String inorder(BinaryTree t) {
        if (t == null) {
            return "";
        }
        String s = "";
        s += inorder(t.getLeft());
        s += t.getValue();
        s += inorder(t.getRight());
        return s;
    }

    public String postorder(BinaryTree t) {
        if (t == null) {
            return "";
        }
        String s = "";
        s += postorder(t.getLeft());
        s += postorder(t.getRight());
        s += t.getValue();
        return s;
    }
}

class BinaryTree {
    private BinaryTree left;
    private BinaryTree right;
    private char value;

    public BinaryTree(char c, BinaryTree left, BinaryTree right) {
        this.value = c;
        this.left = left;
        this.right = right;
    }

    public BinaryTree(char c) {
        this(c, null, null);
    }

    public void setLeft(BinaryTree left) {
        this.left = left;
    }

    public BinaryTree getLeft() {
        return left;
    }

    public void setRight(BinaryTree right) {
        this.right = right;
    }

    public BinaryTree getRight() {
        return right;
    }

    public void setValue(char c) {
        this.value = c;
    }

    public char getValue() {
        return value;
    }
}
```
