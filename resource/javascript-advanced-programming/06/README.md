# 6 面向对象的程序设计

## 6.1 理解对象

  ```javascript
 /**
  * Object构造函数
  */
let person = new Object();        // let person = {}; 这是新建对象的语法糖

person.name = 'Nicholas';
person.age = 29;
person.job = 'Software Engineer';

person.sayName = function () {
    alert(this.name);             // 会被解析为 person.name;
};
```

```javascript
/**
 * 字面量语法
 */
let person = {
    name: 'Nicholas',
    age: 29,
    job: 'Software Engineer',
    sayName: function () {
        alert(this.name);         // 会被解析为 person.name;
    }
};
```

### 6.1.1 属性类型

ECMAScript中有两种属性： **数据属性** 和 **访问器属性** 。
#### 数据属性

数据属性有4个描述其行为的特性;

- [[Configurable]]：表示能否通过delete删除属性从而重新定义属性、能否修改属性的特性，或者能否把属性修改为访问器属性。（直接在对象上定义的属性，该特性默认值为true）。
- [[Enumerable]]：表示能否通过for-in循环返回属性。（直接在对象上定义的属性，该特性默认值为true）。
- [[Writable]]：表示能否修改属性的值。（直接在对象上定义的属性，该特性默认值为true）。
- [[Value]]：包含这个属性的数据值。读取属性值的时候从这个位置读；写入属性值的时候把新值保存在这个位置。（该特性默认值为undefined）。

```javascript
let person = {
    name: 'Nicholas'
}
```

创建了一个名为name的属性，其值为'Nicholas'；也就是，[[Value]]特性将被设置为'Nicholas'。

既然属性有默认特性值，那么它也一定可以被修改，ECMAScript 5（以下简称ES5）就为提供了 **Object.defineProperty()** 这个方法包含三个参数： 属性所在对象、属性的名字和一个描述符对象（该对象的属性必须为：configuration、enumerable、writable、value）。

```javascript
let person = {};
Object.defineProperty(person, 'name', {
   writable: false,
   value: 'Nicholas' 
});

console.log(person.name); // 'Nicholas'
person.name = 'Barry';    // 非严格模式下，此操作忽略，执行下一行代码；严格模式下，该操作会导致抛出错误。
console.log(person.name); // 非严格模式下打印 'Nicholas'
```

```javascript
let person = {};
Object.defineProperty(person, 'name', {
   configurable: false,
   value: 'Nicholas' 
});

// 情况一
console.log(person.name); // 'Nicholas'
delete person.name;       // 非严格模式下，此操作忽略，执行下一行代码；严格模式下，该操作会导致抛出错误。
console.log(person.name); // 非严格模式下打印 'Nicholas'

// 情况二
Object.defineProperty(person, 'name', {
   configurable: true,
   value: 'Nicholas' 
});
// 抛出错误，一旦configurable设置为false，就不能再更改回去；同时再调用Object.defineProperty()方法只能修改writable特性，否则会抛出错误。
```

在调用Object.defineProperty()方法创建一个新属性时，如果不指定，configuration、enumerable、writable特性的默认值都会为false。

*由于实现不彻底，IE8不建议使用Object.defineProperty()方法。*

#### 访问器属性

访问器属性不包含数据值；它们包含一对getter和setter函数（这两个函数不是必须的）；访问器属性拥有4个特性：

- [[Configurable]]：表示能否通过delete删除属性从而重新定义属性、能否修改属性的特性，或者能否把属性修改为访问器属性。（直接在对象上定义的属性，该特性默认值为true）。
- [[Enumerable]]：表示能否通过for-in循环返回属性。（直接在对象上定义的属性，该特性默认值为true）。
- [[Get]]：在读取属性时调用的函数。（默认值为undefined）
- [[Set]]：在写入属性时调用的函数。（默认值为undefined）

访问器属性也不能直接定义，必须使用Object.defineProperty()方法。

```javascript
let book = {
    _year: 2004,        // 添加下划线，表示只能通过对象方法访问的属性。
    edition: 1
};

Object.defineProperty(book, 'year', {
    get: function() {
      return this._year;
    },
    set: function(newValue) {
      if (newValue > 2004) {
          this._year = newValue;
          this.edition += newValue - 2004;
      }
    }
});

book.year = 2005;
console.log(book.year); // 2
```

getter和setter可以不同时设定，单独设置getter（或setter）意味着属性不能写（或读），尝试写入（或读取）属性会被忽略（或返回undefined），在严格模式下都会抛出错误。

### 6.1.2 定义多个属性

当定义多个属性时，ES5又为开发者提供了一个 **Object.defineProperties()方法**，该方法接收两个参数：要添加或修改其属性的对象、对象属性要与第一个对象中要添加或修改的属性一一对应。

```javascript
let book = {};

Object.defineProperties(book, {
    _year: {
        writable: true,
        value: 2004
    },
    edition: {
        writable: true,
        value: 1
    },
    year: {
        get: function() {
          return this._year;
        },
        set: function() {
          if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
          }
        }
    }
});
```

以上代码实现了在book对象中定义了两个数据属性：_year和edition，一个访问器属性：year。

### 6.1.3 读取属性的特性

ES5提供了一个 **Object.getOwnPropertyDescriptor()** 方法，可以取得给定属性的描述符。该方法接收两个参数：属性所在的对象和要读取其描述符的属性名称。返回值为一个对象。

```javascript
let book = {};

Object.defineProperties(book, {
    _year: {
        writable: true,
        value: 2004
    },
    edition: {
        writable: true,
        value: 1
    },
    year: {
        get: function() {
          return this._year;
        },
        set: function() {
          if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
          }
        }
    }
});

let descriptor = Object.getOwnPropertyDescriptor(book, '_year');
console.log(descriptor.value);        // 2004
console.log(descriptor.configurable); // false
console.log(typeof descriptor.get);   // undefined

descriptor = Object.getOwnPropertyDescriptor(book, 'year');
console.log(descriptor.value);        // undefined
console.log(descriptor.enumerable);   // false
console.log(typeof descriptor.get);   // 'function'
```

在JavaScript中，可以针对任何对象——包括DOM和BOM对象，使用Object.getOwnPropertyDescriptor()方法。兼容浏览器有IE9+和现代浏览器。

## 6.2 创建对象

传统的Object构造函数和对象字面量虽然都可以创建单个对象，但是这些方式有个明显缺点：使用同一个接口创建很多对象，产生大量重复代码。 **工厂模式** 出现就是为了解决这个问题。

### 6.2.1 工厂模式

因为无法在ECMAScript中创建类，所以用函数来封装以特定接口创建对象的细节。如：

```javascript
function createPerson(name, age ,job) {
  let obj = new Object();
  
  obj.name = name;
  obj.age = age;
  obj.job = job;
  obj.sayName = function() {
    alert(this.name);
  };
  
  return obj;
}

let p1 = createPerson('Nicholas', 29, 'Software Engineer');
let p2 = createPerson('Barry', 27, 'Doctor');
```

工厂函数虽然解决了创建对个相似对象的问题，但是没有解决如何识别对象类型的问题。 **构造函数模式** 解决了这个问题。

### 6.2.2 构造函数模式

ECMAScript中的构造函数可以用来创建特定类型对象，比如Array和Object这样的原生构造函数；也可以创建自定义的构造函数，从而自定义对象类型的属性和方法。

```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function() {
    alert(this.name);
  }
}

let p1 = new Person('Nicholas', 29, 'Software Engineer');
let p2 = new Person('Barry', 27, 'Doctor');
```

Person()函数和createPerson()函数存在以下差异：

- 没有显示创建对象；
- 直接将属性和方法赋给了this对象；
- 没有return语句；

在创建Person实例时，必须使用new操作符，这种方式调用构造函数实际上经过以下过程：

1. 创建一个新对象；
2. 将构造函数的 **作用域** 赋给新对象（this就指向了新对象）；
3. 执行构造函数中的代码（为新对象添加属性和方法）；
4. 返回新对象；

Person的两个不同实例对象p1、p2都包含着一个constructor属性，该属性最初是用来标识对象类型的。

```javascript
console.log(p1.constructor === Person);
console.log(p2.constructor === Person);
```

但是检测对象类型， **instanceof** 操作符更可靠一些。

```javascript
console.log(p1 instanceof Object);  // true
console.log(p1 instanceof Person);  // true
console.log(p2 instanceof Object);  // true
console.log(p2 instanceof Person);  // true

// p1、p2都是Person的实例，而Person又是Object的实例，所有对象均继承自Object。
```

#### 6.2.2.1 将构造函数当作函数

任何函数，只要通过 new 操作符来调用，那它就可以作为构造函数；而任何函数，如果不通过 new 操作符来调用，那么它跟普通函数没什么区别。

```javascript
// 当做构造函数使用
let person = new Person('Nicholas', 29, 'Software Engineer');
person.sayName();               // Nicholas

// 作为普通函数调用
Person('Greg', 27 , 'Doctor');  // 添加到全局window对象
window.sayName();               // Greg

// 在另一个对象的作用域中调用
let obj = new Object();
Person.call(obj, 'Kristen', 25, 'Nurse');
obj.sayName();                  // Kristen
```

#### 6.2.2.2 构造函数的问题

使用构造函数的主要问题就是，每个方法都要在每个实例上重新创建一遍。例如之前例子中，p1、p2都有一个名为sayName()的方法，但是两个方法不是同一个Function的实例。（ECMAScript中的函数是对象！！！）

```javascript

function Person(name, age, job) {
    // ...略
    this.sayName = new Function('console.log(this.name)');  // 等价于 this.sayName: function() {}
}
```

以这种方式创建，会导致不同的作用域和标识符解析，因此p1和p1的同名函数实际上是不相等的

```javascript

console.log(p1.sayName === p2.sayName); // false
```

为了解决这个问题，我们可以：通过把函数定义转移到函数外部来解决。

```javascript
function Person(name, age, job) {
  // ...略
  this.sayName = sayName;
}

function sayName() {
  console.log(this.name);
}
```

这样一来，我们将 sayName 属性设置成了等于全局的 sayName 函数，p1和p2对象就共享了在全局作用域中定义的sayName()函数。但新问题又出现了：

1. 在全局作用域中定义的函数实际上只能被某个对象调用
2. 而且如果对象需要定义很多方法，那么就需要定义很多歌全局函数，这样就没有封装性可言了。  

好在出现了 原型模式 来解决上述问题。

### 6.2.3 原型模式

**我们创建的每一个函数都有一个 prototype（原型）属性** ，指向的是一个包含可以由特定类型的所有实例共享的属性和方法的对象，换言之： prototype 指向的是通过调用构造函数创建的实例对象的原型对象。。

```javascript
function Person() {  
}

Person.prototype.name = 'Nicholas';
Person.prototype.age = 29;
Person.prototype.job = 'Software Engineer';
Person.prototype.sayName = function() {
  console.log(this.name);
};

let p1 = new Person();
let p2 = new Person();

console.log(p1.sayName());              // Nicholas
console.log(p2.sayName());              // Nicholas
console.log(p1.sayName === p2.sayName); // true
```

#### 6.2.3.1 什么是原型对象？

只要创建了一个新函数，那么就会根据一组特定的规则为该函数创建一个 prototype 属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个 constructor（构造函数）属性，这个属性是一个指向prototype 属性所在函数的指针。

```javascript
console.log(Person.prototype.constructor === Person); // true
```

- 创建自定义构造函数后，其原型对象默认只会取得 constructor 属性，至于其他方法均继承自Object
- 调用构造函数创建新实例后，该实例内部将包含一个指针指向构造函数的原型，ES5中该指针叫 [[prototype]]，Firefox、Chrome和Safari都支持 __proto__ 属性。
- 重要的一点就是：这个连接存在于实例与构造函数的原型对象之间，而不存在于实例与构造函数之间。 

```javascript
// Person构造函数、Person的原型属性和Person的实例之间的关系
console.log(Person.prototype.constructor === Person);
console.log(new Person().__proto__ === Person.prototype); // true
```

我们可以用 **isPrototypeOf()** 方法来确定对象之间是否存在原型的关系，其本质就是：如果 [[prototype]] 指向了调用 isPrototypeOf() 方法的对象（Person.prototype），那么该方法就会返回 true。

```javascript
console.log(Person.prototype.isPrototypeOf(p1));  // true
console.log(Person.prototype.isPrototypeOf(p2));  // true
```

ES5还提供了一个方法 **Object.getPrototypeOf()** ，该方法分会 [[prototype]] 的值，使用 Object.getPrototypeOf() 方法也可以方便的获取一个对象的原型。

```javascript
console.log(Object.getPrototypeOf(p1) === Person.prototype);  // true
console.log(Object.getPrototypeOf(p1).name);  // 'Nicholas'
```

我们可以通过实例访问保存在原型中得值，但是不能通过实例重写原型中的值，当实例中的属性与原型中属性同名了，那么该属性就会屏蔽（不是替换也不是覆盖，而是阻止我们访问原型中的属性，但原型中的属性依然存在，哪怕值为null）原型中的属性。不过，我们也可以通过 delete 操作符完全删除实例中的属性，从而我们又可以重新访问原型中的属性。  
使用 **hasOwnProperty()** 方法可以检测一个属性是存在于实例中还是原型中，返回true表示访问的是实例属性。

```javascript
function Person() {
}

Person.prototype.name = 'Nicholas';

let p1 = new Person();
console.log(p1.name);                   // 'Nicholas' 来自原型
console.log(p1.hasOwnProperty('name')); // false，此时访问的是原型属性

p1.name = 'Greg';
console.log(p1.name);                   // 'Greg' 来自实例
console.log(p1.hasOwnProperty('name')); // true，此时访问的是实例属性

delete p1.name;
console.log(p1.name);                   // 'Nicholas' 来自原型
console.log(p1.hasOwnProperty('name')); // false，此时访问的是原型属性
```

> ES5的 Object.getOwnPropertyDescriptor() 方法只能用于实例属性，想要取得原型属性的描述符，必须直接在原型对象上调用 Object.getOwnPropertyDescriptor() 方法。

#### 6.2.3.2 原型与in操作符

两种使用方式：

- 单独使用 **in** （无论该属性存在于实例中还是原型对象中，in 操作符会在对象通过对象能够访问给定属性时返回 true）。

```javascript
function Person() {
}

Person.prototype.name = 'Nicholas';

let p1 = new Person();
console.log(p1.name); // 'Nicholas' 来自原型
console.log(p1.hasOwnProperty('name')); // false，此时访问的是原型属性
console.log('name' in p1); // true

p1.name = 'Greg';
console.log(p1.name); // 'Greg' 来自实例
console.log(p1.hasOwnProperty('name')); // true，此时访问的是实例属性
console.log('name' in p1); // true

delete p1.name;
console.log(p1.name); // 'Nicholas' 来自原型
console.log(p1.hasOwnProperty('name')); // false，此时访问的是原型属性
console.log('name' in p1); // true
```

同时使用 hasOwnPrototype() 方法和 in 操作符，就可以确定该属性到底存在于对象中还是原型中。

```javascript
function hasPrototypeProperty(object, name) {
  return !Object.hasOwnProperty(name) && (name in object);
}
```

- **for-in** 循环中使用（返回的是所有能够通过对象访问的、可枚举的属性，既包括存在于实例中的属性也包括存在于原型中的属性，即使是不可枚举属性[[enumerable]]设置为false，也能被返回）。

在IE8以及更早的版本中出现一个bug，即可枚举属性[[enumerable]]设置为false的实例属性不会出现在 for-in 循环中。该bug会影响默认不可枚举的所有属性和方法，包括： **hasOwnProperty()**、**propertyIsEnumerable()**、**toLocaleString()**、**toString()** 和 **valueOf()** 。ES5也将 constructor 和 prototype 属性的 [[enumerable]] 特性设置为false。  

想要取得所有可枚举的实例属性，可以使用ES 的 **Object.keys()** 方法。

```javascript
function Person() {
}

Person.prototype.name = 'Nicholas';
Person.prototype.age = 29;
Person.prototype.job = 'Software Engineer';
Person.prototype.sayName = function() {
  console.log(this.name);
};

let keys = Object.keys(Person.prototype);
console.log(keys); // 'name, age, job, sayName'

let p1 = new Person();
p1.name = 'Rob';
p1.age = 31;
let p1Keys = Object.keys(p1);
console.log(p1Keys); // 'name, age'
```

如果你想得到所有实例属性，不管是否可枚举， 可以使用 **Object.getOwnPropertyNames()** 方法。

```javascript
let keys = Object.getOwnPropertyNames(Person.prototype);
console.log(keys); // 'constructor, name, age, job, sayName'
```

#### 6.2.3.3 更简单的原型语法

使用字面量方式来写原型对象：

```javascript
function Person() {
}

Person.prototype = {
    name: 'Nicholas',
    age: 29,
    job: 'Software Engineer',
    sayName: function() {
      console.log(this.name);
    }
};
```

虽然以这种方式写更简便了，但是随之也出现了一个问题： constructor 属性不在指向 Person 了。

> 原因解释：当我们使用字面量创建时，Person.prototype = {} 等同于 Person.prototype = new Object() ，这里我们完全重写了默认的 prototype 对象，因此 constructor 属性也就变成了新对象的 constructor属性（指向 Object 构造函数），不再指向 Person 函数。尽管 instanceof 操作符还能返回正确结果，但是 constructor 已经无法确定对象的类型了。

```javascript
let p1 = new Person();

console.log(p1 instanceof Object); // true
console.log(p1 instanceof Person); // true
console.log(p1.constructor === Person); // false
console.log(p1.constructor === Object); // true
```

解决办法：

```javascript
function Person() {
}

// 解决方案一
Person.prototype = {
    constructor: Person, // 手动将 constructor 指回 Person，但是这样 [[Enumerable]] 会被设置为true，默认原生的 constructor 属性是不可被枚举的
    name: 'Nicholas',
    age: 29,
    job: 'Software Engineer',
    sayName: function() {
      console.log(this.name);
    }
};

// 解决方案二（最佳）
Object.defineProperty(Person.prototype, 'constructor', {
    enumerable: false,
    value: Person
})
```

#### 6.2.3.4 原型的动态性

由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来——即使是先创建了实例后修改了原型也照样如此。

```javascript
let p1 = new Person();

Person.prototype.sayHi = function() {
  console.log('Hi');
};
p1.sayHi(); // 'Hi'
```

即使实例p1是在添加新方法之前创建的，但他任然可以访问这个新方法。其原因是归结为实例与原型之间的松散型连接关系：当我们调用 p1.sayHi() 时，首先会在实例中搜索 sayHi() 方法，当实例中查找不到时，就会去原型中查找。因为实例和原型之间的连接只不过是一个指针，而非一个副本，因此就可以在原型中找到新的 sayHi 属性并返回保存在那里的函数。

但是如果是重写整个原型对象，那么情况就不一样了：构造函数时会为实例添加一个指向最初原型的 [[prototype]] 指针，而把原型修改为另外一个对象就等于切断了构造函数与最初原型之间的联系。

**切记：实例中的指针仅指向原型，而不指向构造函数。**

```javascript
function Person() {
}

let p1 = new Person();

Person.prototype = {
    constructor: Person,
    name: 'Nicholas',
    age: 29,
    job: 'Software Engineer',
    sayName: function() {
      console.log(this.name);
    }
};

p1.sayName(); // error，因为此时 p1 指向的原型中不包含以该名字命名的属性。
```

#### 6.2.3.5 原生对象的原型

原型模式的重要性不仅体现在创建自定义类型方面，原生的引用类型也采用这种模式。所有的原生引用类型（Array、Object、String等等）都在其构造函数的原型上定义了方法。例如：

- Array.prototype 中可以找到 sort() 方法；
- 在String.prototype 中可以找到 subString() 方法；

> 尽管可以这么做，但是不推荐在产品化的程序中修改原生对象的原型。因为如果某个实现中缺少某个方法，就在原生对象的原型中添加这个方法，那么当另一个支持该方法的实现中运行代码时，就有可能会导致命名冲突。也可能会意外重写该方法。

#### 6.2.3.6 原型对象的问题

- 因为它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下取值都相同，会带来一定的不便；
- 原型最大的问题就是由其共享的本性所导致的；

```javascript
function Person() {
}

Person.prototype = {
  constructor: Person,
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  friends: ['Shelby', 'Court'],
  sayName: function() {
    console.log(this.name);
  }
};

let p1 = new Person();
let p2 = new Person();

p1.friends.push('Van');

console.log(p1.friends); // 'Shelby, Court, Van'
console.log(p2.friends); // 'Shelby, Court, Van'，但实际开发中，我们希望的是只改变p1中的friends属性，而p2的friends不应改变
console.log(p1.friends === p2.friends); // true
```

这个问题正是我们很少看到有人单独使用原型模式的原因所在。

### 6.2.4 组合使用构造函数模式和原型模式

创建自定义类型的最常见方式，就是组合使用构造函数模式和原型模式。

- 构造函数模式用于定义实例属性；
- 原型模式用于定义方法和共享的属性；
- 每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度的节省了内存；
- 此外，这种混合模式还支持向构造函数传递参数；

```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.friends = ['Shelby', 'Court'];
}

Person.prototype = {
  constructor: Person,
  sayName: function() {
    console.log(this.name);
  }  
};

let p1 = new Person('Nicholas', 29, 'Software Engineer');
let p2 = new Person('Greg', 27, 'Doctor');

p1.friends.push('Van');
console.log(p1.friends); // 'Shelby, Court, Van'
console.log(p2.friends); // 'Shelby, Court'
console.log(p1.friends === p2.friends); // false
console.log(p1.sayName === p2.sayName); // true
```

这种混合模式是目前在ECMAScript中使用最广泛、认同度最高的一种创建自定义类型的方法。可以说是默认模式了。

### 6.2.5 动态原型模式

动态原型模式的将所有信息都封装在构造函数中，而通过在构造函数中初始化原型（仅在有必要情况下），又保持了同时视同构造函数和原型的优点。

```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  
  if (typeof this.sayName !== 'function') {
      // 只有在初次调用构造函数时才会执行，之后原型已经初始化完了就不需要再作什么修改了。
      Person.prototype.sayName = function() {
        console.log(this.name);
      }
  }
}

let p1 = new Person('Nicholas', 29, 'Software Engineer');
p1.sayName();
```

这个模式创建的对象，还可以使用 instanceof 操作符确定它的类型。

### 6.2.6 寄生构造函数模式

在前几种模式都不适用的情况下，可以使用寄生构造函数模式。

```javascript
function Person(name, age ,job) {
  let obj = new Object();
  
  obj.name = name;
  obj.age = age;
  obj.job = job;
  obj.sayName = function() {
    alert(this.name);
  };
  
  return obj;
}

let p1 = new Person('Nicholas', 29, 'Software Engineer');
p1.sayName(); // 'Nicholas'
```

这个模式可以在特殊的情况下用来为对象创建构造函数。加入我们想创建一个具有额外方法的特殊数组，但是由于不能直接修改 Array 构造函数，因此可以使用这种模式：

```javascript
function SpecialArray() {
  let values = new Array();
  
  values.push.apply(values, arguments);
  
  values.toPipedString = function() {
    return this.join('|');
  };
  
  return values;
}

let colors = new SpecialArray('red', 'blue', 'green');
console.log(colors.toPipedString()); // 'red|blue|green'
```

关于寄生构造函数模式，需要说明：返回的对象与构造函数或者与构造函数的原型属性之间没有关系；也就是说，构造函数返回的对象和在构造函数外部创建的对象没什么不一样。因此不能依赖 instanceof 操作符来确定对象类型。  

一般其他模式可用的情况下，不推荐使用寄生构造函数模式。

### 6.2.7 稳妥构造函数模式

稳妥对象，是由 道格拉·斯克罗克福德（Douglas Crockford）发明的。所谓稳妥对象：指的是没有公共属性，而且其方法也不引用this对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用this和new），或者在防止数据被其他应用程序（如Mashup程序）改动时使用。   

稳妥构造函数遵循与寄生构造函数类似的形式，但有两点不同：

- 新创建对象的实例方法不引用this；
- 不使用new操作符调用构造函数；

```javascript
function Person(name, age ,job) {
  let obj = new Object();
  
  obj.name = name;
  obj.age = age;
  obj.job = job;
  obj.sayName = function() {
    alert(name);
  };
  
  return obj;
}

let p1 = Person('Nicholas', 29, 'Software Engineer');
p1.sayName(); // 'Nicholas'
```

## 6.3 继承

许多OO语言都支持两种继承方式： **接口继承** 和 **实现继承**。   
接口继承只继承方法签名，而实现继承则继承实际的方法。   
由于函数没有签名，在ECMAScript中无法实现接口继承。ECMAScript只支持实现继承，而实现继承主要依靠原型链来实现。

### 6.3.1 原型链

什么是原型链：假如我们让原型对象等于另一个类型的实例，此时原型对象将包含一个指向另一个原型的指针，同时另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条，所谓的原型链。

```javascript
function SuperType() {
  this.property= true;
}

SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
    this.subProperty = false; 
}

// 继承了 SuperType 实例
SuperType.prototype = new SuperType(); 
SuperType.prototype.getSubValue = function() {
  return this.subProperty;
};

let instance = new SubType();
console.log(instance.getSuperValue()); // true
console.log(instance.constructor === SuperType); // true，这是因为原来SubType.prototype 中的 constructor 被重写指向了 SuperType。
```

继承实现的本质就是重写原型对象，代之以一个新类型的实例。

#### 6.3.1.1 默认的原型

所有引用类型默认都继承了 Object，而这个继承也是通过原型链实现的。记住：所有函数的默认原型都是 Object 的实例，因此默认原型都会包含一个内部指针指向 Object.prototype，这也正是所有自定义类型都会继承 toString()、valueOf() 等默认方法的根本原因。

总之，SubType 继承了 SuperType，而SuperType 继承了 Object，当调用 instance.toString() 方法实际上就是调用了保存在 Object.prototype 中的那个方法。

#### 6.3.1.2 确定原型和实例的关系

可以使用两种方式来确定原型和实例之间关系：

- instanceof 操作符，只要拿这个操作符来测试实例与原型链中出现过的构造函数，结果就会返回 true：

```javascript
console.log(instance instanceof Object);    // true
console.log(instance instanceof SuperType); // true
console.log(instance instanceof SubType);   // true
```

由于原型链的关系，我们可以说是 instance 是 Object 、 SuperType 或 SubType 中任何一类型的实例

- isPrototypeOf() 方法，同样也是只要原型链中出现过的原型，都可以说是该原型链派生的实例的原型，因此结果也会返回 true：

```javascript
console.log(Object.prototype.isPrototypeOf(instance));      // true
console.log(SuperType.prototype.isPrototypeOf(instance));   // true
console.log(SubType.prototype.isPrototypeOf(instance));     // true
```

#### 6.3.1.3 谨慎地定义方法

子类型有时候需要覆盖超类型中的某个方法，或者需要添加超类型中不存在的某个方法，这时切记： **给原型添加放的代码一定一定一定要放在替换原型的语句之后！！！**

```javascript
function SuperType() {
  this.property= true;
}

SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
    this.subProperty = false; 
}

// 继承了 SuperType 实例
SuperType.prototype = new SuperType();

// 添加新方法
SuperType.prototype.getSubValue = function() {
  return this.subProperty;
};

// 重写超类型中的方法
SuperType.prototype.getSuperValue = function() {
    return false;
};

let instance = new SubType();
console.log(instance.getSuperValue()); // false
```

另一点需要注意的是， **在通过原型链实现继承时，不能使用对象字面量创建原型方法！！！**，因为这样就会重写原型链：

```javascript
function SuperType() {
  this.property= true;
}

SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
    this.subProperty = false;
}

// 继承了 SuperType 实例
SuperType.prototype = new SuperType();

// 添加新方法
SuperType.prototype = {
    getSubValue: function() {
      return this.subProperty;
    },
    getSuperValue: function() {
        return false;
    }
};

let instance = new SubType();
console.log(instance.getSuperValue()); // error
```

#### 6.3.1.4 原型链的问题

- 主要问题来自包含引用类型的原型。

```javascript
function SuperType() {
  this.colors = ['red', 'blue', 'blue'];
}

function SubType() {
}

SubType.prototype = new SuperType();

let instance1 = new SubType();
instance1.colors.push('black');
console.log(instance1.colors); // 'red, blue, green, black'

let instance2 = new SubType();
console.log(instance2.colors); // 'red, blue, green, black'
```

按照之前了解的知识，应该是 instance1 和 instance2 两个实例分别有自己的数组 colors 属性，即使 SubType.prototype 变成了 SuperType 的一个实例，它也拥有与一个它自己的 colors 属性，但是结果却是 SubType 的所有实例都会共享这一个 colors 属性。

- 在创建子类型的实例时，不能向超类型的构造函数中传递参数。就是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。

综上所述两个问题，实践中我们很少会单独使用原型链。

### 6.3.2 借用构造函数

为解决原型中包含引用类型值所带来的问题，开发人员使用一种叫做 **借用构造函数** 的技术（有时候也叫做伪造对象或经典继承）。即：在子类型构造函数的内部调用超类型构造函数。

```javascript
function SuperType() {
  this.colors = ['red', 'blue', 'green'];
}

function SubType() {
  SuperType.call(this);
}

let instance1 = new SubType();
instance1.colors.push('black');
console.log(instance1.colors); // 'red, blue, green, black'

let instance2 = new SubType();
console.log(instance2.colors); // 'red, blue, green'
```

通过 call() 方法（或 apply() 方法），在新创建的 SubType 实例的环境下调用了 SuperType 构造函数。这样就会在新 SubType 对象上 执行 SuperType() 函数中定义的所有初始化对象，最终 SubType 的每个实例就会都具有自己的 colors 属性副本。

#### 6.3.2.1 传递参数

相对于原型链而言，借用构造函数有一大优势，就是可以在子类型构造函数中向超类型构造函数传递参数。

```javascript
function SuperType(name) {
  this.name = name;
}

function SubType(age) {
  SuperType.call(this, 'Nicholas');
  
  this.age = age;
}

let instance = new SubType(29);
console.log(instance.name); // 'Nicholas'
console.log(instance.age);  // 29
```

#### 6.3.2.2 借用构造函数的问题

如果仅仅是借用构造函数，那么就无法避险构造函数模式存在的问题——方法都在构造函数中定义，因此无法函数复用了。而且在超类型的原型中定义的方法，对于子类型而言是不可见的，结果所有类型都只能使用构造函数模式。因此，借用构造函数的方式也是很少单独使用的。

### 6.3.3 组合继承

有时候也叫伪经典继承，指的是将原型链和借用构造函数的技术组合到一块。

其原理为：使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}

SuperType.prototype.sayName = function() {
  alert(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);
  
  this.age = age;
}

SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
  alert(this.age);
};

let instance1 = new SubType('Nicholas', 29);
instance1.colors.push('black');  // 'red, blue, green, black'
instance1.sayName();             // 'Nicholas'
instance1.sayAge();              // 29

let instance2 = new SubType('Greg', 27);
console.log(instance2.colors);      // 'red, blue, green'
instance2.sayName();                // 'Greg'
instance2.sayAge();                 // 27
```

组合继承避免了原型链和借用构造函数的缺陷，而且， instanceof 和 isPrototypeOf() 也能够用于识别基于组合继承创建的对象。

### 6.3.4 原型式继承

道格拉斯·克罗克福德在2006年写的题为 Prototype Inheritance in JavaScript 中介绍了一种实现继承的方法，该方法并没有严格意义上的构造函数么事借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。

object() 本质上讲就是对传入其中的对象进行了一次浅拷贝。

```javascript
function object(obj) {
  function F() {}       // 临时性构造函数
  F.prototype = obj;    // 将传入的对象作为临时构造函数的原型
  return new F();       // 返回一个临时构造函数的实例
}

let person = {
  name: 'Nicholas',
  friends: ['Shelby', 'Court', 'Van']  
};

let anotherPerson = object(person);
anotherPerson.name = 'Greg';
anotherPerson.friends.push('Rob');

let yetAnotherPerson = object(person);
yetAnotherPerson.name = 'Linda';
yetAnotherPerson.friends.push('Barbie');

console.log(person.friends); // 'Shelby, Court, Van, Rob, Barbie'
```

ES5中新增了 **Object.create()** 方法规范花了原型式继承。该方法接收两个参数：新对象原型的对象 和 （可选）为新对象定义额外属性的对象。

```javascript
let person = {
  name: 'Nicholas',
  friends: ['Shelby', 'Court', 'Van']  
};

let anotherPerson = Object.create(person, {
    // 这里写法同 Object.defineProperties() 方法的第二个参数
    name: {
        value: 'Greg'
    }
});
anotherPerson.friends.push('Rob');

let yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = 'Linda';
yetAnotherPerson.friends.push('Barbie');

console.log(person.friends); // 'Shelby, Court, Van, Rob, Barbie'
```

### 6.3.5 寄生式继承

寄生式继承的思路与寄生构造函数和工厂函数类似，即创建一个仅用于封装继承过程的函数。

```javascript
function createAnother(original) {
  let clone = object(original);     // 通过调用函数创建一个新对象
  clone.sayHi = function() {        // 以某种方式来增强这个对象
    alert('Hi');
  };
  
  return clone;                     // 返回这个对象
}

let person = {
  name: 'Nicholas',
  friends: ['Shelby', 'Court', 'Van']  
};
let anotherPerson = createAnother(person);
anotherPerson.sayHi(); // 'Hi'
```

### 6.3.6 寄生组合式继承

组合继承虽然是常用继承模式，但是也有不足，都会调用两次超类型构造函数：一次在创建子类型原型的时候，另一次是在子类型构造函数内部。

```javascript
function SuperType(name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}

SuperType.prototype.sayName = function() {
  alert(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);           // 第二次调用 SuperType()
  
  this.age = age;
}

SubType.prototype = new SuperType();    // 第一次调用 SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
  alert(this.age);
};
```

而寄生组合式继承，就是通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。基本思路：不必为了指定子类型的原型而调用超类型的构造函数，我们需要的只是超类型的一个副本。

```javascript
function inheritPrototype(subType, superType) {
  let prototype = object(superType.prototype);   // 创建对象，创建超类型原型的一个副本。
  prototype.constructor = subType;              // 增强对象，为副本添加 constructor 属性，从而弥补因重写原型而失去的默认的 constructor 属性。
  subType.prototype = prototype;                // 指定对象，将新创建的副本复制给子类型的原型。
}

function SuperType(name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}

SuperType.prototype.sayName = function() {
  alert(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);           // 第二次调用 SuperType()
  
  this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function() {
  alert(this.age);
}
```

此处高效体现在于它只调用了一次 SuperType 构造函数，并且因此避免了在 SubType.prototype 上面创建不必要的、多余的属性，同时原型链还能保持不变。除此之外， instanceof 和 isPrototypeOf() 还能正常使用。

目前，寄生组合式继承 是引用类型最理想的继承范式。

## 6.4 总结 & FAQ

### 6.4.1 总结

1. ECMAScript支持面向对象编程，但是不支持类或者接口。在没有类的情况下，我们可以采用以下模式创建对象：

- 工厂模式：使用简单函数创建对象，为对象添加属性和方法，然后返回对象。
- 构造函数模式：可以自定义引用类型，使用 new 操作符新建实例化对象。但是其每个成员都无法得到复用，包括函数。
- 原型模式：使用构造函数的 prototype 属性来指定那些应该共享的属性和方法。**组合使用构造函数模式和原型模式** ，使用构造函数定义实例属性，使用原型定义共享的属性和方法。

2. JavaScript通过 原型链 实现继承。原型链的构建是通过将一个类型的实例赋值给另一个构造函数的原型实现的。原型链的问题在于对象实例共享所有继承的属性和方法，而通过 借用构造函数 来解决：在子类型构造函数的内部调用超类型构造函数。除此之外，还存在以下可供选择的继承模式：

- 原型式继承：其本质是执行对给定对象的浅复制。
- 寄生式继承：与原型式继承很相似，基于某个对象或某些信息创建一个对象，然后增强对象，最后返回对象。
- 寄生组合式继承：为了解决组合继承模式由于多次调用超类型构造函数而导致的低效率问题，集寄生式继承和组合继承的优点，是目前实现类型继承的最有效方式。

3. 原型、构造函数和实例的关系：

- 实例.__proto__ === 原型
- 原型.constructor === 构造函数
- 构造函数.prototype === 原型
- 实例.constructor === 构造函数
![关系图](https://user-gold-cdn.xitu.io/2019/2/14/168e9d9b940c4c6f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 6.4.2 FAQ

1. 实例化对象时，new操作符都干了些什么事？
2. 6.3.4原型式继承中 object 函数里面执行下来为什么是浅复制？
