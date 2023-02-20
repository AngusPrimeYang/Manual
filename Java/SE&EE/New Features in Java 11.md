### 目前EOS進程

<table>
<tbody>
	<tr data-sort-method="none">
		<th role="columnheader">Version</th>
		<th role="columnheader">Release</th>
		<th role="columnheader">Active Support</th>
		<th role="columnheader">Security Support</th>
	</tr>
	<tr>
		<td> 8 </td>
		<td> 18 Mar 2014 </td>
		<td> 31 Mar 2022 </td>
		<td> 31 Dec 2030 </td>
	</tr>
	<tr>
		<td> 11 </td>
		<td> 25 Sep 2018 </td>
		<td> 30 Sep 2023 </td>
		<td> 30 Sep 2026 </td>
	</tr>
	<tr>
		<td> 17 </td>
		<td> 14 Sep 2021 </td>
		<td> 30 Sep 2026 </td>
		<td> 30 Sep 2029 </td>
	</tr>
</tbody>
</table>

https://endoflife.date/java

>java將issue命名為JDK Enhancement Proposals(JEPs)，各issue正式表示方法如JEP-181，<br />
可以對應到文件https://openjdk.org/jeps/181，確認詳細說明<br />
以下為列表及簡易說明<br />
跟一般工程師撰寫比較有關的部分，將以 **粗體** 表示<br />
**整體來看，JAVA 9, 10, 11是最大的變革，有辦法使用11後，後續升級相對衝擊小很多**<br />

--


**181: Nest-Based Access Control**

<pre>
	JAVA改變了編譯方式，inner class變可被完整授權
	public class Test {
    
		public static void main(String[] args)
		{
			Inner inner = new Inner();
			inner.setValue(123); //valid
			System.out.println(inner.getValue()); //123
		}
		
		private static class Inner {
			private int value;

			public int getValue() {
				return value;
			}

			private void setValue(int value) {
				this.value = value;
			}
		}
	}
</pre>

309: Dynamic Class-File Constants
315: Improve Aarch64 Intrinsics
	
<pre>
	進一步增加x64效能，補足Math.sin(), Math.cos() and Match.log()支援
</pre>
	
318: Epsilon: A No-Op Garbage Collector
	
<pre>
	外掛Garbage Collector，heap用完JVM會直接shut down
	-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC
</pre>
	
**320: Remove the Java EE and CORBA Modules**
	
<pre>
	1.內部 API不開放存取，許尋找新的程式碼使用，或著使用 --add-exports 或 --add-opens 開放
	2.減肥，部分Modules拔掉變成三方元件(deprecated: 9, removal: 17)
	Removed packages:
		java.xml.ws (JAX-WS)
		java.xml.bind (JAXB)
		java.activation (JAF)
		java.xml.ws.annotation (Common Annotations)
		java.corba (CORBA)
		java.transaction (JTA)
		java.se.ee (Aggregator module for the six modules above)
	Removed Tools:
		wsgen and wsimport (from jdk.xml.ws)
		schemagen and xjc (from jdk.xml.bind)
		idlj, orbd, servertool, and tnamesrv (from java.corba)
</pre>
	
**321: HTTP Client (Standard)**
	
<pre>
	引入新工具java.net.http.*(如java.net.http.HttpClient)!!!!!!!
	
	java.net.http.HttpClient httpClient = java.net.http.HttpClient.newBuilder()
		.version(java.net.http.HttpClient.Version.HTTP_1_1)
		.connectTimeout(java.time.Duration.ofSeconds(10))
		.build();
	java.net.http.HttpRequest request = java.net.http.HttpRequest.newBuilder()
		.GET()
		.uri(java.net.URI.create("https://httpbin.org/get"))
	//		            .setHeader("User-Agent", "Java 11 HttpClient Bot")
		.setHeader("Charset", "utf-8")
		.setHeader("Content-Type", "text/html")
		.setHeader("Connection", "Keep-Alive")
		.setHeader("Cache-Control", "no-cache")
		.build();

	java.net.http.HttpResponse<String> response =
		  httpClient.send(request, java.net.http.HttpResponse.BodyHandlers.ofString());

	java.net.http.HttpHeaders headers = response.headers();
	headers.map().forEach((k, v) -> System.out.println(k + ":" + v));

	System.out.println(response.statusCode());
	System.out.println(response.body());
</pre>
	
**323: Local-Variable Syntax for Lambda Parameters**
	
<pre>
	lambda中的變數類型可使用var(本來就可以指定具體型別如Integer)
	可再搭配lambda annotations(如@NotNull)使用
	上述功能都只是增加程式可讀性，並不影響本來的運作
	使用上請所有變數一致，var就全部var，不要混到具體型別
	String result = list.stream()
          .map((@NotNull var x, var y) -> x.toUpperCase() + y)
          .collect(Collectors.joining(","));
</pre>
	
**324: Key Agreement with Curve25519 and Curve448**
	
<pre>
	新支援ECDH的加密演算模組，Curve25519 and Curve448
		KeyPairGenerator kpg = KeyPairGenerator.getInstance("XDH");
		NamedParameterSpec paramSpec = new NamedParameterSpec("X25519");
		kpg.initialize(paramSpec); // equivalent to kpg.initialize(255)
		// alternatively: kpg = KeyPairGenerator.getInstance("X25519")
        KeyPair kp = kpg.generateKeyPair();
		
		System.out.println("--- Public Key ---");
        PublicKey publicKey = kp.getPublic();
		
        System.out.println("--- Private Key ---");
        PrivateKey privateKey = kp.getPrivate();
</pre>
	
**327: Unicode 10**
	
328: Flight Recorder
	
<pre>
	釋出Java Flight Recorder (JFR) 到openJDK
	-XX:StartFlightRecording=duration=60s,settings=profile,filename=app.jfr MyHelloWorldApp
</pre>
	
329: ChaCha20 and Poly1305 Cryptographic Algorithms
	
<pre>
	Cipher cipher = Cipher.getInstance("ChaCha20");
    ChaCha20ParameterSpec param = new ChaCha20ParameterSpec(nonce, counter);
    cipher.init(Cipher.ENCRYPT_MODE, key, param);
    byte[] encryptedText = cipher.doFinal(pText);
	
	Cipher cipher = Cipher.getInstance("ChaCha20-Poly1305");
	// IV, initialization value with nonce
	IvParameterSpec iv = new IvParameterSpec(nonce);
	cipher.init(Cipher.ENCRYPT_MODE, key, iv);
	byte[] encryptedText = cipher.doFinal(pText);
</pre>
	
**330: Launch Single-File Source-Code Programs**
	
<pre>
	javac HelloJava.java
	java HelloJava
	
	->
	
	java HelloJava.java
	
	以及
	
	Linux Shebang #!
	支援載有Shebang的檔案路徑作為編譯的參數
</pre>
	
331: Low-Overhead Heap Profiling
	
<pre>
	For Virtual Machine Tool Interface (JVMTI) (introduced: 5)
</pre>
	
**332: Transport Layer Security (TLS) 1.3**
	
<pre>
	SSLSocketFactory factory =
		(SSLSocketFactory) SSLSocketFactory.getDefault();
	socket =
		(SSLSocket) factory.createSocket("google.com", 443);
	socket.setEnabledProtocols(new String[]{"TLSv1.3"});
	socket.setEnabledCipherSuites(new String[]{"TLS_AES_128_GCM_SHA256"});
</pre>

333: ZGC: A Scalable Low-Latency Garbage Collector(Experimental)
	
<pre>
	Z Garbage Collector (ZGC) 低延遲GC(<10ms)
	-XX:+UnlockExperimentalVMOptions -XX:+UseZGC
	
	Java 11 support only on Linux/64
	Java 14 support on macOS and Windows
	Java 15 product feature
	
	https://docs.oracle.com/en/java/javase/11/gctuning/z-garbage-collector1.html
	可能需要tuning
</pre>

335: Deprecate the Nashorn JavaScript Engine
	
<pre>
	introduced: 8, deprecated: 11, removal: 15
</pre>
	
336: Deprecate the Pack200 Tools and API
	
<pre>
	deprecated: 11, removal: 14
</pre>
	
--

reference

https://openjdk.org/projects/jdk/11/<br />
https://mkyong.com/java/what-is-new-in-java-11/<br />
https://www.oracle.com/java/technologies/javase/11all-relnotes.html<br />
https://learn.microsoft.com/zh-tw/java/openjdk/transition-from-java-8-to-java-11<br />

