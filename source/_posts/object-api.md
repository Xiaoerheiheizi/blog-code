---
title: JavaScript Object（对象）所有API解析
tags: JavaScript
---

转载自[若川的博客（原链接）](https://www.lxchuan12.cn/js-object-api/) 稍作修改，精简说明内容，以便随时查阅

<!--more-->

MDN上一些非标准API，本文没有列举。

常用API主要有：Object.prototype.toString(), Object.prototype.hasOwnProperty(prop), Object.getPrototypeOf(obj), Object.create(), Object.defineProperty, Object.keys(obj), Object.assign()

# Object 构造器成员
#### Object.prototype
该属性是所有对象的原型（包括Object对象本身），语言中其他对象正是通过该属性上添加东西，来实现它们之间的继承关系的。
```
var s = new String('若川');
Object.prototype.custom = 1;
console.log(s.custom); // 1
```
# Object.prototype 的成员
#### Object.prototype.constructor
该属性指向用来构造该函数对象的构造器
```
Object.prototype.constructor === Object; // true
var o = new Object();
o.constructor === Object; // true
```
#### Object.prototype.toString(radix)
该方法返回的是一个用于描述目标对象的字符串。当目标是一个Number对象时，可以传递一个用于进制数的参数radix，默认值为10
```
var o = {prop : 1};
o.toString(); // '[object Object]'
var n = new Number(255);
n.toString(); // '255'
n.toString(16); // 'ff'  16进制
```
#### Object.prototype.toLocaleString()
与toString()基本相同，不过它有做一些本地化处理。该方法会根据当前对象的不同而被重写，例如：Date(), Number(), Array() ，它们的值会以本地化形式输出。

#### Object.prototype.valueOf()
该方法返回的是用基本类型所表示的this值，如果它可以用基本类型表示的话。Number对象返回它的基本数值，Date对象返回的是一个时间戳（timestamp）。如果无法用基本数据类型表示，返回this本身
```
// Object
var o = {};
typeof o.valueOf(); // 'object'
o.valueOf() === o; // true

// Number
var n = new Number(101);
typeof n; // 'object'
typeof n.valueOf; 'function'
typeof n.valueOf(); 'number'
n.valueOf() === n; // false

// Date
var d = new Date();
typeof d.valueOf(); // 'number'
d.valueOf(); // 150...... (当前时间戳)
```

#### Object.prototype.hasOwnProperty(prop)
该方法仅在目标属性为对象自身属性是返回true，而当属性是从原型链中继承而来或根本不存在时，返回false
```
var o = {prop:1};
o.hasOwnProperty('prop'); // true
o.hasOwnProperty('toString'); // false
o.hasOwnProperty('formString'); // false
```
#### Object.prototype.isPrototypeOf(obj)
如果目标对象是当前对象的原型，该方法就会返回true，而且当前对象所在原型上的所有对象都能通过该测试，并不局限与它的直系关系。
```
var s = new String('');
Object.prototype.isPrototypeOf(s); // true
String.prototype.isPrototypeOf(s); // true
Array.prototype.isPrototypeOf(s); // false
```

#### Object.prototype.propertylsEnumerable(prop)
如果目标属性能在 for in 循环中被显示出来，该方法就返回true
```
var a = [1,2,3];
a.propertyIsEnumerable('length'); // false
a.propertyIsEnumerable(0); // true
```

# ES5中附加的Object属性
在ES3中，除了一些内置属性（如：Math.PI），对象的所有属性在任何时候都可以被修改、插入、删除。在ES5中，我们可以设置属性是否可以被改变或删除，在这之前，这是内置属性的特权。ES5中引入了属性描述符的概念，通过它对所定义的属性有更大的控制权。

属性描述符（特性）：
- value —— 当试图获取属性时所返回的值。 
- writable —— 该属性是否可写。 
- enumerable —— 该属性在for in循环中是否会被枚举 
- configurable —— 该属性是否可被删除。 
- set() —— 该属性的更新操作所调用的函数。 
- get() —— 获取属性值时所调用的函数

value的默认值为undefined以外，其他的默认值都为false

数据描述符其中属性：enumerable, configurable, value, writable 与存取描述符其中属性：enumerable, configurable, set(), get() 之间是有互斥关系。在定义了set()和get()后，描述符会认为存取操作已被定义，再定义value和writable会引起错误
```
// ES3风格的属性定义方式
var person = {};
person.legs = 2;

// 等价的 用ES5通过数据描述符定义属性的方式
var person = {};
Object.defineProperty(person, 'legs', {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
})

// 等价的 用ES5通过存储描述符定义属性的方式
var person = {};
Object.defineProperty(person, 'legs', {
    set: function(v){
        return this.value = v;
    },
    get： function(v){
        return this.value;
    }
    configurable: true,
    enumerable: true
})
person.legs = 2;
```
#### Object.defineProperty(obj, prop, descriptor)
```
var person = {};
Object.defineProperty(person, 'heads', {value: 1});
person.heads = 0; // 0
person.heads; // 1  (改不了)
delete person.heads; // false
person.heads // 1 (删不掉)
```
Vue.js文档： [如何追踪变化](https://cn.vuejs.org/v2/guide/reactivity.html)把一个普通 JavaScript 对象传给 Vue 实例的 data 选项，Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 把这些属性全部转为 getter/setter。Object.defineProperty 是仅 ES5 支持，且无法 shim 的特性，这也就是为什么 **Vue 不支持 IE8 以及更低版本浏览器**的原因。

#### Object.defineProperties(obj, props)
该方法作用于defineProperty()基本相同，只不过它可以用来一次定义多个属性。
```
var glass = Object.defineProperties({}, {
    'color' : {
        value: 'transparent',
        writable: true
    },
    'fullness': {
        value: 'half',
        writable: false
    }
});
glass.fullness; // 'half'
```

#### Object.getPrototypeOf(obj)
在ES3中，需要通过Object.prototype.isPrototypeOf()去猜测某个对象的原型是什么。如今可以直接询问对象“你的原型是什么？”
```
Object.getPrototypeOf([]) === Array.prototype; // true
Object.getPrototypeOf(Array.prototype) === Object.prototype; // true
Object.getPrototypeOf(Object.prototype) === null; true
```

#### Object.create(obj, descr)
该方法主要用于创建一个新对象，并为其设置原型
```
var parent = {hi: 'Hello'};
// 用属性描述符定义对象的原型属性
var o = Object.create(parent, {
    prop: {
        value: 1
    }
});
o.hi; // 'Hello'
// 获得它的原型
Object.getPrototypeOf(parent) === Object.prototype; // true  parent的原型是Object.prototype
Object.getPrototypeOf(o); // {hi: 'Hello'} 
o.hasOwnProperty('hi'); // false 说明hi是原型上的
o.hasOwnProperty('prop'); // true 说明prop是原型上的自身上的属性
```
创建一个完全空白的对象，这在ES3中无法实现
```
var o = Object.create(null);
typeof o.toString(); // 'undefined'
```

#### Object.getOwnPropertyDescriptor(obj, property)
详细查看一个属性的定义。甚至是那些内置的，之前不可见的隐藏属性
```
Object.getOwnPropertyDescriptor(Object.prototype, 'toString')
// {writable: true, enumerable: false, configurable: true, value: ƒ toString()}
```

#### Object.getOwnPropertyNames(obj)
该方法返回一个数组，其中包含了当前对象所有属性的名称（字符串），不论它们是否可枚举。也可以用Object.keys()来单独返回可枚举属性。
```
Object.getOwnPropertyName(Object.prototype);
// ["__defineGetter__", "__defineSetter__", "hasOwnProperty", "__lookupGetter__", "__lookupSetter__", "propertyIsEnumerable", "toString", "valueOf", "__proto__", "constructor", "toLocaleString", "isPrototypeOf"]
Object.keys(Object.prototype);
// []
Object.getOwnPropertyName(Object);
// ["length", "name", "arguments", "caller", "prototype", "assign", "getOwnPropertyDescriptor", "getOwnPropertyDescriptors", "getOwnPropertyNames", "getOwnPropertySymbols", "is", "preventExtensions", "seal", "create", "defineProperties", "defineProperty", "freeze", "getPrototypeOf", "setPrototypeOf", "isExtensible", "isFrozen", "isSealed", "keys", "entries", "values"]
Object.keys(Object);
```

#### Object.preventExtensions(obj)
该方法让有个对象变的不可扩展，也就是永远不能再添加新的属性。并返回原对象。
#### Object.isExtensible(obj)
该方法用于检查某对象是否还可以被添加属性。
```
var deadline = {};
Object.isExtensible(deadline); // true
deadline.date = 'yesterday'; // 'yesterday'
Object.preventExtensions(deadline);
Object.isExtensible(deadline); // false
deadline.date = 'today';
deadline.date; // 'today'  可以修改
// 尽管向某个不可扩展的对象中添加属性不算是一个错误操作，但它没有任何作用。
deadline.report = true;
deadline.report; // undefined
```

#### Object.seal(obj)
该方法让一个对象密封，不能添加新属性，且现有属性不可删除和配置（defineProperty()），可修改。返回被密封后的对象。
#### Object.isSeal(obj)
用于检查某对象是否被密封。
```
var person = {legs:2}；
person === Object.seal(person); // true
Object.isSealed(person); // true
Object.getOwnPropertyDescriptor(person, 'legs');
// {value: 2, writable: true, enumerable: true, configurable: false}
delete person.legs; // false (不可删除，不可配置)
Object.defineProperty(person, 'legs', {value:2});
person.legs; //2
person.legs = 1;
person.legs; // 1 (可修改)
Object.defineProperty(person, 'legs', {get: function() {return 'legs'; } });
```

#### Object.freeze(obj)
该方法冻结一个对象，该对象永远不可变，不能添加新属性，不能修改已有属性的值，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性。
#### Object.isFrozen(obj)
用于检查某对象是否被冻结
```
var deadline = Object.freeze({date: 'yesterday'});
deadline.date = 'tomorrow';
deadline.excuse = 'lame';
deadline.date; // 'yesterday'
deadline.excuse; // undefined
Object.isSealed(deadline); // true
Object.isFrozen(deadline); // true
Object.getOwnPropertyDescriptor(deadline, 'date');
// {value: "yesterday", writable: false, enumerable: true, configurable: false} (不可配置，不可写)
Object.keys(deadline); // ['date'] (可枚举)
```

#### Object.keys(obj)
该方法是种特殊的for-in循环，只返回当前对象的属性，而这些属性也必须是可枚举的。返回值是一个字符串数组
```
Object.prototype.customProto = 101;
Object.getOwnPropertyNames(Object.prototype);
// [..., "constructor", "toLocaleString", "isPrototypeOf", "customProto"]
Object.keys(Object.prototype); // ['customProto']
var o = {own: 202};
o.customProto; // 101
Object.keys(o); // ['own']
```

# ES6中附加的Object属性
#### Object.is(value1, value2)
该方法用来比较两个值是否严格相等。与严格比较运算符（===）行为基本一致。不同之处只有两个：+0不等于-0，NaN等于自身
```
Object.is('1', '1'); // true
Object.is({},{}); // false
Object.is(+0,-0); // false
+0 === -0; // true
Object.is(NaN, NaN); // true
NaN === NaN; // false
```
ES5通过以下代码部署Object.is
```
Object.defineProperty(Object, 'is', {
    value: function(x, y) {
        if(x === y) {
            // 针对+0不等于-0的情况
            return x !== 0 || 1 / x === 1 / y;
        }
        // 针对 NaN的情况
        return x !== x && y !== y;
    },
    configurable: true,
    enumerable: false,
    writable: true
})
```
#### Object.assign(target, ...sources)
该方法将来源对象（sources）所有可枚举的属性复制到目标对象（target），至少需要两个对象，一个参数会报错
```
var target = {a: 1};
var source1 = {b: 2};
var source2 = {c: 3};
obj = Object.assign(target, source1, source2);
target; // {a: 1, b: 2, c: 3}
obj; // {a: 1, b: 2, c: 3}
target === obj; // true
// 如果目标对象与原对象有同名属性，或多个源对象有同名属性，则后面属性会覆盖前面的属性。
var soutce3 = {a: 2, b: 3, c: 4};
Object.assign(target, source3);
target; // {a: 2, b:3, c: 4}
```
Object.assign 只复制自身属性，不可枚举的属性（enumerable 为false）和继承的属性不会被复制。
```
Object.assign({b: 'c'}, 
    Object.defineProperty({}, 'invisible', {
        enumerable: false,
        value: 'hello',
    })
)
// {b: 'c'}
```
属性名为Symbol值的属性，也会被Object.assign()复制
```
Object.assign({a: 'b'}， {[Symbol('c')]: 'd'});
// {a: 'b', Symbol(c): 'd'}
```
嵌套的对象，Object.assign()的处理方式是替换，而不是添加。
```
Object.assign({a: {b: 'c', d: 'e'}}, {a: {b: 'hello'}});
// {a: {b: 'hello'}}
```
对于数组，Object.assign()把数组视为属性名为0、1、2的对象
```
Object.assign([1,2,3],[4,5]);
// [4,5,3]
```
#### Object.getOwnPropertySymbols(obj)
该方法返回一个数组，包含了指定对象自身的（非传承的）所有symbol属性键
```
Object.getOwnPropertySymbols({a: 'b', [Symbol('c')]: 'd'});
// [Symbol(c)]
```
#### Object.setPrototypeOf(obj, prototype)
该方法设置一个指定的对象的原型（内部[Prototype]属性）到另一个对象或null。
```
var dict = Object.setPrototypeOf({}, null)
```
__proto__属性用来读取或设置当前对象的prototype对象。目前，所有浏览器（包括IE11）都部署了这个属性。该属性没有写入ES6的正文，而是写入了附录，__proto__前后双下划线说明它本质上是一个内部属性，而不是正式对外的一个API。无论从语义的角度，还是从兼容性的角度，都不要使用这个属性。而是使用Object.setPrototypeOf()（写操作），Object.getPrototypeOf()（读操作），或Object.create()（生成操作）代替。 在实现上，__proto__调用的Object.prototype.__proto__。 Object.setPrototypeOf()方法的作用与__proto__作用相同，用于设置一个对象的prototype对象。它是ES6正式推荐的设置原型对象的方法。

# ES8中附加的Object属性
#### Object.getOwnPropertyDescriptors(obj)
该方法基本与Object.getOwnPropertyDescriptor(obj, property)用法一致，不过它可以用来获取一个对象的所有自身属性的描述符。
```
Object.getOwnPropertyDescriptor(Object.prototype, 'toString');
// {writable: true, enumerable: false, configurable: true, value: ƒ toString()}
Object.getOwnPropertyDescriptors(Object.prototype); // 自行在浏览器中查看结果
```

#### Object.values(obj)
与Object.keys类似。返回一个给定对象的所有可枚举属性值的数组，值的顺序与使用for...in 循环的顺序相同（区别在于for-in循环枚举原型链中的属性）。
```
var obj = {a: 1, b: 2, c: 3};
Object.keys(obj); // ['a', 'b', 'c']
Object.values(obj); // [1,2,3]
```

#### Object.entries(obj)
返回一个给定对象的可枚举属性[key, value]的数组，数组中键值对的排列顺序和使用for...in循环遍历该对象时返回的循序一致（区别在于一个for...in循环也枚举原型链中的属性）。
```
var obj = {a: 1, b: 2, c: 3};
Object.keys(obj); // ['a', 'b',  'c']
Object.values(obj); // [1, 2, 3]
Object.entries(obj); // [['a', 1], ['b', 2], ['c', 3]]
```

# ES10中附加的Object属性
#### Object.fromEntries(iterable)
返回一个给定可迭代对象（类似Array、Map或其他可迭代对象）对应属性的新对象。是Object.entries()的逆操作。
```
var arr = [['a', 1], ['b', 2], ['c', 3]];
Object.fromEntries(obj); // {a: 1, b: 2, c: 3}
var entries = new Map([
    ['name', '若川'],
    ['age', 18],
]);
Object.fromEntries(entries); // {name: '若川', age: 18}
```