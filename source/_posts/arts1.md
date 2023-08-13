---
title: arts1
date: 2023-08-13 15:14:54
tags:
---
⭐️● Algorithm: 每周至少做一个 LeetCode 的算法题

⭐️● Review: 阅读并点评至少一篇文章

⭐️● Tips: 学习至少一个技巧

⭐️● Share: 分享一篇有观点和思考的文章

### 算法题
```java
/**
 * 如果可以使用以下操作从一个字符串得到另一个字符串，则认为两个字符串 接近 ：
 *
 * 操作 1：交换任意两个 现有 字符。
 * 例如，abcde -> aecdb
 * 操作 2：将一个 现有 字符的每次出现转换为另一个 现有 字符，并对另一个字符执行相同的操作。
 * 例如，aacabb -> bbcbaa（所有 a 转化为 b ，而所有的 b 转换为 a ）
 * 你可以根据需要对任意一个字符串多次使用这两种操作。
 *
 * 给你两个字符串，word1 和 word2 。如果 word1 和 word2 接近 ，就返回 true ；否则，返回 false 。
 * 
 * 解题思路核心在于找到满足的条件，有2:
 * 1、出现的字符数的比例需要一致。
 * 2、字符的种类需要一致
 * 满足这两个条件，则必定可以通过2个操作来达到要求的结果
 * 否则就不行
 * 
 */

 public boolean closeStrings(String word1, String word2) {
        if (word1.length() != word2.length()) return  false;

        int[] arr1 = new int[26];
        int[] arr2 = new int[26];

        for (char c : word1.toCharArray()) {
            arr1[c - 'a']++;
        }
        for (char c : word2.toCharArray()) {
            arr2[c - 'a']++;
        }
        for (int i = 0; i < 26; i++) {
            if (arr1[i] + arr2[i] == 0) continue;

            if (arr1[i]==0 || arr2[i] == 0) return false;
        }
        Arrays.sort(arr1);
        Arrays.sort(arr2);
        for (int i = 0; i < 26; i++) {
            if (arr1[i] != arr2[i]) return false;
        }

        return true;
    }
```

### 文章点评&观点输出
[为什么很多人宁愿 excel 贼 6，也不愿意去用 python？](https://www.zhihu.com/question/53261114/answer/2972862034)

习惯的力量。
一样东西用久了，想要再换很困难。
一样东西已经在这儿了，想把他换掉也需要很大的勇气。

以我自己为例，编程中用惯了的组件，即使我知道有更方便效率更高的其他组件，也是懒得去研究去做替代。

更何况，对于用excel的业务人员，本身看编程相关的内容就像看天书，让他换python岂不是比登天还难。

但是说实话，这样的本性不好。反映出来有几点：一是不愿意接受新事物，对于未知的东西总是恐惧，不愿意尝试。二是浮躁，引入一些新东西，就是对旧东西的破坏。

### 结语
第一次参与这个打卡，但是苦于没有素材，不知道能写什么。甚至这个观点输出，都是临时找的。

其实平时看的文章，视频非常多，但是没有记录，看了也是相当于白看。

希望这个一周一次的打卡，可以帮助我做好记录。