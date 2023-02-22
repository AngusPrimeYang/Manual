## About Version
based on [Maven](https://mvnrepository.com/artifact/org.springframework/spring-core)

<table>
		<tbody><tr data-sort-method="none">
			<th role="columnheader">SpringFramework Ver.</th>
			<th role="columnheader">Java Ver.</th>
			<th role="columnheader">Direct vulnerabilities solved Ver.</th>
			<th role="columnheader">Direct vulnerabilities solved Date</th>
		</tr>
		<tr>
			<td> 5.2 </td>
			<td> support 11 </td>
			<td> 5.2.22.RELEASE </td>
			<td> May 11, 2022 </td>
		</tr>
		<tr>
			<td> 5.3 </td>
			<td> support 17 </td>
			<td> 5.3.20 </td>
			<td> May 11, 2022 </td>
		</tr>
		<tr>
			<td> 6 </td>
			<td> baseline 17 </td>
			<td> 6.0.0 </td>
			<td> Nov 16, 2022 </td>
		</tr>
	</tbody>
</table>

--

## About lib

* basic lib

spring-aop ==> used by spring-webmvc<br />
spring-beans ==> used by spring-webmvc<br />
spring-context ==> used by spring-webmvc<br />
spring-context-support ==> unnecessary<br />
spring-core => used by spring-webmvc<br />
spring-expression ==> used by spring-webmvc<br />
spring-jdbc ==> unnecessary<br />
spring-orm ==> unnecessary，org.springframework.orm<br />
spring-test ==> unnecessary，org.springframework.mock<br />
spring-tx ==> unnecessary<br />
spring-web ==> used by spring-webmvc<br />
spring-webmvc

result

>&lt;dependency&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;org.springframework&lt;/groupId&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;spring-core&lt;/artifactId&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;version&gt;${springframework.version}&lt;/version&gt;<br />
>&lt;/dependency&gt;<br />
>&lt;dependency&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;org.springframework&lt;/groupId&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;spring-web&lt;/artifactId&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;version&gt;${springframework.version}&lt;/version&gt;<br />
>&lt;/dependency&gt;<br />
>&lt;dependency&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;org.springframework&lt;/groupId&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;spring-webmvc&lt;/artifactId&gt;<br />
>&nbsp;&nbsp;&nbsp;&nbsp;&lt;version&gt;${springframework.version}&lt;/version&gt;<br />
>&lt;/dependency&gt;

* other Spring lib

spring-aspects ==> unnecessary，AspectJ<br />
spring-instrument ==> unnecessary<br />
spring-jms ==> unnecessary，jms<br />
spring-mobile-device-1.0.1.RELEASE.jar ==> unnecessary<br />
spring-oxm ==> unnecessary，org.springframework.oxm<br />
spring-struts-3.2.17.RELEASE.jar ==> unnecessary after 4<br />
spring-webmvc-portlet ==> unnecessary after 5<br />

spring-boot-maven-plugin ==> update to at least 2.0.3.RELEASE after 5


* other non Spring lib

xalan ==> update to at least 2.6.0<br />
xalan.serializer ==> update to at least 2.7.1 after 4 ==> update to at least 2.7.2 after 5<br />
saxon-dom ==> keep saxon-dom-9.0 after 4 ==> unnecessary after 5<br />
commons-fileupload ==> update to at least 1.3.3 after 5<br />
aspectjrt ==> unnecessary after 5<br />
aspectjtools ==> unnecessary after 5<br />
aspectjweaver ==> update to at least 1.9.1 after 4 ==> unnecessary after 5<br />
jackson-core ==> change lib to **com.fasterxml**(compatible with wildfly)，or update to at least 2.9.7 after 5.2<br />
jackson-annotation ==> change lib to **com.fasterxml**(compatible with wildfly)，or update to at least 2.9.7 after 5.2<br />
jackson-databind ==> change lib to **com.fasterxml**(compatible with wildfly)，or update to at least 2.9.7 after 5.2

xml-apis ==> maybe leak of package(base on version)，add 1.4.01 to fill up the vacancy

--

## About code

based on [wiki](https://github.com/spring-projects/spring-framework/wiki/What%27s-New-in-Spring-Framework-5.x)

自定義程式部分，可以
* 尋找所有使用到 import org.springframework.* 的地方，比對wiki替換
* 所有lib換上去讓他紅，比對wiki替換
