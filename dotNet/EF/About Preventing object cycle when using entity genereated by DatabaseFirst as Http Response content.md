如題，如果透過dotnet ef dbcontext scaffold生成的結構如下

<pre>
Attachment.cs
        [InverseProperty("AttachmentSnNavigation")]
        public virtual ICollection&lt;ReviewAttachment&gt; ReviewAttachments { get; set; }
        
ReviewAttachment.cs
        [ForeignKey("AttachmentSn")]
        [InverseProperty("ReviewAttachments")]
        public virtual Attachment AttachmentSnNavigation { get; set; } = null!;
</pre>
可以看出有**互相參照**的情形

在預設使用System.Text.Json做反序列化時，會形成無窮迴圈

造成如果以 **生成的Entity作為Reqponse回傳值**，就會死給你看

--

可以考慮是否不要直接把資料表Entity曝露在前端，一律轉view model回傳必要資訊即可

但要維持做法，也不是沒有方法

請拜讀**保哥**的文章[認識 Entity Framework Core 載入關聯資料的三種方法](https://blog.miniasp.com/post/2022/04/21/Loading-Related-Data-in-EF-Cor)
<ol>
  <li>可以另外對變數指定[JsonIgnore]，但是在**每次重新DatabaseFirst生成Entity時，要手動重做**</li>

  <li>
  可以在program.cs指定不循環參照(Core6.0+，但效率奇差，建議僅於快速升級時使用，事後還是要改寫)
  <pre>
  builder.Services.AddControllers().AddJsonOptions(options =>
      //物件表示在序列化期間偵測到參考週期時，忽略
      options.JsonSerializerOptions.ReferenceHandler = System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles;
  });
  </pre>
  </li>

  <li>如保哥文章(以及網路上大多數人的建議)，改採用NewtonsoftJson</li>
</ol>

--

#### 我傾向dll越少越好盡可能原生，所以轉view model回傳
