MSSQL建置全文檢索，請另外參照文章 [全文檢索設定](https://github.com/AngusPrimeYang/Manual/blob/main/DB/MSSqlServer/FullText%20Search%20Configration.md)

### 本篇著重於使用HQL下全文檢索語法

--

normal fulltext search in MSSQL
<pre>
SELECT KEYDAY, VOLNUM, MAIOBJ FROM TB WHERE CONTAINS(MAIOBJ, 'keyword')
</pre>
HQL command
<pre>
select tb.keyday, tb.volnum, tb.maiobj from TB tb where  CONTAINS(tb.maiobj, 'keyword')
</pre>

由於 **CONTAINS** 這個函式，並沒有註冊，因此要手動處理，在Dialect增加registerFunction定義

但是registerFunction必須定義回傳值，當我們在下HQL時，最後轉換的結果如下
<pre>
//define
registerFunction("CONTAINS", new SQLFunctionTemplate(StandardBasicTypes.BOOLEAN, "CONTAINS(?1, ?2)"));
//query by HQL
WHERE CONTAINS(tb.maiobj, 'keyword') 
//parsed result will be
WHERE CONTAINS(tb.maiobj, 'keyword') = 1
</pre>
會產生一個不上不下的錯誤結果

<br />

因此只能這樣處理
<pre>
//define
registerFunction("CONTAINS", new SQLFunctionTemplate(StandardBasicTypes.BOOLEAN, "CONTAINS(?1, ?2) AND 1"));
//query by HQL
WHERE CONTAINS(tb.maiobj, 'keyword') = true
//parsed result will be
WHERE CONTAINS(tb.maiobj, 'keyword') AND 1 = 1
</pre>
自己把查詢字串接正確，同時對應好回傳

--

Reference

從根本下手，高手在民間 

https://stackoverflow.com/questions/9488094/hibernate-mssql-fulltext-search-via-contains
