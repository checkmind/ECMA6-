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
此时，Parent上面的方法，Child都可以用。但Parent内部的属性和方法是调用不到的。  
### 继承内部属性方法  
如果想继承Parent内部的属性和方法。则应该  
```javascript
	function Parent(age) {
		this.age = age;
		this.friends = [];
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
	Child.prototype =new Parent(2);  //继承原型上的方法和对象
	let people = new Child();
	people.age = 56;
	people.friends.push('duhao');
	let people2 = new Child();
	console.log(people.friends,people2.age); //['duhao'] 2
```
到这里发现,虽然子类实例可以获取到父类的属性，但是如果属性是数组或对象（引用传值）则不同实例共享该属性。所以这种继承还是有缺陷。
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
但这个方法同样存在问题，我们无法得到Parent原型链上的方法和属性。  
因此我们可以结合以上两种继承方式。  
### 组合继承
```javascript
	function Parent(age) {
		this.age = age;
		this.friends = [];
		this.sayAge=function()
		{
		console.log(this.age);
		}
	}
	Parent.prototype.sayParent = function() {
		alert("this is parentmethod!!!");
	}
	function Child(arg){
		Parent.call(this,arg);
	}
	Child.prototype =new Parent(2);  //继承原型上的方法和对象
	Child.prototype.constructor = Child; //把构造函数重新指向Child
```
这样子，基本上以上问题都被解决了。但是还有问题，Parent类被执行了两次，一次在Child伪造对象的时候，  
另一次在创建Parent实例的时候。并且，子类继承父类的属性继承了两次，一次在伪造对象的时候，另一次在原型继承的时候。  
因此这个方法效率比较低。所以就有了下面的解决方法。  
### 寄生组合继承  
先想想上面的组合继承为什么Parent类被执行两次之后，在什么业务下效率低？？  
应该是随着Parent代码量增大，每次构造的时候运行的时间增长，此时执行两次当然效率会低下。那么如果我们能减少调用的次数，不就可以提高效率。  
再考虑一个问题，当我们用伪装继承的时候，已经获取了一遍父类内部的属性和方法，然而在上面的组合继承中，我们又 new了一次父类，再一次的把父类内部的属性和方法赋给了子类实例。所以，这一步我们也做了两次。  
因此下面的方法解决了这几个问题。  
```javascript
	function Parent(arg){

	}
	function Child(arg){
		Parent.call(this,arg)
	}
	function extend(obj){
		function fn(){};
		fn.prototype = obj;
		return new fn();
	}
	function initProto(child,parent){
		let proto = extend(parent.prototype);
		child.prototype.constructor = child;
		child.prototype = proto;
	}
	initProto(Child,Parent);
```
在extend方法里，我们创建了一个空的构造函数，并将父类的原型赋值给它(这只让这个空类继承了原型上的方法和属性)  
然后，再让子类原型继承这个空构造函数。此时子类用有了父类原型上的方法，然后再通过伪造继承，子类继承父类的内部属性和方法。  
至此，继承完结撒花...



