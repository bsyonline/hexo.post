---
title: huffman tree and huffman code
tags:
  - Data Structure
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2020-02-14 08:23:31
thumbnail:
---

哈夫曼树的定义为假设有 m 个权值｛w1,w2,w3,...,wn｝，可以构造一棵含有 n 个叶子结点的二叉树，每个叶子结点的权为 wi ，则其中**带权路径长度 WPL **最小的二叉树称做最优二叉树或哈夫曼树。如图有 4 个节点 a b c d ，其权值分别为 wa = 7 ，wb = 5 ，wc = 2 ，wd = 4 ，它们可以构造出不同的二叉树，从根节点到每一个节点的路径长度和节点权值的乘积之和为 **带权路径长度 WPL **，WPL 最小的树即为哈夫曼树。

<img src="https://s2.ax1x.com/2020/02/14/1XEpL9.png" alt="1XEpL9.png" border="0" style="width:500px"/>

从上图可以看出，要构造一颗哈夫曼树，权值越大，路径越短，则 WPL 越小。所以，如果我们需要构造一棵哈夫曼树，可以使用如下步骤：
1. 先将节点按照权值升序排列，将前 2 个节点取出，作为树的叶子节点， 2 个叶子节点的权值之和记为父节点的权值。
2. 将父节点加入到节点列表中，重新排序，再重复步骤 1 ，直到最后一个节点。

利用哈夫曼树可以生成哈夫曼编码，哈夫曼树是一种变长编码，主要用于数据的压缩和解压场景。利用哈夫曼树生成哈夫曼编码的方法很简单，将左子树路径记为 0 ，右子树路径记为 1 即可。

<img src="https://s2.ax1x.com/2020/02/14/1X11xI.png" alt="1X11xI.png" border="0"  style="width:300px"/>



```
public class HuffmanTreeExample {

    public static void main(String[] args) {
        Node node1 = new Node(7, 'a');
        Node node2 = new Node(5, 'b');
        Node node3 = new Node(2, 'c');
        Node node4 = new Node(4, 'd');
        List<Node> list = new ArrayList<>();
        list.add(node1);
        list.add(node2);
        list.add(node3);
        list.add(node4);
        HuffmanTree huffmanTree = new HuffmanTree();
        Node root = huffmanTree.create(list);
        huffmanTree.preOrder(root);
        System.out.println("--");
        Map<Character, String> huffmanCode = huffmanTree.createHuffmanCode(root);
        System.out.println(huffmanCode);
    }
}

class HuffmanTree {

    public void preOrder(Node root) {
        System.out.println(root);
        if (root.left != null) {
            preOrder(root.left);
        }
        if (root.right != null) {
            preOrder(root.right);
        }
    }

    public Map createHuffmanCode(Node node) {
        HashMap codeMap = new HashMap();
        return createHuffmanCode(node, "", new StringBuffer(), codeMap);
    }

    /**
     * 将左子树的 pathNo 记为 0， 又子树的 pathNo 记为 1
     *
     * @param node
     * @param pathNo
     * @param code
     * @param map
     * @return
     */
    public Map createHuffmanCode(Node node, String pathNo, StringBuffer code, Map<Character, String> map) {
        StringBuffer sb = new StringBuffer(code);
        if (node != null) {
            sb.append(pathNo);
            if (node.value == null) {
                createHuffmanCode(node.left, "0", sb, map);
                createHuffmanCode(node.right, "1", sb, map);
            } else {
                map.put(node.value, sb.toString());
            }
        }
        return map;
    }

    public Node create(List<Node> nodes) {
        while (nodes.size() > 1) {
            Collections.sort(nodes);
            Node node0 = nodes.get(0);
            Node node1 = nodes.get(1);
            Node parentNode = new Node(node0.weight + node1.weight, null);
            parentNode.left = node0;
            parentNode.right = node1;
            nodes.remove(node0);
            nodes.remove(node1);
            nodes.add(parentNode);
        }
        return nodes.get(0);
    }
}

class Node implements Comparable<Node> {
    int weight;
    Character value;
    Node left;
    Node right;

    public Node(int weight, Character value) {
        this.weight = weight;
        this.value = value;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }

    public Character getValue() {
        return value;
    }

    public void setValue(Character value) {
        this.value = value;
    }

    public Node getLeft() {
        return left;
    }

    public void setLeft(Node left) {
        this.left = left;
    }

    public Node getRight() {
        return right;
    }

    public void setRight(Node right) {
        this.right = right;
    }

    @Override
    public int compareTo(Node o) {
        return this.weight - o.weight;
    }

    @Override
    public String toString() {
        return "Node{" +
                "weight=" + weight +
                ", value=" + value +
                '}';
    }
}
```