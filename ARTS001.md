# ARTS 第一周

> 每周完成一个ARTS： 每周至少做一个 leetcode 的算法题、阅读并点评至少一篇英文技术文章、学习至少一个技术技巧、分享一篇有观点和思考的技术文章。（也就是 Algorithm、Review、Tip、Share 简称ARTS）

## Algorithm

> 开始之前先推荐一下LeetCode刷题神器 [vscode-leetcode](https://marketplace.visualstudio.com/items?itemName=shengchen.vscode-leetcode) ，见名知义，这是一个vscode插件，具体用法见链接文档。

LeetCode20，这是一道难度为easy的题目，给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：
左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。

注意空字符串可被认为是有效字符串。

**时间复杂度:O(N)**

**空间复杂度: O(N)** 

```java
class Solution {
    String v = "<[{()}]>";
    int len = v.length();
    int mid = len / 2;
    int high = len - 1;

    public boolean isValid(String s) {
        int[] stack = new int[s.length()];
        int r = 0;
        for (char c : s.toCharArray()) {
            int x = v.indexOf(c);
            if (x > -1 && x < mid) {
                stack[r++] = x;
            } else if (x > -1) {
                if (r == 0)
                    return false;
                if ((stack[r - 1] + x) != high) {
                    return false;
                } else {
                    r--;
                }
            } else {
                return false;
            }
        }
        return r == 0;
    }
}
```



## Review

[Cheat Sheets for AI, Neural Networks, Machine Learning, Deep Learning & Big Data](https://becominghuman.ai/cheat-sheets-for-ai-neural-networks-machine-learning-deep-learning-big-data-678c51b4b463)

这是一篇关于人工智能、神经网络、机器学习、深度学习、大数据的检查表的集合。作者介绍了以上各个部分的重要算法列表与概念，列出各种该领域的重要程序库与框架，Scikit-Learn，Numpy，SciPy，TensorFlow…等。通过该文章，能够较全面的了解该领域的知识图谱，从而指导具体的学习，避免盲人摸象。



[A Bird’s-Eye View on Java Concurrency Frameworks](https://dzone.com/articles/a-birds-eye-view-on-java-concurrency-frameworks-1) 

本文介绍评估分析了Java中几个常用的并发框架(ExecutorService、RxJava、Disruptor、Akka)功能与异同点，并介绍了各个框架的使用场景。合适的工具就应该在合适的场景中使用，深入了解是为了在某个场景中能够快速的拿出合适的方案。本文也有中文翻译:[Java 并发框架全览，这个牛逼！](http://www.10tiao.com/html/27/201903/2650494868/3.html)。ps：此文起先是在某个公众号上看到，本想转载作为分享的。然后去找源头，结果发现是篇翻译，找到了英文原文。

## Tip

看了耗子哥的 [**打造高效的工作环境 – SHELL 篇**](https://coolshell.cn/articles/19219.html), 在之前使用[**oh-my-zsh**](https://ohmyz.sh/)基础之上加装了 [**zsh-autosuggestions**](https://github.com/zsh-users/zsh-autosuggestions)插件之后相当好用，一些常用的历史命令能够自动提示，非常的好。

对于macOS用户来讲，[alfred](https://www.alfredapp.com/)是一个非常好用的工具，不论是启动程序还是一些快捷操作都能做到非常的高效，特别是购买了Powerpack之后使用 可以找到各类workflow [Workflows List](http://alfredworkflow.com/)快捷使用，最重要的是你可以通过简单编程自定义功能。

## Share

[MySQL通过frm、ibd文件恢复innodb数据](https://zhuanlan.zhihu.com/p/60327406)