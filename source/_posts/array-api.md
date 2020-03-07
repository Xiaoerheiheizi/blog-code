---
title: JavaScript Array（数组）所有API解析
tags: JavaScript
---

转载自[路易斯博客（原链接）](https://louiszhai.github.io/2017/04/28/array/) 稍作修改，精简说明内容，以便随时查阅。**新增最新API**
<!--more-->

类数组对象字符串（String），像join方法（不改变当前对象自身）就完全适用，可惜的是 Array.prototype 中很多方法均会去试图修改当前对象的 length 属性，比如说 pop、push、shift, unshift 方法，操作 String 对象时，由于String对象的长度本身不可更改，这将导致抛出TypeError错误

Array.prototype 的各方法间的共性：
- 所有插入元素的方法, 比如 push、unshift，一律返回数组新的长度；
- 所有删除元素的方法，比如 pop、shift、splice 一律返回删除的元素，或者返回删除的多个元素组成的数组；
- 部分遍历方法，比如 forEach、every、some、filter、map、find、findIndex，它们都包含function(value,index,array){} 和 thisArg 这样两个形参。
- 所有方法均具有鸭式辨型这种神奇的特性。它们不止可以用来处理数组对象，还可以处理类数组对象。

# 构造器
## Array构造器
用于创建一个新的数组。通常推荐对象字面量创建数组，但一些情况下构造器会更合适。

- new Array(arg1, arg2,…)，参数长度为0或长度大于等于2时，传入的参数将按照顺序依次成为新数组的第0至N项（参数长度为0时，返回空数组）。
- new Array(len)，当len不是数值时，处理同上，返回一个只包含len元素一项的数组；当len为数值时，根据如下规范，len最大不能超过32位无符号整型，即需要小于2的32次方（len最大为Math.pow(2,32) -1或-1>>>0），否则将抛出RangeError。
```
// Array 构造器
var a = Array(8); // [undefined × 8] 没有new会默认new一个实例
// 使用对象字面量
var b = [];
b.length = 8; // [undefined × 8]
```
## ES6新增构造函数方法
### Array.of
用于将参数依次转化为数组中的一项，返回这个新数组，不管参数是数字或其他。与Array构造器功能一致，唯一区别在于单个数字参数的处理上。需要使用数组包裹元素时，推荐优先使用Array.fo方法
```
Array.of(9.0); // [8]
Array(9.0); // [empty x 8]
```
其他版本浏览器不支持时，实现一个polyfill
```
if(!Array.of){
    Array.of = function () {
        return Array.prototype.slice.call(arguments)
    }
}
```

### Array.from(arrayLike[, mapFunction[, thisArg]])
- arrayLike：必传参数，想要转换成数组的伪数组对象或任意一个可迭代对象。
- mapFunction：可选参数，mapFunction(item，index){...} 是在集合中的每个项目上调用的函数。返回的值将插入到新集合中。
- thisArg：可选参数，执行回调函数 mapFunction 时 this 对象。这个参数很少使用。
任何一种编程语言都具有超出基本用法的功能，它得益于成功的设计和试图去解决广泛问题。

Array.from的设计初衷是**从一个类似数组的可迭代对象创建一个新的数组实例**。只要有一个对象有迭代器，Array.from就能把它变成一个数组（返回新的数组，不改变原对象）。

Array.from() 方法接受类数组对象以及可迭代对象，它可以接受一个 map 函数，并且，这个 map 函数不会跳过值为 undefined 的数值项。这些特性给 Array.from() 提供了很多可能。

Array.from：允许在 JavaScript 集合(如: 数组、类数组对象、或者是字符串、map 、set 等可迭代对象) 上进行有用的转换。
```
// String
Array.from('abc'); // ['a', 'b', 'c']
// Set
Array.from(new Set(['abc', 'def']); // ['abc', 'def']
// Map
Array。from(new Map([[1, 'abc'], [2, 'def']])); // [[1, 'abc'], [2, 'def']]
// 类数组对象arguments
function fn(){
    return Array.from(arguments)
}
fn(1,2,3); // [1,2,3]

// 扩展。如：生成一个从0到指定数字的新数组
Array.from({length: 10}, (v, i) => i); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**1. 将类数组的每一项乘以2**
```
const someNumbers = { '0': 10, '1': 15, length: 2 };

Array.from(someNumbers, value => value * 2); // => [20, 30]
```

**2. 对函数的参数求和（可将类数组转换成数组）**

Array.from(arguments) 将类数组对象 arguments 转换成一个数组，然后使用数组的 reduce 方法求和。
```
function sumArguments() {
    return Array.from(arguments).reduce((sum, num) => sum + num);
}

sumArguments(1, 2, 3); // => 6
```

**3. 克隆一个数组**

浅拷贝实现
```
const numbers = [3, 6, 9];
const numbersCopy = Array.from(numbers);

numbers === numbersCopy; // => false
```
深拷贝实现 （递归）
```
function recursiveClone(val) {
    return Array.isArray(val) ? Array.from(val, recursiveClone) : val;
}

const numbers = [[0, 1, 2], ['one', 'two', 'three']];
const numbersClone = recursiveClone(numbers);

numbersClone; // => [[0, 1, 2], ['one', 'two', 'three']]
numbers[0] === numbersClone[0] // => false

```

**4. 使用值填充数组**
```
const length = 3;
const init   = 0;
const result = Array.from({ length }, () => init);
// fill也可以
const fillResult = Array(length).fill(init);

result; // => [0, 0, 0]
```

**5. 使用对象填充数组**

当初始化数组的每个项都应该是一个新对象时，Array.from() 是一个更好的解决方案
```
const length = 3;
const resultA = Array.from({ length }, () => ({}));
const resultB = Array(length).fill({});

resultA; // => [{}, {}, {}]
resultB; // => [{}, {}, {}]

resultA[0] === resultA[1]; // => false
resultB[0] === resultB[1]; // => true
```

**6. 生成数字范围**

当初始化数组的每个项都应该是一个新对象时，Array.from() 是一个更好的解决方案
```
function range(end) {
    return Array.from({ length: end }, (_, index) => index);
}

range(4); // => [0, 1, 2, 3]
```


**7. 数组去重**

Array.from() 的入参是可迭代对象，因而我们可以利用其与 Set 结合来实现快速从数组中删除重复项。
```
function unique(array) {
  return Array.from(new Set(array));
}

unique([1, 1, 2, 3, 3]); // => [1, 2, 3]
```

# Array.isArray
判断一个变量是否是数组类型
```
Array.isArray([]); // true
Array.isArray({0:'a', length: 1}); // false
```
以下四种判断一个值是否是数组的方法，都基于继承，当手动指定了某个对象的__proto__属性为Array.prototype，以下四种方法都将无效：
```
var a = {
    __proto__ : Array.prototype
};
// 1. 基于instanceof
a instanceof Array; // true
// 2. 基于constructor
a.constructor === Array; // true
// 3. 基于Object.prototype.isPrototypeOf
Array.prototype.isPrototypeOf(a); // true
// 4. 基于Object.getPrototypeOf
Object.getPrototypeOf(a) === Array.prototype; // true
```
通过Object.prototype.toString去判断一个值的类型，是各大主流库的标准
```
// 基于Object.prototype.toString
Object.prototype.toString(a) === '[object Array]'; // false
```
Array.isArray的polyfill
```
if(!Array.isArray){
    Array.isArray = function(arg) {
        return Object.prototype.toString.call(arg) === '[object Array]'
    }
}
```

# 原型 Array.prototype
js中所有数组方法均来自于Array.prototype（基于继承）
```
Array.isArray(Array.prototype); // true
console.log(Array.prototype.length); // 0
console.log([].__proto__.length); // 0
console.log([].__proto__); // [Symbol(Symbol.unscopables): Object]
```
数组原型提供的方法非常之多，主要分为三种，一种是会改变自身值的，一种是不会改变自身值的，另外一种是遍历方法。

由于 Array.prototype 的某些属性被设置为[[DontEnum]]，因此不能用一般的方法进行遍历，我们可以通过如下方式获取 Array.prototype 的所有方法：
```
Object.getOwnPropertyNames(Array.prototype); // ["length", "constructor", "concat", "copyWithin", "fill", "find", "findIndex", "lastIndexOf", "pop", "push", "reverse", "shift", "unshift", "slice", "sort", "splice", "includes", "indexOf", "join", "keys", "entries", "values", "forEach", "filter", "flat", "flatMap", "map", "every", "some", "reduce", "reduceRight", "toLocaleString", "toString"]
```

## 改变自身值的方法（8个）
### pop()
删除一个数组中最后一个元素，并返回这个元素。如果是栈的话，这个过程是栈顶弹出。
```
var array = ["cat", "dog", "cow", "chicken", "mouse"];
var item = array.pop();
console.log(array); // ["cat", "dog", "cow", "chicken"]
console.log(item); // mouse
```
由于设计上的巧妙，pop方法可以应用在类数组对象上，即 鸭式辨型. 如下：
```
var o = {0:"cat", 1:"dog", 2:"cow", 3:"chicken", 4:"mouse", length:5}
var item = Array.prototype.pop.call(o);
console.log(o); // Object {0: "cat", 1: "dog", 2: "cow", 3: "chicken", length: 4}
console.log(item); // mouse
```
但如果类数组对象不具有length属性，那么该对象将被创建length属性，length值为0。如下：
```
var o = {0:"cat", 1:"dog", 2:"cow", 3:"chicken", 4:"mouse"}
var item = Array.prototype.pop.call(o);
console.log(array); // Object {0: "cat", 1: "dog", 2: "cow", 3: "chicken", 4: "mouse", length: 0}
console.log(item); // undefined
```
### push()
添加一个或者多个元素到数组末尾，并且返回数组新的长度。如果是栈的话，这个过程是栈顶压入。
```
var array = ["football", "basketball", "volleyball", "Table tennis", "badminton"];
var i = array.push("golfball");
console.log(array); // ["football", "basketball", "volleyball", "Table tennis", "badminton", "golfball"]
console.log(i); // 6
```
同pop方法一样，push方法也可以应用到类数组对象上，如果length不存在或不能转成一个数值，则插入元素索引为0，不存在时将会创建length。push方法根据length属性决定从那里插入给定的值。
```
var o = {0:"football", 1:"basketball"};
var i = Array.prototype.push.call(o, "golfball");
console.log(o); // Object {0: "golfball", 1: "basketball", length: 1}
console.log(i); // 1
```
利用push根据length属性插入元素这个特点，可以实现数组的合并
```
var array = ["football", "basketball"];
var array2 = ["volleyball", "golfball"];
var i = Array.prototype.push.apply(array,array2);
console.log(array); // ["football", "basketball", "volleyball", "golfball"]
console.log(i); // 4
```
### shift()
删除数组的第一个元素，并返回这个元素。如果是栈的话，这个过程就是栈底弹出。
```
var array = [1,2,3,4,5];
var item = array.shift();
console.log(array); // [2,3,4,5]
console.log(item); // 1
```
同样受益于鸭式辨型，对于类数组对象，shift仍然能够处理。
```
var o = {0:"a", 1:"b", 2:"c", length:3};
var item = Array.prototype.shift.call(o);
console.log(o); // Object {0: "b", 1: "c", length: 2}
console.log(item); // a
```
如果类数组对象length属性不存在，将添加length属性，并初始化为0
```
var o = {0:"a", 1:"b", 2:"c"};
var item = Array.prototype.shift.call(o);
console.log(o); // Object {0: "a", 1: "b", 2:"c" length: 0}
console.log(item); // undefined
```
### reverse()
颠倒数组中元素的位置，第一个成为最后一个，最后一个成为第一个，返回对数组的引用。
```
var array = [1,2,3,4,5];
var array2 = array.reverse();
console.log(array); // [5,4,3,2,1]
console.log(array2===array); // true
```
reverse 也是鸭式辨型的受益者，颠倒元素的范围受length属性制约，如果length属性小于2或不为数值，原类数组对象将没有变化。length属性不存在也不会去创建。当length属性较大时，类数组对象的[索引]会尽可能向length看齐。
```
var o = {0:"a", 1:"b", 2:"c",length:100};
var o2 = Array.prototype.reverse.call(o);
console.log(o); // Object {97: "c", 98: "b", 99: "a", length: 100}
console.log(o === o2); // true
```
### sort([comparefn])
对数组元素进行排序，并返回这个数组的引用。comparefn是可选的，如果省略，数组元素将按照各自转换为字符串的Unicode(万国码)位点顺序排序。如果指明了comparefn，数组将按照调用该函数的返回值来排序。

使用数组的sort方法需要注意一点：各浏览器的针对sort方法内部算法实现不尽相同，**排序函数尽量只返回-1、0、1三种不同的值**，不要尝试返回true或false等其它数值，因为可能导致不可靠的排序结果。
```
var array = ["apple","Boy","Cat","dog"];
var array2 = array.sort();
console.log(array); // ["Boy", "Cat", "apple", "dog"]
console.log(array2 == array); // true

array = [10, 1, 3, 20];
var array3 = array.sort();
console.log(array3); // [1, 10, 20, 3]
```
sort一样受益于鸭式辨型，若类数组对象不具有length属性，它并不会进行排序，也不会为其添加length属性
```
var o = {0:'互',1:'联',2:'网',3:'改',4:'变',5:'世',6:'界',length:7};
Array.prototype.sort.call(o,function(a, b){
  return a.localeCompare(b);
});
console.log(o); // Object {0: "变", 1: "改", 2: "互", 3: "界", 4: "联", 5: "世", 6: "网", length: 7}
```
#### 使用映射改善排序
comparefn 如果需要对数组元素多次转换以实现排序，那么使用map辅助排序将是个不错的选择。基本思想就是将数组中的每个元素实际比较的值取出来，排序后再将数组恢复。
```
// 需要被排序的数组
var array = ['dog', 'Cat', 'Boy', 'apple'];
// 对需要排序的数字和位置的临时存储
var mapped = array.map(function(el, i) {
  return { index: i, value: el.toLowerCase() };
})
// 按照多个值排序数组
mapped.sort(function(a, b) {
  return +(a.value > b.value) || +(a.value === b.value) - 1;
});
// 根据索引得到排序的结果
var result = mapped.map(function(el){
  return array[el.index];
});
console.log(result); // ["apple", "Boy", "Cat", "dog"]
```
### splice(start[, deleteCount[, item1[, item2[, ...]]]])
用新元素替换旧元素的方式来修改数组。

- start 指定修改的开始位置（从0计数）。如果超出了数组的长度，则从数组末尾开始添加内容；如果是负值，则表示从数组末位开始的第几位（从-1计数，这意味着-n是倒数第n个元素并且等价于array.length-n）；如果负数的绝对值大于数组的长度，则表示开始位置为第0位。

- deleteCount （可选）

整数，表示要移除的数组元素的个数。
如果 deleteCount 大于 start 之后的元素的总数，则从 start 后面的元素都将被删除（含第 start 位）。
如果 deleteCount 被省略了，或者它的值大于等于array.length - start(也就是说，如果它大于或者等于start之后的所有元素的数量)，那么start之后数组的所有元素都会被删除。
如果 deleteCount 是 0 或者负数，则不移除元素。这种情况下，至少应添加一个新元素。

- itemN （可选）要添加进数组的元素,从start 位置开始。如果不指定，则 splice() 将只删除数组元素。

- 返回值 由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组。
```
var array = ["apple","boy"];
var splices = array.splice(1,1);
console.log(array); // ["apple"]
console.log(splices); // ["boy"] ,可见是从数组下标为1的元素开始删除,并且删除一个元素,由于itemN缺省,故此时该方法只删除元素

array = ["apple","boy"];
splices = array.splice(2,1,"cat");
console.log(array); // ["apple", "boy", "cat"]
console.log(splices); // [], 可见由于start超过数组长度,此时从数组末尾开始添加元素,并且原数组不会发生删除行为

array = ["apple","boy"];
splices = array.splice(-2,1,"cat");
console.log(array); // ["cat", "boy"]
console.log(splices); // ["apple"], 可见当start为负值时,是从数组末尾开始的第-start位开始删除,删除一个元素,并且从此处插入了一个元素

array = ["apple","boy"];
splices = array.splice(-3,1,"cat");
console.log(array); // ["cat", "boy"]
console.log(splices); // ["apple"], 可见即使-start超出数组长度,数组默认从首位开始删除

array = ["apple","boy"];
splices = array.splice(0,3,"cat");
console.log(array); // ["cat"]
console.log(splices); // ["apple", "boy"], 可见当deleteCount大于数组start之后的元素总和时,start及之后的元素都将被删除
```
splice一样受益于鸭式辨型。如果类数组对象没有length属性，splice将为该类数组对象添加length属性，并初始化为0
```
var o = {0:"apple",1:"boy",length:2};
var splices = Array.prototype.splice.call(o,1,1);
console.log(o); // Object {0: "apple", length: 1}, 可见对象o删除了一个属性,并且length-1
console.log(splices); // ["boy"]
```
如果需要删除数组中一个已存在的元素
```
var array = ['a','b','c'];
array.splice(array.indexOf('b'),1);
```
### unshift(element1, …, elementN)
用于在数组开始处插入一些元素（栈底插入），并返回新数组的长度。
```
var array = ["red", "green", "blue"];
var length = array.unshift(["yellow"]);
console.log(array); // [["yellow"], "red", "green", "blue"]
console.log(length); // 4, 可见数组也能成功插入
```
unshift也受益于鸭式辨型。如果类数组对象不指定length属性，shift会认为数组长度为0，此时将从对象下标为0的位置开始插入，相应位置属性将被替换，此时初始化类数组对象的length属性为插入元素个数。
```
var o = {0:"red", 1:"green", 2:"blue",length:3};
var length = Array.prototype.unshift.call(o,"gray");
console.log(o); // Object {0: "gray", 1: "red", 2: "green", 3: "blue", length: 4}
console.log(length); // 4
```
### copyWithin(target[, start[, end]])  // ES6
浅复制数组的一部分到同一数组中的另一个位置，并返回它，不会改变原数组的长度。

参数 target、start 和 end 必须为整数

taget 指定被替换元素的索引，start 指定替换元素起始的索引，end 可选，指的是替换元素结束位置的索引。

如果start和end为负，将从末尾开始计算。
```
var array = [1,2,3,4,5]; 
var array2 = array.copyWithin(0,3);
console.log(array===array2,array2); // true [4, 5, 3, 4, 5]

var array = [1,2,3,4,5]; 
console.log(array.copyWithin(0,3,4)); // [4, 2, 3, 4, 5]

var array = [1,2,3,4,5]; 
console.log(array.copyWithin(0,-2,-1)); // [4, 2, 3, 4, 5]
```
copyWithin一样受益于鸭式辨型
```
var o = {0:1, 1:2, 2:3, 3:4, 4:5,length:5}
var o2 = Array.prototype.copyWithin.call(o,0,3);
console.log(o===o2,o2); // true Object { 0=4,  1=5,  2=3,  更多...}
```
### fill(value[, start[, end]])  // ES6
用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。

value：用来填充数组元素的值。start：可选 起始索引，默认值为0。end：可选 终止索引，默认值为 this.length。

```
var array = [1,2,3,4,5];
var array2 = array.fill(10,0,3);
console.log(array===array2,array2); // true [10, 10, 10, 4, 5], 可见数组区间[0,3]的元素全部替换为10
// 其他的举例请参考copyWithin
```
fill 一样受益于鸭式辨型
```
var o = {0:1, 1:2, 2:3, 3:4, 4:5,length:5}
var o2 = Array.prototype.fill.call(o,10,0,2);
console.log(o===o2,o2); true Object { 0=10,  1=10,  2=3,  更多...}
```
## 不会改变自身方法
### concat(value1, value2, …, valueN)
将传入的数组或者元素与原数组合并，组成一个新数组并返回
```
var array = [1, 2, 3];
var array2 = array.concat(4,[5,6],[7,8,9]);
console.log(array2); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log(array); // [1, 2, 3], 可见原数组并未被修改
```
若concat方法中不传入参数，那么将基于原数组浅复制生成一个一模一样的新数组（指向新的地址空间）
```
var array = [{a: 1}];
var array3 = array.concat();
console.log(array3); // [{a: 1}]
console.log(array3 === array); // false
console.log(array[0] === array3[0]); // true，新旧数组第一个元素依旧共用一个同一个对象的引用
```
concat 一样受益于鸭式辨型。类数组对象合并后返回的是依然是数组，并不是期望的对象
```
var o = {0:"a", 1:"b", 2:"c",length:3};
var o2 = Array.prototype.concat.call(o,'d',{3:'e',4:'f',length:2},['g','h','i']);
console.log(o2); // [{0:"a", 1:"b", 2:"c", length:3}, 'd', {3:'e', 4:'f', length:2}, 'g', 'h', 'i']
```
### slice([start[, end]])
将数组中一部分元素**浅复制**存入新的数组对象，并且返回这个数组对象

参数 start 指定复制开始位置的索引，end如果有值则表示复制结束位置的索引（不包括此位置）。

如果 start 的值为负数，假如数组长度为 length，则表示从 length+start 的位置开始复制，此时参数 end 如果有值，只能是比 start 大的负数，否则将返回空数组。

slice方法参数为空时，同concat方法一样，都是浅复制生成一个新数组。
```
var array = ["one", "two", "three","four", "five"];
console.log(array.slice()); // ["one", "two", "three","four", "five"]
console.log(array.slice(2,3)); // ["three"]
```
利用下 slice 方法第一个参数为负数时的特性，我们可以非常方便的拿到数组的最后一项元素
```
console.log([1,2,3].slice(-1));//[3]
```
slice 一样受益于鸭式辨型
```
var o = {0:{"color":"yellow"}, 1:2, 2:3, length:3};
var o2 = Array.prototype.slice.call(o,0,1);
console.log(o2); // [{color:"yellow"}] ,毫无违和感...
```

### join([separator = ‘,’])
将数组中的所有元素连接成一个字符串。

separator可选，缺省默认为逗号
```
var array = ['We', 'are', 'Chinese'];
console.log(array.join()); // "We,are,Chinese"
console.log(array.join('+')); // "We+are+Chinese"
console.log(array.join('')); // "WeareChinese"
```
join 一样受益于鸭式辨型
```
var o = {0:"We", 1:"are", 2:"Chinese", length:3};
console.log(Array.prototype.join.call(o,'+')); // "We+are+Chinese"
console.log(Array.prototype.join.call('abc')); // "a,b,c"
```
### toString()
返回数组的字符串形式，该字符串由数组中的每个元素的 toString() 返回值经调用 join() 方法连接（由逗号隔开）组成。
```
var array = ['Jan', 'Feb', 'Mar', 'Apr'];
var str = array.toString();
console.log(str); // Jan,Feb,Mar,Apr
```
当数组直接和字符串作连接操作时，将会自动调用其toString() 方法。
```
var str = ['Jan', 'Feb', 'Mar', 'Apr'] + ',May';
console.log(str); // "Jan,Feb,Mar,Apr,May"

// 鸭式辨型
var o = {
  0:'Jan', 
  1:'Feb', 
  2:'Mar', 
  length:3, 
  join:function(){
    return Array.prototype.join.call(this);
  }
};
console.log(Array.prototype.toString.call(o)); // "Jan,Feb,Mar"
```
### toLocaleString()
类似toString()的变型，该字符串由数组中的每个元素的 **toLocaleString()** 返回值经调用 join() 方法连接（由逗号隔开）组成。

数组中的元素将调用各自的 toLocaleString 方法：
- Object：Object.prototype.toLocaleString()
- Number：Number.prototype.toLocaleString()
- Date：Date.prototype.toLocaleString()
```
var array= [{name:'zz'}, 123, "abc", new Date()];
var str = array.toLocaleString();
console.log(str); // [object Object],123,abc,2016/1/5 下午1:06:23

// 鸭式辩型
var o = {
  0:123, 
  1:'abc', 
  2:new Date(), 
  length:3, 
  join:function(){
    return Array.prototype.join.call(this);
  }
};
console.log(Array.prototype.toLocaleString.call(o)); // 123,abc,2016/1/5 下午1:16:50
```
### indexOf(element, fromIndex=0)
使用严格相等（即使用 === 去匹配数组中的元素）查找元素在数组中第一次出现时的索引，如果没有，则返回-1。

element 为需要查找的元素。

fromIndex 为开始查找的位置，缺省默认为0。如果超出数组长度，则返回-1。如果为负值，假设数组长度为length，则从数组的第 length + fromIndex项开始往数组末尾查找，如果length + fromIndex < 0 则整个数组都会被查找。

```
var array = ['abc', 'def', 'ghi','123'];
console.log(array.indexOf('def')); // 1
console.log(array.indexOf('def',-1)); // -1 此时表示从最后一个元素往后查找,因此查找失败返回-1
console.log(array.indexOf('def',-4)); // 1 由于4大于数组长度,此时将查找整个数组,因此返回1
console.log(array.indexOf(123)); // -1, 由于是严格匹配,因此并不会匹配到字符串'123'

// 鸭式辩型
var o = {0:'abc', 1:'def', 2:'ghi', length:3};
console.log(Array.prototype.indexOf.call(o,'ghi',-4)); // 2
```
### lastIndexOf(element, fromIndex=length-1)
使用严格相等（即使用 === 去匹配数组中的元素）查找元素在数组中**最后**一次出现时的索引，如果没有，则返回-1

element 为需要查找的元素。

fromIndex 为开始查找的位置，缺省默认为数组长度length-1。如果超出数组长度，由于是逆向查找，则查找整个数组。如果为负值，则从数组的第 length + fromIndex项开始往数组开头查找，如果length + fromIndex < 0 则数组不会被查找。

### includes(element, fromIndex=0) // ES7
用来判断当前数组是否包含某个指定的值

element 为需要查找的元素。

fromIndex 表示从该索引位置开始查找 element，缺省为0，它是正向查找，即从索引处往数组末尾查找。

```
var array = [-0, 1, 2];
console.log(array.includes(+0)); // true
console.log(array.includes(1)); // true
console.log(array.includes(2,-4)); // true

// 鸭式辩型
var o = {0:'a', 1:'b', 2:'c', length:3};
var bool = Array.prototype.includes.call(o, 'a');
console.log(bool); // true
```
唯一区别：includes能够发现NaN，而indexOf不能。
```
var array = [NaN];
console.log(array.includes(NaN)); // true
console.log(arra.indexOf(NaN)>-1); // false
```

## 遍历方法（12个）
### forEach(callback(currentValue [, index [, array]])[, thisArg]);
callback 为数组中每个元素执行的函数，该函数接收三个参数：
- currentValue      ：数组中正在处理的当前元素。
- index 可选    ：数组中正在处理的当前元素的索引。
- array 可选    ：forEach() 方法正在操作的数组。

thisArg 可选
可选参数。当执行回调函数 callback 时，用作 this 的值。

返回undefined

无法直接退出循环，只能使用return 来达到for循环中continue的效果
```
var array = [1, 3, 5];
var obj = {name:'cc'};
var sReturn = array.forEach(function(value, index, array){
  array[index] = value * value;
  console.log(this.name); // cc被打印了三次
},obj);
console.log(array); // [1, 9, 25], 可见原数组改变了
console.log(sReturn); // undefined, 可见返回值为undefined


// 得益于鸭式辨型，虽然forEach不能直接遍历对象，但它可以通过call方式遍历类数组对象
var o = {0:1, 1:3, 2:5, length:3};
Array.prototype.forEach.call(o,function(value, index, obj){
  console.log(value,index,obj);
  obj[index] = value * value;
},o);
// 1 0 Object {0: 1, 1: 3, 2: 5, length: 3}
// 3 1 Object {0: 1, 1: 3, 2: 5, length: 3}
// 5 2 Object {0: 1, 1: 9, 2: 5, length: 3}
console.log(o); // Object {0: 1, 1: 9, 2: 25, length: 3}
```
### every(callback[, thisArg])
使用传入的函数测试所有元素，只要其中有一个函数返回值为 false，那么该方法的结果为 false；如果全部返回 true，那么该方法的结果才为 true。

callback 用来测试每个元素的函数，它可以接收三个参数：
- element： 用于测试的当前值。
- index可选：用于测试的当前值的索引。
- array可选：调用 every 的当前数组。
thisArg
执行 callback 时使用的 this 值。

```
function isBigEnough(element, index, array) {
  return element >= 10;
}
[12, 5, 8, 130, 44].every(isBigEnough);   // false
[12, 54, 18, 130, 44].every(isBigEnough); // true

// 鸭式辩型
var o = {0:10, 1:8, 2:25, length:3};
var bool = Array.prototype.every.call(o,function(value, index, obj){
  return value >= 8;
},o);
console.log(bool); // true
```

### some(callback(element[, index[, array]])[, thisArg])
数组中有至少一个元素通过回调函数的测试就会返回true；所有元素都没有通过回调函数的测试返回值才会为false。

callback 用来测试每个元素的函数，接受三个参数：
- element： 数组中正在处理的元素。
- index可选：数组中正在处理的元素的索引值。
- array可选：some()被调用的数组。
thisArg
执行 callback 时使用的 this 值。

### filter(callback(element[, index[, array]])[, thisArg])
使用传入的函数测试所有元素，并返回所有通过测试的元素组成的新数组。它就好比一个过滤器，筛掉不符合条件的元素。

鸭式辨型写法参考every方法
```
var array = [18, 9, 10, 35, 80];
var array2 = array.filter(function(value, index, array){
  return value > 20;
});
console.log(array2); // [35, 80]
```
### map(function callback(currentValue[, index[, array]])[, thisArg])
遍历数组，使用传入函数处理每个元素，并返回函数的返回值组成的新数组

参数介绍同 forEach 方法的参数介绍

```
var array = [1, 4, 9];
var roots = array.map(Math.sqrt);//map包裹方法名
// roots is now [1, 2, 3], array is still [1, 4, 9]
var array = [1, 4, 9];
var doubles = array.map(function(num) {//map包裹方法实体
  return num * 2;
});
// doubles is now [2, 8, 18]. array is still [1, 4, 9]

// 鸭式辩型
var elems = document.querySelectorAll('select option:checked');
var values = Array.prototype.map.call(elems, function(obj) {
  return obj.value;
});
```

### reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])
接收一个方法作为累加器，数组中的每个值(从左至右) 开始合并，最终为一个值

callback
执行数组中每个值 (如果没有提供 initialValue则第一个值除外)的函数，包含四个参数：

- accumulator ： 累计器累计回调的返回值; 它是上一次调用回调时返回的累积值，或initialValue（见于下方）。
- currentValue ： 数组中正在处理的元素。
- index 可选 ： 数组中正在处理的当前元素的索引。 如果提供了initialValue，则起始索引号为0，否则从索引1起始。
- array可选 ： 调用reduce()的数组

initialValue可选
作为第一次调用 callback函数时的第一个参数的值。 如果没有提供初始值，则将使用数组中的第一个元素。 在没有初始值的空数组上调用 reduce 将报错。

返回值
函数累计处理的结果

当 callback 第一次执行时：
- 如果 initialValue 在调用 reduce 时被提供，那么第一个 previousValue 将等于 initialValue，此时 item 等于数组中的第一个值；
- 如果 initialValue 未被提供，那么 previousVaule 等于数组中的第一个值，item 等于数组中的第二个值。此时如果数组为空，那么将抛出 TypeError。
- 如果数组仅有一个元素，并且没有提供 initialValue，或提供了 initialValue 但数组为空，那么fn不会被执行，数组的唯一值将被返回。
```
var array = [1, 2, 3, 4];
var s = array.reduce(function(previousValue, value, index, array){
  return previousValue * value;
},1);
console.log(s); // 24
// ES6写法更加简洁
array.reduce((p, v) => p * v); // 24
```

以上回调被调用4次，每次的参数和返回见下表：
callback |	previousValue	|currentValue	|index	|array |	return value
---|---|---|---|---|---
第1次 |	1|	1|	0|	[1,2,3,4] |	1
第2次 |	1|	2|	1|	[1,2,3,4] |	2
第3次 |	2|	3|	2|	[1,2,3,4] |	6
第4次 |	6|	4|	3|	[1,2,3,4] |	24
reduce 一样支持鸭式辨型，具体请参考every方法鸭式辨型写法。

#### 计算出现的次数（同样适用于字符）
```
const nums = [3, 5, 6, 82, 1, 4, 3, 5, 82];

const result = nums.reduce((tally, amt) => {
    tally[amt] ? tally[amt]++ : tally[amt] = 1;
    return tally;
}, {});

console.log(result);
//{ '1': 1, '3': 2, '4': 1, '5': 2, '6': 1, '82': 2 }
```

#### 获取一个数组并将其转换为显示某些条件的对象
假设我们有一个数组，我们希望基于一组条件创建一个对象。reduce 在这里非常适用！现在，我们希望从数组中任意一个数字项创建一个对象，并同时显示该数字的奇数和偶数版本。
```
const nums = [3, 5, 6, 82, 1, 4, 3, 5, 82];

// we're going to make an object from an even and odd
// version of each instance of a number
const result = nums.reduce((acc, item) => {
  acc[item] = {
    odd: item % 2 ? item : item - 1,
    even: item % 2 ? item + 1 : item
  }
  return acc;
}, {});

console.log(result);
// { '1': { odd: 1, even: 2 },
//   '3': { odd: 3, even: 4 },
//   '4': { odd: 3, even: 4 },
//   '5': { odd: 5, even: 6 },
//   '6': { odd: 5, even: 6 },
//   '82': { odd: 81, even: 82 } 
// }
```

### reduceRight(callback(accumulator, currentValue[, index[, array]])[, initialValue])
接收一个方法作为累加器，数组中的每个值（从右至左）开始合并，最终为一个值。除了与reduce执行方向相反外，其他完全与其一致，请参考上述 reduce 方法介绍


### entries() // ES6
返回一个新的Array Iterator对象（数组迭代器对象），该对象包含数组中每个索引的键/值对。
```
var array = ["a", "b", "c"];
var iterator = array.entries();
console.log(iterator.next().value); // [0, "a"]
console.log(iterator.next().value); // [1, "b"]
console.log(iterator.next().value); // [2, "c"]
console.log(iterator.next().value); // undefined, 迭代器处于数组末尾时, 再迭代就会返回undefined

// 鸭式辩型
var o = {0:"a", 1:"b", 2:"c", length:3};
var iterator = Array.prototype.entries.call(o);
console.log(iterator.next().value); // [0, "a"]
console.log(iterator.next().value); // [1, "b"]
console.log(iterator.next().value); // [2, "c"]
```

### find(callback[, thisArg]) 和 findIndex(callback[, thisArg])  // ES6
find() 方法基于ECMAScript 2015（ES6）规范，返回数组中第一个满足条件的元素（如果有的话）， 如果没有，则返回undefined。

findIndex() 方法也基于ECMAScript 2015（ES6）规范，它返回数组中第一个满足条件的元素的索引（如果有的话）否则返回-1。

语法与forEach等十分相似，其实不光语法，find（或findIndex）在参数及其使用注意事项上，均与forEach一致。其鸭式辨型写法也与forEach方法一致
```
var array = [1, 3, 5, 7, 8, 9, 10];
function f(value, index, array){
  return value%2==0; // 返回偶数
}
function f2(value, index, array){
  return value > 20; // 返回大于20的数
}
console.log(array.find(f)); // 8
console.log(array.find(f2)); // undefined
console.log(array.findIndex(f)); // 4
console.log(array.findIndex(f2)); // -1
```
### keys()  // ES6
返回一个包含数组中每个索引键的Array Iterator对象。
```
var array = ["abc", "xyz"];
var iterator = array.keys();
console.log(iterator.next()); // Object {value: 0, done: false}
console.log(iterator.next()); // Object {value: 1, done: false}
console.log(iterator.next()); // Object {value: undefined, done: false}
```
索引迭代器会包含那些没有对应元素的索引
```
var array = ["abc", , "xyz"];
var sparseKeys = Object.keys(array);
var denseKeys = [...array.keys()];
console.log(sparseKeys); // ["0", "2"]
console.log(denseKeys);  // [0, 1, 2]
```
生成一个从0到指定数字的新数组
```
[...Array(10).keys()]; // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[...new Array(10).keys()]; // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### values() // ES6
返回一个新的 Array Iterator 对象，该对象包含数组每个索引的值
```
var array = ["abc", "xyz"];
var iterator = array.values();
console.log(iterator.next().value);//abc
console.log(iterator.next().value);//xyz
```

### [Symbol.iterator]()
同 values 方法功能相同
```
var array = ["abc", "xyz"];
var iterator = array[Symbol.iterator]();
console.log(iterator.next().value); // abc
console.log(iterator.next().value); // xyz
```

### flat([depth]) // ECMAScript 2019
会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回

depth 可选：指定要提取嵌套数组的结构深度，默认值为 1。

```
var arr1 = [1, 2, [3, 4]];
arr1.flat(); 
// [1, 2, 3, 4]

var arr2 = [1, 2, [3, 4, [5, 6]]];
arr2.flat();
// [1, 2, 3, 4, [5, 6]]

var arr3 = [1, 2, [3, 4, [5, 6]]];
arr3.flat(2);
// [1, 2, 3, 4, 5, 6]

//使用 Infinity，可展开任意深度的嵌套数组
var arr4 = [1, 2, [3, 4, [5, 6, [7, 8, [9, 10]]]]];
arr4.flat(Infinity);
// [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// 会移除数组中的空项
var arr4 = [1, 2, , 4, 5];
arr4.flat();
// [1, 2, 4, 5]
```

### flatMap(function callback(currentValue[, index[, array]])[, thisArg]) // ECMAScript Latest Draft (ECMA-262)
首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。它与 map 连着深度值为1的 flat 几乎相同，但 flatMap 通常在合并成一种方法的效率稍微高一些

参数介绍同 forEach 方法的参数介绍
```
let arr1 = ["it's Sunny in", "", "California"];

arr1.map(x => x.split(" "));
// [["it's","Sunny","in"],[""],["California"]]

arr1.flatMap(x => x.split(" "));
// ["it's","Sunny","in", "", "California"]
```