在使用native sql執行sp時，EF可以用下列方法達成
<pre><code class="csharp">
string p1 = "參數";
decimal p2 = 88.1234;
DbSet<SP_GetResult>.FromSqlRaw("EXECUTE dbo.SP_GetResult @param0 = {0}, @param1 = {1}, p1, p2).ToList();
</code></pre>


此時我們預想MSSQL會幫我們執行成
<pre><code class="sql">
EXEC	@return_value = [dbo].[SP_GetResult]
		@param0 = N'參數',
		@param1 = 88.1234
</code></pre>


但經過實際監測，其實會執行成
<pre><code class="sql">
exec sp_executesql N'EXECUTE dbo.SP_GetResult @param0 = @p0, @param1 = @p1, @TotalCapacity = @p2',
	N'@p0 nvarchar(4000),@p1 decimal(18,2),
	@p0=N'參數',@p1=88.12
</code></pre>

你會發現C#的decimal，在FromSqlRaw轉換成執行語法時，會自動轉成decimal(18,2)，導致小數點第三位以後被4捨5入，造成執行結果不正確

--

由於下列執行方法也是可以被MSSQL接受的
<pre><code class="sql">
EXEC	@return_value = [dbo].[SP_GetResult]
		@param0 = N'參數',
		@param1 = '88.1234'
</code></pre>

因此，較為合理的解法是
<pre><code class="csharp">
string p1 = "參數";
decimal p2 = 88.1234;
DbSet<SP_GetResult>.FromSqlRaw("EXECUTE dbo.SP_GetResult @param0 = {0}, @param1 = {1}, p1, p2.ToString()).ToList();
</code></pre>



