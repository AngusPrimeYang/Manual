提供我很佩服的公司ABNB的說明 respect

https://github.com/airbnb/javascript#ecmascript-6-es-2015-styles

除了提供ES6+ features外，還有Guideline建議使用方法

已有繁體中文，不看此文直接去看Airbnb JavaScript Style Guide也行

https://github.com/jigsawye/javascript

--

* instanceof
<pre>
[1, 2, 3] instanceof Array // true
/abc/ instanceof RegExp // true
({}) instanceof Object // true
(function(){}) instanceof Function // true
</pre>
* Math
<pre>
2 ** 10 // Math.pow(2, 10)
~~2.5 // Math.floor(2.5, 10)
~2.5 // Math.floor(-2.5, 10)
</pre>
* String
<pre>
String(123) // 123 + ""
let str = +"123"; parseInt("123")

const [a, b, c, d, e] = "12345"
console.log(c) // 3

// template strings
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`
}
getFullName({firstName: 'fn', lastName: 'ln'}) //'fn ln'
</pre>
* Boolean
<pre>
Boolean(0) // false
let result0 = !"1"; // false
let result1 = !!"1"; // true
</pre>
* Nunmer
<pre>
"123" >> 0 // 123 only 32-bit Int
"128" << 1 // 256
"128" >> 1 // 64
Number("123") // parseInt(inputValue, 10) // 123

typeof NaN === 'number'; // true
Number.isNaN(Number('1.2.3')); //NaN, true
Number.isFinite(parseInt('2e3', 10)); //2, true
</pre>
* Array
<pre>
const itemsCopy = [...items]

const [x, ...y] = [1, 2, 3]
console.log(x) //1
console.log(y) //[2,3]
</pre>
* JSON
<pre>
const { x, y } = { x: 5, y: 10 } // x=5, y=10
const { user: x } = { user: 5 } // x=5

// Rest Properties
let { x = 0, y, ...z } = { x: 1, y: 2, a: 3, b: 4 }
console.log(x) // 1
console.log(z) // { a: 3, b: 4 }

// Spread Properties
let n = { x, y, ...z }
console.log(n) // { x: 1, y: 2, a: 3, b: 4 }

// pratice
const original = { a: 1, b: 2 }
const copy = { ...original, c: 3 } // copy = { a: 1, b: 2, c: 3 }
const { a, ...noA } = copy // noA = { b: 2, c: 3 }

// default's default 
function func({a = 3, b = 5} = {a: 7, b: 11}) {
  return a + b
}
func({a: 1, b: 2}) // 3
func({a: 1}) // 6
func({b: 2}) // 5
func({}) // 8
func() // 18

//lambda
const f1 = ((a) => {
	const {a1, a2} = a;
	return a1;
});
f1({a1:1,a2:2}) // 1
</pre>
</pre>
* JSON + for
<pre>
let people = [
	{name: "a1", family: { father: "a", mother: "1" }},
	{name: "a2", family: { father: "a", mother: "2" }}
];
for (let {name: n, family: { father: f } } of people) {
  console.log('Name: ' + n + ', Father: ' + f)
}
// Name: a1, Father: a
// Name: a2, Father: a
</pre>

--

### 防止汙染
> 一般定義在js中的function，都是直接註冊在最上層，如果同時引用2個js，裡面包含同名函式，會有汙染問題<br />
> 為了防止，有幾個方法可以參考

* Immediately Invoked Function Expression(IIFE)
<pre>
(function IIFE() {
  console.log("Immediately");
})();

console.log(IIFE); //error!
</pre>
或使用在定義觸發事件時，資料來源可以有很好的隔離
<pre>
// parameters already defined before trigger
function (mark, msgBox, target) {
    return function () {
        msgBox.open(target.area, mark);
    }
}(marker, msgBox, this);
</pre>
<br />

* modularization
<pre>
var a = 1;
let module = (function(parm){
	var a = 2;
	console.log('a:' + a); // a:2
	console.log('parm:' + parm); // parm:p
	f1 = (g) => g+1
	f2 = (h) => h-1
	return { f1: f1, f2: f2 }
})('p'); // IIFE
console.log('a:' + a); // a:1
module.f1(3); // 4
</pre>

--

Reference

https://github.com/airbnb/javascript#ecmascript-6-es-2015-styles

https://github.com/airbnb/javascript#ecmascript-6-es-2015-styles

https://eslint.org/docs/latest/rules/

https://eyesofkids.gitbooks.io/javascript-start-from-es6/content/
