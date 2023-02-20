在Program.cs(或Startup.cs)做服務註冊時，要注意服務生命週期，主要有下列三種模式

* Singleton
　整個 Process 只建立一個 Instance，任何時候都共用它。
 <pre>
　services.AddSingleton&lt;IDataService, DataService&gt;();
 </pre>
* Scoped
　在網頁 Request 處理過程(指接到瀏覽器請求到回傳結果前的執行期間)共用一個 Instance。
 <pre>
　services.AddScoped&lt;IDataService, DataService&gt;();
 </pre>
* Transient
　每次要求元件時就建立一個新的，永不共用。
 <pre>
　services.AddTransient&lt;IDataService, DataService&gt;();
 </pre>

--

其中服務註冊的方法相當彈性
<pre>
//通用寫法
services.Add(new ServiceDescriptor(typeof(IDataService), typeof(DataService), ServiceLifetime.Transient));
//精簡寫法
services.AddTransient&lt;IDataService, DataService&gt;();
//未另宣告介面，直接使用實作型別
services.AddTransient&lt;DataService&gt;();
//進階管理
services.AddTransient&lt;IDataService, DataService&gt;((ctx) =&gt;
{
    IOtherService svc = ctx.GetService&lt;IOtherService&gt;();
    //GetService&lt;T&gt;找不到服務時會傳回null，若不允許可改用GetRequiredService&lt;T&gt;
    //找不到時丟出例外
    //IOtherService svc = ctx.GetRequiredService&lt;IOtherService&gt;();
    return new DataService(svc);
});

//一個介面一個 Instance
services.AddSingleton&lt;IDataService, DataService&gt;();
services.AddSingleton&lt;ISomeInterface, DataService&gt;();
//兩個介面共用一個 Instance
var dataService = new DataService();
services.AddSingleton&lt;IDataService&gt;(dataService);
services.AddSingleton&lt;ISomeInterface&gt;(dataService);

//相依服務
IServiceProvider provider = services.BuildServiceProvider();
IOtherService otherService = provider.GetRequiredService&lt;IOtherService&gt;();
var dataService = new DataService(otherService);
services.AddSingleton&lt;IDataService&gt;(dataService);
services.AddSingleton&lt;ISomeInterface&gt;(dataService);
</pre>
--

依賴注入
<pre>
public class HomeController : Controller
{
    //透過建構子注入
    private readonly IDataService _dataService;

    public HomeController(IDataService dataService)
    {
        _dataService = dataService;
    }
    
    //從函式參數注入
    [HttpGet]
    public IActionResult Index([FromServices] IDataService dataService2)
    {
        IDataService dataService3 = HttpContext.RequestServices.GetService&lt;IDataService&gt;();
        return View();
    }
}
</pre>

服務中可以直接注入管理連線的HttpContext
<pre>
public class DataService : IDataService
{
    private readonly HttpContext _httpContext;

    public DataService(IHttpContextAccessor contextAccessor)
    {
        _httpContext = contextAccessor.HttpContext;
    }
    //...
}
</pre>

--


Reference

官方文件

https://learn.microsoft.com/en-us/aspnet/core/blazor/fundamentals/dependency-injection?view=aspnetcore-7.0

黑暗執行緒

https://blog.darkthread.net/blog/aspnet-core-di-notes/
