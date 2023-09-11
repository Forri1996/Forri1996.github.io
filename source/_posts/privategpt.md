---
title: 如何基于私有问答对微调大模型
date: 2023-09-11 11:16:30
tags: 私有化大模型
---

为了实现私人的问答助手，主要有几种思路：
1. prompt提示词工程。将私有知识库作为prompt发给gpt。

优点：实现简单

缺点：私有知识库内容如果超出token限制，就无法玩转了。

2. promot提示词工程Plus。将超长私有知识库分片后，再发给gpt。相当于是任务分片。

对于这种方案，有两种场景。一是分片之间没有关联性，二是分片之间存在关联性。

第一种场景比较简单，就是N个任务的重复。第二种场景相对复杂，一般参考滑动任务窗口的方案来解决

优点：实现相对来说比较简单

缺点：使用不方便，效果不一定好。性能也很差。

参考：[如何解决超长文本gpt](https://zis0qwtriqo.feishu.cn/wiki/WB1LwGSNhiu0lok2pqWcRyFXn5A)

3. 大模型微调
https://blog.csdn.net/taoshihan/article/details/129053834

https://zhuanlan.zhihu.com/p/632882735

https://zhuanlan.zhihu.com/p/625794605

https://xie.infoq.cn/article/c73ff709a5e632d65010f1d27
4. embedding向量数据库
