# ES6 新特性学习第六篇

## 第一篇前言

本文建立在学习阮一峰老师的 ES6 教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp 'ES6 全套教程 ECMAScript6 原著:阮一峰 ') 和 [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ '阮一峰ES6')

### class

其实在另一篇 blog 里已经介绍了 ES5 创建对象的语法[https://youzixr.github.io/2019/03/12/JS-%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1/](https://youzixr.github.io/2019/03/12/JS-%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1/)，但为了使 ES6 更易用而引入了`class`，有其他 OO 语言经验的人能快的上手这部分内容。

**类可以看做是ES5构造函数的语法糖**

来看一个例子：

    class Person {
        constructor(name, age) {
            this.name = name
            this.age = age
        }
        growUp() {
            return this.age++
        }
    }
    let p = new Person('?', 10)
    typeof Person // function
    Person == Person.prototype.constructor // true
    p.constructor == Person.prototype.constructor // true

**类方法都是定义在 prototype 上的**

    // 上述写法相当于ES5的：
    Person.prototype = {
        constructor() {},
        growUp() {}
    }
    // 如果需要继续添加方法可以使用
    Object.assign(Person.prototype, {
        methods() {
            // ...
        }
    })

**类内部定义的方法都是不可枚举的，enumerable == false**

#### constructor

类方法有一个默认的方法，当用`new`生成实例的时候，会自动调用这个方法；一个类必须有`constructor`方法，如果没有显示定义，一个空的`constructor`方法会被默认添加。

    class Point {
    }

    // 等同于
    class Point {
      constructor() {}
    }

另外`constructor`方法默认会返回实例`this`，也可以指定返回另一个对象，但是这样用`instanceof`运算符的时候会有问题。

#### 实例

生成实例必须用`new`，如果用函数方法调用就会报错。实例的属性会定义在原型对象上，除非用`this`给实例赋一次值。（其实相当于用原型模式来创建实例对象，上篇 blog 有提过）

**所有的实例对象共享同一个原型对象，**proto**属性指向同一个对象**

尽量不推荐使用`__proto__`属性去改写原型对象，会影响到所有实例对象。

#### get / set 方法

类内部可以使用`get / set`方法去劫持某个属性的存取行为；一般写法：

```javascript
class Person {
  get name() {
    return this.name;
  }
  set name(val) {
    this.name = `hello ${val}`;
  }
}
```

#### 属性表达式

例子：

```javascript
const method = 'getName';
class Person {
  [method]() {}
}
```

#### 类名表达式

类的表达式与函数类似，可以用这种方法建立类似私有类；

```javascript
const AnotherPerson = class Person {
  getName() {
    return Person.name;
  }
};
```

上面代码的`Person`是类名，但是这个类名只能在类的内部使用，外部调用需要用`AnotherPerson`；特别的，如果在类内部没有需要用到类名的地方，可以像匿名函数那样定义类；

```javascript
const Person = class {
  // ...
};
```

另外，也可以像立即执行函数那样，写成立即执行类；

```javascript
let person = new (class {
  constructor() {}
  getName() {}
})();
```

#### 静态方法

在一个方法前加上`static`，表示这个方法是静态方法，不会被类的实例继承，而是被类对象直接调用。

**静态方法里的`this`指向类对象本身**

**父类静态方法会被子类继承**

**静态方法也可以从`super`对象调用**

例子：

    class Person {
        constructor(name, age) {
            this.name = name
            this.age = age
        }
        static growUp() {
            return this + '?'
        }
    }
    Person.growUp() //class Person ... + '?'
    let p = new Person('?', 10)
    // p.growUp() // Uncaught TypeError: p.growUp is not a function
    let someMethods = '???'
    class Male extends Person {
        static [someMethods]() {
            return super.growUp() + '\n?'
        }
    }
    Male['???']()
    /* "class Male extends Person {
            static [someMethods]() {
                return super.growUp() + '\n?'
            }
        }?
    ?" */

#### 静态属性

跟静态方法类似，是类对象本身的属性，不会被实例对象继承，但是语法不支持在属性变量前加`static`属性，所以改为直接在类对象上添加属性`Person.property = '???'`

#### 私有方法和私有属性

在命名上区别私有对象，但是没啥用，该访问到的还是能访问到。

将私有方法移出类对象，例子：

    class Widget {
      foo (baz) {
        bar.call(this, baz);
      }
      // ...
    }

    function bar(baz) {
      return this.snaf = baz;
    }

利用`Symbol`值的唯一性，将私有方法的名字命名成`Symbol`值

    const bar = Symbol('bar');
    const snaf = Symbol('snaf');
    export default class myClass{
      // 公有方法
      foo(baz) {
        this[bar](baz);
      }
      // 私有方法
      [bar](baz) {
        return this[snaf] = baz;
      }
      // ...
    };

### 继承

类对象可以通过`extends`关键字来实现继承，ES5 语法中需要通过修改原型链实现继承，语法上简单不少；

**子类`constructor`方法中一开始就要调用`super`方法**，否则新建实例时会报错；具体原因是因为子类自己的`this`对象必需通过父类的构造函数，得到和父类实例对象一样的属性和方法，才能构造出子类的`this`，后续再对子类进一步添加属性方法等等操作。

**\*ES6 的继承机制是先将父类实例对象的属性和方法，添加到子类的`this`上，再用子类的构造函数修改子类实例对象的`this`；而 ES5 的继承，先创建子类实例对象`this`，再将父类方法添加到`this`上（`Parent.apply(this)`）**；

另外，子类如果没有显式定义`constructor`方法，这个方法会被默认添加，**只有调用`super`之后，才能使用`this`关键字**

**父类的静态方法会被子类继承**

    class Super {
        constructor(x, y) {
            this.x = x
            this.y = y
        }
        method() {
            return `${this.x} + ${this.y}`
        }
    }
    class Sub extends Super {
        constructor(x, y, z) {
            super(x, y) // 调用父类的constructor(x, y)
            this.z = z
        }
        method() {
            return this.z + super.method() // 调用父类的method()方法
        }
    }

#### 判断继承

用`Object.getPrototypeOf()`方法从子类上取得其父类，所以可以这样判断：`Object.getPrototypeOf(sub) == super`

#### super

这个保留字在 ES6 可以当做函数使用，也可以当做对象使用。

在前面的代码中可以看到，`super`作为函数调用时，代表了父类的构造函数，刚刚也提过，在子类的构造函数里必须执行一次`super()`；这里的`super`虽然是父类的构造函数，但其实返回的是子类的实例对象，意思是在`super`内的`this`指向了子类的实例对象，所以在上一段代码中，子类调用的`super(x,y)`，参数都被绑定在子类的实例上了；**super()作为函数时，只能在子类的构造函数 constructor 方法中使用**

被当做对象使用时，在普通方法中，**`super`指向了父类的原型对象**，所以定义在构造函数上的属性无法被调用；**子类普通方法通过 super 调用父类方法时，方法内部的 this 指向当前子类的实例对象**；**在静态方法中，`super`指向父类**。

    class Parent {
        constructor() {
            this.x = '?'
            return this.x
        }
        method() {
            console.log(this.x)
            return this.x
        }
    }
    class Child extends Parent {
        constructor(x) {
            super()
            this.x = x
            console.log(`${super.x} + ${this.x}`);
            /* 这里是取不到super.x这个值的，
            因为super作为对象时，指向的是父类Parent的原型对象，
            而父类的x是定义在实例上的，无法通过super调用 */
        }
        method() {
            return `${super.method()} + ?`
            /* 这里子类的普通方法调用了父类的方法，
            ES6规定此时方法内部this指向子类的实例 */
        }
    }
    let child = new Child('!')
    // undefined + !
    child.method()
    // !
    // "! + ?"

一个挺有意思的例子：

    class A {
      constructor() {
        this.x = 1;
      }
    }
    class B extends A {
      constructor() {
        super();
        this.x = 2;
        super.x = 3;
        super.y = '?';
        console.log(this.y); // '?'
        console.log(super.x); // undefined
        console.log(this.x); // 3
      }
    }
    let b = new B();

上面的例子说明了，**通过`super`对属性赋值，此时的`super`指向子类实例，赋值的属性会变成子类实例的属性**。换句话说，赋值的时候是给实例赋值的；但读取的时候，读的是父类原型对象。

刚也提到过，`super`作为对象在子类静态方法中调用时，指向**父类对象本身**（不是父类原型对象），来看一个例子。

    class Parent {
    static myMethod(msg) {
        console.log('static', msg);
    }
    myMethod(msg) {
        console.log('instance', msg);
    }
    }
    class Child extends Parent {
    static myMethod(msg) {
        super.myMethod(msg);
    }
    myMethod(msg) {
        super.myMethod(msg);
        console.log('?');
    }
    }
    Child.myMethod(1); // static 1
    var child = new Child();
    child.myMethod(2);
    // instance 2
    // ?

首先明确：父类静态方法会被子类继承，类的静态方法不会被类的实例继承。所以上面这段代码，`Child.myMethod`调用的是父类的静态方法， `child.myMethod`调用的是实例的方法；这里要明确实例对象`child`本身继承了子类的普通方法，另外也继承了父类的普通方法，但是因为两个方法同名，所以在调用时查找原型链会先找到子类方法，也可以通过`child.__proto__.__proto__.myMethod()`找到父类的这个方法。

**在子类的静态方法中通过 super 调用父类方法时，方法内部的 this 指向子类，而不是子类实例**

#### 总结

这里做一个关于`this`、`static`、`super`的总结：

- 类的普通方法定义在类对象的原型上；而类的静态方法定义在类对象自身上。
- `super`被当做对象时，在子类普通方法中指向父类的原型对象；在子类的静态方法中指向父类对象本身；所以如果在子类的静态方法里通过`super`只能调用父类的静态方法。
- 类的实例`__proto__`指向同一个原型。
- 子类的普通方法通过`super`调用父类的普通方法，父类方法里的`this`指向子类的实例对象；子类普通方法通过`super`无法访问到父类的静态方法。
- 静态方法里的`this`指向类对象本身，子类里就指向子类本身，父类里还是指向子类本身。
- 使用`super`时必须显式指定是作为函数还是对象来使用，否则会报错。
- 由于对象总是继承自其它对象，所以在任一个对象中都可以使用`super`关键字。

### 类的 prototype 和**proto**

首先明确，类同时具有函数的`prototype`属性和对象的`__proto__`属性。

1. 子类的`__proto__`属性，指向父类，表示构造函数的继承。
2. 子类的`prototype`属性的`__proto__`属性表示方法的继承，总是指向父类的`prototype`属性。

出现这样的结果是因为类的继承是分为两步的：

1. 子类的实例继承父类的实例： `Object.setPrototypeOf(Sub.prototype, Parent.prototype)`
2. 子类继承父类的静态属性： `Object.setPrototypeOf(Sub, Parent)`

而`Object.setPrototypeOf()`方法实现是以下方法：

    Object.setPrototypeOf = function (obj, proto) {
        obj.__proto__ = proto;
        return obj;
    }

#### 实例的**proto**属性

子类实例的`sub.__proto__.__proto__`指向父类实例的`super.__proto__`，子类的原型的原型，是父类的原型。

### 原生构造函数的继承

ES6 允许继承原生构造函数，使用`extends`关键字：

    Boolean()
    Number()
    String()
    Array()
    Date()
    Function()
    RegExp()
    Error()
    Object()

前面也提到过，ES5 是先创建子类的实例，再给实例添加属性和方法，由于原生构造函数的属性和方法是无法获取的，导致了 ES5 环境下无法继承原生构造函数。而 ES6 是先创建父类的实例，父类的实例本身就继承了父类的所有属性和普通方法，然后再用子类的构造函数`constructor`，使得子类的构造函数继承父类的构造函数，所以可以通过`extends`在原生数据结构的基础上构建自己的数据结构。

### Mixin

Mixin 指多个对象合成一个新的对象，新对象具有各个组成员的接口，简单实现如下：

    const a = {
        a: 'a'
    }
    const b = {
        b: 'b'
    }
    const c = {...a, ...b}

完整的实现可以参考：

    function mix(...mixins) {
    class Mix {
        constructor() {
        for (let mixin of mixins) {
            copyProperties(this, new mixin()); // 拷贝实例属性
        }
        }
    }

    for (let mixin of mixins) {
        copyProperties(Mix, mixin); // 拷贝静态属性
        copyProperties(Mix.prototype, mixin.prototype); // 拷贝原型属性
    }

    return Mix;
    }

    function copyProperties(target, source) {
    for (let key of Reflect.ownKeys(source)) {
        if ( key !== 'constructor'
        && key !== 'prototype'
        && key !== 'name'
        ) {
        let desc = Object.getOwnPropertyDescriptor(source, key);
        Object.defineProperty(target, key, desc);
        }
    }
    }
    // 继承
    class DistributedEdit extends mix(Loggable, Serializable) {
        // ...
    }
