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
## Set Map
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



