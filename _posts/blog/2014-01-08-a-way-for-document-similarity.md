---
layout: post
title: 计算两个文档的相似度的一种思路
description: 看到之前的一种计算思路，先记录下来，再慢慢来增加和细化。
category: blog
---

###计算两个文档的相似度的一种思路：

1.构建K-shingle集合

2.Hash 压缩，标准化

3.再次Hash生成最小hash签名矩阵

4.用Locality-Sensitive Hash 把最小hash签名矩阵降维

5.计算相似度。

算法中间一共有5个参数要设置：K，最小hash签名的长度n，LSH降维中的行带宽度b，行带中的列的条数r，对每个行带hash计算结果的一致性判断threshold t。