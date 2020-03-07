---
title: JavaScript 对象所有API解析
tags: JavaScript
---

转载自[路易斯博客（原链接）](https://louiszhai.github.io/2016/01/12/js.String) 稍作修改，精简说明内容，以便随时查阅。**新增最新API**


常用方法：charAt、indexOf、lastIndexOf、match、replace、search、slice、split、substr、substring、toLowerCase、toUpperCase、trim、valueof 等这些

# String构造器方法
#### String.fromCharCode(num1, ..., numN) 
返回使用指定的Unicode序列创建的字符串，也就是说传入Unicode序列，返回基于此创建的字符串。只能处理UCS-2编码，即所有字符都是2个字节，无法处理4字节字符

传入的参数均为数字
```
String.fromCharCode(65, 66, 67); // "ABC"
String.fromCharCode(97, 98, 99); // "abc"
String.fromCharCode(42); // "*"
String.fromCharCode(43); // "+"
String.fromCharCode(45); // "-"
String.fromCharCode(47); // "/"
```

#### String.fromCodePoint(num1[, ...[, numN]])  // ES6
作用和语法同fromCharCode方法，该方法主要扩展了对4字节字符的支持
```
// "𝌆" 是一个4字节字符，我们先看下它的数字形式
"𝌆".codePointAt(); // 119558
//调用fromCharCode解析之，返回乱码
String.fromCharCode(119558); // "팆"
//调用fromCodePoint解析之，正常解析
String.fromCodePoint(119558); // "𝌆"

// 规范了错误处理，无效的Unicode编码 会报错
String.fromCodePoint('abc'); // RangeError: Invalid code point NaN
String.fromCodePoint(Infinity); // RangeError: Invalid code point Infinity
String.fromCodePoint(-1.23); // RangeError: Invalid code point -1.23
```

#### String.raw(callSite, ...substitutions)
String.raw`templateString`

是一个模板字符串的标签函数，用来获取一个模板字符串的原始字面量

- callSite ： 一个模板字符串的“调用点对象”。类似{ raw: ['foo', 'bar', 'baz'] }。
- ...substitutions ： 任意个可选的参数，表示任意个内插表达式对应的值。
- templateString ： 模板字符串，可包含占位符（${...}）。

作为函数调用时，若第一个参数不是一个符合标准格式的对象，执行将抛出TypeError错误

String.raw作为前缀的用法
```
// 防止特殊字符串被转义
String.raw`a\nb\tc`; // 输出为 "a\nb\tc"
// 支持内插表达式
let name = "louis";
String.raw`Hello \n ${name}`;  // "Hello \n louis"
// 内插表达式还可以运算
String.raw`1+2=${1+2},2*3=${2*3}`; // "1+2=3,2*3=6"
```
String.raw作为函数来调用
```
// 对象的raw属性值为字符串时，从第二个参数起，它们分别被插入到下标为0，1，2，...n的元素后面
String.raw({raw: 'abcd'}, 1, 2, 3); // "a1b2c3d"
// 对象的raw属性值为数组时，从第二个参数起，它们分别被插入到数组下标为0，1，2，...n的元素后面
String.raw({raw: ['a', 'b', 'c', 'd']}, 1, 2, 3); // "a1b2c3d"
```
String.raw作为函数调用时，基本与ES6的tag标签模板一样
```
// 如下是tag函数的实现
function tag(){
  const array = arguments[0];
  return array.reduce((p, v, i) => p + (arguments[i] || '') + v);
}
// 回顾一个simple的tag标签模板
tag`Hello ${ 2 + 3 } world ${ 2 * 3 }`; // "Hello 5 world 6"
// 其实就想当于如下调用
tag(['Hello ', ' world ', ''], 5, 6); // "Hello 5 world 6"
```

# 原型 String.prototype
和其他所有对象一样，字符串实例的所有方法均来自String.prototype。以下是它的属性特性：

属性名称 | 状态
---|---
writable|	false
enumerable	|false
configurable|	false
字符串属性不可编辑，任何试图改变它属性的行为都将抛出错误
```
Object.getOwnPropertyNames(String.prototype);
// ["length", "constructor", "anchor", "big", "blink", "bold", "charAt", "charCodeAt", "codePointAt", "concat", "endsWith", "fontcolor", "fontsize", "fixed", "includes", "indexOf", "italics", "lastIndexOf", "link", "localeCompare", "match", "matchAll", "normalize", "padEnd", "padStart", "repeat", "replace", "search", "slice", "small", "split", "strike", "sub", "substr", "substring", "sup", "startsWith", "toString", "trim", "trimStart", "trimLeft", "trimEnd", "trimRight", "toLocaleLowerCase", "toLocaleUpperCase", "toLowerCase", "toUpperCase", "valueOf"]
```

## 属性
String.prototype共有两个属性，如下：

#### String.prototype.constructor 指向构造器(String())
#### String.prototype.length 表示字符串长度

## 方法
字符串原型方法分为两种，一种是html无关的方法，一种是html有关的方法。无论字符串方法如何厉害，都不至于强大到可以改变原字符串。

### HTML无关的方法

#### str.charAt(index)
返回字符串中指定位置的字符

index 为字符串索引（取值从0至length-1），如果超出该范围，则返回空串。
```
console.log("Hello, World".charAt(8)); // o, 返回下标为8的字符串o
```

#### str.charCodeAt(index)
返回指定索引处字符的 Unicode 数值

index 为一个从0至length-1的整数。如果不是一个数值，则默认为 0，如果小于0或者大于字符串长度，则返回 NaN。

Unicode 编码单元（code points）的范围从 0 到 1,114,111。开头的 128 个 Unicode 编码单元和 ASCII 字符编码一样.

charCodeAt() 总是返回一个小于 65,536 的值。因为高位编码单元需要由一对字符来表示，为了查看其编码的完成字符，需要查看 charCodeAt(i) 以及 charCodeAt(i+1) 的值
```
console.log("Hello, World".charCodeAt(8)); // 111
console.log("前端工程师".charCodeAt(2)); // 24037, 可见也可以查看中文Unicode编码
```

#### str.concat(string2, string3[, ..., stringN])
将一个或多个字符串与原字符串连接合并，形成一个新的字符串并返回

但是 concat 的性能表现不佳，强烈推荐使用赋值操作符（+或+=）代替 concat。”+” 操作符大概快了 concat 几十倍
```
console.log("早".concat("上","好")); // 早上好
```

#### str.indexOf(searchValue [, fromIndex=0]) 和 str.lastIndexOf(searchValue [, fromIndex=0])
indexOf方法：返回调用它的 String 对象中第一次出现的指定值的索引，从 fromIndex 处进行，从左往右查找搜索。如果未找到该值，则返回 -1。

lastIndexOf 则从右往左查找，其它与前者一致

- searchValue ：一个字符串表示被查找的值。如果没有提供确切地提供字符串，searchValue 会被强制设置为 "undefined"， 然后在当前字符串中查找这个值。
- fromIndex 可选 ：表示开始查找的位置。可以是任意整数，默认值为 0。如果 fromIndex 小于 0，则查找整个字符串（等价于传入了 0）。如果 fromIndex 大于等于 str.length，则必返回 -1。

```
console.log("".indexOf("",100)); // 0
console.log("IT改变世界".indexOf("世界")); // 4
console.log("IT改变世界".lastIndexOf("世界")); // 4
```
#### referenceStr.localeCompare(compareString[, locales[, options]])
返回一个数字来指示一个参考字符串是否在排序顺序前面或之后或与给定字符串相同。

该方法实现依赖具体的本地实现，不同的语言下可能有不同的返回。

新的 locales 、 options 参数能让应用程序定制函数的行为即指定用来排序的语言。  locales 和 options 参数是依赖于具体实现的，在旧的实现中这两个参数是完全被忽略的。

```
// The letter "a" is before "c" yielding a negative value
'a'.localeCompare('c'); 
// -2 or -1 (or some other negative value)

// Alphabetically the word "check" comes after "against" yielding a positive value
'check'.localeCompare('against'); 
// 2 or 1 (or some other positive value)

// "a" and "a" are equivalent yielding a neutral value of zero
'a'.localeCompare('a'); 
// 0
```
#### str.match(regexp)
检索返回一个字符串匹配正则表达式的的结果

regexp： 一个正则表达式对象。如果传入一个非正则表达式对象，则会隐式地使用 new RegExp(obj) 将其转换为一个 RegExp 。如果你没有给出任何参数并直接使用match() 方法 ，你将会得到一 个包含空字符串的 Array ：[""] 。

该方法返回包含匹配结果的数组，如果没有匹配项，则返回 null。

- 若正则表达式没有 g 标志，则返回同 RegExp.exec(str) 相同的结果。而且返回的数组拥有一个额外的 input 属性，该属性包含原始字符串，另外该数组还拥有一个 index 属性，该属性表示匹配字符串在原字符串中索引（从0开始）。
- 若正则表达式包含 g 标志，则该方法返回一个包含所有匹配结果的数组，没有匹配到则返回 null。

**相关 RegExp 方法**
- 若需测试字符串是否匹配正则，请参考 RegExp.test(str)。
- 若只需第一个匹配结果，请参考 RegExp.exec(str)。
```
var str = "World Internet Conference";
console.log(str.match(/[a-d]/i)); // ["d", index: 4, input: "World Internet Conference"]
console.log(str.match(/[a-d]/gi)); // ["d", "C", "c"]
// RegExp 方法如下
console.log(/[a-d]/gi.test(str)); // true
console.log(/[a-d]/gi.exec(str)); // ["d", index: 4, input: "World Internet Conference"]
```
由上可知，RegExp.test(str) 方法只要匹配到了一个字符也返回true。而RegExp.exec(str) 方法无论正则中有没有包含 g 标志，RegExp.exec将直接返回第一个匹配结果，且该结果同 str.match(regexp) 方法不包含 g 标志时的返回一致。

#### str.matchAll(regexp)  // Draft阶段，有兼容问题
返回一个包含所有匹配正则表达式的结果及分组捕获组的迭代器

参数同match
```
let regexp = /t(e)(st(\d?))/g;
let str = 'test1test2';

let array = [...str.matchAll(regexp)];

console.log(array[0]);
// expected output: Array ["test1", "e", "st1", "1"]

console.log(array[1]);
// expected output: Array ["test2", "e", "st2", "2"]
```

#### str.replace(regexp|substr, newSubStr|function)
[参考](https://louiszhai.github.io/2015/12/11/js.replace/)

返回一个由替换值（replacement）替换一些或所有匹配的模式（pattern）后的新字符串。模式可以是一个字符串或者一个正则表达式，替换值可以是一个字符串或者一个每次匹配都要调用的回调函数。

- regexp (pattern) ：一个RegExp 对象或者其字面量。该正则所匹配的内容会被第二个参数的返回值替换掉。
- substr (pattern) ：一个将被 newSubStr 替换的 字符串。其被视为一整个字符串，而不是一个正则表达式。仅第一个匹配项会被替换。
- newSubStr (replacement) ：用于替换掉第一个参数在原字符串中的匹配部分的字符串。该字符串中可以内插一些特殊的变量名。参考下面的使用字符串作为参数。
- function (replacement) ：一个用来创建新子字符串的函数，该函数的返回值将替换掉第一个参数匹配到的结果。参考下面的指定一个函数作为参数。

```
const p = 'The quick brown fox jumps over the lazy dog. If the dog reacted, was it really lazy?';

const regex = /dog/gi;

console.log(p.replace(regex, 'ferret'));
// "The quick brown fox jumps over the lazy ferret. If the ferret reacted, was it really lazy?"

console.log(p.replace('dog', 'monkey'));
// "The quick brown fox jumps over the lazy monkey. If the dog reacted, was it really lazy?"
```

#### str.search(regexp)
用于测试字符串对象是否包含某个正则匹配，相当于正则表达式的 test 方法，且该方法比 match() 方法更快。

regexp： 一个正则表达式（regular expression）对象。如果传入一个非正则表达式对象 obj，则会使用 new RegExp(obj) 隐式地将其转换为正则表达式对象。

返回值：如果匹配成功，则 search() 返回正则表达式在字符串中首次匹配项的索引;否则，返回 -1。

**与indexOf的区别**：search默认会将子串转化为正则表达式形式，而indexOf不做此处理，也不能处理正则。
```
var str = "abcdefg";
console.log(str.search(/[d-g]/)); // 3, 匹配到子串"defg",而d在原字符串中的索引为3

// 不支持全局匹配（正则中包含g参数）
console.log(str.search(/[d-g]/g)); // 3, 与无g参数时,返回相同
```

#### str.split([separator[, limit]])
把原字符串分割成子字符串组成数组，并返回该数组。

两个参数均是可选的
- separator ：表示分隔符，它可以是字符串也可以是正则表达式。如果忽略 separator，则返回的数组包含一个由原字符串组成的元素。如果 separator 是一个空串，则 str 将会被分割成一个由原字符串中字符组成的数组。
- limit ： 表示从返回的数组中截取前 limit 个元素，从而限定返回的数组长度。
```
var str = "today is a sunny day";
console.log(str.split()); // ["today is a sunny day"]
console.log(str.split("")); // ["t", "o", "d", "a", "y", " ", "i", "s", " ", "a", " ", "s", "u", "n", "n", "y", " ", "d", "a", "y"]
console.log(str.split(" ")); // ["today", "is", "a", "sunny", "day"]

// 使用limit限定返回的数组大小，如下：
console.log(str.split(" ", 1)); // ["today"]

// 使用正则分隔符（RegExp separator）
console.log(str.split(/\s*is\s*/)); // ["today", "a sunny day"]

// 若正则分隔符里包含捕获括号，则括号匹配的结果将会包含在返回的数组中。
console.log(str.split(/(\s*is\s*)/)); // ["today", " is ", "a sunny day"]
```

#### str.substr(start[, length])
返回字符串指定位置开始的指定数量的字符。

- start : 开始提取字符的位置。如果为负值，则被看作 strLength + start，其中 strLength 为字符串的长度（例如，如果 start 为 -3，则被看作 strLength + (-3)）。
- length : 可选。提取的字符数。

```
var str = "Yesterday is history. Tomorrow is mystery. But today is a gift.";
console.log(str.substr(47)); // today is a gift.
console.log(str.substr(-16)); // today is a gift.
```

#### str.substring(indexStart[, indexEnd])
返回一个字符串在开始索引到结束索引之间的一个子集, 或从开始索引直到字符串的末尾的一个子集。

- indexStart ：需要截取的第一个字符的索引，该索引位置的字符作为返回的字符串的首字母。
- indexEnd ：可选。一个 0 到字符串长度之间的整数，以该数字为索引的字符不包含在截取的字符串内，缺省则到末尾。
返回值 ：包含给定字符串的指定部分的新字符串。
```
var str = "Get outside every day. Miracles are waiting everywhere.";
console.log(str.substring(1,1)); // ""
console.log(str.substring(0)); // Get outside every day. Miracles are waiting everywhere.
console.log(str.substring(-1)); // Get outside every day. Miracles are waiting everywhere.
console.log(str.substring(0,100)); // Get outside every day. Miracles are waiting everywhere.
console.log(str.substring(22,NaN)); // Get outside every day.
```

#### str.toLocaleLowerCase([locale, locale, ...]) 和 str.toLocaleUpperCase([locale, locale, ...])
toLocaleLowerCase() 方法返回调用该方法的字符串被转换成小写的值，转换规则根据本地化的大小写映射。而toLocaleUpperCase() 方法则是转换成大写的值。

- locale 可选：参数 locale 指明要转换成小写格式的特定语言区域。 如果以一个数组 Array形式给出多个locales,  最合适的地区将被选出来应用（参见best available locale）。默认的locale是主机环境的当前区域(locale)设置。

返回值：根据任何特定于语言环境的案例映射规则将被调用字串转换成小写格式的一个新字符串。
```
console.log('ABCDEFG'.toLocaleLowerCase()); // abcdefg
console.log('abcdefg'.toLocaleUpperCase()); // ABCDEFG
```

#### str.toLowerCase() 和 str.toUpperCase()
toLowerCase() 方法将字符串转换为相应的小写。toUpperCase()转换为大写形式。
```
console.log('ABCDEFG'.toLowerCase()); // abcdefg
console.log('abcdefg'.toUpperCase()); // ABCDEFG
```

#### str.toString() 和 str.valueOf()
toString() 方法返回指定对象的字符串形式。
valueOf() 方法返回一个String对象的原始值（primitive value）。
```
var str = "abc";
console.log(str.toString()); // abc
console.log(str.toString()==str.valueOf()); // true
```

对于对象而言，toString和valueOf也是非常的相似，它们之间有着细微的差别
```
var x = {
    toString: function () { return "test"; },
    valueOf: function () { return 123; }
};

console.log(x); // {toString: ƒ, valueOf: ƒ}
console.log("x=" + x); // "x=123"
console.log(x + "=x"); // "123=x"
console.log(x + "1"); // 1231
console.log(x + 1); // 124
console.log(["x=", x].join("")); // "x=test"
```
当 “+” 操作符一边为数字时，对象x趋向于转换为数字，表达式会优先调用 valueOf 方法，如果调用数组的 join 方法，对象x趋向于转换为字符串，表达式会优先调用 toString 方法。

#### str.trim()
从一个字符串的两端删除空白字符。在这个上下文中的空白字符是所有的空白字符 (space, tab, no-break space 等) 以及所有行终止符字符（如 LF，CR等）。返回一个新字符串。
```
console.log(" a b c ".trim()); // "a b c"
```
trim() 方法是 ECMAScript 5.1 标准加入的，它并不支持IE9以下的低版本IE浏览器。polyfill写法：
```
if (!String.prototype.trim) {
  String.prototype.trim = function () {
    return this.replace(/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g, '');
  };
}
```

##### str.trimEnd() 或 str.trimRight()
从一个字符串的末端、右侧移除空白字符。返回一个新字符串。
```
const greeting = '   Hello world!   ';
console.log(greeting.trimEnd()); // "   Hello world!";
```
##### str.trimStart() 或 str.trimLeft()
从字符串的开头、左侧删除空格。返回一个新字符串。

#### str.codePointAt(pos) // ES6
返回使用UTF-16编码的给定位置的值的非负整数。
```
console.log("a".codePointAt(0)); // 97
console.log("\u4f60\u597d".codePointAt(0)); // 20320
```

#### str.includes(searchString[, position]) // ES6
用来判断一个字符串是否属于另一个字符

- subString ： 表示要搜索的字符串。
- position ： 表示从当前字符串的哪个位置开始搜索字符串，默认值为0。
```
var str = "Practice makes perfect.";
console.log(str.includes("perfect")); // true
console.log(str.includes("perfect",100)); // false
```

#### str.startsWith(searchString[, position]) // ES6
用来判断当前字符串是否以另外一个给定的子字符串开头，并根据判断结果返回 true 或 false。

- searchString： 要搜索的子字符串。
- position 可选： 在 str 中搜索 searchString 的开始位置，默认值为 0，也就是真正的字符串开头处。
```
var str = "Where there is a will, there is a way.";
console.log(str.startsWith("Where")); // true
console.log(str.startsWith("there",6)); // true
```

#### str.endsWith(searchString[, length]) // ES6
用来判断当前字符串是否是以另外一个给定的子字符串“结尾”的，根据判断结果返回 true 或 false。

- searchString ： 要搜索的子字符串。
- length ： 可选。作为 str 的长度。默认值为 str.length。
```
var str = "Learn and live.";
console.log(str.endsWith("live.")); // true
console.log(str.endsWith("Learn",5)); // true
```

#### str.normalize([form]) // ES6
会按照指定的一种 Unicode 正规形式将当前字符串正规化。（如果该值不是字符串，则首先将其转换为一个字符串）。

- form 可选：四种 Unicode 正规形式（Unicode Normalization Form）"NFC"、"NFD"、"NFKC"，或 "NFKD" 其中的一个, 默认值为 "NFC"。
这四个值的含义分别如下：
1. "NFC"：Canonical Decomposition, followed by Canonical Composition.
1. "NFD"：Canonical Decomposition.
1. "NFKC"：Compatibility Decomposition, followed by Canonical Composition.
1. "NFKD"：Compatibility Decomposition.

返回值：含有给定字符串的 Unicode 规范化形式的字符串。
```
var str = "\u4f60\u597d";
console.log(str.normalize()); // 你好
console.log(str.normalize("NFC")); // 你好
console.log(str.normalize("NFD")); // 你好
console.log(str.normalize("NFKC")); // 你好
console.log(str.normalize("NFKD")); // 你好
```

#### str.repeat(count) // ES6
返回重复原字符串多次的新字符串

- count：介于0和正无穷大之间的整数 : [0, +∞) 。表示在新构造的字符串中重复了多少遍原字符串。
```
var str = "A still tongue makes a wise head.";
console.log(str.repeat(0)); // ""
console.log(str.repeat(1)); // A still tongue makes a wise head.
console.log(str.repeat(1.5)); // A still tongue makes a wise head.
console.log(str.repeat(-1)); // RangeError:Invalid count value
```
#### str.padStart(targetLength [, padString]) // ECMAScript 2017
功能同padEnd，唯一不同的是 填充从当前字符串的开始(左侧)应用的。
```
'abc'.padStart(10);         // "       abc"
'abc'.padStart(10, "foo");  // "foofoofabc"
'abc'.padStart(6,"123465"); // "123abc"
'abc'.padStart(8, "0");     // "00000abc"
'abc'.padStart(1);          // "abc"
```

#### str.padEnd(targetLength [, padString]) // ECMAScript 2017
用一个字符串填充当前字符串（如果需要的话则重复填充），返回填充后达到指定长度的字符串。从当前字符串的末尾（右侧）开始填充。

- targetLength ：当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。
- padString 可选 ：填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断。此参数的缺省值为 " "（U+0020）。

```
'abc'.padEnd(10);          // "abc       "
'abc'.padEnd(10, "foo");   // "abcfoofoof"
'abc'.padEnd(6, "123456"); // "abc123"
'abc'.padEnd(1);           // "abc"
```

### HTML相关的方法（均已废弃）
