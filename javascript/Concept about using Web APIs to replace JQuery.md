有Web APIs可以用了，就很想拔掉jqeury，因為一般使用jquery不外乎只會用到幾個
* selector
* ajax
* event handler
現在這些Web APIs都有提供簡單的對應方法，一點點語法糖就可以達成

除非已經完全採用前端引擎、或著已自行發展出生態系、或著三方元件包袱過重

直接換掉可以省下近100k*million times的傳輸成本(現在jquery-min約88k)

而且這塊已經是瀏覽器大戰戰後的結果，nothing to lose

--

* selector

document.querySelector()與document.querySelectorAll()，都是由css Selectors出發。跟jquery異曲同工，所以很親民

最大的差別在於回傳值是原生物件，沒有再封裝一層成jquery object形式
<pre>
// select by TAG
target.querySelector("input").value = "";
//select by Class
divE.querySelectorAll(".divExt"); //array
//select by Id
document.querySelector(".divExt .changeExt").click();
</pre>
那，跟傳統的getElement(s)差在哪?
<pre>
getElementById
getElementsByClassName
getElementsByName
getElementsByTagName
</pre>
主要在於querySelector(All)取得的像是副本Copy，取得之後如果對象內容有異動，不會作用在取得結果上
<pre>
let el = myOl.getElementsByTagName("li");
let sl = myOl.querySelectorAll("li");

console.log(el.length); // 5
console.log(sl.length); // 5

myOl.appendChild(document.createElement('li'));

console.log(el.length); // 6
console.log(sl.length); // 5

sl.appendChild(document.createElement('li'));

console.log(el.length); // 7
console.log(sl.length); // 5
</pre>
會不斷變動內容，且變動後會繼續用來操作的對象，建議維持使用getElement(s)

--

* ajax

以前為了方便，自己弄了一個通用函式讓程式碼乾一點，以下僅為範例
<pre>
// jquery.ajax example

function callAjax(url, data, successCallBack, successParams) {
	return $.ajax({
		url:url,
		type : "POST",
		data : (typeof data === "undefined" ? {} : data),
		success : function(data) {
			if (typeof successCallBack === "function") {
				successCallBack(data,successParams);
			}
		},
		error:function(xhr) {
			createAJAXErrorMessage(xhr);
		}
	});
}
</pre>
後來改用Fetch寫，幾乎可以簡單應付各種情況<br />
multipart的話data要以FormData格式直接放到body，headers裡的格式不要設定，遇到FormData時會自動以multipart處理
<pre>
const FetchType = {
    GET: "GET",
    POST: "POST",
    PUT: "PUT",
    DELETE: "DELETE",
    IsQueryType: (type) => type == FetchType.GET || type == FetchType.DELETE
}
//multipart mode，data type of date must be FormData, no need to set header types
async function FetchAPI(url, type, data, callback) {
    if (FetchType.IsQueryType(type) && data) {
        url += "?" + new URLSearchParams(data).toString();
    }

    let result = await fetch(url, {
        credentials: 'include',
        headers: {
            "Content-Type": "application/json",
            "Accept": "application/json",
        },
        method: type,
        mode: 'cors',
        cache: 'default',
        body: FetchType.IsQueryType(type) ? null : JSON.stringify(data ?? {})
    })
    .then(resp => resp.status == 200 ? resp.json() : null)
    .catch((obj) => {
        alert('Ajax request 發生錯誤:' + obj.statusText);
    });

    if (typeof callback == "function") {
        callback(result);
    }
    return result;
}
</pre>
其中特別要注意的是那個async，如果取得的資料，不透過callback function，但接著還是要拿來利用，請務必做好await
dotNet使用者應該很快能夠理解，否則可以用jquery的async: false去想像
<pre>
let result = await FetchAPI("Api/GetData", FetchType.GET, { sysId: sysId });
if (data.some(d => d.gid)) {
  for (let d in data) {
    ...
  }
}
</pre>

--

* event handler

事件的話簡單寫一些語法糖就可以取代，但建議改用前端引擎好好規劃
<pre>
Element.prototype.trigger = function (eventType) {
    this.dispatchEvent(new Event(eventType));
};

Element.prototype.click = function () {
    this.trigger("click");
};

Element.prototype.change = function (val) {
    if (this.value == val) {
        return;
    }
    this.value = val;
    this.trigger("change");
};
</pre>

--

Reference

css Selectors
[Level 1](https://www.w3.org/TR/selectors-api/)
[Level 2](https://www.w3.org/TR/DOM-Level-2-Style/)
[Level 3](https://www.w3.org/TR/selectors-3/#selectors)

https://developer.mozilla.org/en-US/docs/Web/API

fetch
https://developer.mozilla.org/en-US/docs/Web/API/fetch

event
https://developer.mozilla.org/zh-TW/docs/Web/API/Event/Event

HTMLElement (contextual menu, etc...)
https://developer.mozilla.org/zh-TW/docs/Web/API/HTMLElement




