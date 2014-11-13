title: 关于call 的面试题两则之我的分析
date: 2014-11-13 18:54:02
tags: javascript
categories: javascript
---
**第一题：**

	function f1(){
	        console.log('this is function f1!');
	}
	
	function f2(){
	         console.log('this is function f2!');
	}
	
	var f3 = f1.call;
	f1.call(f2);
	f3.call(f2);

**第二题：**

	function fn(a,b){
		 console.log(this);
		 this.a=a;
		 this.b=b;
		 console.log(this.a+":"+this.b);
	}
	fn.call.call(fn,8,7);
<!--more-->
前几天在前端群里面看见的两个面试题，想了几天，尼玛理解能力太差，直到昨天晚上洗澡的时候静下心来想了一下，终于想通了，又对call方法进一步了解了。

首先了解一下call方法，看看ECMAScript里面的解释：


15.3.4.4 Function.prototype.call (thisArg [ , arg1 [ , arg2, … ] ] )
When the call method is called on an object func with argument thisArg and optional arguments arg1, arg2
etc, the following steps are taken:

1. If IsCallable(func) is false, then throw a TypeError exception.
2. Let argList be an empty List.
3. If this method was called with more than one argument then in left to right order starting with arg1 append
each argument as the last element of argList
4. Return the result of calling the [[Call]] internal method of func, providing thisArg as the this value and
argList as the list of arguments.
The length property of the call method is 1.

NOTE The thisArg value is passed without modification as the this value. This is a change from Edition 3, where a
undefined or null thisArg is replaced with the global object and ToObject is applied to all other values and that result is
passed as the this value.

翻译出来就是：

15.3.4.4 Function.prototype.call (thisArg [ , arg1 [ , arg2, … ] ] )
当以 thisArg 和可选的 arg1, arg2 等等作为参数在一个 func 对象上调用 call 方法，采用如下步骤：


1.	如果 IsCallable(func) 是 false, 则抛出一个 TypeError 异常。
2.	令 argList 为一个空列表。
3.	如果调用这个方法的参数多余一个，则从 arg1 开始以从左到右的顺序将每个参数插入为 argList 的最后一个元素。
4.	提供 thisArg 作为 this 值并以 argList 作为参数列表，调用 func 的 [[Call]] 内部方法，返回结果。

call 方法的 length 属性是 1。

注：在外面传入的 thisArg 值会修改并成为 this 值。thisArg 是 undefined 或 null 时它会被替换成全局对象，所有其他值会被应用 ToObject 并将结果作为 this 值，这是第三版引入的更改。
call方法是属于Function.prototype的，在声明函数的时候，每个函数是Function的一个实例，自然每个函数就带上了call方法，当然call也是一个函数，call也是Function的一个实例，call.call这是理所当然的。

首先来看看第一题：

	function f1(){
	        console.log('this is function f1!');
	}
	
	function f2(){
	         console.log('this is function f2!');
	}
	
	var f3 = f1.call;
	f1.call(f2);
	f3.call(f2);

首先声明了f1、f2两个函数，即为Function的一个实例，自然可以call辣。
然后声明了一个变量f3，值为f1.call。
执行：f1.call(f2)；
这儿应该不难吧，根据call的解释，f1.call在执行的时候，this是指向的是f2的，记为f1.this -> f2，如果在f1中console.log(this)，打印的肯定是f2函数，f1.call执行，我的理解是f1会执行，只是在执行的时候this变成f2了，相当于f2调用f1，即f2.f1()，这儿自然打印的是 ：

	this is function f1!

再来看看f3.call(f2)，根据f1.call(f2)的解释，在f3执行的时候,f3的this指向的是f2，记为f3.this -> f2，相当于f2.f3()，此时调用f3的应该是f2，由于f3是等于f1.call的，那么调用f1.call的应该是f2，即f2.call();那么这儿自然就打印：

	this is function f2!

OK，第一题就到这儿。

再看看第二题，

	function fn(a,b){
	    console.log(this);
	    this.a=a;
	    this.b=b;
	    console.log(this.a+":"+this.b);
	}
	fn.call.call(fn,8,7);

声明了一个函数fn,参数有a和b，接下来执行fn.call.call(fn,8,7)，为了便于理解，这儿把fn.call看作一个整体，记为var fn1 = fn.call，
可见 fn1.call(fn,8,7)，看看第一题，fn1的this变成了fn，记为 fn1.this -> fn,则调用fn1函数的实际是fn.fn1，即fn.fn1()，根据call方法的官方解释，把参数带过来，即fn.fn1(8,7)，因为fn1等于fn.call，那么调用fn.call的应该是fn,即fn.call()，参数带过来，fn.call(8,7)，因为只传递了一个参数，那么fn中的a就等于7，b就是undefined，这儿自然打印的是：

	Number {[[PrimitiveValue]]: 8}
	7:undefined

因为，8是一个数字，所以console.log(this)，打印出的就是Number {[[PrimitiveValue]]: 8}，因为8.call(7);


看到这儿，我相信可以自己模拟一个call方法了，下边是我自己模拟call方法的代码：

	Function.prototype.mycall = function(thisObj){
        var args = [].slice.call(arguments,1);
        return this.apply(thisObj,args);
	}

改写代码：
**第一题**

	function f1(){
		console.log('this is function f1!');
	}

	function f2(){
		console.log('this is function f2!');
	}
   
	var f3 = f1.mycall;
	f1.mycall(f2);
	f3.mycall(f2);

打印结果：

	this is function f1! 
	this is function f2! 

**第二题**

	function fn(a,b){
	    console.log(this);
	    this.a=a;
	    this.b=b;
	    console.log(this.a+":"+this.b);
	}
	fn.mycall.mycall(fn,8,7);

打印结果：

	Number {[[PrimitiveValue]]: 8} 
	7:undefined 

OVER，以上仅是个人理解，不代表完全正确，有不对的地方还请指正。:)

