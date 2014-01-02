---
layout: post
title: 计算MD5值
description: MD5是常用的hash方法，它可以有效的进行用于认证或者作为特定范围内数据的唯一标识，这篇文章记录一个实现，用于备用。
category: blog
---

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

	public static String calcualte32BITMD5(String text) {
		try {
			MessageDigest md = MessageDigest.getInstance("MD5");
			md.update(text.getBytes());
			return byteArrayToHEX(md.digest());
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
			return null;
		}

	}

	public static String calculate16BITMD5(String text) {
		String MD532bit = calcualte32BITMD5(text);
		if (MD532bit != null) {
			String MD516bit = MD532bit.substring(8, 24);
			return MD516bit;
		} else {
			return null;
		}
	}
}

