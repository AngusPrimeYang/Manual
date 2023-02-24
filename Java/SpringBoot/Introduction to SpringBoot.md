
## 緣由 
    (一) springboot好快，漸漸POC、REST APIs(springboot是為微型服務量身打造)等都採用
    (二) 預設使用tomcat(war)且積極擁抱java17。
    (三) 本身就是spring framework延伸*1，spring基礎架構並沒有改變，畢竟是嫌麻煩但又覺得好用的鄉民變出來的(追溯到2012)，
    　　　朝springboot前進的話，很多模組其實不需要調整，但有強調採用standalone mode。
    (四) maven的dependency有處理過，很多版控操作會自動弄好。
    (五) 透過各種annotation快速設定就可以架起簡易站台，Eclipse也有支援快速生成專案、增減模組的starter，
    　　　非常方便，甚至很多設定檔像web.xml都可以省掉。

## 目的
    簡單說明springboot怎麼建置，並且實作在java+wildfly環境中做為案例。

## 使用範圍
    版本使用2.7.2，maven從2.x升上來沒甚麼要調整，因為後面其他相依元件版本會自動處理好，
    　　　只要指定使用哪些dependency就好，也不用指定版本。
    要調整的是code的部分，弄好後spring framework的controller甚至可以直接搬進來(如果是要架站而非架API的話)。
    開發環境使用eclipse，java用8，wildfly為搭配及快速建置採用23，關於java或wildfly升級與springboot無關，在此不贅述。
    內容基本上做到了Controller控制、Filter控制與wildfly協作(導入springframwork的東西)、前端資源管理。
    header的部分因為本人推從分工透過AP Server管理(像wildfly卡在最前面)，所以不寫在這邊(有些需要另外處理或覆寫的東西，還是會做)，
    　　　同時當你header是透過Server管理的話，改用springboot也根本不用動到這塊。
    另外提供改用JPA連線資料庫的方法，不同於EJB寫起來像struts2，或著說struts2因為本身就採用了不少annotation，
    　　　像是@Autowired等，所以比較相似。

## Eclipse新建springboot專案

    (一) 新建專案，選Starter
    
![新建專案，選Starter](https://user-images.githubusercontent.com/30335993/221069033-ae22b5fd-58a1-4804-81a5-c05fa39f2da7.jpg)

    (二) 管理好使用java的版本，建議直接使用Eclipse內建的java，可以省掉設定環境的時間，以及環境造成跟其他專案衝突。
    　　　另外因為使用tomcat一脈(wildfly)，所以專案用war檔形式輸出即可。

![java版本](https://user-images.githubusercontent.com/30335993/221069314-d3b1d4ee-ba1d-4e5d-bf6b-e47206bff846.jpg)

    (三) 除了最主要的boot版本外，左下還提供你下拉方式帶入你要的模組，右下會顯示你選了那些，中上還會記錄你常用模組供你快速選擇。

![帶入模組](https://user-images.githubusercontent.com/30335993/221069406-de613f68-44cf-4473-9eb4-2bcca1575a7c.jpg)

    (四) 想要調整模組的使用，對專案點右鍵還可以再開啟starter增減。
    
![維護模組](https://user-images.githubusercontent.com/30335993/221069475-c17c803d-0f9b-47cc-a761-5dd9ac74c80f.jpg)

    (五) 基本上他就是一個常見的java web project，有src跟resource，沒甚麼不同。

![專案屬性](https://user-images.githubusercontent.com/30335993/221069558-feff4f18-716b-4e6e-8552-426fe6137126.jpg)

    (六) 見pom檔可以發現甚麼都弄好了，建議之後要管理boot相關dependency時一樣使用starter管理就好，才不會弄亂。
    
![pom結果](https://user-images.githubusercontent.com/30335993/221069676-211f9f44-9441-4a98-bf06-a7fbca2b45ac.jpg)

    (七) 上述步驟就設定完，一個不需網頁的API架構，基本上就完成了。

## springboot專案設定

### springboot Rest APIs新建

    (一) 承Eclipse新建springboot專案，只要再補上個@RestController，對外服務就開張了。以下提供個Hello World。

![服務開張](https://user-images.githubusercontent.com/30335993/221070005-68e92613-39f2-46d2-b969-f190b6744490.jpg)

    (二) 透過Eclipse把專案run起來很簡單，直接run as Spring Boot App就好。從這邊也可以看出就是內鍵tomcat作為預設Server使用。
    
![run](https://user-images.githubusercontent.com/30335993/221070107-3ca91dfe-ee80-450b-b2d9-65ba6cec963c.jpg)

    (三) 結果如下。
    
![result](https://user-images.githubusercontent.com/30335993/221070167-45359972-3cf0-4eba-bc68-f263cf4a42fb.jpg)

### springboot MVC新建

    (一) 承springboot Rest APIs新建，些著新建其他Mapping，定義同springframework。
    　　　一般controller回傳的字串，其實是指導向哪個頁面，除非增加@ResponseBody定義才會將string做為內容直接回傳
       　另外，因為沒有做任何設定，所以回傳要寫清楚是導向.html
        
![自訂Mapping](https://user-images.githubusercontent.com/30335993/221070730-2a8cdd97-d6f9-4bda-ae83-85ceefa6188a.jpg)

    (二) 管理靜態資源的方法。請讓@SpringBootApplication引用WebMvcConfigurer，並複寫addResourceHandlers中的資源對應。
    　　　依照專案基本配置，src跟resource都會在編譯後被放到編譯完的資料夾，也就是編完後static與templates都會直接跟java資料夾下的東西放在一起，
    　　　因此靜態來源需標註為”classpath:target”形式。前面”/**”表示，你來源結構長怎樣，我網頁路徑就長怎樣，
    　　　例如說login_cover.png在static的images資料夾下，來源標註為classpath:/static/”，那抓取網址就相當於http://localhost:8080/images/login_cover.png
       
![管理靜態資源](https://user-images.githubusercontent.com/30335993/221070913-ade77fd0-f97f-4abc-b915-843f31a778e8.jpg)

    (三) 管理Request的方法。過去坊間依照官方文件皆建議可使用方法@EnableWebSecurity，搭配extends WebSecurityConfigurerAdapter，但已deprecated。
    　　　2.7以後直接一併在WebMvcConfigurer處理。

![管理Request](https://user-images.githubusercontent.com/30335993/221071045-0ec0f6bd-975b-4119-a019-6cd09eb372d9.jpg)

    (四) 初始化時載入外部系統參數做法。因為啟動時會執行SpringBootServletInitializer的configure，可以直接把動作引流到這。

![載入外部系統參數](https://user-images.githubusercontent.com/30335993/221071511-90108406-80b4-4df7-acf4-29675538aee0.jpg)

### 使用Filter管理封包

    (一) 客製化Filter的做法。可以透過引用OncePerRequestFilter來達到過去引用javax.servlet.Filter的效果，
    　　　OncePerRequestFilter依照官方說明有個很重要的好處，就是一個request只會跑一次，
    　　　不像過去其實所有靜態物件讀取時都會進Filter一次，並非我們想過濾處理的封包。

![客製化Filter](https://user-images.githubusercontent.com/30335993/221071851-08443436-89ff-49bb-ab45-3abc4de00adf.jpg)

### 使用JPA管理資料庫

    (一) 用@Repository的方式建立與資料庫的對應，Entity使用舊有的@Entity就可以了都不用改，
    　　　且語法管理也直接用@Query等封裝，不寫在xml引用，但我想用外部參數的概念做還是可以辦到。
       
![Repo](https://user-images.githubusercontent.com/30335993/221072191-cb951be0-e5ce-4895-a46c-665bae7f9d59.jpg)

    (二) 承上，JpaRepository這個interface其實就已經包含了許多基本的查詢函式，
    　　　單純的表格甚至不用寫任何內容在自定義的interface內，就已經可以使用很多CRUD函式，參考*2。
       
![JpaRepo](https://user-images.githubusercontent.com/30335993/221072332-e02e26c7-d807-40b7-9f53-304cd17c6d36.jpg)

    (三) 用@Service連結主機端，定義函式、取用實作出來的@Repository。
    　　　要記得@Repository前面是寫成interface，理應有實作去完成功能才對，
    　　　但因為直接交給JPA去完成了，躺著過，只要在@Service透過@Autowired把@Repository串起來standby好就行了。
       
![Autowired](https://user-images.githubusercontent.com/30335993/221072526-ea4ab171-eaa5-4caf-88b0-8d27fabab702.jpg)

    (四) 在package結構趨向複雜的情況下，要管理好各種類型annotition啟動時掃描的範圍，才不會搞混在一起，其中
    1. @EntityScan會定義Entity存放(掃描)位置
    2. @EnableJpaRepositories會定義Repo存放(掃描)位置
    3. @ComponentScan會定義各種系統服務存放(掃描)位置

![其他annotition](https://user-images.githubusercontent.com/30335993/221072676-d1380f34-3a8c-45fa-97b9-b80db558c715.jpg)

    (五) 資料庫連線設定，透過application.properties管理，
    　　　只要把連線設定填好，啟動時就會自動建立資料庫連線。
    　　　其中genereate-ddl一類設定是在連線時管理JPA是否會自動對應資料庫規格，順向建立Entity，或反向建立Table的功能，這邊先不做這塊的解析。

![資料庫連線設定](https://user-images.githubusercontent.com/30335993/221073245-88b00f44-4b18-4a02-a899-893a3bd2c529.jpg)

    (六) 關於設定檔內容資料加密，可以透過jasypt元件管理。請先從pom檔導入元件。

![設定檔加密](https://user-images.githubusercontent.com/30335993/221073681-de64ccf8-7805-435c-8c47-4c9c180b52ec.jpg)

    (七) 指定好jasypt的演算法、IV與金鑰，就可以將部分的value值加密起來，這邊使用預設的演算法做說明。

![加密演算法設定](https://user-images.githubusercontent.com/30335993/221073763-a5d2b428-f87e-428d-a484-1670f4120212.jpg)

    (八) 對應預設演算法生成加密後資料的語法如下。

![對應預設演算法加密](https://user-images.githubusercontent.com/30335993/221073814-f16ddb44-b0e3-403d-a85f-7132c32977f1.jpg)

    (九) @SpringBootApplication檔需加入@EnableEncryptableProperties，
    　　　如果使用預設的演算法，則不用額外設定@Configuration，只要加註在@SpringBootApplication處就可以了

![EnableEncryptableProperties](https://user-images.githubusercontent.com/30335993/221073908-73c97d51-6f8b-4c5f-a438-9e28ebf69b5c.jpg)

    (十) 如果想要自訂加密演算法，可以跳過步驟(七)~(九)，改用一個單獨的@Configuration管理，透過PooledPBEStringEncryptor.encrypt加密字串。

![自訂加密演算法](https://user-images.githubusercontent.com/30335993/221073984-73daf070-4746-447b-aaa4-1a3880bf8591.jpg)

    (十一) 維持使用EJB或其他連線方法管理也是可以的，只要第三方元件使用上不要彼此產生衝突就好。
    (十二) @Repository使用上也是透過@Autowired把服務實作起來使用即可。

![Autowired](https://user-images.githubusercontent.com/30335993/221074226-b354c763-4bb9-4cb7-9fe5-e8ec4c80e82c.jpg)

    (十三) Spring Boot 三層式架構簡述可以參考其他開發者說明*3。

## 結合Wildfly

    不外乎就是處理好元件之間的版本管理，
    springboot當前版本用的底層元件或三方元件，跟wildfly當前版本用的，有沒有衝突，
    這邊以springboot 2.7.2與wildfly23，配合java8為案例，說明衝突解決的方法。

    (一) 主要衝突在是logger之間的使用，建議springboot排除logback-classic，將logger交還wildfly。

![springboot排除](https://user-images.githubusercontent.com/30335993/221074787-c19e2322-67f5-41ab-9107-f87f6b5fb5fb.jpg)

    (二) 雖然wildfly基礎框架是師承tomcat(WAR檔封裝結構是源自於tomcat)，
    　　　但畢竟還是兩個產品，在各自的相輔相成的路上，還是會有衝突的時候，
    　　　目前將專案打包成war的支援是透過spring-boot-starter-tomcat這個lib，但是很容易遇到以下問題。
       
       javax.el.ELException: Provider # Licensed to the Apache Software Foundation (ASF) under one or more not found
       
       此時建議排除tomcat的作法。其中並沒有對錯，只是在元件衝突時，選擇了優先採用wildfly程式碼而已。
       
![排除tomcat](https://user-images.githubusercontent.com/30335993/221075195-a0773643-d6fe-41c8-b2b2-e670d05cf7a8.jpg)

    (三) 將finalName改為artifactId，這個是將project輸出的結果作為編譯資料使用的習慣，才不會站台帶有版號的情形。

![0631](https://user-images.githubusercontent.com/30335993/221075367-880dbee5-b0ae-417a-be6f-837a640f144f.jpg)
![0632](https://user-images.githubusercontent.com/30335993/221075371-f602193a-dd5a-4161-987b-196a3b383ffe.jpg)
![0633](https://user-images.githubusercontent.com/30335993/221075373-3534ee94-6e8b-4b84-b047-1335869331f0.jpg)
![0634](https://user-images.githubusercontent.com/30335993/221075375-fd6efafe-f5cb-43a6-b7d1-e446eaf4cd80.jpg)

--

Reference

1. [關於spring與spring boot的差異](https://www.interviewbit.com/blog/spring-vs-spring-boot/)

2. [Spring JPA](https://www.baeldung.com/spring-data-jpa-query)

3. [Spring Boot 三層式架構簡述](https://ithelp.ithome.com.tw/articles/10240138)

4. [tutorialspoint.com](https://www.tutorialspoint.com/spring_boot/spring_boot_introduction.htm)
