### 確保所有元件最新，請重建專案，再把程式逐步放進來。
### 新增專案步驟請參照圖片(含runtime安裝與vs升級)。

--

專案升級注意事項<br />


　1. Server端<br />
　　1. Error.cshtml交由server 端管理，client side如果有記得砍掉或移到server side<br />
　　2. 請保持乾淨，刪除垃圾<br />


　2. Client端<br />
　　1. Shared跟Pages記得清空後再貼進去，避免殘留垃圾<br />
　　2. Data如果有(ViewModel)，請放到Shared端，namespace請重新管理<br />
　　3. Service如果有，建議新建Service資料夾分開，並且寫在Blazor端，namespace請重新管理<br />
　　4. 承上，如果用builder.Services.AddSingleton<>()，建議改用AddScoped<>()，然後集中管理HttpClient，寫法如下:<br />
<pre>
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.Configuration.GetValue<string>("AjaxUrl")??"") });
builder.Services.AddScoped<PostService>();
</pre>
　　5. 承上，其中 builder.Configuration，是透過appsettings.json管理靜態資源(固定值)，請新建在wwwroot下，寫法參照server 端如下:
<pre>
{
  "AjaxUrl": "https://api.revo.org.tw/revotripublic/api/OfficialSite/"
}
</pre>

　　6. 承上，此時Service的 HttpClient要透過Constructor注入，如下:
<pre>
public class ClientService
{
　readonly HttpClient client;

　public ClientService(HttpClient client) {
　　this.client = client;
　}
}
</pre>
　　7. 請保持乾淨，刪除垃圾<br />


　3. Shared端<br />
　　1. 請集中管理ViewModel到這，給Server端與Client端共用<br />
　　2. 請保持乾淨，刪除垃圾<br />