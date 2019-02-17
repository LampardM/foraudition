### audition纪要
***
#### 调用栈相关
* [宏任务和微任务](https://juejin.im/post/59e85eebf265da430d571f89)
* macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
* micro-task(微任务)：Promise，process.nextTick

```
    setTimeout(function(){
        console.log('1')
    });

    new Promise(function(resolve){
        console.log('2');
        for(var i = 0; i < 10000; i++){
            i == 99 && resolve();
        }
    }).then(function(){
        console.log('3')
    });

    console.log('4');

```
Promise是微任务，第一个函数的回调是立即执行的而then的回调是要放在微任务列表里的，process.nextTick的回调是微任务，需要放在微任务列表。setTtimeout是宏任务，回调注册到宏任务列表，也就是微任务回调2先执行，3放入微任务列表，执行4然后微任务列表里3开始执行，最后执行宏任务回调1。
注意：一个宏任务里包含多个微任务，一定是需要把此宏任务里的微任务全部执行完毕才能进入下一个宏任务的执行。
***
#### 基本类型和引用类型
* 基本类型：string number boolean unll undefined symbol。基本类型的数据存储在栈中，而引用类型的数据存储在堆中，栈中存储的是指针
* js中所有数字都是浮点数，所以在计算时会存在意想不到情况，例如：0.1 + 0.2 === 0.3是false
* number boolean 都有toString()方法，并返回一个字符串
* parseFloat('11.22.33') // 11.22 第二个小数点会被忽略

```
var a = new Array()
a instanceof Array // true
a instanceof Object // true
typeof null // object

```
函数传参为引用类型时则传递的是一个引用，传给函数的是数值的一个引用，函数中对其属性的修改外部可见，但用新引用覆盖其则在外部不可见
```
var a = new Object()

function set(obj) {
    obj.name = '1'
    obj = new Object() // 新引用覆盖对外部不可见
    obj.name = '2'
}
set(a)
console.log(a) // '1' //函数内部创建的对象会在函数执行完毕后销毁

```
这是更好的一道题：[引用传参](https://blog.fundebug.com/2017/08/09/explain_value_reference_in_js/)
```
function changeAgeAndReference(person) {
    person.age = 25;
    person = {
        name: 'John',
        age: 50
    };
    
    return person;
}
var personObj1 = {
    name: 'Alex',
    age: 30
};
var personObj2 = changeAgeAndReference(personObj1);
console.log(personObj1); // -> ?
console.log(personObj2); // -> ?
```
***
#### 运算符和类型判断以及类型转换
* 实现isNaN可利用其不等于自身来实现
```
function isNaN(n) {
    return n !== n
}
```
* 判断是null
```
function isNull(m) {
    // 不是false不是undefined也不是0
    return !m && typeof m != 'undefined' && m != 0
}
```
* == 和 != 比较若类型不同，先尝试转换类型，再作值比较，最后返回值比较结果，而 === 和 !== 只有在相同类型下，才会比较其值。
* 关于!!：!可以将变量转换为boolean类型，null undefined 0和空字符串都转换为true，其他都转换为false。此链接第一句话有误：[!!的转换](http://www.cnblogs.com/tison/p/8111712.html)
