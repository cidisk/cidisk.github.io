---
layout: post
title: 计算MD5值
description: MD5是常用的hash方法，它可以有效的进行用于认证或者作为特定范围内数据的唯一标识，这篇文章记录一个实现，用于备用。
category: blog
published: true
---

在各种应用系统的开发中，经常需要存储用户信息，很多地方都要存储用户密码，而将用户密码直接存储在服务器上显然是不安全的，如何进行相对安全的身份认证;另外一个常用场景是在文件传输时或文件校验时。在工作中的情形我们常用MD5加密算法。

MD5: Message-Digest Algorithm 5(信息摘要算法)缩写，广泛用于加密和解密技术，常用于文件校验。不管文件多大，经过MD5后都能生成唯一的MD5值。一个消息摘要就是一个数据块的数字指纹。即对一个任意长度的一个数据块进行计算，产生一个唯一指印。消息摘要是一种与消息认证码结合使用以确保消息完整性的技术。主要使用单向散列函数算法，可用于检验消息的完整性，和通过散列密码直接以文本形式保存等。

消息摘要有两个基本属性：
1. 两个不同的报文难以生成相同的摘要
2. 难以对指定的摘要生成一个报文，而可以由该报文反推算出该指定的摘要

也就是说MD5算法可以生产不可逆的唯一指纹。尽管山东大学王小云教授关于MD5破解的研究取得了极大进展，并成功找出了通过已加密MD5伪造身份的方法，从而在法律意义上推翻了MD5作为身份指纹的合理性，但是MD5在当前工程环境中仍然是相对安全的。

那么MD5校验怎么用呢？当然是把当前信息经过MD5算法后产生MD5的值和已存储的MD5值进行比较。现在在网上下载的ISO等镜像文件，都会附带一个原始镜像生成的md5文件以和当前下载镜像进行校验，以防止被篡改植入病毒或文件损坏。

在Java中，java.security.MessageDigest(rt.jar中)已经定义了 MD5 的计算，所以我们只需要简单地调用即可得到MD5的128位整数。然后将此128位计16个字节转换成 16进制表示即可,计算其64位MD5已取中法获得。

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