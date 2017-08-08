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
`array[Symbol.iterator]()`  
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
