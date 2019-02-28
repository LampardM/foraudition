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
    下面是更复杂的的一道题：
    ```
    console.log('1');

    setTimeout(function() {
        console.log('2');
        process.nextTick(function() {
            console.log('3');
        })
        new Promise(function(resolve) {
            console.log('4');
            resolve();
        }).then(function() {
            console.log('5')
        })
    })
    process.nextTick(function() {
        console.log('6');
    })
    new Promise(function(resolve) {
        console.log('7');
        resolve();
    }).then(function() {
        console.log('8')
    })

    setTimeout(function() {
        console.log('9');
        process.nextTick(function() {
            console.log('10');
        })
        new Promise(function(resolve) {
            console.log('11');
            resolve();
        }).then(function() {
            console.log('12')
        })
    })
    ```
* 加入async和await后的执行顺序[async和执行顺序相关](https://mp.weixin.qq.com/s/sPpdVb5VerfFV960s--_PQ)

***
#### 基本类型和引用类型
* 基本类型：string number boolean unll undefined symbol。基本类型的数据存储在栈中，而引用类型的数据存储在堆中，栈中存储的是指针
* js中所有数字都是浮点数，所以在计算时会存在意想不到情况，例如：0.1 + 0.2 === 0.3是false
* number boolean 都有toString()方法，并返回一个字符串
* parseFloat('11.22.33') // 11.22 第二个小数点会被忽略
* 数组实例的instanceof

    ```
    var a = new Array()
    a instanceof Array // true
    a instanceof Object // true
    typeof null // object

    ```
* 函数传参为引用类型时则传递的是一个引用，传给函数的是数值的一个引用，函数中对其属性的修改外部可见，但用新引用覆盖其则在外部不可见
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
* 关于NaN需要注意的是：全局函数isNaN的作用是在检查一个值能否被Number成功转换，能就返回false，否则返回true，所以isNaN('abc') isNaN(undefined) isNaN({})都是true，尽管传入的值并不是NaN。实现isNaN可利用其不等于自身来实现，或者可以使用Number.isNaN()
    ```
    function isNaN(n) {
        return n !== n
    }
    ```
* 判断是null
    ```
    function isNull(m) {
        // 不是false不是undefined也不是0也不是NaN
        return !m && typeof m != 'undefined' && m != 0 && m == m
    }
    ```
* == 和 != 比较若类型不同，先尝试转换类型，再作值比较，最后返回值比较结果，而 === 和 !== 只有在相同类型下，才会比较其值。
* 关于!!：!可以将变量转换为boolean类型，null NaN undefined 0和空字符串都转换为true，其他都转换为false。此链接第一句话有误：[!!的转换](http://www.cnblogs.com/tison/p/8111712.html)
* {}转换成数字是NaN而""转成0，undefined转换为数字时会被转换成NaN [详细类型转换表](https://juejin.im/post/59ad2585f265da246a20e026)
* +很神奇的地方：1 + 2 + '3' // '33'而不是'123'
* 判断条件的转换：if || && 会把0 -0 undefined null "" NaN转换为false
* point函数会把x，y为0的情况忽略，所以还是用!!较为保险。
    ```
    function point(x, y) {
        if (!x) {
            x = 320;
        }
        if (!y) {
            y = 240;
        }
        return { x: x, y: y };
    }
    ```
* 关于toString()，数组的toString方法经过重新定义，所以[1]会被转换成"1"，而Object.prototype.toString.call([1])的结果是"[object Array]"
* 关于JSON.stringify()的一个应用：
    ```
    var a = {
        b: 42,
        c: "42",
        d: [1, 2, 3]
    };

    JSON.stringify(a, ["b", "c"]); // "{"b":42,"c":"42"}"
    JSON.stringify(a, function (k, v) {
        if (k !== "c") return v;
    });
    // 返回key不为c的值
    ```
* 神奇的+
  ```
  +'3.14' // 3.14
  +new Date() // 会直接得到时间戳
  ```
* 神奇的～
  ```
  var a = 'abc'
  ~a.indexOf('a') // true
  ~a.indexOf('d') // false
  // ~只有在indexOf返回为-1时将其转换为假值0，其余一律转换为真值
  ```
* 神奇的转换
  ```
  [] == ![] // true
  [1] == [1] //false
  [1] == 1 // true
  ['1'] == 1 // true
  "" == {} // false
  0 == {} // false
  null == false // false 但是!null == true是true
  null == true // false
  null == undefined // 设计缺陷
  ```
* 关于ToPrimitive：此方法将值转换为原始类型的值，所谓原始类型就是Null undefined number boolean string。其他都是非原始的值，如Array Function Object。
  转换原则：
  ToPrimitive(input,hint)转换为原始类型的方法，根据hint目标类型进行转换。
  hint只有两个值：String和Number
  如果没有传hint，Date类型的input的hint默认为String，其他类型的input的hint默认为Number
  Number 类型先判断 valueOf()方法的返回值，如果不是原始值，再判断 toString()方法的返回值
  String 类型先判断 toString()方法的返回值，如果不是原始值，再判断 valueOf()方法的返回值
  ```
  [10].valueOf() // [10] 还不是原始类型的值
  [10].toString() // "10" 是原始类型的值，故[10] == 10 true
  ```
***
#### typeof和instanceof
* 什么是instanceof：用来判断某个构造函数(a)的prototype是否在要判断对象(b)的原型链上
  ```
  var a = function() {}
  var b = new a()
  b instanceof a // true
  Object.getPrototypeOf(b) === a.prototype 

  typeof NaN // 'number'
  ```
* Function instanceof Object == true // true Object instanceof Funtion // true ?
  ```
  Function.__proto__.__proto__ === Object.prototype
  Object.__proto__ === Function.prototype
  Object.constructor === Function
  ```
  [彻底搞懂Function和Object的关系](https://juejin.im/post/58358606570c35005e4142bd)

***
#### 深拷贝和浅拷贝
* [基本实现](https://www.cnblogs.com/Chen-XiaoJun/p/6217373.html)
  ```
  function deepClone(initObj, finalObj) {
      obj = finalObj || {}
      for(var i in initObj) {
          var pro = initObj[i]
          if(pro === obj) {
              continue // 避免相互引用
          }

          if(typeof initObj[i] === 'object'){
              obj[i] = Object.prototype.toString.call(initObj[i] === '[object array]' ? [] : {})
              deepClone(initObj[i, ]obj[i])
          } else {
              obj[i] = initObj[i]
          }
      }

      return obj
  }
  ```

  浅拷贝就更简单了
  ```
  function lowClone(initObj, finalObj) {
      var obj = finalObj || {}
      for(var i in initObj) {
          obj[i] = initObj[i]
      }

      return obj
  }
  ```
* 实现Object.assign
  ```
  Object.prototype.meAssign = function(target) {
      for(var i = 1;i < arguments.length;i++){
          var source = arguments[i]

          for(var key in source) {
              if(Object.prototype.hasOwnProperty.call(source, key)) {
                  target[key] = source[key]
              }
          }
      }

      return target
  }
  ```
***
#### 关于jQuery的扩展
* [插件的两种形式](https://www.cnblogs.com/shy1766IT/p/5762707.html)
***
#### 什么是原型链
* 这就是原型链
  ```
  每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。那么，假如我们让原型对象等于另一个类型的实例，结果会怎么样呢?显然，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数 的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条，这就是原型链。

  a.__proto__.__proto__ .....

  ```
* 原型链的图解：[图解原型链](https://www.cnblogs.com/libin-1/p/5820550.html)
***
#### 对象，继承以及各种继承的优缺点(重头戏)
* Object.getOwnPropertyDescriptor()和Object.getOwnPropertyDescriptors()的区别
  ```
  var obj = {
      name: 'zlong',
      sex: 'male'
  }

  Object.getOwnPropertyDescriptor(obj, 'name') // 返回这个对象name属性的数据属性
  Object.getOwnPropertyDescriptors(obj) // {name:{...}, sex: {...}} 返回一个对象包含这个对象所有属性的数据属性
  相同点就是获取的都是对象实例属性，而不是对象原型属性
  ```
* 访问器属性和数据属性
  数据属性：Configurable Enumerable Writable Value
  访问器属性：Configurable Enumerable get set
* Object.defineProperty()和Object.defineProperties()的区别
  ```
  var a = { 
      name: 'zlong',
      sex; 'male'
    }
  
  Object.defineProperty(a, 'name', {...})
  Object.defineProperties(a, {
      name: {...},
      sex: {...}
  })
  ```
* 先来说说对象
  ```
  // 字面量的方式
  var obj = {
      name: 'zl'
  }

  // 工厂模式，显示的创建对象
  function createObj() {
    var o = new Object()
    0.name = 'zl'

    return o
  }

  var obj = createObj()

  // 构造函数模式(优点在于每个实例通过instaneof都可以确定其类型，也就是其构造函数是什么)
  function Person() {
      this.name = 'zl'
  }

  var obj1 = new Person()
  var obj2 = new Person() // 当然确定也很明显，就是每创建一个实例，构造函数的属性和方法都要创建一遍

  // 原型模式
  function Person() {}

  Person.prototype.name = 'zl'
  Person.prototype.gifts = ['fe']

  var obj1 = new Person()
  var obj2 = new Person()

  obj1.gifts.push('js')
  obj2.gifts // ['fe', 'js']
  // 原型模式的好处在于不用重复创建属性，但是缺点在于引用类型的属性被所有实例共享，这是原型模式最大的缺点，替代的方案就是组合式，也就是原型和构造函数相结合的方式，构造函数负责实例的属性，而原型负责共享的属性和方法

  function Person() {
      this.name = 'zl'
      this.gifts = ['fe']
  }

  Person.prototype.sex = 'male'
  Person.prototype.sayHi = function() {
      console.log('hi')
  }

  * 很重要的一点是理解什么是原型对象：无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个constructor(构造函数)属性，这个属性包含一个指向prototype属性所在函数的指针。就拿前面的例子来说，Person.prototype.constructor指向Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

  * 原型对象上的属性不会被实例重写，最多只能是被屏蔽

  * hasOwnProperty可以知道实例属性到底是实例属性还是原型属性
  function Person() {
      this.name = 'zl'
  }

  var pone = new Person()
  var ptwo = new Person()

  pone.name = 'zlong'
  pone.hasOwnProperty('name') // true
  ptwo.hasOwnProperty('name') // false

  * 还有一点要注意
  pone.constructor === Person.prototype.constructor // true 其实就是实例的构造函数当然等于构造函数本身

  // create形式
  Object.create(obj)

  function meCerate(obj) {
      // 实现create
      function f() {}
      f.prototype = obj

      return new f()
  }
  ```
* in操作符和hasPrototypeProperty
  ```
  function Person() {
      this.name = 'zl'
  }
  var p1 = new Person()

  p1.hasPrototypeProperty('name') // false
  'name' in Person // true
  
  // 所以实现自己的hasPrototypeProperty：
  function meHasPrototypeProperty(name) {
      return !(name in object) && !object.hasOwnProperty(name)
  }

  还有一点需要注意，操作符in会获取所有实例和原型对象的属性，而Object.keys()获取的是实例属性
  function Person() {}

  Person.prototype.name = 'zl'

  var p1 = new Person()
  p1.sex = 'male'

  Object.keys(Person.prototype) // name
  Object.keys(p1) // sex

  // Object.getOwnPropertyNames可以获取实例所有属性，不管是否可以枚举

  ```

* 为什么我们习惯原型对象一个一个添加属性，而不是全部赋值给一个新的对象
  ```
  function Person() {}
  Person.prototype.name = 'zl'
  Person.prototype.sex = 'male'

  而不是：
  Person.prototype = {
      name: 'zl',
      sex: 'male'
  }
  其实这样也可以，但是是在对原型对象的constructor属性要求不严格的情况下，因为此时Person的原型对象的constructor属性已经不再是构造函数的constructor了，而是变成了新的对象的constructor，也就是Object，即便通过强制设置Person.prototype的constructor为Person，但是会导致constructor变成可枚举，可通过Object.defineProperty(Person.prototype, 'constructor', { enumerable: false })再将其设置为不可枚举，这样就太麻烦了，得不偿失。
  ```

* 关于重写原型对实例的影响
  ```
  function Person() {
      this.saiHi = function() {
          console.log('hi')
      }
  }

  var p1 = new Person()

  Person.prototype = {
      constructor: Person,
      sayHi: function() {
          console.log('hi')
      }
  }

  p1.sayHi() // error 重写原型切断了已经创建实例和原型对象之间的关系
  ```
* 再来说说继承
  继承之后子实例的constructor:
  ```
  function animal() {
      this.name = 'animal'
  }

  function dog() {
      this.name = 'dog'
  }

  dog.prototype = new animal()

  var tedy = new dog()

  tedy.costructor == animal.prototype.constructor // 而不是dog，因为重写了原型对象

  * 所有函数的默认原型都是Object，在给子类新增方法的时候一定要写在原型链继承之后
  ```
* 原型链继承
  ```
  function Person() {
      this.name = ['zl']
  }

  function me() {
      this.sex = 'male'
  }

  me.prototype = new Person()

  var m1 = new me()
  var m2 = new me()

  m1.name.push('js')
  m2.name // ['zl', 'js'] 这是因为现在me的所有实例的prototype都指向Person的一个实例的prototype，所有属性都是共享的，另外一个缺点就是在创建子类实例的时候无法向超类的构造函数传递参数，其实可以传参，但是会影响所有实例，所以一般不能传参。
  ```
* 构造函数的继承
  ```
  首先注意一点
  function animal() {
      this.name = 'animal'
  }

  animal.prototype.gifts = ['eat']

  function dog() {
      animal.call(this) // 此处只是把构造函数的属性赋值给dog，而不是原型上的属性
  }

  var tedy = new dog()
  tedy.gifts // undefined

  * 借用构造函数可以把超类构造函数的属性全部赋值给子类的构造函数，这样在子类的所有实例里，引用类型的属性不会相互影响。还有个优点就是可以子类的构造函数可以向超类构造函数传参

  * 那么缺点也很明显，超类的方法都在构造函数中定义，每创建一个实例都要创建一遍方法，无法实现共享
  ```

* 组合继承
  ```
  function animal(){
      this.name = 'animal'
      this.gifts = ['eat']
  }

  animal.prototype.sayHi = function(){
      console.log('hi')
  }

  function dog() {
      animal.call(this) // 实现子类的实例属性不互相影响
  }

  dog.prototype = new animal() // 实现子类实例的方法共享
  ```

* 原型式继承
  ```
  function animal() {
      this.name = 'animal'
      this.sex = ['male']
  }

  animal.prototype.gifts = ['eat']

  function extend(obj) {
      function f(){}
      f.prototype = obj

      return new f()
  }

  var dog1 = extend(animal)
  var dog2 = extend(animal)

  dog1.sex.push('fmale')
  dog2.sex // ['male', 'fmale']

  dog.name // animal
  dog.gifts // undefined 原型式的继承也只是拿到超类构造函数的属性，而不是超类原型对象上的属性，拿到的这些属性将是返回的实例的共享属性，因为在extend函数内都将超类构造函数的属性全部赋值给了f的prototype，也就是变成了f的原型属性。接下来将被所有f的实例共享。

  ES5实现了这个方法，也就是Object.create()
  ```
* 寄生继承
  ```
  function parasitic(obj) {
      var clone = Object.create(obj)

      clone.saiHi = function() {
          console.log('hi')
      }

      return clone
  }
  ```

* 寄生组合继承
  ```
  首先明白为什么要用寄生组合继承，因为组合继承会调用两次超类的构造函数

  function animal() {
      this.name = 'animal'
  }

  function dog() {
      animal.call(this) // 第一次
      this.name = 'dog'
  }

  dog.prototype = new animal() // 第二次

  其实在第二次，无非子类需要的就是超类的原型副本而已，那么解决方案就是：
  function extendPrototype(child, super) {
      prototype = Object.create(super.prototype)
      prototype.constructor = child
      child.prototype = prototype
  }

  所以寄生组合继承就是：
  function animal() {
      this.name = 'animal'
  }

  function dog() {
      animal.call(this)
      this.dog = 'dog'
  }

  extendPrototype(child, animal)

  ```
  
* 继承优缺点：[继承的优缺点](http://www.cnblogs.com/lanyueff/p/7792009.html)
***
#### New到底做了什么
* [New的实现步骤](http://www.cnblogs.com/faith3/p/6209741.html)
  ```
  var a = function(){}
  var b = new a()

  1 创建了一个新对象
  2 将构造函数的作用域赋给这个新对象，this指向这个新对象
  3 执行构造函数中的代码，为这个新对象添加属性
  4 返回新对象

  ```
* 如何实现一个New
  ```
  function meNew() {
      // 创建一个新对象
      var o = new Object()

      // 将构造函数的作用域赋值给这个新对象，this指向这个新对象
      Constructor = [].shift.call(arguments) // 拿到构造函数，也就是第一个参数
      o.__proto__ = Constructor.prototype

      // 执行构造函数，把属性赋值给新对象
      Constructor.call(o, arguments)

      return o
  }
  ```
***
#### setTimeout那些神奇的问题
* [实现轮询](https://www.cnblogs.com/Mainz/archive/2009/04/27/1444691.html)
* for循环和setTimeout的经典问题：[setTimeout和作用域](https://www.cnblogs.com/Trista-l/p/7380830.html)，[这个解释也很好](https://www.jianshu.com/p/9b4a54a98660)
* 如何解决上述问题：[闭包let和匿名函数](https://www.jb51.net/article/122489.htm)
***
#### async和await
* [深入理解async和await](https://segmentfault.com/a/1190000007535316)
* 很重要的一句话：Promise的特点是无等待(就是为什么Promise的第一个参数会立即执行)，所以在没有await的情况下执行async函数，它会立即执行，返回一个Promise对象，并且，绝不会阻塞后面的语句。
* await后面是Promise的话，那么会阻塞await表达式下一行的代码，等外部同步的代码执行完毕后再回到这里，如果await后面不是Promise的话，那么await也会阻塞await表达式下一行的代码。后面是Promise返回结果的方式也是如下：
  ```
  Promise.reslove().then(function(){
      // 此任务也被推入微任务列表等待执行，执行完毕后再执行await的下一行代码
  })
  ```
***
#### 烦人的this
* 先从call bind apply开始吧[学会this这篇文章就够了](https://www.jianshu.com/p/6b4333e78bf5)
  ```
  function say(content1, content2) { 
      console.log("From " + this + ": Hello "+ content1 + "Ha "content2); 
  }
  say.call("Bob", "World", "Hi"); //==> From Bob: Hello World Ha Hi
  函数执行的的this指向第一个参数，tetfunction()也就是testfunction.call(window, xxx)的语法糖
  ```
* 改变this指向的实现：[call apply和bind的实现](https://github.com/Abiel1024/blog/issues/16)
  ```
  基础版本call：
  Function.prototype.meCall = function(context) {
      context.fn = this // 此处this就是.meCall这个.前的函数
      context.fn()
      delete context.fn
  }

  带参数的call：
  Function.prototype.meCall = function(context, ...params) {
      context.fn = this
      context.fn(...params)
      delete context.fn
  }

  apply同理，只是注意传参为数组

  bind的实现(返回一个改变this指向的函数，而不是立即执行)：
  Function.prototype.bind = function(context, ...params) {
      var me = this
      return function(...params) {
          me.call(context, ...params)
      }
  }
  ```
* this的值并不会按作用域链去找（非箭头函数）
  ```
  var name = 'big'

  var a = {
      fn: function() {
          console.log(this.name)
      }
  }

  window.a.fn() // undefined而不是去找作用域链上层的'big'
  ```
* 箭头函数的this： [有营养的教程](https://juejin.im/post/59bfe84351882531b730bac2)
  非常重要的概念：箭头函数的this始终指向函数定义时的this，而非执行时(一定要理解这句话，只是说定义的时候绑定了this，但this具体是什么值要看执行情况)。箭头函数需要记着这句话：箭头函数中没有this绑定，必须通过查找作用域链来决定其值，如果箭头函数被非箭头函数包含，则this绑定的是最近一层非箭头函数的this，否则，this为 undefined。理解下来就是箭头函数的确定是在定义时，绑定的是最近一层非箭头函数的this，但是注意！注意！注意！并不是意味着箭头函数的this就是不变的，只是说箭头函数的this绑定了最近一层非箭头函数的this，具体this是哪个值还是要看最近一层函数的执行情况。
  ```
  var name = "windowsName";

  var a = {
      name : "Cherry",

      func1: function () {
        console.log(this.name)     
  },

  func2: function () {
      setTimeout( () => {
        this.func1()
      },100);
    }
  };

  var c = a.func2
  c() // 此时this是window，this.func1()会报错
  a.func2()     // Cherry

  ```
***
#### 作用域的那些事
* 现在我明白了！！！到底什么是词法作用域，也就是静态作用域。javaScript使用的是词法作用域，也就是函数的作用域是在函数定义的时候确定的，而不是函数执行时！牢记这句话！
  ```
  var a = 'window'
  function bar() {
      console.log(a)
  }

  function foo() {
      var a = 'foo'

      bar()
  }

  foo() // window
  ```
* 关于变量提升：
  ```
  function foo() {
      console.log('foo1')
  }

  foo()

  function foo() {
      console.log('foo2')
  }

  foo()

  console两次foo2，因为函数声明会被提升
  ```
  还有一个经典问题：
  ```
  function foo() {
      console.log(a)
      a = 'test'
  }

  foo() // a is not defined
  a // test
  // 函数声明内部不带var的变量是全局变量，全局变量在给其赋值的函数体内并不存在提升，所以console不是undefined，而是直接报错
  ```
  函数声明和变量声明提升的优先级和注意点
  ```
  console.log(foo)

  function foo() {
      console.log('foo')
  }

  var foo = 1

  console是函数foo，因为在进入执行上下文时，首先会处理函数声明，其次会处理变量声明，如果如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性。
  ```
***
#### 同样烦人的闭包
* 不使用全局变量记录一个函数调用次数
  ```
  function count() {
      var count = 0

      return function() {
          count++
          console.log(count)
      }
  }

  var num = count()
  num() // 1
  num() // 2

  或者
  var countNumber = (function(){
    var num = 0;
    return function(){
        return ++num;
    };
  })();
    
  ```
* 小小的注意点：
  ```
  var obj = {
      fn: function() {
          console.log(fn)
      } 
  }

  obj.fn()

  会报错，在对象内部的函数表达式访问不到存放当前函数的变量，但是可以通过this访问
  ```
* 闭包的应用
  ```
  * 设计私有的方法和变量
  function animal() {
      var name = 'dog'

      var say = function() {
          return name
      }

      return say
  }

  * 减少全局变量的使用
  var objEvent = objEvent || {};
  (function(){ 
      var addEvent = function(){ 
          console.log(1)
      };
      function removeEvent(){
          console.log(2)
      }

      objEvent.addEvent = addEvent;
      objEvent.removeEvent = removeEvent;
  })();
  ```
* 区别闭包的变量访问和this访问
  ```
  function run() {
      var name = 'zl'

      setTimeout(function() {
          console.log(name)
          console.log(this)
      },1000)
  }

  run() // 'zl' Window

  ```
***
#### 柯里化和函数式编程
* 柯里化的定义：是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。即是接受一个参数，返回一个函数。
  ```
  这是一个打折的函数，接受一个商品价格和折扣
  function discount(price, discount){
      return price * discount
  }

  但是这样不够通用，因为每次都要输入两个参数，柯里化的处理方式
  function newDiscount(discount){
      return function(price){
          return price * discount
      }
  }

  var priceResult = newDiscount(0.1) // 普通用户
  priceResult(500) // 500的商品的折扣价格
  var vipPriceResult = newDiscount(0.2) // vip用户
  vipPriceResult(500) // 500商品vip用户的价格
  ```
***
#### escape encodeURI encodeURIComponent的使用场景
* [详细教程](https://www.cnblogs.com/season-huang/p/3439277.html)
  总的来说：escape对字符串编码，encodeURI对整个url进行编码，encodeURIComponent对url的参数编码
#### 关于reduce
```
[1, 2, 3].reduce(function(pre, next) {
    return pre + next
})

原生实现reduce
Array.prototype.ruduce = function(callback, initvalue){
    var pre = initvalue
    var k

    if(typeof initvalue === 'undefined') {
        pre = this[0]
        k = 1
    }

    for(k;k<this.length;k++) {
        pre = callback(pre, this[k])
    }

    return pre
}
```
***
#### vue是如何实现双向绑定的
* [实现自己的vue](https://www.cnblogs.com/libin-1/p/6893712.html)
***
#### 实现trim
* 原生实现trim()
  ```
  String.prototype.metrim = function() {
      var reg = /^\s+|\s+$/g

      return this.replace(reg, '')
  }
  ```
***
#### canvas和svg的异同点
* [异同点](https://www.cnblogs.com/Renyi-Fan/p/9223071.html)
***
#### 前端缓存相关
* 必会的缓存基础
  [前端缓存基础](https://mp.weixin.qq.com/s/b_vo_epjycDsGvczU6ol3Q)
***
#### HTTP相关
* TCP/IP协议族的分层
  ```
  应用层 传输层 网络层 链路层
  应用层：FTP DNS HTTP
  传输层：TCP(传输控制协议) UDP(用户数据协议) [UDP和TCP区别](https://www.cnblogs.com/xiaomayizoe/p/5258754.html)
  网络层：IP 传输数据包
  链路层：处理硬件
  ```
* 一次HTTP请求的大致过程
  ```
  应用层HTTP发送请求，在传输层TCP将报文分割并添加序号和端口号，到网络层增加作为通信目的地的MAC地址传输给链路层
  ```
* 什么是字节流服务
  ```
  TCP为了进行可靠传输，将报文切割成数据包，并未了确认传输到对方，采用三次握手策略
  ```
* 什么是持久链接
  ```
  初始的HTTP协议采取的是非持久链接，每一次请求就要断开TCP链接，这样增加通信量的开销，而持久化的链接可以在不明确断开链接的情况下一直请求，这样减少服务器消耗，增加网页响应速度，并且可以让请求实现管线化，一往是必须等待一个请求完成之后才能进行下一个请求，而管线化可以让请求并行发送
  ```
* HTTP状态码
  ```
  204：请求已经成功，但是为实体内容
  206：范围请求
  301：永久重定向，资源已经被分配新的地址
  302：临时重定向，资源位置已经改变，并且可能还要变
  303：和302类似，资源已经有新地址，并且指定使用get方法访问
  304：和重定向无关，if-match if-modified-since，如果缓存没有过期，那么就不向服务器发送请求，返回缓存资源
  307：和302相比不会强制转换为get方法
  400：请求报文中有语法错误
  401：发送的请求需要有通过HTTP认证的认证信息
  403：服务其拒绝响应
  404：资源未找到
  500：服务器在执行请求过程中发生错误
  502：网关或者代理服务器尝试请求时，上有服务器无响应
  503：服务器暂时处于超载情况
  ```
* HTTP的缺点
  ```
  采用明文传输，内容可能被窃听
  不验证通信双方身份，可能会被伪装
  无法验证报文完整性，报文可能遭到篡改
  ```
* HTTPS的解决方案
  ```
  通信加密，使用SSL或者TSL的安全套接层，建立安全通信线路
  内容加密，对请求报文进行内容加密
  证书验证通信身份，并保证报文完整性
  ```
* HTTP + 加密 + 认证 + 完整性保护 = HTTPS
  ```
  共享密钥加密：也就是密钥是共享的，使用同一个密钥进行加密解密，但是存在的问题就是如果密钥被劫持就会存在危险
  公开密钥加密：使用两把非对称密钥，一把似有密钥，一把共有密钥，似有密钥任何人都不能知道，而共有密钥是可以所有人获取的，首先交换对方的共有密钥，在传输时使用对方的共有密钥加密，然后对方收到信息使用似有密钥解密
  HTTPS使用二者结合的方式，在交换密钥的时候使用公开密钥加密，建立通信后使用共享密钥，安全认证机构负责确认交换的密钥真实可靠
  ```
* HTTPS请求过程
  ```
  发送报文，开始SSL通信
  服务器可进行SSL通信，返回server hello作为回应
  服务器返回证书
  服务器返回server hello done，最初阶段SSL握手结束
  客户端发送clientkey change
  客户端发送change cipher change
  客户端发送finished
  服务端发送change cipher change
  服务端发送finished
  开始http通信
  ```
***
#### 正则相关
* 邮箱正则
  ```
  var reg = /^[A-Za-z\d]+([-_.][A-Za-z\d]+)*@([A-Za-z\d]+[_-]{0,1}\.)+[A-Za-z\d]{2,4}$/

  var email = "lam.long.163@sina.com.cn"
  var email2 = 'lam-2.sina@163.com'

  // 技巧：
  * 开始为字母数字[A-Za-z\d]，开始之后和@之前的可以为-xxx _xxx .xxx 故[A-Za-z\d]+([_-.][A-Za-z\d]+)*@
  * 紧跟@的是xxx- xxx_ xxx. 但.是必须的-_是非必须的，故@([A-Za-z\d]+[-_]{0,1}\.)+
  * 最后[A-Za-z\d]{2, 4}

  ```
***
#### 算法和数据结构相关
***
#### Vue相关
***
#### React相关
* React路由原理
  [深入路由原理](https://blog.csdn.net/sinat_17775997/article/details/83477138)
***
#### 移动端适配1px
***
#### 防抖和节流
***
#### from表单可以跨域么
***