## 闭包引发的类比

[TOC]

### 1. 什么是闭包？ 

大部分人是从JavaScript开始接触的，也就是`函数内部定义的函数，内部函数可以访问外部函数定义的局部变量`，其实也就是一个作用域确定的问题。

由于JavaScript实际是把function当作Object，所以引申到其他面向对象的语言比如java，就是内部类对于外部类的一个访问问题。



### 2. 正常闭包 

#### 2.1 JavaScript

```javascript
var x = 10;
(function () {
	var x = 100;
	(function() {
	  console.log(x);
	})();
})();
```

输出结果:

100

#### 2.2 dart

> 这篇文章实际是在学习dart的时候，发现dart跟JavaScript类似的定义，而展开的。

```dart
var x = 10;
((){
  var x = 100;
  ((){
    print(x);
  })();
})();
```

输出结果:

100

#### 2.3 kotlin

##### 2.3.1 函数

```kotlin
var x = 10
val fn: (() -> Unit) = {
    var x = 100
    val fm: (() -> Unit) = { println(x) }
    fm()
}
fn()
```

输出结果:

100

##### 2.3.2 面向对象

> 这里用了kotlin作为例子，就不再用java/dart/python举例了

```kotlin
class ClassA {
    var x = 10
    inner class ClassB {
        var x = 100
        fun test() {
            println(x)
        }
    }
}
ClassA().ClassB().test()
```

输出结果:

100

#### 2.4 python

```python
x = 10
def fn():
    x = 100
    def fm():
        print(x)
    fm()
fn()
```

输出结果:

100

### 3. "陷阱"闭包

> 所谓陷阱闭包(我取的名字，不要太在意)，可能是初学者看完闭包后容易犯的错误吧，P.S.:话说我不知道为啥会搞错
>
> 把函数或者对象作为参数传递（当然js/dart/kotlin里都把函数定义为对象），并非在内部定义

#### 3.1 JavaScript

```javascript
var x = 10;
(function (f) {
	var x = 100;
	f();
})(function() {
	console.log(x);
})();
```

输出结果:

10

#### 3.2 dart

```dart
var x = 10;
((dynamic f){
  var x = 100;
  f();
})((){
  print(x);
});
```

输出结果:

10

#### 3.3 kotlin

##### 3.3.1 函数

```kotlin
var x = 10
val fn: ((() -> Unit) -> Unit) = { var x = 100;it() }
fn({ println(x) })
```

输出结果：

10

##### 3.3.2 面向对象

```kotlin
var x = 10
abstract class ITest{
   abstract fun test()
}
class ClassTest(val f:ITest) {
    var x = 100
    init {
        f.test()
    }
}
ClassTest(object :ITest(){
    override fun test() {
        println(x)
    }
})
```

输出结果：

10

再看个例子

```kotlin
var x = 10
abstract class ITest{
    abstract fun test()
}
class ClassTest {
    var x = 100
    init {
        object :ITest(){
            override fun test() {
                println(x)
            }
        }.test()
    }
}
ClassTest()
```

输出是什么呢？^_^

#### 3.4 python

```python
x = 10
def test(f):
    x = 100
    f()
    
def fn():
    print(x)
    
test(fn)
```

输出结果：

10