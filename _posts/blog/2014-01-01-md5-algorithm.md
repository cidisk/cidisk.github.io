---
layout: post
title: 计算MD5值
description: MD5是常用的hash方法，它可以有效的进行用于认证或者作为特定范围内数据的唯一标识，这篇文章记录一个实现，用于备用。
category: blog
published: true
---

在各种应用系统的开发中，经常需要存储用户信息，很多地方都要存储用户密码，而将用户密码直接存储在服务器上显然是不安全的，本文简要介绍工作中常用的MD5加密算法，希望能抛砖引玉。

一个消息摘要就是一个数据块的数字指纹。即对一个任意长度的一个数据块进行计算，产生一个唯一指印（对于SHA1是产生一个20字节的二进制数组）。消息摘要是一种与消息认证码结合使用以确保消息完整性的技术。主要使用单向散列函数算法，可用于检验消息的完整性，和通过散列密码直接以文本形式保存等，目前广泛使用的算法有MD4、MD5、SHA-1。

消息摘要有两个基本属性：

1. 两个不同的报文难以生成相同的摘要
2. 难以对指定的摘要生成一个报文，而可以由该报文反推算出该指定的摘要

在Java中，java.security.MessageDigest （rt.jar中）已经定义了 MD5 的计算，所以我们只需要简单地调用即可得到MD5的128位整数。然后将此128位计16个字节转换成 16进制表示即可。 

下面先贴出代码，方便以后使用。

    import java.security.MessageDigest;
    import java.security.NoSuchAlgorithmException;
    
    public class MD5 {
        private final static String[] HEX = { "0", "1", "2", "3", "4", "5", "6",
                "7", "8", "9", "a", "b", "c", "d", "e", "f" };
    
        private static String byteArrayToHEX(byte[] b) {
            StringBuffer buf = new StringBuffer();
            for (int i = 0; i < b.length; i++) {
                buf.append(byteToHEX(b[i]));
            }
            return buf.toString();
        }
    
        private static String byteToHEX(byte b) {
            int n = b;
            if (n < 0) {
                n += 256;
            }
            int d1 = n / 16;
            int d2 = n % 16;
            return HEX[d1] + HEX[d2];
        }
    
        public static String calcualte128BITMD5(String text) {
            try {
                MessageDigest md = MessageDigest.getInstance("MD5");
                md.update(text.getBytes());
                return byteArrayToHEX(md.digest());
            } catch (NoSuchAlgorithmException e) {
                e.printStackTrace();
                return null;
            }
    
        }
    
        public static String calculate64BITMD5(String text) {
            String MD5128bit = calcualte128BITMD5(text);
            if (MD532bit != null) {
                String MD564bit = MD5128bit.substring(8, 24);
                return MD564bit;
            } else {
                return null;
            }
        }
    }