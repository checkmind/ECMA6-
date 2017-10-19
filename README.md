# ECMA6-
记录ECMA6学习过程
## 一 类数组转化为数组  
在之前，我们想类数组调用数组的方法，需要这么做：  
```
let div = document.querySelectorAll('div')
Array.prototype.map.call(div,function(val){
    console.log(val);
})
```
这样看来很繁琐，现在我们尝试用其他方法实现以上效果。  
###  Array.from  
这个方法可以将含有Iterator的对象转化为数组，进而可以使用数组的方法  
```
let div = document.querySelectorAll('div')  
Array.from(div);
div.map(function(val){
    console.log(val);
})
```
现在代码是不是更加简洁明了了。  
那么，怎么才能知道对象是否含有Iterator属性呢  
拥有Iterator属性的对象内置了 Symbol.iterator 方法，
```array[Symbol.iterator]()```
运行内置方法，如果有iterator属性则返回一个Iterator对象
## 二 数组去重
在ES5时代，我们数组去除也给予了很多方法，比如 indexOf 哈希表 for循环，  
在此主要列举一下哈希表去重。  
```
function removeRepeat(arr){
	var hash = {};
	var newArray = [];
	for(var i = 0,len=arr.length;i<len;i++){
		if(hash[arr[i]])
			break;
		else{
			hash[arr[i]] = true;
			newArray.push(arr[i]);
		}
	}
	return newArray;
}
```
哈希表去重明显提升了速度，但其结构复杂，可读性差。  
在ES6引入了 map 和 set 的数据结构。  
### Set 数据结构
其中 set 让我不禁想起了初中数学  
集合的定义：  
-互异性：即集合内的元素和其他元素各不相同  
-无序性：即集合内元素顺序与大小无关  
-确定性：集合内的每一个元素都是确定的  
这里的互异性和我们的数组去重方向一致。
```
var arr = [...new Set([2,3,3,4])]
```
就一句话，完成了我们 removeRepeat 函数的功能
首先我们将数组作为 Set 函数的参数传入，在Set进行互异性处理之后，利用...重新解构为数组  
即完成了数组去重功能。
Set 本身也内置了 iterator。所以我们也可用上一节的 Array.from 函数将 Set 转化为数组格式
```
function removeRepeat(arr){
  return Array.from( new Set(arr) );
}
```
## 三 遍历器
在 ES6 引入了遍历器的概念。
在对象里内置 Symbol.iterator 函数，用该函数，可以遍历对象成员。用法如下：
```
let array = [2,3,4]
arrayIterator = array[Symbol.iterator]()
arrayIterator.next();   //2
arrayIterator.next();   //3
arrayIterator.next();   //4
```
引入遍历器的原因之一就是它只会遍历具有数字下标的成员。现在回忆一下我们之前遍历对象的方式。
### for...in
**for...in除了枚举自身成员，还会枚举继承的成员属性**
比如如下代码：
```
let arrx = [2,3,4];
Array.prototype.name = 'duhao';
for(var i in arrx)
	console.log(arrx); // 2 3 4 'duhao'
```
像这个例子，‘duhao’字符串是继承来的属性，也被 for...in 枚举了。  
并且 for...in 还存在另一个问题。***如果数组同时拥有对象属性和数组元素，返回的属性名很可能是按照创建的顺序而非数值的大小排列***(Javascript权威指南)  
### map 和 forEach
ES5 数组的新api，也是我平时用的最多的方法之一。  
···
array.map(function(val){
	console.log(val) //数组的成员
})
array.forEach(function(val,index){
	console.log(val,index)  // 数组成员与成员下标
})
### 自己写的遍历器
首先看原声遍历器的用法，首先创建了一个对象的内置 Symbol.iterator 函数，该函数的返回值便是一个遍历器对象，  
依次调用该遍历器的 next 方法，即得到了从下标0开始升序的对象成员。  
代码如下：  
```
Array.prototype.iterator = function(){
	var index = -1,
	    that = this;
	return {
		next (){
			index++;
			return (index>=that.length)?undefined : that[index]
		}
	}
};		
arriter = arr.iterator();
console.log(arriter.next());
console.log(arriter.next());
console.log(arriter.next());
console.log(arriter.next());
```
至此，我们实现了数组的遍历器。我们可以用循环将遍历器指向我们想遍历的对象。
代码如下：
```
 var iterator = (function(){
 	var object = [Array,String];
	object.map(function(val){
		val.prototype.iterator = function(){
			var index = -1,
				that = this;
			return {
				next (){
					index++;
					return (index>=that.length)?undefined : that[index]
				}
			}
		};		
	})
 })();
```
## 四 Set Map
### Set
在之前我们也用过了Set这个新的数据格式，他的特点便和集合的特点一样（互异性）。  
主要的api有 
```
var set = new Set();
set.add(2); //增加一个对象
set.delete(2); //减少一个对象
set.has(2);  //检查是否拥有对象
```
也可在创建Set对象的时候传入一个数组来对其初始化。Set也内置了遍历器，所以可以使用  
for...of 来对其进行遍历。
### WeakSet
也有一个特殊的Set对象，WeakSet，它的内部只允许储存对象格式。其他api无异于 Set对象
```
var weakSet = new WeakSet();
weakSet.add(1);  //这是不被允许的。 Invalid value used in weak set
```
WeakSet内部的对象都是弱引用，也就是说随时会被垃圾回收机制回收。  
在研究 WeakSet 之前需要知道js的垃圾回收机制。    
**垃圾回收机制**主要有两种。标记清除和引用计数。    
**引用计数**：在对象被重新赋予新变量的时候，则标记数+1，在变量的值被修改为其他值时，则标记数-1。为0时，则被回收。代码如下：  
```
var obj = new Object();  //标记数+1
var obj2 = obj; //标记数+1
obj2 = 2;  //变量值修改为其他值了。标记数-1
obj = null; // 引用了对象的所有变量都为null时，对象直接被回收。
```
**标记清除**：在对象进入环境的时候，理论上讲不能被回收，当环境消失的时候，  
该对象会被回收。也就是说函数结束了，如果变量没有被外部引用，则会被清除。  
垃圾回收装置就是这样，问题是 **WeakSet** 不会被垃圾回收机制注意。即使 他引用了某个对象，该对象  
的标记位也不会+1。也就是说，如果该对象除了 WeakSet 引用之外的变量都被设置为null或者其他引用值时，该变量就会被清除。  
那么WeakSet内部的对象，随时都可能被清除。  
因此以上决定了WeakSet的另一个特点：不能被for...of遍历。因为很有可能在遍历结束后，内部对象被清除了。  
*WeakSet的这个特性适用于后面的WeakMap*
### Map
Map函数是用来弥补Object只能用字符串当键值的不足的。它可以用任意对象当键值，甚至是Element对象。  
```
let map = new Map();
let o = {
	name : 'duhao'
}
map.set(o,'hello');
console.log(map.has(o));
console.log(map.get(o));
```
以上用o对象作为map的键值。用set方法设置value值。has方法检查是否拥有该属性。get得到该属性值。  
还可以在创建Map的时候进行初始化  
```
let map = new Map([['names','lala'],['name','heiheihei']]);
console.log(map);  //Map(2) {"names" => "lala", "name" => "heiheihei"}
```
这里初始化的时候很容易让人迷惑，传入了一个二维数组，实际上内部实现是这样的。
```
let map = new Map();
const arr = [['names','lala'],['name','heiheihei']];
arr.forEach(function([key,value]){  
	map.set(key,index);
})
```
注意forEach的回调函数的参数是 ``[key,value]``,而不是 ``key,value``,   
这就意味着对数组的第一个元素进行了解构赋值。也意味着，如果二维数组的第二维数组有了第2位值，会被自动忽略。
### WeakMap
他和 WeakSet 的两个特性大体相同，这里一概而过  
- 键值必须是对象  
- 不能被遍历，因为不会被垃圾清除系统标记  


## 五 Proxy
今天在浏览别人博客的时候看见了一篇讲Proxy的文章。  
在之前看过曽探写的《javascript设计模式》，里面讲代理模式举了这么一个例子。  
大致是，小明喜欢女神，但不敢自己给女神送花，只好找了女神的闺蜜，把花交给闺蜜，  
然后让闺蜜将花交给女神。这个过程，闺蜜就是中间的代理，整个过程不变的是花，也就是我们所说的接口。  
首先代理的条件： 代理函数接口和被代理对象的接口一致。  
那么ES6引入这个Proxy有什么用呢。首先来看Proxy的用法：  
```
let target = {};
let handler = {
 	set() {
	    console.log('cannot set this');
	}
};
let proxy = new Proxy(target,handler);
proxy.send = 'flower'; // cannot set this
```
在这个例子里出现了两个对象，target和handler。target就是被代理对象，也就是故事里的女神，   
handler是代理处理方法，proxy就是闺蜜，当我们想给女神送花的时候，经过代理方法，被handler拦截请求，   
由handler处理我们的请求，决定是否将花送给女神，也就是是否设置该属性。  
在handler的set方法里提供了四个参数，分别是 target,key,value,receiver,target是目标对象，receiver是接受者，    
在这里即proxy。    
Proxy不止能代理对象，它还可以代理函数。  
```
var obj = {
	apply(target,binding,args){
		console.log("call me")
	},
	construct(target,args){
		console.log(args)
		return {value:args[0]}
	},
}
var obj2 = new Proxy(function(ar1,ar2){
	return ar1+ar2
},obj)

obj2(2,3)
var obj3 = new obj2(3,4)
console.log(obj3)
```
此时，当作为普通函数调用函数时，实际上调用了拦截器的apply方法，而此时的binding则指向全局。  
当作为构造器使用时，则调用了construct方法。
## 六 Reflect
____________________________________________________________________
## 七 Object
### Object.assign()   注意有坑
这个方法用来对指定对象的“拷贝”，因为js里面对象都是基于引用传值的。
```
let obj = {
    name : 'dh'
};
let obj2 = obj;
obj2.name = 'zqq';
console.log(obj2===obj1);  //true
```
上面这段代码就说明了，js的对象传值是基于引用的，新创建的obj2并没有开辟一个新的内存区  
而是将指针指向了{name:'dh'},(这里不是obj，因为obj也是指向了左边这个对象)。  
让我们用Object.assign() 重写一遍。  
```
let obj = {
    name : 'dh'
};
let obj2 = Object.assign({},obj);
obj2.name = 'zqq';
console.log(obj2===obj); // false
```
可以看到，改变了obj2的属性之后，obj的属性并没有受到影响。如果到这里，你就以为他是深拷贝，那就太天真了。  
我们再来看一个例子。  
```
var b = {
	name : 'b',
	num : 2,
	title : {
		name : 'bname'
	}
}
var c = Object.assign({},b);
c.title.name = 'c';
console.log(c,b) //可以看到两个结构一样
console.log(c===b); //但又不全等
```
到这里就很好奇了，b.title.name 确实由于 c.title.name的赋值改变到了，也就是说，在title这一层级，  
是浅克隆的，但是c又不全等于b，这是由于，在name和num层级是深克隆的，这就是assign的蛋疼之处，也就是说，  
他只支持对象的第一层深度克隆，其他层采用浅克隆。  
关于这个问题，我也去找个官方的解释。  
*针对深度拷贝，需要使用其他方法，因为 Object.assign() 拷贝的是属性值。假如源对象的属性值是一个指向对象的引用，它也只拷贝那个引用值。*   ---MDN
 `https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign`  
 也就是说，如果我们想实现真正的深度拷贝，还是需要之前的方法。
 ### 深度拷贝
 让我回忆一下之前深度拷贝的方法。  
 1，JSON.parse 和 JSON.stringify
 ```
 let obj = {
 	name : 'duhao',
	son : 'xiaochuizi'
 };
 let obj2 = JSON.parse(JSON.stringify(obj));
 ```
 这个方法实现很简单，但它首先将对象转为字符串，再将字符串转为对象，  
 因此效率很低。我们可以用第二种方法。
 2, construct 和 typeof
 ```
 let obj = {
 	name : 'duhao',
	son : 'xiaochuizi'
 };
 function deepClone(obj){
	var result;
	if(obj.constructor===Object){
		result = {};
	}
	else if(obj.constructor===Array){
		result = [];
	}
	else
		return result;
	for(var key in obj){
		if(typeof obj[key]==='object'){
			result[key] = deepClone(obj[key]);
		}else{
			result[key] = obj[key];
		}
	}
	return result;
}
deepClone(obj)
 ```
这个方法的原理是，如果是基本数据类型，则直接复制，因为js会开辟新的内存空间，但是  
如果是引用类型，Object和Array，那就再进入对象内部，直到找到基本数据类型，然后复制。  
如此回调。则完成了一层层克隆。这个方法的效率比上面的快不少。
```
default: 1706.0478515625ms
default: 340.9189453125ms
```
分别运行十万次的比较，差距还是很明显的！！！  

``所以，深度拷贝请务必用以上方式。Object.assign()不具备深度拷贝功能``
## Class  
其实这里的Class就是prototype的语法糖，它只是看上去更像面向对象里的类，如果以前学过typescript，则  
会发现，这像极了typescript（应该说typescript像极了它）。关于它的基本使用。  
```
class person{
	constructor (name,year){
		this.name = name;
		this.year = year;
	}
	callName (){
		var that = this;
		console.log(`${this.name} is ${this.year} old`);
	}
}
var me = new person('duhao',19);
me.callName();
```
constructor 就好比面向对象语言里面的构造函数，其实这里实际上指函数的构造类。  
打印类的construct属性，发现是： `ƒ Function() { [native code] }` ，也说明了Class的实现与Function实现无异。  




