- 本人前端小白一枚，整理这篇文章也是为了帮助自己学习。废话不多说，直接开始吧~
# 关于自学前端知识和遇到的面试题的整理。
## 原型对象和原型链
* **原型链的原理**
> **什么是原型链**？<br>
想要搞清楚`原型`链，首先需要搞清楚什么是`原型对象`。想要搞清楚什么是`原型对象`，首先要搞清楚什么是`普通对象`和`函数对象`。(小学语文体育老师教的，谅解~~)
<br><br>
**那么什么是普通对象和函数对象呢？** <br>
在JavaScript中万物皆对象。对象又分为普通对象和函数对象。函数对象顾名思义就是通过`new Function()`创建的对象就叫做函数对象。

```
function object(){alert(1)}
var object1 = function(){alert(1)};
var object2 = new function('demo','alert(1)');
```
> object、object1、object2实际上都是通过`new function()` 创建的`函数对象`。除了函数对象，其他的都叫做`普通对象`。
<br><br>
**什么是原型对象**<br>
每一个对象在创建的过程中都会有一些预定义的属性，其中`_proto_`就是其中一个预定义属性。而每一个函数对象中都会有一个`prototype`属性(只有函数对象中有`prototype`属性，普通对象中没有)。当我们通过`new`关键字去创建一个函数对象的实例的时候。这个函数对象实例的`_proto_`属性就会指向这个函数对象的`prototype`属性。而`object.prototype`就是原型对象。

```
function Person(){
    Person.prototype.name = 'Jack';
    Person.prototype.age  = 21;
    Person.prototype.say = function(){
        alert('人说话')
    }
}

//man为Person函数对象的一个实例所以man对象的_proto_属性就指向Person对象的prototype属性
var man = new Person(); 
```
> 所以每一个对象的`_proto_`属性都会指向其构造器的`prototype`属性。由于函数对象的构造器就是函数本身,所以函数对象的`_proto_`属性都是指向`Function.prototype`属性。每一个对象都会继承它的原型对象的所有方法和属性，而前面说到每一个对象都会有一个`_proto_`属性。所以每一个对象在调用方法的时候，首先会找他的原型对象（prototype）中的方法和属性，如果没有就找原型对象的`_proto_`属性，一直向上查询直到Object对象。

****
**总结** <br>
1、原型和原型链是JS实现继承的一种模型。<br>
2、原型链的形成是真正是靠`__proto__ ` 而非`prototype`


