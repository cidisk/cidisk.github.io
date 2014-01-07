---
layout: post
title: "使用apache-commons httpclient 4.3.X访问https安全认证链接"
description: 需要使用httpclient访问一些需要证书的安全链接，却发现网上基本都是在老的3.X版本的httpclient上的实现。
category: blog
published: true
---

占位编辑 

##完整代码

下面内容只是一个简单的实例代码，可根据需要修改。
	
    package com.ibm.bluemix.hamm.crawler;

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
		
			X509TrustManager tm = new X509TrustManager() {
				public void checkClientTrusted(X509Certificate[] xcs, String string) {
				}

				public void checkServerTrusted(X509Certificate[] xcs, String string) {
				}

				public X509Certificate[] getAcceptedIssuers() {
					return null;
				}
			};

			SSLContext sslcontext = SSLContext.getInstance("TLS");

			sslcontext.init(null, new TrustManager[] { tm }, null);

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