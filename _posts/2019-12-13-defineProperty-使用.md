## `Object.defineProperty`在 Vue2.0 的应用

在 ES6 之前，没有`Proxy`类用于代理劫持对象，通常会通过`Object.defineProperty`来实现对象属性的拦截，在 Vue2.0 版本中，使用这一属性实现了双向数据绑定，即数据的变化会引起 vDOM 的更新，从而更新实际 DOM。

### 基础语法

我们知道一个对象的属性，具有如下六个描述：

- `value`：属性值，不允许和`get`共用；
- `writable`：可写性，为`false`时属性不可写，不允许和`set`共用；
- `configurable`：为`false`时，属性不可被删除，属性的描述也不能被修改；
- `enumerable`：为`false`时，属性不可枚举，即遍历对象时，使用的`for in | Object.keys | Object.assign | JSON.stringify | Object.keys`，会忽略`enumerable`为`false`的属性；
- `get`：属性访问器函数，可以利用这个函数来劫持对象属性的访问；
- `set`：属性设置器函数，可以劫持对象属性的设值；

### 实现简单的 DOM 和数据双向绑定

```html
<div>
  <input type="number" id="nameInput" />
</div>
<script>
  let nameInput = document.querySelector('#nameInput');
  let tmpObj = {};
  Object.defineProperty(tmpObj, 'name', {
    get: () => {
      return nameInput.value;
    },
    set: val => {
      nameInput.value = parseFloat(val) + 1;
    }
  });
  const onNameInput = e => {
    tmpObj.name = parseFloat(e.target.value) + 1;
    console.log(tmpObje.name);
  };
  nameInput.addEventListener('input', onNameInput);
</script>
```

可以看到代码中设置了一个对象，类似`Vue`中的`data`属性，其中`name`是作为绑定`DOM`的值，在更新`name`的同时也会对输入框的值进行更新，在更新输入框的值时也会对`name`进行更新；

#### 实现对数组的监听

我们知道，在一般情况下除非整体替换整个数组，否则单独更新数组元素是不会触发数组的`set`方法的，来看下面的例子；

```javascript
let obj = { array: [] };
let initialVal = [];
Object.defineProperty(obj, 'array', {
  set: val => {
    console.log('in setter: ' + val);
    initialVal = val;
  },
  get: () => {
    return initialVal;
  }
});
obj.array[0] = 10; // 10
obj.array.push(20); // 4
obj.array = [1, 2, 3]; // "in setter [1, 2, 3]" [1, 2, 3]
```

可以看到给单独给数组元素赋值或者调用数组方法`push`，不会触发`set`，而整体替换时可以触发；

其实`Vue`可以对数组实现监听，这是因为源码对数组的一部分原型方法进行了重写，这部分感觉对我来说有点复杂了，下面是一个 demo：

```javascript
var arrayProto = Array.prototype;

var arrayMethods = Object.create(arrayProto);

['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(function(method) {
  // 使用 Object.defineProperty 进行监听
  Object.defineProperty(arrayMethods, method, {
    value: function testValue() {
      console.log('数组被访问到了');
      const original = arrayProto[method];
      // 使类数组变成一个真正的数组
      const args = Array.from(arguments);
      original.apply(this, args);
    }
  });
});
```

代码中重写了部分方法，直接改写了`value`，不改变这些方法的实现（`original.apply(this, args)`直接使原生方法调用结果不变），但可以在调用这些方法时加入我们需要的其他代码，比如添加订阅者响应等，这里的订阅者就是`console.log('数组被访问到了')`；

而对每个数组元素，也需要对它们改写`get | set`函数，使得访问数组元素时也能添加我们自己的代码，从而实现双向绑定；

来看一个 demo：

```javascript
// 为了不影响Array的原生方法，我们应该新生成Array原型的子类arrayMethods，继承自Array.prototype，这样我们去覆盖子类的部分方法时，就不会影响到其他普通数组去调用这部分方法。
let arrayProto = Array.prototype;
let arrayMethods = Object.create(arrayProto);

['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(method => {
  // 对这些方法进行改写，添加自己的代码，并使用apply让方法正常运行
  Object.defineProperty(arrayMethods, method, {
    value(...args) {
      console.log('数组被访问到了', method, args);
      const original = arrayProto[method];
      // 此时this指向调用时的实例
      original.apply(this, [...args]);
    }
  });
});
// 定义观察者类，在初始化时调用一次
class Observer {
  constructor(data) {
    this.data = data;
    traversal(data);
  }
}
// 添加拦截器，这里把对象和数组都作为可以添加拦截器的对象，所以针对数组元素也会受到影响，最直接的就是单独访问数组元素时也能触发拦截器
const defAry = (key, val, o) => {
  // debugger;
  Object.defineProperty(o, key, {
    enumerable: true,
    configurable: true,
    get() {
      console.log(key, 'getters', val);
      return val;
    },
    set(newVal) {
      console.log(key, 'setters', newVal);
      val = newVal;
    }
  });
};
// 遍历函数，用于遍历对象的属性，如果是嵌套对象也会进行递归调用
const traversal = o => {
  Object.entries(o).forEach(val => {
    let [key, value] = [...val];
    if (typeof value === 'object') {
      if (Array.isArray(value)) {
        // 这一步是将当前数组的__proto__指向子类arrayMethods，从而可以访问被覆盖的数组方法，比如push等
        value.__proto__ = arrayMethods;
        value.forEach((el, index) => {
          debugger;
          defAry(index, el, value);
        });
      } else {
        // 如果是对象的话，递归调用
        traversal(value);
      }
    }
    // 对对象的每个属性都要执行添加拦截器操作
    defAry(key, value, o);
  });
};
// 例子
let data = {
  a: 10,
  b: [1, 2, 3]
};
let observer = new Observer(data);
data.b[1]; // 1 getters 2
data.push(4); // 数组被访问到了 push 4
```

那么其实在 Vue 源码中有更完整的实现过程，主要是 2.0 的版本，详见 Vue 源码分析[从源码角度再看数据绑定](https://github.com/answershuto/learnVue/blob/master/docs/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E5%86%8D%E7%9C%8B%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A.MarkDown)
