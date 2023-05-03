### 目的

* wildfly24+
* java17

--

### 使用範圍

* 已先實作wildfly23與java升級(8/11/17)
* 需要安全性的全面提升

--

### 關於Wildfly

* 下載請至Wildfly官方網站取得最新版: https://www.wildfly.org/downloads/
* Wildfly 16開始引進了Elytron技術，統整安全管理機制，整合Wildfly帳密與權限管理、資料庫帳密登入管理、站台SSL憑證管理等。於Wildfly 24以後已停用原來的jboss:domain:security機制，僅保留wildfly:elytron，因此本來所有的picketbox、Vault等已不再建議使用或已無法使用。
* 下述任一步驟如果完成後下一步無法執行，建議到Runtime對Server做reload，或每做完一步reload一次也行。如果使用HAL Management Console也就是所謂網頁管理介面，可以看到許多步驟完成後會自行reload。
這裡盡量使用HAL Management Console作為輔助，增加友善度，但是部分功能網頁管理介面支援不完全，還是會以cli處理。
> 1. 取代picketbox
> > (1) 因應安全性問題，已被WildFly Elytron tool的mask功能取代，生成方法透過bin下的elytron-tool工具，生成結果以MASK-開頭辨識，或參考*1手冊4.1.5.9.
> > (2) 生成範例如下，並附上參數說明，可以看到結果開頭為MASK-。
![image](https://user-images.githubusercontent.com/30335993/235827234-987ecf8a-3ece-4e76-a8c3-fc7490d819dd.png)
> > (3) 可惜生成的加密內容不能完全取代以往picketbox的腳色，例如說資料庫密碼<password>節點就不接受使用MASK-開頭辨識，因此必須改用其他方法處理，所幸wildfly也提供不只一個選擇取代。
> 2. 取代Vault
> > (1) *picketbox依然存在，但建議直接改用elytron以獲得高安全性。*
> 3. 建立Credential Store
> > (1) elytron主要透過外部金鑰作為加密依據，也就是另外生成一個自用的keystore專門用來管理wildfly，金鑰檔使用java的keytool生成即可，或參考*1手冊1.2.9.1.
<pre><code class="shell">
keytool -genkeypair -alias "你的alias" -storetype jks -keyalg RSA -keysize 2048 -keypass "你的alias密碼" -keystore "你的ks路徑" -storepass "你的ks密碼"
</code></pre>
> > (2) 承上，注意手冊已經在生成金鑰時要求加上 -validity 730 -v 參數，表示本金鑰有效時間僅兩年， *請依照機關長期規劃決定有效期限，並定期維護之。*
> > (3) 承上，一般jks金鑰生成後，都會建議轉成pkcs12，但是經確認轉了之後wildfly無法使用，這個要等後續wildfly處理了(2022/07)，請不要轉換pkcs12。
> > (4) 要讓wildfly與外部金鑰關聯起來，必須建立 credential-storer節點，請先至Configuration>Subsystems>Security>Other settings[View]。
![image](https://user-images.githubusercontent.com/30335993/235827256-dbf2868c-f2e3-431c-a95d-fa716ce29c81.png)
> > (5) 承上，請在Credential Store點選Add。
![image](https://user-images.githubusercontent.com/30335993/235827278-ac522a99-7886-42a9-ac53-55240212a4ef.png)
> > (6) 承上，輸入必要資訊，並切換Create為ON後新增，或參考*1手冊4.1.4.1.
![image](https://user-images.githubusercontent.com/30335993/235827289-dff842d8-741f-486b-a8ca-aeb334f28993.png)
> > (7) 採用外部金鑰方式建置的credential-store，可以用來作為密碼管理、加解密、憑證管理使用，參考*1手冊4.1.3.
![image](https://user-images.githubusercontent.com/30335993/235827306-aad7c111-3e0b-4316-b303-eebc5a2393fc.png)
> 4. 以SecretKeyCredential使用Encrypted expressions遮蔽機敏文字
> > (1) WildFly Elytron tool 的mask功能，產出的數值還是可以看出salt跟iteration這些加密參數，可以被用來作為拆解的提示，因此wildfly另外提供了 Encrypted expressions 功能，進一步透過 SecretKeyCredential機制，來加密設定檔文字，加密結果會以${ENC::}形式呈現。
> > (2) SecretKeyCredential的設定目前(Wildfly26)在HAL Management Console上沒有良好的支援，以下將會直接使用cli執行，參考*1手冊4.1.6.，開啟cli方法如下，在站台啟動的情況下，執行jboss_cli.bat，再connect。
![image](https://user-images.githubusercontent.com/30335993/235827322-5167ffe9-0e86-4360-b6d8-61027f30f01b.png)
> > (3) 首先透過credential-store建立SecretKeyCredential管理密碼。使用elytron提供的generate-secret-key功能鍵立，語法如下，參考*1手冊4.1.4.6.。
<pre><code class="shell">
/subsystem=elytron/credential-store=你的CredentialStore名稱:generate-secret-key(alias=你的SecretKeyCredential名稱)
</code></pre>
> > (4) 再為SecretKeyCredential建立一個resolver加密用以文字，參考*1手冊4.1.7.
<pre><code class="shell">
/subsystem=elytron/expression=encryption:add(resolvers=[{name=你的resolver名稱, credential-store=你的CredentialStore名稱, secret-key=你的SecretKeyCredential名稱}])
</code></pre>
> > (5) 執行加密前，因為要輸入加密文字，請避免留下command history，設定不紀錄command。
<pre><code class="shell">
history --disable
</code></pre>
> > (6) 執行 create-expression 產生加密文字，參考*1手冊4.1.7。
<pre><code class="shell">
/subsystem=elytron/expression=encryption:create-expression(resolver=你的resolver名稱, clear-text=你要加密的文字)
</code></pre>
> > (7) 執行加密後，重新把command history打開。
<pre><code class="shell">
history --enable
</code></pre>
> 5. 以PasswordCredential取代Datasources的password欄位。
> > (1) 透過credential-store建立PasswordCredential管理密碼，請先至Runtime>Server>Security>Stores[view]
![image](https://user-images.githubusercontent.com/30335993/235827366-248d0a80-87f1-4fc5-99ab-ea9bc597373a.png)
> > (2) 承上，這時可以看到前面建好的credential-store，點選Aliases管理。
![image](https://user-images.githubusercontent.com/30335993/235827376-24e23dd5-794b-4386-838d-47d8f4904132.png)
> > (3) 承上，點選Add alias。
![image](https://user-images.githubusercontent.com/30335993/235827384-58168f03-a00c-4acb-9897-53ad6681b25a.png)
> > (4) 承上，指定外部金鑰檔中的alias後加入，作為PasswordCredential使用。
![image](https://user-images.githubusercontent.com/30335993/235827389-1347115f-15c1-43ce-8cc2-4b91bebd9648.png)
> > (5) 接下來要用PasswordCredential取代原來的資料庫密碼，請到Configuration>Subsystems>Datasources&Drivers>Datasources>Datasource[view]
![image](https://user-images.githubusercontent.com/30335993/235827401-6392ac00-5951-4fee-b27e-73f97fd50d32.png)
> > (6) 切換到Security頁籤檢查，本來有密碼的情況請先把密碼拔掉。
![image](https://user-images.githubusercontent.com/30335993/235827407-e37b4224-75e9-4c7f-a1a9-2f0ba0199dc9.png)
> > (7) 請將剛才完成CredentialStor與Alias帶入此處，就完成密碼取代了。
![image](https://user-images.githubusercontent.com/30335993/235827422-6f4b3af5-0443-47d9-9d72-1eb6838a6b5e.png)
> 6. MASK-與${ENC::}的混用
> > (1) 幾乎所有的設定檔文字都可以使用Encrypted expressions加密，不論是屬性值還是節點值，唯有一個東西不能，就是Credential Store的密碼，因為你必須先開啟Credential Store，才可以使用裡面的SecretKeyCredential、PasswordCredential，如果你使用${ENC::}遮蔽Credential Store的密碼，wildfly會掉入無限迴圈不斷找密碼去解密碼，所以至少Credential Store的密碼還是要使用MASK-去遮蔽的。
> > (2) 因為不論是屬性值還是節點值都能用Encrypted expressions處理，所以當設定好SecretKeyCredential後，就不一定要使用PasswordCredential去處理資料庫連線的password節點，也可以直接把密碼轉成${ENC::}後放到節點作為加密處理的方法就好。
<pre><code class="xml">
&lt;security>
&lt;user-name>${ENC::}&lt;/user-name>
&lt;password>${ENC::}&lt;/password>
&lt;/security>
</code></pre>
> > (3) 另外PasswordCredential沒有地方需要輸入金鑰檔中alias層的密碼，我覺得很神奇，或許他根本沒有把alias解開使用，又或許是甲骨文神秘力量。
> 7. 採用ssl-context管理站台憑證
> > (1) 因應資安長遠考量，ssl請至少使用TLSv1.2以上的版本，過往jboss/wildfly的升級，請參考上面【wildfly設定檔繼承】設定ssl憑證部分
> > (2) 新版本的wildfly中可以看到https-listener不再預設使用security-realm="ApplicationRealm"管理憑證，而是改成使用ssl-context="applicationSSC"處理
![image](https://user-images.githubusercontent.com/30335993/235827444-e981686c-3613-4708-b895-2335ab25d11a.png)
> > (3) 既然他都預設好了，就不客氣直接拿來使用了(其實是因為https-listener的security-realm節點已被標註為deprecated，遲早要換)，故以下步驟皆採用*1手冊1.5.2.1.僅Using Elytron Subsystem Commands部分，章節亦提供透過wildfly自建金鑰方法generate-key-pair，作為測試使用
> > (4) 承上，以applicationSSC回頭去找尋server-ssl-context設定本身，將本來https-listener的enabled-protocols與enabled-cipher-suites屬性轉移進去，請至Configuration>Subsystems>Security>Other settings[View]
![image](https://user-images.githubusercontent.com/30335993/235827454-cfc40b9d-e679-47ea-b49e-3d2b764980e2.png)
![image](https://user-images.githubusercontent.com/30335993/235827461-453fb224-a628-41a3-ac00-7352cb9207c1.png)
> > * 屬性protocols，用來取代https-listener中的 enabled-protocols屬性
> > * 屬性cipher-suite-filter，用來取代https-listener中的enabled-cipher-suites屬性，一樣間隔以:或,或 處理，僅適用TLSv1.2以下
> > * 屬性cipher-suite-names，如果演算法使用TLSv1.3，cipher請放在此屬性中，間隔以:處理
> > (5) 承上，以applicationKM回頭去找key-manager設定本身，屬性generate-self-signed-certificate-host請拿掉，不然他在找不到憑證時會自行生成，不是我們需要的情境，credential-reference的clear-text改成核發金鑰中alias指定的密碼，即本來的key-password
![image](https://user-images.githubusercontent.com/30335993/235827804-ac0aaa54-9845-4223-b465-96bac38ca366.png)
![image](https://user-images.githubusercontent.com/30335993/235827810-b4910d83-c119-445f-9af7-5258da266f6a.png)
> > * 屬性algorithm，依照憑證類型選用PKIX(X509)，詳細，參考*5
> > * 屬性alias-filter，用來取代keystore中的alias屬性
> > * credential-reference的clear-text，用來取代keystore節點中的key-password
> > (6) 承上，以applicationKS回頭去找key-store設定本身，將path改為核發金鑰的檔名，credential-reference的clear-text改成核發金鑰密碼，即本來的keystore-password
![image](https://user-images.githubusercontent.com/30335993/235827829-ebe4d053-98d2-41ba-b60c-0c2a2531c98b.png)
![image](https://user-images.githubusercontent.com/30335993/235827847-3f73986a-5a38-4214-bc9c-e0acd9702958.png)
> > * 屬性path，用來取代keystore中的path屬性
> > * 屬性relative-to，用來取代keystore中的relative-to屬性
> > * 屬性Type，依照我們憑證核發的規範填上JKS
> > (7) 驗證憑證是否有設定成功的方法，可透過cli執行查詢keystore的alias功能，他會把整個alias的詳細內容列出，語法如下
<pre><code class="shell">
/subsystem=elytron/key-store=applicationKS/:read-alias(alias=你的alias名稱)
</code></pre>

--

### 關於Java與Wildfly整合

Wildfly24以後，Hibernate又對EJB的運作，做了更多限制與效能提升，另外也整併了不少三方元件。

> 1. 所有內含資料庫insert/update/delete函式，不能再賦予SUPPORTS屬性，是說本來就是錯的，只是以前會被忽略，以下為錯誤示範。
![image](https://user-images.githubusercontent.com/30335993/235827879-89fc2fe6-47a4-4116-85af-27f723c9942f.png)
> 2. 承上，這邊同時講解一下TransactionAttributeType的意義，當函式用此屬性包覆時，表示整個函式允許的執行行為與commit的範圍，預設值為REQUIRED，所以沒幹嘛的話不用特別加，除非要限定範圍內只允許查詢(SUPPORTS)，或希望另開一個transaction先commit部分(REQUIRES_NEW)，否則一律是所有sql指定完成後commit(範圍內使用其他DAO的函式也一樣，除非範圍內某個函式使用了REQUIRES_NEW，那那個函式會獨立開一條transaction執行)，因此TransactionAttribute僅在連續呼叫函式且需要先後處理的情況有意義。
![image](https://user-images.githubusercontent.com/30335993/235827886-65d07a68-b5fb-4384-ae24-02661350c26d.png)
> 3. 所有語法中的 ?，不再允許交由程式判斷(不管是用EntityManager還是NativeSQL都一樣)，都要補上數字管理如 ?0，而:param的方法還是可以繼續使用，下為改寫後範例。
![image](https://user-images.githubusercontent.com/30335993/235827894-383fd247-7e4d-40f0-afd8-6eed94c1b69e.png)
> 4. 使用createSQLQuery直接以Native Sql查詢的情況，請不要再使用org.hibernate.Query接，一律改用org.hibernate.query.NativeQuery<>，其中<>中的內容與本來回傳類型一致即可，查詢明確型別欄位時如下，可以使用如<Integer>，Hibernate會協助轉型
![image](https://user-images.githubusercontent.com/30335993/235827899-a98183a3-9fe8-4697-9f0c-50a682224e3a.png)
> 5. 承上，遇到使用executeUpdate增刪寫時，有需要可以用<?>處理
![image](https://user-images.githubusercontent.com/30335993/235827914-c0a5ba2c-84a0-4df1-9478-b34a2c6c30b8.png)
> 6. 承上，這邊順便講解一下使用EntityManager 做資料庫語法IN的方法，只要一個?丟陣列進去就可以解決
![image](https://user-images.githubusercontent.com/30335993/235827922-63cd4204-c0c2-47ec-b0f4-9281ca0d7b7b.png)
> 7. JSON已備wildfly整併到僅使用com.fasterxml.jackson，請一律改寫import對象
> 8. org.apache.xmlbeans已不提供支援，請改寫或自行至maven抓安全版本下來使用，因安全性問題請至少挑xmlbeans-3.1.0以上版本
> 9. org.apache.commons.digester已不提供支援，請改用digester3，或著就直接棄用，詳見meven的commons-digester3
> 10. 針對已不支援或移除的元件，如果有從jboss-deployment-structure.xml做dependencies，請移除
[!clipboard-202305031121-8xpr7.png!](https://jcsap2.westus2.cloudapp.azure.com/attachments/download/114/clipboard-202305031121-8xpr7.png)

--

Reference

1. RedHat EAP 7.4說明書(安全性設定部分)
	https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.4/html/how_to_configure_server_security/index
2. Wildfly 26 Elytron使用說明書
	https://docs.wildfly.org/26/WildFly_Elytron_Security.html
3. Wildfly 26 升級指南
	https://docs.wildfly.org/26/Migration_Guide.html
4. Wildfly 26 Model 規格
	https://docs.wildfly.org/26/wildscribe/index.html
5. Implementations Supplied by SunJSEE
	https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#SupportClasses
