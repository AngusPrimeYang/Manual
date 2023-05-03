### 目的

* 系統java運行環境升到java 8的最新版，並把wildfly升到23

* 在已穩定運作下可以再升級至java11，或透過進階章節升級到24+，最後可以升級至java17

--

### 使用範圍

* 最低可以接受範圍從java7&jboss7開始，一路升到java8&wildfly23， *java11+、24+為進階升級選項，主要在安全性的全面提升，請視需要使用。*

* 不贅述java7升java8的細節，因為都是容易追溯作法的程式改寫工作，尤其java架構最主要的差異在java6升java7，許多寫法在java7就已經邁向棄用，到了java8後正式停用，要說最大的改變在6升7也不能說錯。總之就是升上去透過IDE的proposals支援改寫法，把所有problem解光就對了。

* 在2022年大部分產品已經成熟轉移到java11，eclipse、wildfly也以java11為主流在開發，2022下半年後大量vulnerability也已解掉大部分，網路對外的產品建議整體上去到至少java11環境。

![image](https://user-images.githubusercontent.com/30335993/235822874-5ed68445-e956-408d-812f-120e94573624.png)

--

### 關於JAVA

* 下載OpenJDK請至adoptium計畫網站取得最新版(以java8為例): https://adoptium.net/temurin/releases/?version=8

* 請注意安裝位置預設在Program Files\Eclipse Adoptium
* 安裝時可以直接設JAVA_HOME，或後續自行增加環境變數

* MS SQLServer，JAVA在一定版本後會強制要求使用TLS1.2連線，請務必確保資料庫有升級到已支援版本，不然站台會打不開，
說明: https://support.microsoft.com/en-us/topic/kb3135244-tls-1-2-support-for-microsoft-sql-server-e4472ef8-90a9-13c1-e4d8-44aad198cdbe

* 如果有對java的檔案做過調整(最常見會動到的是cacerts)請務必記得同步過去，關於java keytool 的import在此不贅述。

--

### 關於Wildfly

* 下載請至Wildfly官方網站取得最新版: https://www.wildfly.org/downloads/

* 如果原專案有針對啟動時帶的JAVA_OPTS做調整，請記得繼承。

* 依照模式更新設定，以standalone mode為例

> 1. 設定datasources的datasource，最好保留新的結構，僅更新以下數值：
> > * 屬性jndi-name，請改寫成java:jboss/datasources/xxxxxxxDS
> > * 屬性pool-name，直接繼承
> > * 節點connection-url，直接繼承
> > * 節點security，如果沒有針對密碼做加密，不需改動， *有做security-domain或Vault的話直接繼承，否則請引進Elytron技術*
> > * 其他有客製化的節點如min-pool-size與max-pool-size等
> > * 注意statistics-enabled屬性，這決定wildfly是否啟用運行環境分析，standalone預設為false不分析
> 2. 設定datasources的driver，請升級至sqljdbc4.2+
> 3. 調整完datasources後，在jboss:domain:ee這個model中有一個<default-bindings>，注意不要對到不存在的datasource
> 4. jboss:domain:io如果有做過tuning，直接複製過來就可以。此部分主要在設定<worker>的task-max-threads跟io-threads屬性，決定向CPU發起的thread數量，增加wildfly的運算能力
> 5. jboss:domain:undertow為wildfly重要設計，如果從wildfly升級，基本上直接複製過來就可以，其中：
> > * <https-listener>跟<http-listener>，如果站台無法支援http2，務必把 enable-http2="true" 拿掉，減少檢誤時間
> > * <https-listener>的cipher-suites至少設定為以下強度，此組合支援2010/02以後的所有常用系統＋瀏覽器組合(至少win7+ie11)，若沒有要這麼久遠，請拿掉其中的TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
<pre><code class="xml">
&lt;https-listener name="httpsServer" enabled-protocols="TLSv1.2" enabled-cipher-suites="TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256" security-realm="ApplicationRealm" socket-binding="https"/&gt;
</code></pre>
> > * *Wildfly 24+結構不同，請參考wildfly24+升級文章*
> > * <host>中務必增加header防護，大致如下，其中Strict-Transport-Security的max-age不見得要那麼久，Content-Security-Policy請配合站台調整，X-Frame-Options最好是設為DENY
<pre><code class="xml">
&lt;host&gt;
&lt;filter-ref name="X-Content-Type-Options"/&gt;
&lt;filter-ref name="Content-Security-Policy"/&gt;
&lt;filter-ref name="X-XSS-Protection"/&gt;
&lt;filter-ref name="Strict-Transport-Security"/&gt;
&lt;filter-ref name="X-Frame-Options"/&gt;
&lt;/host&gt;
</code></pre>
<pre><code class="xml">
&lt;filtery&gt;
&lt;response-header name="X-Content-Type-Options" header-name="X-Content-Type-Options" header-value="nosniff"/&gt;
&lt;response-header name="Content-Security-Policy" header-name="Content-Security-Policy" header-value="default-src 'self'; media-src 'self' data:; img-src 'self' data:; script-src 'unsafe-eval' 'self' 'unsafe-inline'; style-src 'unsafe-inline' 'self'"/&gt;
&lt;response-header name="X-XSS-Protection" header-name="X-XSS-Protection" header-value="1"/&gt;
&lt;response-header name="Strict-Transport-Security" header-name="Strict-Transport-Security" header-value="max-age=31536000; includeSubDomains"/&gt;
&lt;response-header name="X-Frame-Options" header-name="X-Frame-Options" header-value="SAMEORIGIN"/&gt;
&lt;/filtery&gt;
</code></pre>
> 6. 設定SSL憑證。如果wildfly本來有設定ssl憑證，直接複製過來就可以， *Wildfly 24+結構不同，請參考wildfly24+升級文章*
![image](https://user-images.githubusercontent.com/30335993/235822935-2312678d-d3de-400f-a521-8e221c646240.png)
> 7. 設定session-cookie防護。增加http-only防護能力防護。另外如果有設定ssl，請增加secure確保session經過保護
![image](https://user-images.githubusercontent.com/30335993/235822951-6ad1883c-6698-45b1-b519-cda49db90779.png)
> 8. 控制公開/管理對外IP設定。請把interface的public(公開)address正式改為站台對外IP，wildfly才會開放以該IP接收連線，否則僅能在本機連線(可用於測試)，interface的management(管理)address即為網頁管理模式可使用的IP。另外，wsdl服務可獨立用wsdl-host節點管理可使用的IP。
![image](https://user-images.githubusercontent.com/30335993/235822962-0f29904b-12fe-4f62-91d8-22b2101fb770.png)
> 9. 注意socket-binding-group中http與https的port，是否與現行站台符合。
![image](https://user-images.githubusercontent.com/30335993/235822981-1d1bf9f5-cd8c-4b73-96e1-ac2e1bab9b58.png)
> 10. 依照需要延長啟動等待時間。每次啟動時，windows defender會針對war資料夾掃毒，可能導致等待過久最後逾時。有遇到此情形的話，可以在最初的<extensions>下增加系統設定區塊<system-properties>，再修改逾時時間jboss.as.management.blocking.timeout。
![image](https://user-images.githubusercontent.com/30335993/235822994-715f8d26-42a4-4ab9-9df4-805e28168e78.png)
> 或直接加在JAVA_OPTS也可以
<pre><code class="shell">
set "JAVA_OPTS=%JAVA_OPTS% -Djboss.as.management.blocking.timeout=900"
</code></pre>
> 11. wildfly將logger設定拉出來還給了在configuration資料夾下logging.properties，作為standalone/domain全域設定位置，請注意好server.log的路徑是否符合wildfly資料夾所在位置
![image](https://user-images.githubusercontent.com/30335993/235823038-282e75b3-55ee-495a-9347-123b7a780b6a.png)
> 一般專案都有自己的log4j.properties，如果本來就使用wildfly管理logger，請繼續沿用。 *建議將自定義logger合併給Wildfly管理，就不需要煩惱log4j跟log4j2之間的gap*
> 如果要直接使用Wildfly管理(即透過jboss:domain:logging管理)，須關閉use-deployment-logging-config屬性，或刪除專案中的自定義properties。關閉擴充自定義logger方法如下
![image](https://user-images.githubusercontent.com/30335993/235823032-e26818ea-e97d-4de2-966c-5fd4a02e0492.png)
> Wildfly預設自動掃描使用者自定義logger設定，掃描區域為 META-INF與WEB-INF/classes，且為以下名稱之一
<pre>
logging.properties
jboss-logging.properties
log4j.properties
log4j.xml
jboss-log4j.xml
</pre>

--

### 關於Java與Wildfly整合

* java與wildfly升級期間陸陸續續針對Hibernate做了許多調整，後來針對EJB的Lookup方法做了修補，另外許多東西用法變得更嚴謹、更有效率，近幾年(2020~2022)JAVA發生了不少重大資安事件，也是升級要點之一。

> 1. 承【關於Wildfly】，請在persistence.xml的jta-data-source節點，增加jboss/datasources/前置
![image](https://user-images.githubusercontent.com/30335993/235823050-aa15f4eb-7e5c-43ba-9c78-246d61aa04f9.png)
> 2. 新的開發環境其實對Java 7以下已經相當不友善(如eclipseg使用java17執行、專案使用java7)，舊語法一律都要改寫，請透過IDE提示協助逐一處理，其中特別注意
> > * Base64已整併進java內建函式java.util.Base64，不須到處掛lib
> > * 有繼承java.lang.AutoCloseable的串流請務必放進try()中處理
> 3. JAVA11以後，注意JAVA核心元件都已經須拆分自行引用，但所需的幾乎Wildfly都已自行提供，因此只要表明哪些會由Wildfly提供就好，如下圖所示，scope列為provided，maven打包時就不會再另外多包lib，執行時使用wildfly提供的lib取代就可以了
![image](https://user-images.githubusercontent.com/30335993/235823060-29ee1c37-4d4e-4aa1-8663-38218d509fad.png)
> 4. 承上，特別要提到javax元件(如javax.servlet、javax.xml)，已經不再內建於JAVA，但貿然直接升版三方元件很容易造成跟wildfly衝突， *因為三方元件都是基於某一版JAVA元件開發三方程式*，又 *JAVA元件在一般情況下都是向下支援的* ，建議的做法是僅保留第三方元件少數外掛，其他盡可能使用wildfly減少版本衝突。
> 如下，核心功能一律exclude掉，使用wildfly內建的去滿足，這樣可以直接避免掉所有會衝突的可能，最多是第三方元件沒有跟上JAVA，但在極端保守前進的情況下，基本上不會發生這種情形。
![image](https://user-images.githubusercontent.com/30335993/235823070-311cff88-0dfc-418d-94b4-286640a3ca24.png)
> 5. 承上，springframework與quartz一類重要元件，現在都有良好的向下支援，升級基本上不用擔心，確認好java版本對應的lib版本就可以了。
> 6. 承上，請記得使用新的maven元件，以良好相容，maven-ejb-plugin也記得採用3以上。
> 7. 如果有Wildfly已不支援或移除的元件，請從jboss-deployment-structure.xml移除，務必整理一遍。例如在程式已經完成將log4j升級至log4j2的情況下，需要確認是否排除wildfly的log4j(基本上就看誰舊排除誰)，排除路徑改變如下。
![image](https://user-images.githubusercontent.com/30335993/235823078-a6ad266a-9226-49a9-b7ed-bfc0f69e7c16.png)
> 8. EJB的部分，請也都直接採用wildfly元件provided，最小化衝突的可能，注意wildfly支援 *hibernate皆尚有vulnerability，直到5.4.24.Final才真的解決。*
> 9. 專案絕對建議從Log4j升到Log4j2，或著交還給wildfly管理，以解決重大資安事件後續帶來的影響。
> 如果專案都只是如下述方法在使用預設logger，恭喜你升級元件就好，只要log4j跟slf4j版本搭好，引用走org.apache.logging.log4j包，不需要再掛串接lib如slf4j-log4j12一類。
![image](https://user-images.githubusercontent.com/30335993/235823088-a2b83616-9bba-4000-afa1-5f0eb03eef1b.png)
![image](https://user-images.githubusercontent.com/30335993/235823089-7a582265-4a3a-4283-bd4f-465957c9f217.png)
> 10. 承上，如果有自定義Logger，建議改成用程式控制讀取的設定檔，減少被抓到的可能，以下為log4j讀取外部的log4j.properties(非站台啟動能抓到的位置)，在自行組完properties後設定至站台的方法， *log4j2的方法請參考下一段落。*
![image](https://user-images.githubusercontent.com/30335993/235823104-d7384b9f-9012-4522-bcea-7c48143dcc8b.png)
> 11. 承上，log4j2在properties做了不少規格異動，為與系統權限全面斷開，下方直接給予定義頁面，以及如何將log4j.properties中的DailyRollingFileAppender方法程式化，可參考
> https://logging.apache.org/log4j/2.x/manual/customconfig.html
<pre><code class="java">
	String appenderName = "rolling";
	String loggerName = "mail";
	String logDir = "/mail/2022/07/";
	ConfigurationBuilder<BuiltConfiguration> builder = ConfigurationBuilderFactory
.newConfigurationBuilder();
	//對應.layout
	LayoutComponentBuilder layoutBuilder = builder.newLayout("PatternLayout")
	//對應.layout.ConversionPattern
	    .addAttribute("pattern", "%d{MM/dd HH:mm:ss} %5p %C:%L - %m%n");
	//cron設定為每日
	ComponentBuilder triggeringPolicy = builder.newComponent("Policies")
.addComponent(builder.newComponent("CronTriggeringPolicy").addAttribute("schedule", "0 0 12 1/1 * ? *"));
	//對應DailyRollingFileAppender
	AppenderComponentBuilder appenderBuilder = 
builder.newAppender(appenderName, "RollingFile")
	.addAttribute("fileName", logDir.concat("mail.log")) //對應.File
	.addAttribute("filePattern", logDir.concat("mail-%d{MM-dd-yyyy}.log")) //對應.DatePattern
	.add( layoutBuilder )
	.addComponent(triggeringPolicy);
	builder.add(appenderBuilder);
	builder.add( builder.newLogger( loggerName, Level.INFO )
			    .add( builder.newAppenderRef( appenderName ) )
			    .addAttribute( "additivity", false ) ); //不要同時寫到CONSOLE
	builder.add( builder.newRootLogger( Level.INFO )
			    .add( builder.newAppenderRef( appenderName ) ) ); //對應log4j.logger設定
	Configurator.initialize(builder.build());
</code></pre>

--

reference

Hibernate Log Categories
https://docs.jboss.org/hibernate/stable/core.old/reference/en/html/configuration-logging.html
use-deployment-logging-config早期說明文件
https://docs.jboss.org/author/display/WFLY10/Logging%20Configuration.html#91947146_LoggingConfiguration-%7B%7Busedeploymentloggingconfig%7D%7D
