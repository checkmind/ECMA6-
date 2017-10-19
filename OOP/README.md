### 继承原型链条属性方法  
如果只是继承原型上面的方法或属性的话，用原型继承就可以了。  
```javascript
	function Parent(age) {
		this.age = age;
		this.sayAge=function()
		{
		console.log(this.age);
		}
	}
	Parent.prototype.sayParent = function() {
		alert("this is parentmethod!!!");
	}
	function Child(){

	}
	Child.prototype = Parent.prototype;  //继承原型上的方法和对象
```
### 继承内部属性方法
此时，Parent上面的方法，Child都可以用。但Parent内部的属性和方法是调用不到的。  
如果想继承Parent内部的属性和方法。则应该  
```javascript
	Child.prototype = new Parent(5);
	let people = new child();
	people.age = 5;
	let people2 = new child();
	console.log(people2.age) //5
```
到这里发现child不同实例是共享Parent的属性的。所以这种继承实际上没有什么用处。因为我们做出的修改  
都是在Parent的原型链上修改的。任意一个实例修改了原型链上的属性或者方法，其他实例势必会受到影响。  
并且，我们在new child的时候，没有办法对Parent进行传参，也就没办法对Parent内部属性进行初始化。所以  
这种继承方法是不可取的。  
### 构造函数继承(伪造对象)  
```javascript
	function child(arg){
		Parent.call(this,arg)
	}
```
这样子只需要在child构造函数里，用call改变Parent的调用对象，改为child的实例调用。  
我们就实现了，1，在new child的时候传参给Parent。2，child的实例不共享Parent的属性和方法。  
但这个方法同样存在问题，我们
```javascript
	let people2 = new child();
	console.log(people2 instanceof Parent)  //false
```
用这种方法创建出来的实例不属于Parent的实例。  


