Blazor 的 跨來源資源共用（Cross-Origin Resource Sharing (CORS)）政策預設是嚴格的

因此做為API或內部站台，建議限制可以Request過來的對象

過去在開發時，通常都直接配個Access-Control-Allow-Origin: *

相當於把header關掉

現在Blazor提供在Program.cs建置時，就把header給管理好

順勢善加利用Environment.IsDevelopment()判斷，就不會因為每次要切來切去，導致使用意願降低

可用幾種方法設定，以下逐一說明。

--

* 新增Policy並引用為預設
<pre>
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =&gt;
{
    options.AddPolicy("CorsPolicy", policy =&gt;
    {
        if (builder.Environment.IsDevelopment()) //development mode
        {
            policy.AllowAnyOrigin();
        }
        else
        {
            policy.WithOrigins("https://www.site1.org.tw", "https://www.site2.org.tw");
        }
        policy.AllowAnyHeader()
            .AllowAnyMethod();
            //.AllowCredentials(); // for cookie
            //.WithHeaders(HeaderNames.ContentType, //HeaderNames.Server, "x-custom-header", "x-path", "x-record-in-use", 
            //HeaderNames.AccessControlAllowHeaders, HeaderNames.AccessControlExposeHeaders, HeaderNames.ContentDisposition); // for related headers
    });
});

//...

var app = builder.Build();

//...

app.UseCors("CorsPolicy"); // before UseAuthorization & MapControllers

app.UseAuthorization();

app.MapControllers();

app.Run();

</pre>


* 直接蓋過預設Policy
<pre>
var builder = WebApplication.CreateBuilder(args);

//...

var app = builder.Build();

//...

app.UseCors(policy =&gt;
{
    if (app.Environment.IsDevelopment())
    {
        policy.AllowAnyOrigin();
    }
    else
    {
        policy.WithOrigins("https://*.site.org.tw")
                .SetIsOriginAllowedToAllowWildcardSubdomains(); // allow all subdomain
    }
    policy.AllowAnyHeader()
        .AllowAnyMethod();
});

app.UseAuthorization();

app.MapControllers();

app.Run();
</pre>


* 新增Policy但不引用，後續再詳細於Controller中個別定義
<pre>
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =&gt;
{
    options.AddPolicy("Site1Policy", policy =&gt;
    {
        if (builder.Environment.IsDevelopment())
        {
            policy.AllowAnyOrigin();
        }
            else
        {
            policy.WithOrigins("https://www.site1.org.tw");
        }
        policy.AllowAnyHeader()
            .AllowAnyMethod();
    });
    options.AddPolicy("Site2Policy", policy =&gt;
    {
        if (builder.Environment.IsDevelopment())
        {
            policy.AllowAnyOrigin();
        }
            else
        {
            policy.WithOrigins("https://www.site2.org.tw");
        }
        policy.AllowAnyHeader()
            .AllowAnyMethod();
    });
});

//...

var app = builder.Build();

//...

app.Run();
</pre>
個別定義可以從Class層級指定
<pre>
// 定義給整個Controller
[EnableCors("MyPolicy")]
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    // GET api/values
    [HttpGet]
    public IActionResult Get() =&gt;
        ControllerContext.MyDisplayRouteInfo();

    // 特別排除
    // GET: api/values/GetValues2
    [DisableCors]
    [HttpGet("{action}")]
    public IActionResult GetValues2() =&gt;
        ControllerContext.MyDisplayRouteInfo();

}
</pre>
或從Function層級指定
<pre>
[Route("api/[controller]")]
[ApiController]
public class WidgetController : ControllerBase
{
    // 採用自訂政策AnotherPolicy
    // GET api/values
    [EnableCors("AnotherPolicy")]
    [HttpGet]
    public ActionResult&lt;IEnumerable&lt;string&gt;&gt; Get()
    {
        return new string[] { "green widget", "red widget" };
    }

    // 採用某一自訂政策Policy1
    // GET api/values/5
    [EnableCors("Policy1")]
    [HttpGet("{id}")]
    public ActionResult&lt;string&gt; Get(int id)
    {
        return id switch
        {
            1 => "green widget",
            2 => "red widget",
            _ => NotFound(),
        };
    }
}
</pre>

--

Reference

https://learn.microsoft.com/zh-tw/aspnet/core/security/cors?view=aspnetcore-7.0

https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS
