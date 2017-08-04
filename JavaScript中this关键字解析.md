# JavaScript中this关键字解析 #

this关键字是javascript中最复杂的机制之一，它是一个很特别的关键字，被自动定义在所有的函数作用域中。

## 主线： ##

我们需要根据 "调用位置" 上函数的 "调用方式" 来确定函数中this使用的   "绑定规则"。

函数调用位置上的调用形式。
	
## 调用形式与绑定规则##


### 独立函数调用，默认绑定规则 ###			

独立函数调用。可以把这条规则看作是无法应用其他规则时的**默认规则**。如果使用了非严格模式，**this 会绑定到全局对象(window)**；如果使用严格模式（ strict mode ），**this 会绑定到undefine**。
		
这里有一个微妙但是非常重要的细节，虽然 this 的绑定规则完全取决于调用位置，但是只有 foo()运行在非 strict mode 下时，默认绑定才能绑定到全局对象；严格模式下调用foo()不会影响默认绑定规则。
	
无论函数是在哪个作用域中被调用,只要是独立调用则就会按默认绑定规则被绑定到全局对象或者undefined上。


**示例代码 **

	<script>
	//独立函数调用,如果使用了非严格模式,this 会绑定到全局对象（window）
		function foo() {
			console.log( this.a );
		}
		var a = 2;
		
		(function(){
			"use strict";
			foo(); // 2
		})();

	//独立函数调用,如果使用严格模式（ strict mode ），this 会绑定到 undefine
		function bar() {
			"use strict"
			console.log( this.a );
		}
		var a = 2;
		
		bar();// 报错Uncaught TypeError: Cannot read property 'a' of undefined
	</script>

				
### 隐式函数调用（对象·的形式），隐式绑定规则 ###

this的**隐式绑定规则**，**this绑定到离函数最近的对象**。

隐式绑定的规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。

对象属性引用链中只有最顶层或者说最后一层会影响调用位置。

**示例代码**

	<script>

		function foo() {
			console.log( this.a );
		}
		var obj = {
			a: 2,
			foo: foo
		};
		obj.foo();// 2


		//对象属性引用链中只有最顶层或者说最后一层会影响调用位置
		function foo() {
			console.log( this.a );
		}
		var obj2 = {
			a: 22,
			foo: foo
		};
		var obj1 = {
			a: 2,
			obj2: obj2
		};
		obj1.obj2.foo(); // 22
	</script>

**注意：**

隐式丢失的实现

- 将函数通过隐式调用的形式赋值给一个变量。

- 将函数通过隐式调用的形式进行传参。

- setTimeout函数传入隐式调用的形式。

一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用**默认绑定**，从而把** this 绑定到全局对象或者 undefined **上，取决于是否是严格模式。

参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值。

如果把函数传入语言内置的函数而不是传入你自己声明的函数，结果是一样的，没有区别JavaScript环境中内置的 setTimeout() 函数实现和下面的伪代码类似：

	function setTimeout(fn,delay) {
		// 等待delay毫秒
		fn(); // <-- 调用位置！
	}

**示例代码 **

	<script>
		//将函数通过隐式调用的形式赋值给一个变量。
		function foo() {
			console.log( this.a );
		}
		var a = "oops, global"; 
		var obj = {
			a: 2,
			foo: foo
		};
		
		var bar = obj.foo; 
		bar(); // oops, global

		//将函数通过隐式调用的形式进行传参。
		function foo() {
			console.log( this.a );
		}

		function doFoo(fn) {
			fn(); 
		}
		
		var a = "oops, global"; 

		var obj = {
			a: 2,
			foo: foo
		};
		
		doFoo( obj.foo ); // oops, global

		//setTimeout函数传入隐式调用的形式
		function foo() {
			console.log( this.a );
		}
		
		var a = "oops, global"; 
		var obj = {
			a: 2,
			foo: foo
		};
		
		setTimeout( obj.foo, 1000 );// oops, global
	</script>
						
### call、apply和bind函数调用，显式绑定规则 ###

this的**显式绑定规则** ，**this绑给callbin、apply和bind的指定对象**。

我们不想在对象内部包含函数引用，而想在某个对象上强制调用函数具体点说，可以使用函数的 call(..) 和 apply(..) 方法来实现显示绑定。

**示例代码 **

	<script>
		// call(..) 和 apply(..) 方法来实现显示绑定
		function foo(a,b) {
			console.log( this.a,a,b );
		}

		var obj = {
			a:2
		};

		foo.call( obj,"a","b"); //2 a b
		foo.apply(obj,["a","b"])//2 a b
	</script>

**硬绑定**

1、我们来看看这个显式绑定变种到底是怎样工作的。我们创建了函数 bar() ，并在它的内部手动调用了 foo.call(obj) ，因此强制把 foo 的 this 绑定到了 obj 。无论之后如何调用函数 bar ，它总会手动在 obj 上调用 foo 。这种绑定是一种显式的强制绑定，因此我们称之为硬绑定。

	<script>
		//硬绑定的bar不可能再修改它的this(指的是foo中的this)
		function foo() {
			console.log( this.a );/
		}
		
		var a =1;

		var obj = {
			a:2
		};

		var obj_test = {
			a:"test"
		};

		var bar = function() {
			console.log( this.a );
			foo.call( obj );
		};
		
		bar(); // 1 2

		setTimeout( bar, 1000 ); // 1 2

		bar.call( obj_test ); //test  2

	</script>

2、硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值。

	<script>
		function foo(arg1,arg2) {
			console.log( this.a,arg1,arg2);
			return this.a + arg1;
		}
		var obj = {
			a:2
		};
		var bar = function() {
			return foo.apply( obj, arguments);
		};
		
		
		var b = bar(3,2); // 2 3 2
		console.log( b ); // 5
	</script>

3、类bind函数的实现

	<script>
		function foo(something,otherthing) {
			console.log( this.a+" "+ something+" "+  otherthing);
			return this.a + something;
		}
		// 简单的辅助绑定函数    bind函数的作用：返回一个新的函数，并且指定该新函数的this指向
		function bind(fn, obj) {
			return function() {
					return fn.apply( obj, arguments );
				};
		}
		
		var obj = {
			a:2
		};
		var obj_test = {
			a:22
		};
		
		
		var bar = bind( foo, obj);
		var b = bar(3); // 2 3 undefined
		console.log( b ); // 5
		
		bar.call(obj_test,3);//2 3 undefined
	</script>

**硬绑定函数**

	<script>
		//对于上面这道题目，答案并不是太难，主要考点就是this指向的问题，
		//altwrite()函数改变了write的this的指向，让它指向global或window对象，
		//导致执行时提示非法调用异常
		document.write("test");
		var altwrite = document.write;
			
		//三种方法解决
		altwrite.bind(document)(" hello");
		altwrite.call(document, " call")
		altwrite.apply(document, [" apply"])
	</script>


### 函数的“构造调用”，new 绑定 ###

this的**new 绑定**，**this指定构造出来的实例对象**。

我们重新定义一下JavaScript中的“构造函数”。JavaScript，构造函数只是一些使用 new 操作符时被调用的函数。

它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被 new 操作符调用的普通函数而已。


!!!!!实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”。

使用 new 来调用函数，或者说发生构造函数调用时，对于我们的this来说。这个新对象会绑定到函数调用的 this 。

使用 new 来调用 foo(..) 时，我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this 上。 new 是最后一种可以影响函数调用时 this 绑定行为的方法，我们称之为 new 绑定。		

**示例代码 **

	<script>
		function foo(a) {
			this.a = a;
		}
		var bar = new foo(2);
		console.log( bar.a ); // 2
	</script>


### 绑定规则优先级 ###

new绑定 > 显式绑定 > 隐式绑定 

### 绑定例外 ###

**ES6中的箭头函数 **

	<script>
		function foo() {
		  setTimeout(() => {
		    console.log('id:', this.id); 
		  }, 100);
		}
		
		var id = 21;
		
		foo.call({ id: 42 })// id:42
	</script>	

**被忽略的this**

如果你把 null 或者 undefined 作为 this 的绑定对象传入 call 、 apply 或者 bind ，这些值在调用时会被忽略，实际应用的是默认绑定规则。

	<script>
		function foo() {
			console.log( this.a );
		}

		var a = 2;

		foo.call( null ); // 2
	</script>

### 柯里化 ###

柯里化:给目标函数预绑定一些参数。

	<script>
		function foo(a,b) {
			
			console.log( "a:" + a + ", b:" + b );
		}

		// 把数组“展开”成参数
		foo.apply( null, [2, 3] ); // a:2, b:3
		
		// 使用 bind(..) 进行柯里化
		var bar = foo.bind( null, [2] );
		bar( 3 ); // a:2, b:3
	</script>

不会污染全局柯里化。

	<script>
		function foo(a,b) {
			this
			console.log( "a:" + a + ", b:" + b );
		}

		// 我们的DMZ空对象,“DMZ”（demilitarized zone，非军事区）
		var ø = Object.create( null );//{}

		// 把数组展开成参数
		foo.apply( ø, [2, 3] ); // a:2, b:3

		// 使用bind(..)进行柯里化
		var bar = foo.bind( ø, 2 );
		bar( 3 ); // a:2, b:3
	</script>