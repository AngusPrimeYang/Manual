前言: EF6後繼的版本不再提供Model First， **edmx就這麼從此消失了。**

>先有AP再建DB、跟先有DB再建AP，相當於雞生蛋生雞的問題，過去<br /><br />
前者是使用Code First概念，其實就是指純手工，從頭刻下去，最後在站台啟動時加個 customDbContext.Database.EnsureCreated(); 如果沒有資料庫，直接透過程式中的entity定義建回資料庫/表<br /><br />
後者是使用Model First概念，透過工具產生edmx做中介，連線資料庫取得定義生成entity，方便是方便但是真的有夠慢的<br />

**於是乎就出現了Code First From Database**，講白了就是手動管理的Model First，不要整天比來比去蓋來蓋去，下語法執行，速度快<br /><br />
其實類似eclipse透過JPA tool生成entity，但是這邊是連EntityManager+各表對應getter/setter都幫你弄好。

--

以下為做法
* 要寫專案，SDK先準備好
* 開啟CMD，確保EntityFramework元件已安裝，輸入指令尋找獨角獸

![尋找獨角獸](https://user-images.githubusercontent.com/30335993/220062671-9bc6c722-4cc4-454a-8fa3-5add086d876b.jpg)

* 沒有的話請更新元件
<pre>
dotnet tool update --global dotnet-ef
</pre>
* 要使用的是scaffold功能

![scaffold功能](https://user-images.githubusercontent.com/30335993/220062864-f2f669b7-db5e-4cec-8d5a-619431b7aeec.png)


* 在 **有EntityFrameworkCore元件的專案下，從專案跟目錄開啟命令提示字元** ，執行語法(以下全部為一行，方便解說才切開)
<pre>
dotnet ef dbcontext scaffold "Server=.;Database=DBName;TrustServerCertificate=True;Trusted_Connection=True;" // 連線字串，信任憑證、採window驗證，否則請用帳密登入 user id=user;password=pwd;
 Microsoft.EntityFrameworkCore.SqlServer -c DBContext --data-annotations // 指定連到mssql，將最終結果寫到DBContext.cs檔(繼承DbContext)，且entity包含詳細定義
 --force -o DbEntity // 生成後強制蓋過已存在的同名檔案，且放在名為DbEntity的資料夾下
 --no-onconfiguring // DbContext檔不預設產生OnConfiguring函式
 -t Address -t AddressNo // 如果只是想產出部分表格而非所有資料表，請使用多個-t指定
</pre>

--

接著使用請參考[服務註冊與依賴注入](https://github.com/AngusPrimeYang/Manual/blob/main/dotNet/Core/Add%20Service%20and%20Dependency%20Injection.md)

供參

