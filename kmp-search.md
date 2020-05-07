---
title: KMP Search
tags:
  - Interview
category:
  - Data Structure and Algorithm
author: bsyonline
lede: 没有摘要
date: 2020-02-12 21:38:39
thumbnail:
---


和 BF 算法不同，KMP 算法按照前缀和后缀找到公共前后缀，构造出一个 prefix table，然后在匹配失败的时候，不是回溯到开头的位置，而是按照 prefix table 的索引位置进行回溯，从而减少回溯次数。

我们来看看 KMP 的查找过程：
目标字符串为 text ，索引记为 i ，需要匹配的字符串为 pattern ，索引记为 j 。根据 pattern 先生成 prefix table 。然后从第一个元素开始匹配。第一个位置匹配， i 和 j 都后移一位。
<img src="https://s2.ax1x.com/2020/02/12/1bG0IJ.png" alt="20200212213949" border="0" style="width:500px">
当第二个位置不匹配，根据 prefix table ，将 pattern 的 0 位置对齐到 text 的 i 位置。
<img src="https://s2.ax1x.com/2020/02/12/1bGdZF.png" alt="20200212214027" border="0" style="width:500px">
然后重新开始匹配，pattern 第一个位置就不匹配，根据 prefix table ，j = -1 ，所以 pattern 整体后移一位。
<img src="https://s2.ax1x.com/2020/02/12/1bGNrT.png" alt="20200212214046" border="0" style="width:500px">
当匹配到 pattern 最后一个位置不匹配，根据 prefix table ，将 pattern 的 0 位置对齐到 text 的 i 位置。
<img src="https://s2.ax1x.com/2020/02/12/1bGUqU.png" alt="20200212214106" border="0" style="width:500px">
pattern 最后一个位置不匹配，根据 prefix table，pattern 的 0 位置对齐到 text 的 i 的位置。
<img src="https://s2.ax1x.com/2020/02/12/1bGJx0.png" alt="20200212214129" border="0" style="width:500px">
继续匹配，当匹配到 pattern 的最后一个位置符合，则找到一个匹配的字符串。
<img src="https://s2.ax1x.com/2020/02/12/1bGDi9.png" alt="20200212214201" border="0" style="width:500px">
继续向后找，找出数组长度则退出，查找完成。

```
public class KMPExample {
    public static void main(String[] args) {
        String s1 = "ABAABCDAABAB";
        String pattern = "AABA";
        int[] next = next(pattern);
        int[] prefix = prefixTable(next);
        System.out.println(Arrays.toString(next));
        System.out.println(Arrays.toString(prefix));
        kmpSearch(s1, pattern);
    }

    /**
     * @param s1
     * @param pattern
     */
    public static void kmpSearch(String s1, String pattern) {
        int[] next = next(pattern);
        int[] prefixTable = prefixTable(next);
        int i = 0;
        int j = 0;
        boolean found = false;
        while (i < s1.length()) {
            // j 等于 pattern.length 并且 pattern 最后一位置和 s1 的 i 位置匹配，说明整体匹配，即找到
            if (j == pattern.length() - 1 && s1.charAt(i) == pattern.charAt(j)) {
                System.out.println("found at " + (i - j));
                found = true;
                // 如果需要继续找，则从 前缀表的 j 位置继续匹配
                j = prefixTable[j];
            }
            if (s1.charAt(i) == pattern.charAt(j)) {
                i++;
                j++;
            } else {
                // 如果不匹配，则从前缀表的 j 位置开始重新匹配
                j = prefixTable[j];
                // 如果第一个位置都不匹配，则整体后移一位
                if (j == -1) {
                    i++;
                    j++;
                }
            }
        }
        if (!found) {
            System.out.println("not found");
        }
    }

    /**
     * 构造 prefix table， next 数组整体后移 1 位，第一个位置置为 -1
     *
     * @param next
     * @return
     */
    private static int[] prefixTable(int[] next) {
        int[] prefix = new int[next.length];
        prefix[0] = -1;
        for (int i = 0; i < next.length - 1; i++) {
            prefix[i + 1] = next[i];
        }
        return prefix;
    }

    /**
     * 前缀表
     *
     * @param pattern
     * @return
     */
    private static int[] next(String pattern) {
        int[] next = new int[pattern.length()];
        next[0] = 0;
        for (int i = 1, j = 0; i < pattern.length(); i++) {
            while (j > 0 && pattern.charAt(i) != pattern.charAt(j)) {
                j = next[j - 1];
            }
            // 如果
            if (pattern.charAt(i) == pattern.charAt(j)) {
                j++;
            }
            next[i] = j;
        }
        return next;
    }
}


```