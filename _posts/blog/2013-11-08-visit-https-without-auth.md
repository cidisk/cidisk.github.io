---
layout: post
title: 使用apache-commons httpclient 4.3.X访问https安全认证链接
description: 需要使用httpclient访问一些需要认证的HTTPS安全链接，却发现网上基本都是在者需要手工安装证书的实现或者是基于Httpclient 3.X老版本的无需认证实现。本文将对https连接进行介绍，并给出使用httpclient 4.3.X的通用无需证书代码实现。
category: blog
published: true
---

##HTTPS介绍

HTTPS（Hypertext Transfer Protocol over Secure Socket Layer），字面上就是SSL上的HTTP，安全套接层上的超文本转移协议，也就是HTTP下加入SSL层，加密的详细内容就需要SSL。SSL (Secure Socket Layer)最早为Netscape提出，利用数据加密技术，可确保数据在网络上之传输过程中不会被截取及窃听。它已被广泛地用于Web浏览器与服务器之间的身份认证和加密数据传输。SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。

显而易见，现在HTTPS已被广泛接受为万维网上安全通讯标准，基本上所有的电子交易和支付都基于HTTPS。

HTTPS协议相比较于HTTP，其服务器需要到专门的认证机构申请证书，通常来说这些证书都是收费的。HTTPS因为建立在SSL之上，所以其构建的链接需要进过身份认证并且传输的信息都是经过加密的，HTTPS默认使用443端口。因为在HTTPS协议上传输的信息就都是加密的，即使被黑客截获，也没有任何意义，因为他没有密钥，篡改也就没有意义了。

HTTPS的主要作用可以分为两种：一种是建立一个信息安全通道，来保证数据传输的安全；另一种就是确认网站的身份，防止伪造。

通常的https，就是服务器有一个证书，目的是保证服务器就是防止服务器被伪造，因为证书是被唯一颁发给这个服务器的。但是在少许情况下，服务器也会要求客户端必须有一个证书，即对客户的身份也要进行认证。因为个人证书一般来说是别人无法伪造的，所有这样能够确认自己的身份，保证用户访问的安全性。目前通常只有网银是这种做法，具体证书可能是拿U盘（即U盾）作为一个备份的载体。

##Java中访问Https的两种方法

JSSE，即Java Secure Socket Extension，Java安全套接扩展。JSSE是基于安全算法和握手机制之上的合成体，极大减轻了开发者的负担，使得开发者可以很轻松的将SSL整合到程序中。

基于上文中是否要求客户端提供证书的前提，访问HTTPS站点有两种方法，一种方法是把该证书导入到Java的TrustStore文件中，另一种是自己实现并覆盖JSSE缺省的证书信任管理器类。两种方法各有优缺点，第一种方法不会影响JSSE的安全性，但需要手工导入证书；第二种方法使用起来简单方便，不用手工导入证书，对于是否要求证书的网站均可直接访问，但使用时必须格外小心，毕竟要求证书是出于安全性考虑，而这种方法直接绕过了证书，会带来一些安全隐患。

因为httpclient4.3.X的api已经发生了较大变化，网上一番搜索没发现新版本httpclient的无需认证的实现，本人查阅文档和调试，下面就直接给出方便通用的第二种无需导入证书的方法实现。即通过自定义实现JSSE中证书信任管理器类接口X509TrustManager，让它信任所有的证书，当然包括不要求证书的情况。
       
##完整代码

下面内容只是一个简单的实例代码，可根据需要修改。
	
	package cf.hadoop.crawler;

	import java.io.BufferedReader;
	import java.io.InputStream;
	import java.io.InputStreamReader;
	import java.security.cert.X509Certificate;
	import java.util.Arrays;

	import javax.net.ssl.SSLContext;
	import javax.net.ssl.TrustManager;
	import javax.net.ssl.X509TrustManager;
	import org.apache.http.HttpResponse;
	import org.apache.http.client.HttpClient;
	import org.apache.http.client.config.AuthSchemes;
	import org.apache.http.client.config.CookieSpecs;
	import org.apache.http.client.config.RequestConfig;
	import org.apache.http.client.methods.HttpPost;
	import org.apache.http.config.Registry;
	import org.apache.http.config.RegistryBuilder;
	import org.apache.http.conn.socket.ConnectionSocketFactory;
	import org.apache.http.conn.socket.PlainConnectionSocketFactory;
	import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
	import org.apache.http.impl.client.HttpClients;
	import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;

	public class HttpsTest {
		public static void main(String[] args) throws Exception {
		
			String url = "https://isp.housingauthority.gov.hk/ispWeb/?locale=zh_HK";
			
            //实现X509TrustManager接口，如下三个方法为必须实现，将其实现为什么都不做
			X509TrustManager tm = new X509TrustManager() {
				public void checkClientTrusted(X509Certificate[] xcs, String string) {
				}

				public void checkServerTrusted(X509Certificate[] xcs, String string) {
				}

				public X509Certificate[] getAcceptedIssuers() {
					return null;
				}
			};
			
            //用自定义X509TrustManager初始化
			SSLContext sslcontext = SSLContext.getInstance("TLS");
			sslcontext.init(null, new TrustManager[] { tm }, null);
			//新建SSLConnectionSocketFactory
			SSLConnectionSocketFactory socketFactory = new SSLConnectionSocketFactory(
					sslcontext, SSLConnectionSocketFactory.STRICT_HOSTNAME_VERIFIER);

			// //////////////////
			RequestConfig defaultRequestConfig = RequestConfig
					.custom()
					.setCookieSpec(CookieSpecs.BEST_MATCH)
					.setExpectContinueEnabled(true)
					.setStaleConnectionCheckEnabled(true)
					.setTargetPreferredAuthSchemes(
							Arrays.asList(AuthSchemes.NTLM, AuthSchemes.DIGEST))
					.setProxyPreferredAuthSchemes(Arrays.asList(AuthSchemes.BASIC))
					.build();
			// //////////////////
			Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder
					.<ConnectionSocketFactory> create()
					.register("http", PlainConnectionSocketFactory.INSTANCE)
					.register("https", socketFactory).build();
			PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager(
					socketFactoryRegistry);
			HttpClient httpClient = HttpClients.custom()
					.setConnectionManager(connManager)
					.setDefaultRequestConfig(defaultRequestConfig).build();
			// //////////////////
			// 获得HttpGet对象
			HttpPost post = new HttpPost(url);
			// 发送请求
			HttpResponse response = httpClient.execute(post);
			// 输出返回值
			InputStream is = response.getEntity().getContent();
			BufferedReader br = new BufferedReader(new InputStreamReader(is));
			String line = "";
			while ((line = br.readLine()) != null) {
				System.out.println(line);
			}
		}
	}
