### 本文簡述全文檢索建立方式

--

首先請確定資料庫支援全文檢索
<pre>
select FullTextServiceProperty('IsFullTextInstalled')
</pre>
沒有安裝的話，請重新打開SETUP，把Full-Text Search補裝進去

接著要注意全文檢索使用的語言
<pre>
SELECT * FROM sys.fulltext_languages ORDER BY lcid
</pre>
可以看到繁中為1028(Traditional Chinese)

或[參照官方文件](https://learn.microsoft.com/en-us/openspecs/office_standards/ms-oe376/6c085406-a698-4e12-9d4d-c3b0ee3dbc4a)

後續會用到

--

* 建立FULLTEXT CATALOG
<pre>
--檢查現有清單
SELECT * FROM sys.fulltext_catalogs
--取得指定FULLTEXT CATALOG明細
SELECT fulltextcatalogproperty('FullTextSearchCatalog_MaiObj', 'ItemCount') AS ItemCount,
fulltextcatalogproperty('FullTextSearchCatalog_MaiObj', 'IndexSize') AS IndexSize,
fulltextcatalogproperty('FullTextSearchCatalog_MaiObj', 'MergeStatus') AS MergeStatus,
DATEADD(ss,fulltextcatalogproperty('FullTextSearchCatalog_MaiObj', 'PopulateCompletionAge'), '1/1/1990') AS PopulateCompletionAge,
fulltextcatalogproperty('FullTextSearchCatalog_MaiObj', 'PopulateStatus') AS PopulateStatus,
fulltextcatalogproperty('FullTextSearchCatalog_MaiObj', 'UniqueKeyCount') AS UniqueKeyCount,
fulltextcatalogproperty('FullTextSearchCatalog_MaiObj', 'AccentSensitivity') AS AccentSensitivity

--新增FULLTEXT CATALOG
CREATE FULLTEXT CATALOG TestFullTextSearchCatalog;
</pre>
參照[參照官方文件](https://learn.microsoft.com/zh-tw/sql/t-sql/statements/create-fulltext-catalog-transact-sql?view=sql-server-ver16)

<br />

* 建立FULLTEXT INDEX
<pre>
--檢查現有清單
SELECT COLUMNPROPERTY(OBJECT_ID('[dbo].[TB01]'), 'TB01_COL01', 'IsFulltextIndexed');
SELECT INDEXPROPERTY(OBJECT_ID('[dbo].[TB01]'), 'PK_TB01',  'IsFulltextKey');

--新增FULLTEXT INDEX
CREATE FULLTEXT INDEX ON [dbo].[TB01]
(
	[TB01_COL01]
		language 1028 --Chinese - Taiwan / zh-TW
)
KEY INDEX [PK_TB01]
ON TestFullTextSearchCatalog
WITH STOPLIST = SYSTEM  --使用系統的停用字
,CHANGE_TRACKING OFF --關閉自動追蹤
,NO POPULATION --不要立即擴展
;
</pre>
[參照官方文件](https://learn.microsoft.com/zh-tw/sql/t-sql/statements/create-fulltext-index-transact-sql?view=sql-server-ver16)

因為設定為 **關閉自動追蹤** 及 **不要立即擴展** ，後續要排定處理這些事情

<br />

* CHANGE_TRACKING [ = ] { MANUAL | AUTO | OFF [ , NO POPULATION ] }
<pre>
指定 (更新、刪除或) 插入對全文檢索索引所涵蓋之資料表資料行所做的變更，是否會由SQL Server傳播至全文檢索索引。 
</pre>
每次當欄位異動時，如果都要再重新涵蓋全文檢索索引，會影響整體執行效率

因此會比較建議採用MANUAL，再自行決定執行時間一次處理
<pre>
--CHANGE_TRACKING採用MANUAL時的擴展方法
ALTER FULLTEXT INDEX ON [dbo].[TB01] START UPDATE POPULATION;
</pre>
另外只有在 CHANGE_TRACKING 是 OFF 時，才能使用 NO POPULATION 選項。 指定 NO POPULATION 時，SQL Server在建立索引之後不會填入索引。

這是避免建立索引同時直接開始執行擴展，於NO POPULATION時要手動重新擴展，語法如下
<pre>
ALTER FULLTEXT INDEX ON [dbo].[TB01] START FULL POPULATION;
--或
ALTER FULLTEXT INDEX ON [dbo].[TB01] START INCREMENTAL POPULATION;
</pre>
也可在建立索引後，再執行變更CHANGE_TRACKING
<pre>
--自動
ALTER FULLTEXT INDEX ON [dbo].[TB01] SET CHANGE_TRACKING AUTO;
--或手動
ALTER FULLTEXT INDEX ON [dbo].[TB01] SET CHANGE_TRACKING = MANUAL;
</pre>

<br />

* STOPLIST
停用字詞表，是拿來過濾掉作為全文檢索的關鍵字來說沒有意義的字詞，例如「的」、「因為」這些對搜尋沒甚麼幫助

系統本身會有預設的字詞表，依照不同語言各自有各自的組件
<pre>
--篩選清單
EXEC sp_help_fulltext_system_components 'filter'; 
--斷詞工具元件清單
EXEC sp_help_fulltext_system_components 'wordbreaker'; 
</pre>
當然也可以自建
<pre>
--檢查現有清單
SELECT * FROM sys.fulltext_catalogs
--自建
CREATE FULLTEXT STOPLIST TestStopList FROM SYSTEM STOPLIST --以任一現有清單為基底
</pre>
[參照官方文件](https://learn.microsoft.com/zh-tw/sql/t-sql/statements/alter-fulltext-stoplist-transact-sql?view=sql-server-ver16)

--

* 使用方法(查詢語法)
<pre>
SELECT * FROM [dbo].[TB01] WHERE CONTAINS(TB01_COL01, 'keyword')
</pre>
關鍵字本身可以模糊查詢，但使用的是斷字規則下去查關鍵字，例如 *'"keyword*"'* 指的是，使用採用所有帶有"keyword"這三個字為開頭的關鍵字來查詢

[參照官方文件](https://learn.microsoft.com/zh-tw/sql/relational-databases/search/query-with-full-text-search?view=sql-server-ver16)

<br />

* CONTAINS 和 FREETEXT

不同於CONTAINS關鍵字搜尋，FREETEXT會連關鍵字的同義字也納入查詢

<br />

* CONTAINSTABLE 和 FREETEXTTABLE

表示以TableValue形式回傳，可用來做JOIN等運算

<br />

* 自建同義詞典

[參照官方文件](https://learn.microsoft.com/zh-tw/previous-versions/office/sharepoint-server-2010/cc263242(v=office.14))

--

Reference

https://learn.microsoft.com/zh-tw/sql/relational-databases/search/query-with-full-text-search?view=sql-server-ver16



