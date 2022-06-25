# 第一篇：面向对象编程

## 类式继承

> 类的原型对象的作用就是为类的原型添加共有方法，但是类不能访问这些属性，
>
> 必须通过原型prototype来访问，而当我们实例化一个父类时，
>
> 新创建的对象赋值了父类原型对象上的属性和方法，并将原型\_proto\_指向了父类的原型对象

```js
//类式继承
  function SuperClass(){
      this.superValue = true
      this.books = ['a','b']
    }
//    为父类添加共用方法
  SuperClass.prototype.getSuperValue = function (){
      return this.superValue;
    }
//    声明子类
  function SubClass(){
    this.subValue = false
  }
//    继承父类
//    这里是直接把父类的一个实例当做了子类的prototype
  SubClass.prototype = new SuperClass();
  SubClass.prototype.getSubValue = function (){
    return this.subValue;
  }
```

类式继承是有两个严重的问题的

1. 因为类式继承是通过prototype继承父类，导致子类的修改会影响到父类
2. 在继承父类时，由于使用了SubClass.prototype = new SuperClass();所以是无法对父类进行初始化的，也无法对父类传参

关于上面问题的测试

```js
let superA = new SuperClass()
console.log(superA.books)
let subA = new SubClass()
console.log(subA.books)
let subB = new SubClass()
console.log(subB.books)
//    输出正常，都只有a,b
//如果A向里添加了数据，这是B也会收到影响，因为A和B的prototype指向的是一块内存
subA.books.push('c')
console.log(subA.books)
console.log(subB.books)
```

## 构造函数继承

为了解决子类无法调用父类的构造函数的问题，提出了复用父类的构造函数的继承方法

```js
// 声明父类
    function SuperClass(id) {
      //引用类型共有属性
      this.books = ['a','b','c'];
      //值类型共有属性
      this.id = id ;
    }
//    父类声明原型方法
    SuperClass.prototype.showBooks = function (){
      console.log(this.books)
    }
//    声明子类
function Subclass(id){
  //继承父类
  SuperClass.call(this,id);
}
```

这种构造函数继承方式使用call调用了父类的构造函数，这样可以很方便的对父类进行合理化

测试

```js
//第一个子类实例
let instance1 = new Subclass()
//第二个子类实例
let instance2 = new Subclass()

//实现子类的实例是否会互相影响
instance1.books.push('d')

console.log(instance1.books)
console.log(instance2.books)
//结果是不会互相影响
//下面这个会报错，因为是通过子类调用父类的构造函数实现的继承，在子类的原型上并没有父类的函数
instance1.showBooks()
```

## 组合继承

> 为了解决类式继承不能实例化父类，构造函数继承不能拿到父类的原型的问题，提出了组合继承，将两个继承方式的组合在一起

```js
//  声明父类
    function SuperClass(name) {
    //  值类型共有属性
      this.name = name;
    //  引用类型共有属性
      this.books = ['a','b','c'];
    }

//    父类原型共有方法
    SuperClass.prototype.getName = function (){
      console.log(this.name);
    }

//    声明子类,这里是构造函数式，用来获取构造函数
    function SubClass(name,time){
    //  构造函数式继承父类name属性
      SuperClass.call(this,name);
    //  子类中新增共有属性
      this.time = time;
    }

//    类式继承，子类原型继承父类，用来获取父类的prototype
    SubClass.prototype = new SuperClass();

//    子类原型方法
    SubClass.prototype.getTime = function (){
      console.log(this.time)
    }
```

测试一下

```js
    let instance1 = new SubClass('js book',2014);
    instance1.books.push('test');
    console.log(instance1.books);
    instance1.getName();
    instance1.getTime();

    let instance2 = new SubClass('js book2',2016);
    instance2.books.push('test2');
    console.log(instance2.books);
    instance2.getName();
    instance2.getTime();
```

从结果可以看出，两个实例之间是相互独立的，并且也拿到了父类的prototype，同样的，在子类的构造函数中也实现了对子类的重新构造

## 原型式继承

> 但是，组合继承虽然大致解决了继承的核心问题，但是还是有一些问题，
>
> 比如在使用构造函数继承时执行了一次父类的构造函数，而在实现子类原型的类式继承时，又调用了一边父类的构造函数。

```js
//原型式继承
   function inheritObject(o){
    //声明一个过渡函数对象
      function F(){}
    //过渡对象的原型继承父对象
      F.prototype = o;
    //返回过渡对象的一个实例，该对象的原型继承了父对象
      return new F();
    }
```

原型继承与类式继承相似，都是通过prototype进行继承，仍然存在类式继承的问题，但是它不再通过子类的prototype进行继承，而是通过一个中间对象，让他去继承父类，然后返回一个new F（）。

类式继承是让子类的prototype等于父类的一个实例，使用时就将子类new实例化如何赋值给实例对象。而在原型式继承中，不需要去改变子类的prototype，只要去改变一个过渡对象的prototype，让他的prototype等于父类的实例，然后让过渡对象实例化，把这个对象返回给真正的对象。这个流程有点像一个工厂，这个工厂的目的就是创造子类的对象

## 寄生式继承

> ```JS
> 寄生继承就是基于原型继承，在使用过渡对象继承返回一个新的实例对象之后，可以对这个对象进行增强添加新的属性和方法。
> ```

```JS
//声明基对象
let book = {
  name: 'js book',
  alikeBook:['css book','html book']
};
function createBook(obj){
  //通过原型继承方式创建新对象
  let o = new inheritObject(obj);
  //拓展新对象
  o.getName = function (){
    console.log(name)
  }
  return o;
}
```

## 寄生组合式继承（继承的终极形态）

> 根据组合继承的方式，融合了寄生式继承和构造函数继承的优点

```JS
//传递参数：subClass 子类
    //传递参数：superClass 父类
    // 原型式继承
    function inheritObject(o){
      //声明一个过渡函数对象
      function F(){}
      //过渡对象的原型继承父对象
      F.prototype = o;
      //返回过渡对象的一个实例，该对象的原型继承了父对象
      return new F();
    }
    //定义
    function inheritPrototype(subClass,superClass){
    //  复制一份父类的原型副本保存在变量中
      let p = inheritObject(superClass.prototype);
    //  因为在一开始用寄生式继承时，用的是父类的构造函数，
    //  如果不修改子类的构造函数的话，就只能修改父类的构造函数
    //  修正因为重写子类原型导致的constructor属性被修改
      p.constructor = subClass;
      //  设置子类的原型
      subClass.prototype = p;
    }
```

测试

```JS
    //  声明父类
    function SuperClass(name) {
      //  值类型共有属性
      this.name = name;
      //  引用类型共有属性
      this.books = ['a','b','c'];
    }

    //    父类原型共有方法
    SuperClass.prototype.getName = function (){
      console.log(this.name);
    }

    //    声明子类,这里是构造函数式，用来获取构造函数
    function SubClass(name,time){
      //  构造函数式继承父类name属性
      SuperClass.call(this,name);
      //  子类中新增共有属性
      this.time = time;
    }

    //    寄生式组合式继承
    inheritPrototype(SubClass,SuperClass)

    //    子类原型方法
    SubClass.prototype.getTime = function (){
      console.log(this.time)
    }


    //    测试代码
    let instance1 = new SubClass('js book',2014);
    instance1.books.push('test');
    console.log(instance1.books);
    instance1.getName();
    instance1.getTime();

    let instance2 = new SubClass('js book2',2016);
    instance2.books.push('test2');
    console.log(instance2.books);
    instance2.getName();
    instance2.getTime();
```

## 多继承

### 什么是单继承？

```JS
  //单继承：其实就是把对象的属性一个一个全部复制给新对象
  let extend = function (target,source){
  //  遍历对象中属性
    for(let property in source){
    //  将源对象中的属性复制到目标对象中
      target[property] = source[property];
    }
    return target
  }
```

### 多继承就是多次的单继承

```JS
  let mix = function (){
    let i = 1;
    let len = arguments.length
    let target = arguments[0]
    let arg = undefined;
  //  遍历被继承的对象
    for(;i<len;i++){
    //  缓存当前的对象
      arg = arguments[i];
      for(let property in arg){
        target[property] = arg[property];
      }
    }
  }
```

## 多态

> 多态: 因为在JS中，函数的参数是以数组的形式存在的，所以在编译时是不会产生多态的可以在函数体内，根据参数进行不同的操作，形成类似多态的功能

```JS
    function add(){
    //  获取参数
      let arg = arguments;
      let len = arg.length;
      switch (len){
      //  如果没有参数
          case 0:
            return 10;
          //  如果只有一个参数
          case 1:
            return 10 + arg[0];
      }
    }
```

# 第二篇：结构性设计模式

## 简单工厂模式（神奇的魔术师）

> 简单工厂模式：又叫静态工厂方法，由一个工厂对象决定创建某一种产品对象类的实例

下面这是一个创建运动类的工厂，这个函数的目的就是返回一个对象，用户不需要去关注到底怎么实现的，只需要向工厂提需求，就可以得到对应的对象。

```JS
function createSport(name){
    let o = new Object();
    o.intro = name;
    switch (name) {
        case '篮球':
            o.getMember = function (){
                console.log('篮球要五个人');
            }
            o.getBallSize = function () {
                console.log('篮球很大');
            }
            break
        case '足球':
            o.getMember = function (){
                console.log('足球要11个人');
            }
            o.getBallSize = function () {
                console.log('足球一般大');
            }
            break;
        case '乒乓球':
            o.getMember = function (){
                console.log('乒乓球要2个人');
            }
            o.getBallSize = function () {
                console.log('乒乓球很小');
            }
            break;
    }
    return o;
}
```

测试

```JS
//提供了简单的数据
let sportList = ['乒乓球','足球','篮球'];
//对数据进行对象的创建
for (let i = 0; i < sportList.length; i++) {
    let o = createSport(sportList[i]);
    o.getBallSize();
    o.getMember();
}
```

> 总结：主要用来生成同一类对象,在团队项目开放中，对全局变量的限制非常大，所以我们要尽量少的创建全局变量，对于同一类对在不同需求中的重复限制，很多时候不需要重复创建，可以提取他们的共用的资源成为一个工厂。

## 工厂方法模式（给我一张名片）

> ```
> 工厂方式模式：通过对产品类的抽象使其创建业务主要负责用于创建类产品的实例
> ```

### 安全模式

> 为了防止用户在使用时，不知道到底要不要用new ，干脆使用安全模式保证不管用不用new都可以安全运行

```JS
//    安全模式下的构造函数
let  Factory = function (type,content) {
    if(this instanceof Factory){
        return new this[type](content);
    }else {
        return new Factory(type,content)
    }
}
```

### 工厂原型

```js
//    工厂原型中设置创建所有类型数据对象的基类
Factory.prototype = {
    Java: function (content){
        console.log('Java',content)
    },
    Python: function (content){
        console.log('Python',content)
    },
    JS: function (content){
        console.log('JS',content)
    }
}
```

测试

```js
let data = [
    {type:'Java',content:'java是世界上最好的语言'},
    {type:'Python',content:'Python是世界上最好的语言'},
    {type:'JS',content:'JS是世界上最好的语言'}
]

for (let i = 0; i < data.length ; i++) {
    //使用new
    let x = new Factory(data[i].type,data[i].content);
    //不使用new，这得益于安全模式
    Factory(data[i].type,data[i].content);
}
```

> 总结：对于创建多类对象，前面学过的简单工厂模式就不太适用了，工厂方法模式可以让我们轻松创建出多个类的实例对象，这样工厂方法对象在创建对象的方式也避免了使用者与对象类的耦合，用户不需要关系创建对象的具体类，只需调用工厂方法即可。

## 抽象工厂模式（出现的都是幻觉）

> 抽象工厂模式：通过对类的工厂抽象使其业务用于产品类簇的创建，而不负责创建某一类产品的实例

### 抽象类

因为在一些大型应用中，总有一些子类要去继承另一些父类，这些父类会定义一些必须要实现的方法，却没有具体的实现

```JS
//抽象工厂方法
let VehicleFactory = function (subType,superType) {
  if (typeof VehicleFactory[superType] === 'function') {
    //缓存类
    function F () {}
    //继承父类属性和方法
    F.prototype = new VehicleFactory[superType]();
    //将子类constructor 指向子类
    subType.constructor = subType;
    //子类原型继承父类
    subType.prototype = new F();
  } else {
    throw new Error('未创建该抽象类');
  }
}
  //  小汽车抽象类
  VehicleFactory.Car = function (){
    this.type = 'car'
  }
  VehicleFactory.Car.prototype = {
    getPrice: function (){
      return new Error('抽象方法不能调用')
    },
    getSpeed: function (){
      return new Error('抽象方法不能调用')
    }
  }
```

抽象类的实现

```JS
//  宝马汽车子类
  let BMW = function (price,speed){
    this.price = price;
    this.speed = speed;
  }
//  抽象工厂实现对Car抽象类的继承
  VehicleFactory(BMW,'Car')
  //重写抽象父类的函数
  BMW.prototype.getPrice = function (){
    return this.price
  }
  BMW.prototype.getSpeed = function (){
    return this.speed
  }
```

测试

```JS
  let myCar = new BMW(100000,23333);
  console.log(myCar.getPrice());
  console.log(myCar.getSpeed());

```

> 抽象工厂模式定义的是类的结构，但是由于JS不支持抽象化创建于虚拟方法，所以在JS中抽象方法用的并不是很广泛

## 建筑者模式（分即使合）

> 建造者模式：把多个工厂模式结合在一起，将一个复杂的对象相互分离，用不同的类去表示，然后再用一个总的类来包括

创建多个原型类

```JS
//  创建一位人类
let Human = function (param) {
    //  技能
    this.skill = param ? param.skill : '保密'
    //  兴趣爱好
    this.hobby = param ? param.hobby : '保密'
}

//  人类原型方法
Human.prototype = {
    getSkill : function (){
        return this.skill
    },
    getHobby : function (){
        return this.hobby
    }
}

//  实例化姓名类
let Named = function (name) {
  let that = this;
//  构造器
//  构造函数解析姓名中的姓与名
  (function (name,that){
    that.wholeName = name;
    if(name.indexOf(' ') > -1){
      that.FirstName = name.slice(0,name.indexOf(' '));
      that.secondName = name.slice(name.indexOf(' '))
    }
  })(name,that)
}

//  实例化职位类
let Work = function (work) {
  let that = this;
//  构造器
//  构造函数中通过传入的职位特征来设置相应职位以及描述
  (function (work,that){
    switch (work){
      case 'code':
        that.work = '工程师'
        that.workDescript = '每天沉迷编程'
      case 'UI':
        that.work = '设计师'
        that.workDescript = '每天沉迷画画'
    }
  })(work,that)
}

//更换期望的职位
Work.prototype.changeWork = function (work){
      this.work = work
}

//添加
Work.prototype.changeDescript = function (setence){
  this.workDescript = setence
}
```

创建一个建造者类

```JS
let Person = function (name,work){
    //创建应聘者缓存对象
    let _person = new Human()
    //创建应聘者姓名
    _person.name = new Named(name);
    //创建应聘者期望职位
    _person.work = new Work(work)
    //将创建的应聘者对象返回
    return _person
}
```

测试一下

```JS
//创建实例
let person = new Person('yi jiamu','code')
console.log(person.getSkill())
console.log(person.getHobby())
console.log(person.name)
console.log(person.work)
console.log(person.skill)
```

> 建筑者模式与之前的工厂模式不同，工厂模式是只在乎最后得到的对象，而建筑者模式关注的是对象的创建过程，我们通常会将创建对象的类模块化，然后组合在一起得到一个完整的个体。

## 原型模式（语言之魂）

### 非原型模式继承

> 原型模式：用原型实例指向创建对象的类，使用创建新的对象的类共享原型独享的属性以及方法

```JS
//    图片轮播类
    let LoopImages = function (imgArr,container){
      this.imagesArray = imgArr;
      this.container = container;
      this.createImage = function (){}
      this.changeImage = function (){}
    }
//  上下滑动切换类
    let SlideLoopImg = function (imgArr,container){
    //  构造函数继承图片轮播类
    //  这里调用的LoopImages的构造函数
      LoopImages.call(this,imgArr,container);
      this.changeImage = function (){
        console.log('SlideLoopImg')
      }
    }
//  创建一个实例
    let fadeImg = new SlideLoopImg([
      '1',
      '2',
      '3'
    ],'Slide')
    fadeImg.changeImage()
```

分析一下这个代码，实例调用了构造函数，在构造类函数中，重新调用了父类的构造函数，并且增强了构造函数，这就出现了当父类的构造函数极其复杂时，导致性能较差。

```js
    //    图片轮播类
    let newLoopImages = function (imgArr,container){
      this.imagesArray = imgArr;
      this.container = container;
    }
    newLoopImages.prototype = {
      createImage : function (){
        console.log('newLoopImg')
      },
      changeImage : function (){
        console.log('changeLoopImg')
      }
    }
    //  上下滑动切换类
    let newSlideLoopImg = function (imgArr,container){
      //  构造函数继承图片轮播类
      newLoopImages.call(this,imgArr,container);
    }
    newSlideLoopImg.prototype = new newLoopImages()
    //  创建一个实例
    let newfadeImg = new newSlideLoopImg([
      '1',
      '2',
      '3'
    ],'Slide')
    newfadeImg.changeImage()
```

将复杂的函数放在原型中，使得继承父类的子类共用一个函数，这样可以提高性能，继承的方式可以选择组合继承，可以选择更加的寄生组合式继承。

### 原型继承

> 但是原型模式更多的是用在对对象的创建上，比如创建一个实例对象的构造函数比较复杂，或者耗时比较长，或者通过多个对象来实现，这时我们最好不要用new关键字来复制这些基类，应该通过对这些对象属性和方法进行复制来创建，这就是原型模式的思想。

原型继承

```JS
  /*多对象的原型继承*/
    function prototypeExtend(){
      let F = function (){};
      let args = arguments;
      let i = 0;
      let len = args.length;
      for(;i<len;i++){
        for(let j in args[i]){
          F.prototype[j] = args[i][j]
        }
      }
      return new F()
    }
```

实现一下

```JS
//    实现
    let penguin = prototypeExtend({
      speed:20,
        swim: function (){
        console.log('游泳')
      }
    },{
      run: function (){
        console.log('跑步')
      }
    },{
      jump: function (){
        console.log('跳跃')
      }
    })
    console.log(penguin)
    penguin.swim()
    penguin.run()
    penguin.jump()
```

> 原型模式可以让多个对象分享同一个原型对象的属性和方法，这也是一种继承方式，这种方式是不需要创建的，而是将原型对象分享给那些继承的对象，当然有时候也需要让某个对象独立拥有一份原型对象，此时就需要对原型对象进行复制

## 单例模式（一个人的寂寞）

> 单例模式：有称为单体模式，是只被允许实例化一次的对象类，有时我们使用这个对象来实现一个命名空间，井井有条的管理对象上的属性和方法

### 命名空间

```js
//用于命名空间的管理，防止出现命名重复的问题
let YJM = {
    g : function (id){
        return document.getElementById(id);
    },
    css : function (id,key,value){
        this.g(id).style[key] = value;
    }
}
//用于管理代码中的各个模块,这样代码非常简洁
// baidu.dom.addClass
// baidu.dom.append
// baidu.event.stopPropagation
```

### 静态变量

```js
//    单例模式下的应用：静态变量，使用闭包实现
let Conf = (function (){
    let conf = {
        Max_num : 100,
        Min_num : 1
    }
    return {
        get : function (name){
            return conf[name] ? conf[name] : null
        }
    }
})();

let count = Conf.get('Min_num')
console.log(count)
```

### 惰性单例

```js
//    惰性单例模式
let LazySingle = (function (){
    let _instance = null;
    //单例
    function Single () {
        //  这里定义私有属性和方法
        return {
            publicMethods:function (){},
            publicProperty:'1.0'
        }
    }

    console.log('create')
    return function (){
        //  如果为创建单例将创建单例
        //  这里会out两次，因为在下面会调用一次就会运行一次这个函数
        //  但是只有第一次创建才会实例化，才会调用create
        console.log('out')
        if(!_instance){
            _instance = Single();
        }
        return _instance
    }
})()
console.log(
    LazySingle().publicProperty
)
console.log(
    LazySingle().publicProperty
)
```



# 第三篇：结构型设计模式

## 外观模式（套餐服务）

> 外观模式：为一组复杂的子系统接口提供一个更高级的统一接口，通过这个接口使得对子系统的访问更加容易，在JS中有时也会用于对底层结构兼容性进行统一封装。

外观模式实现

```js
//    外观模式实现
function addEvent (dom,type,fn) {
    //  对于支持DOM2级事件处理程序addEventListener 方法的浏览器
    if(dom.addEventListener){
        dom.addEventListener(type,fn,false)
        //  对于不支持addEventListener 方法但是支持attachEvent的浏览器
    }else if(dom.attachEvent){
        dom.attachEvent('on'+type,fn);
        //  对于都不支持的浏览器
    }else{
        dom['on'+type] = fn
    }
}
```

调用

```js
//    外观模式调用
let myInput = document.getElementById('myInput')
addEvent(myInput,'click',function (){
    console.log('我是点击事件')
})
addEvent(myInput,'move',function (){
    console.log('我是移动事件')
})
```

> 当一个复杂的系统提供一系列复杂的接口方法时，为系统的管理方便会造成接口的使用极其复杂，在多人的团队开发中，撰写的成本是很大的，通过外观模式，对接口的二次封装隐藏起复杂性，并简化其使用是一种很不错的实践。

## 适配器模式（水管弯弯）

> 适配器模式：将一个类（对象）的接口转化成另一个接口以满足用户的需求，使类（对象）之间的接口不兼容问题得到解决

## 代理模式（牛郎织女）

> 代理模式：由于一个对象不能直接引用另一个对象，所以需要通过代理对象在这两个对象之间起中介的作用。
>
> 代理对象可以完全解决被代理对象与外部对象之间的耦合问题，当然从被代理的页面角度来看是一种保护代理，然而从服务器角度看又是一种远程代理。

## 装饰者模式（房子装修）

> 装修者模式：在不改变原对象的基础上，通过对其包装拓展（添加属性或者方法）使其满足用户更复杂的要求。

其实就是通过一个函数，给定需要装饰的对象，和需要装饰的“装饰物”，这个函数会自动帮忙把装饰物加到对象上

```js
//定义装饰者模式函数
let decorator = function (input , fn ){
//  获取事件源
  var input = document.getElementById(input);
//  如果事件源已经绑定事件
  if(typeof input.onclick === 'function'){
  //  缓存事件源原来的事件
    let oldEvent = input.onclick;
    //为事件源定义新的事件
    input.onclick = function (){
    //    事件源原来的事件
      oldEvent()
      //执行新添加的事件
      fn()
    }
  }else{
    //如果没有给事件源绑定事件的话就直接把填加回调函数
    input.onclick = fn;
  }
}
```

实现

```JS
//实现函数
decorator('tel_input',function (){
  document.getElementById('tel').style.display = 'none'
})
```

> 关于适配器和装饰者的区别：
>
> 适配器是对对象进行重组，而装饰者只是在外部进行拓展，不触碰原本的功能

## 桥接模式（城市间的公寓）

> 桥接模式：在系统沿着多个维度变化的同时，又不增加其复杂度并达到解耦
>
> 将功能的逻辑和实现剥离开，达到解耦的效果

实现函数

```JS
 //这些都是实现层
//运动单元
function Speed(x,y){
    this.x = x;
    this.y = y;
}
Speed.prototype.run = function (){
    console.log('跑起来');
}
//着色单元
function Color(cl){
    this.color = cl;
}
Color.prototype.draw = function (){
    console.log('绘制色彩')
}
//变形单元
function Shape(sp){
    this.shape = sp;
}
Shape.prototype.change = function (){
    console.log('改变颜色');
}
//说话单元
function Speek(wd){
    this.word = wd;
}
Speek.prototype.say = function (){
    console.log('书写字体');
}
```

逻辑函数

```JS
  //下面的都是抽象层
//    创建一个球类，他可以运动，可以着色
function Ball(x,y,cl){
    this.speed = new Speed(x,y);
    this.color = new Color(cl);
}
Ball.prototype.init = function (){
    this.speed.run()
    this.color.draw()
}
//我们想创建一个人物类，他可以运动和说话
function Person(x,y,f){
    this.speed = new Speed(x,y);
    this.speek = new Speek(f);
}
Person.prototype.init = function (){
    this.speed.run()
    this.speek.say()
}
let p = new Person(10,12,16)
p.init()
```

> 桥接模式的最大特点就是将实现层和抽象层解耦分离，使两部分可以独立变化，由此可以看出桥接模式主要是对结构之间的结构。
>
> 实现实现层和抽象层分开处理，避免需求的改变造成对象内部的修改，体现了面对对象对拓展的开放和对修改的关闭原则

## 组合模式（超值午餐）

> 组合模式：又称之为部分-整体模式，将对象组合成树形结构以表示“部分整体”的层次结构，组合模式使得用户对单个对象和组合对象的使用有一致性

组合模式就是一层又一层的嵌套

在原生JS中，DOM树的创建是可以依赖于类与类之间的关系的，如在本例子中的各种子类，他们之间有add函数，该函数就是构建DOM树，其子类的add是可以嵌套使用的，所以可以像HTML一样，去构建虚拟DOM树。每个子类都有相同的属性和方法，但是属性相同但方法不相同，所以需要一个抽象父类,来为子类提供共享的属性和规定死的方法

定义一个顶层父类

```JS
//一个抽象父类,来为子类提供共享的属性和规定死的方法
let News = function (){
    //  子组件容器
    this.children = [];
    //  当前组件元素
    this.element = null;
}
News.prototype = {
    init : function () {
        throw new Error('请重写你的方法')
    },
    add : function () {
        throw new Error('请重写你的方法')
    },
    getElement : function () {
        throw new Error('请重写你的方法')
    },
}
```

定义继承方式

```js
//定义寄生组合式继承
function inheritObject(o){
    //声明一个过渡函数对象
    function F(){}
    //过渡对象的原型继承父对象
    F.prototype = o;
    //返回过渡对象的一个实例，该对象的原型继承了父对象
    return new F();
}
function inheritPrototype(subClass,superClass){
    //  复制一份父类的原型副本保存在变量中
    let p = inheritObject(superClass.prototype);
    //  因为在一开始用寄生式继承时，用的是父类的构造函数，
    //  如果不修改子类的构造函数的话，就只能修改父类的构造函数
    //  修正因为重写子类原型导致的constructor属性被修改
    p.constructor = subClass;
    //  设置子类的原型
    subClass.prototype = p;
}
```

定义一个容器类

```js
// 容器类构造函数
let Container = function (id,parent){
    //  构造函数继承父类
    News.call(this);
    //  模块id
    this.id = id;
    //  模块的父容器
    this.parent = parent;
    //  构建方法
    this.init()
}
//寄生式继承父类原型方法
inheritPrototype(Container,News)

//重写方法
//构建方法
Container.prototype.init = function (){
    this.element = document.createElement('ul');
    this.element.id = this.id;
    this.element.className = 'new-container';
}
//添加子元素方法
Container.prototype.add = function (child){
    //在子元素容器中插入子元素
    this.children.push(child);
    //  插入当前组件元素树中
    this.element.appendChild(child.getElement());
    return this;
}
Container.prototype.getElement = function (){
    return this.element;
}
//显示方法
Container.prototype.show = function (){
    this.parent.appendChild(this.element);
}
```

定义容器下的各个子类

```js
//子类成员：Item
let Item = function (classname){
    News.call(this);
    this.classname = classname || '';
    this.init();
}
inheritPrototype(Item,News);
Item.prototype.init = function (){
    this.element = document.createElement('il');
    this.element.className = this.classname;
}
//添加子元素方法
Item.prototype.add = function (child){
    //在子元素容器中插入子元素
    this.children.push(child);
    //  插入当前组件元素树中
    this.element.appendChild(child.getElement());
    return this;
}
Item.prototype.getElement = function (){
    return this.element;
}
//子类成员：NewGroup
let NewGroup = function (classname){
    News.call(this);
    this.classname = classname || '';
    this.init();
}
inheritPrototype(NewGroup,News);
NewGroup.prototype.init = function (){
    this.element = document.createElement('div');
    this.element.className = this.classname;
}
//添加子元素方法
NewGroup.prototype.add = function (child){
    //在子元素容器中插入子元素
    this.children.push(child);
    //  插入当前组件元素树中
    this.element.appendChild(child.getElement());
    return this;
}
NewGroup.prototype.getElement = function (){
    return this.element;
}

//子类成员：ImageNews
let ImageNews = function (url,href,classname){
    News.call(this);
    this.url = url || '';
    this.href = href || '#';
    this.classname = classname || '';
    this.init();
}
inheritPrototype(ImageNews,News);
ImageNews.prototype.init = function (){
    this.element = document.createElement('a');
    let img = new Image();
    img.src = this.url;
    this.element.appendChild(img);
    this.element.className = 'image-news' + this.classname;
    this.element.href = this.href;
}
//添加子元素方法
ImageNews.prototype.add = function (){}
ImageNews.prototype.getElement = function (){
    return this.element;
}

//子类成员：IconNews
let IconNews = function (text,href,type){
    News.call(this);
    this.text = text || '';
    this.href = href || '#';
    this.type = type || 'video';
    this.init();
}
inheritPrototype(IconNews,News);
IconNews.prototype.init = function (){
    this.element = document.createElement('a');
    this.element.innerHTML = this.text;
    this.element.href = this.href;
    this.element.className = 'icon' + this.type;
}
//添加子元素方法
IconNews.prototype.add = function (){}
IconNews.prototype.getElement = function (){
    return this.element;
}
//子类成员：EasyNews
let EasyNews = function (text,href){
    News.call(this);
    this.text = text || '';
    this.href = href || '#';
    this.init();
}
inheritPrototype(EasyNews,News);
EasyNews.prototype.init = function (){
    this.element = document.createElement('a');
    this.element.innerHTML = this.text;
    this.element.href = this.href;
    this.element.className = 'text';
}
//添加子元素方法
EasyNews.prototype.add = function (){}
EasyNews.prototype.getElement = function (){
    return this.element;
}

//子类成员：TypeNews
let TypeNews = function (text,href,type,pos){
    News.call(this);
    this.text = text || '';
    this.href = href || '#';
    this.type = type || '';
    this.pos = pos || 'left';
    this.init();
}
inheritPrototype(TypeNews,News);
TypeNews.prototype.init = function (){
    this.element = document.createElement('a');
    if(this.pos === 'left'){
        this.element.innerHTML = '[' + this.type + ']' + this.text;
    }else{
        this.element.innerHTML = this.test + '[' + this.type + ']' ;
    }
    this.element.href = this.href;
    this.element.className = 'text';
}
//添加子元素方法
TypeNews.prototype.add = function (){}
TypeNews.prototype.getElement = function (){
    return this.element;
}
```

创建一个新闻对象实例

```js
//创建新闻
let new1 = new Container('news',document.body);
new1.add(
    new Item('normal').add(
        new IconNews('梅西不拿金球也伟大','#','video')
    )
).add(
    new Item('normal').add(
        new NewGroup('has-img').add(
            new ImageNews('img/1.jpg','#','small')
        ).add(
            new EasyNews('从240斤胖子成功变型男','#')
        ).add(
            new EasyNews('五大雷人跑步机','#')
        )
    ).add(
        new Item('normal').add(
            new TypeNews('AK47不愿为费城打球','#','NBA','left')
        )
    ).add(
        new Item('normal').add(
            new TypeNews('火炮彪6分钟新高','#','CBA','left')
        )
    )
)
```

> 组合模式能给我们提供一个清晰的组成结构，组合对象类通过继承同一个父类使其具有统一的方法，也方便了我们统一管理和使用，这时单体对象和组合体成员行为表现就比较一致了。这样也就模糊了简单对象和组合对象之间的区别。

## 享元模式（城市公交车）

> 享元模式：运用共享技术有效的支持大量颗粒度的对象，避免对象间拥有相同内容造成多余的开销。

```JS
 let Flyweight = function (){
  //  已创建的元素
    let created = [];
  //  创建一个新闻包装对象
    function create(){
      let dom = document.createElement('div');
    //  将容器放在新闻列表中
      document.getElementById('container').appendChild(dom);
    //  缓存新创建的对象
      created.push(dom);
    //  返回创建的新元素
      return dom;
    }
    return {
    //  获取创建新闻元素方法
      getDiv : function (){
      //  如果已创建的元素小于当前页元素总个数，则创建
        if(created.length < 5){
          return create();
        }else{
        //  获取第一个元素，并插在最后面
          let div = created.shift();
          created.push(div);
          return div;
        }
      }
    }
  }()

  let paper = 0;
  let num = 5;
  let len = article.length;
//  添加5条新闻
  for(let i = 0 ; i < 5; i++){
    if(caticle[i]){
    //  通过享元类获取创建的元素并写入新闻内容
      Flyweight.getDiv().innerHTML = article[i];
    }
  }
//  下一页按钮绑定事件
  document.getElementById('next_page').onclick = function (){
  //  如果新闻内容不足5条就返回
    if(article.length < 5){
      return
    }
    let n = ++paper * num % len;//获取当前页的第一条新闻索引
    let j = 0;//循环变量
    for(let j = 0;j < 5;j++){
    //  如果纯在第n+j条则插入
      if(article[n+j]){
        Flyweight.getDiv().innerHTML = article[n+j];
      //  否则插入起始位置第n+j-len条
      }else if(article[n+j-len]){
        Flyweight.getDiv().innerHTML = article[n+j-len];
      //  如果都不存在则插入空字符串
      }else{
        Flyweight.getDiv().innerHTML = '';
      }
    }
  }
```

> 享元模式的应用是为了提高程序的执行效率， 就是把一些比较有重复性的代码抽象提取，重复调用，和桥接模式有一定区别，
>
> 桥接模式是将实现类和逻辑类相互分离，细节的改变不会引起总体项目的剧变，
>
> 而享元模式是在实现类内部，将实现类的代码抽象提取

# 第四篇：行为型设计模式

## 模板方法模式（照猫画虎）

> 模板方式模式：在父类定义一组操作系统算法骨架，而将一些实现步骤推迟到子类中，使得子类可以不改变父类的算法逻辑结构的同时，可以重新定义算法中某些实现类

```JS
//    模板类 基础提示框，data渲染数据
    let Alert = function(data){
    //  没有数据就返回，防止后面程序执行
      if(!data)
        return
    //  设置内容
      this.content = data.content;
    //  创建提示框面板
      this.panel = document.createElement('div');
    //  创建提示内容组件
      this.contentNode = document.createElement('p')
    //  创建确定按钮组件
      this.confirmBtn = document.createElement('span')
    //  创建关闭按钮组件
      this.closeBtn = document.createElement('b')
      //为提示框面板添加类
      this.panel.className = 'alert';
      //为关闭按钮组件添加类
      this.closeBtn.className = 'a-close';
      //为确定按钮组件添加类
      this.confirmBtn.className = 'a-confirm';
      //为确定按钮组件添加文案
      this.confirmBtn.innerHTML = data.confirm || '确认';
      //为提示内容添加文本
      this.contentNode.innerHTML = this.content;
      //点击确定按钮执行方法 如果data中有success方法则为success方法，否则为空函数
      this.success = data.success || function (){};
      //点击关闭按钮执行方法
      this.fail = data.fail || function (){};
    }
//    模板类的原型方法
    Alert.prototype = {
    //  创建方法
      init:function (){
      //  生成提示框
        this.panel.appendChild(this.closeBtn);
        this.panel.appendChild(this.contentNode);
        this.panel.appendChild(this.confirmBtn);
      //  插入页面中
        document.body.appendChild(this.panel);
      //  绑定事件
        this.bindEvent();
      //  显示提示框
        this.show();
      },
      bindEvent: function (){
        let me = this;
      //  关闭按钮点击事件
        this.closeBtn.onclick = function (){
        //  执行关闭取消方法
          me.fail();
        //  隐藏弹层
          me.hide();
        }
        this.confirmBtn.onclick = function () {
        //  执行关闭确认方法
          me.success();
        //  隐藏弹层
          me.hide();
        }
      },
    //  隐藏弹层方法
      hide : function (){
        this.panel.style.display = 'none';
      },
    //  显示弹层方法
      show : function (){
        this.panel.style.display = 'block';
      }
    }
```

在模板方法的基础上继承父类

```JS
//    右侧按钮提示框
    let RightAlert = function (data){
    //  继承基本提示框构造函数
      Alert.call(this,data);
    //  为确认按钮添加right类设置位置居右
      this.confirmBtn.className = this.confirmBtn.className + 'right';
    }
    //继承父类的方法
    RightAlert.prototype = new Alert();
//    还可以按照上面的方法创建其他类型的相似子类
//    标题提示框
    let TitleAlert = function (data){
      //  继承基本提示框构造函数
      Alert.call(this,data);
      //设置标题内容
      this.title = data.title;
      //创建标题组件
      this.titleNode = document.createElement('h3');
      //在标题内部写入标题内容
      this.titleNode.innerHTML = this.title;
    }
    //继承父类的方法
    TitleAlert.prototype = new Alert();
    TitleAlert.prototype.init = function (){
    //  插入标题
      this.panel.insertBefore(this.titleNode,this.panel.firstChild);
    //  继承基本提示框init 方法
      Alert.prototype.init.call(this);
    }


//    继承模板的继承类也同样可以作为模板类
//    带有取消按钮的弹出框
    let CancelAlert = function (data){
    //  继承标题提示框的构造函数
      TitleAlert.call(this,data);
    //  取消标题框自己独立的构造函数
      //  取消按钮文案
      this.cancel = data.cancel
      //创建取消按钮
      this.cancelBtn = document.createElement('span')
      //为取消按钮添加类
      this.cancelBtn.className = 'cancel';
      //为确定按钮组件添加文案
      this.cancelBtn.innerHTML = data.cancel || '取消';
    }
//    继承提示框的原型方法,这里继承Alert和TitleAlert都是可以的，
//    因为继承Alert后，init是增强，继承TitleAlert是重写
//    为了出现继承没有重写的问题， 还是继承一个比较干净的Alert比较好
    CancelAlert.prototype = new Alert();
    CancelAlert.prototype.init = function (){
      // 继承标题提示框创建方法
      TitleAlert.prototype.init.call(this);
       //  由于取消按钮要添加在末尾，所以在创建完其他组件后添加
      this.panel.appendChild(this.cancelBtn);
    }
    CancelAlert.prototype.bindEvent = function (){
      let me = this;
      //  标题提示框绑定事件方法继承
      TitleAlert.prototype.bindEvent.call(me);
      this.cancelBtn.onclick = function (){
        //  执行关闭取消方法
        me.fail();
        //  隐藏弹层
        me.hide();
      }
    }

```

实例化对象

```JS
//  创建一个提示框
new CancelAlert({
    title:'提示标题',
    content:'提示内容',
    success: function (){
        console.log('ok')
    },
    fail:function (){
        console.log('cancel')
    }
}).init()
```

> 模板方法的核心在于对方法的重用，他将核心方法封装在基类中，然子类继承基类的方法实现基类方法的共享，达到方法共用

## 观察者模式（通信卫星）

> 观察者模式：又被称作发布-订阅者模式或消息机制，他定义了一种依赖关系，解决了主体对象与观察者之间的耦合。

一般来说，一个观察者对象会有一个消息容器，三个方法：分别是订阅消息方法，取消订阅方法，发送订阅的方法

```JS
//将观察者放在闭包里，当页面加载就立即执行
let Observer = (function (){
  let _messages = {};
  return {
  //  注册信息接口
    regist : function (type,fn){
      //如果消息不存在，就创建一个新的消息类型
      if(typeof _messages[type] === 'undefined'){
        _messages[type] = [fn];
      }
    //  如果消息存在
      else {
        // 将该动作方法推入该消息对应的动作执行序列中
        _messages[type].push(fn);
      }
    },
  //  发布信息接口
    fire : function (type,args){
    //  如果该类型没有注册，则直接返回
      if(!_messages[type]){
        return
      }
    //  定义消息信息
      let event = {
        type:type,
        args:args || {}
      }
      for(let i = 0 ; i < _messages[type].length;i++){
        _messages[type][i].call(this,event)
      }
    },
  //  移除信息接口
    remove : function (type,fn){
    //    如果消息存在
      if(_messages[type] instanceof Array){
      //  从最后一个元素遍历】
        for(let i = _messages[type].length - 1 ;i>=0;i--){
          //判断一下函数是否相同，
          _messages[type][i] === fn && _messages[type].splice(i,1);
        }
      }
    }
  }
})()
```

简单测试一下

```JS
//简单测试一下
//订阅消息
Observer.regist('test',function (e){
  console.log(e.type,e.args)
})
//发布
Observer.fire('test','测试一下')
//取消订阅
Observer.remove('test',function (e){
  console.log(e.type,e.args)
})
//这里再发布信息就无法触发了
Observer.fire('test','测试一下')
```

一个观察者模式的例子

```JS
//学生类
let Student = function (result){
  let that = this;
  that.result = result;
  that.say = function () {
    console.log(that.result)
  }
}
Student.prototype.answer = function (question) {
  Observer.regist(question,this.say)
}

Student.prototype.sleep = function (question) {
  console.log(this.result+ ' '+ question +'注销')
  Observer.remove(question,this.say)
}

//教师类
let Teacher = function () {}
//教师提问问题的方法
Teacher.prototype.ask = function (question){
  console.log('问题是'+ question)
  Observer.fire(question)
}

let student1 = new Student('学生一回答问题');
let student2 = new Student('学生二回答问题');
let student3 = new Student('学生三回答问题');
//每个同学订阅两个
student1.answer('什么是设计模式');
student1.answer('简述观察者模式');
student2.answer('什么是设计模式');
student2.answer('简述观察者模式');
student3.answer('什么是设计模式');
student3.answer('简述观察者模式');
//第三个取消订阅
student3.sleep('简述观察者模式');

let teacher = new Teacher();
teacher.ask('什么是设计模式');
teacher.ask('简述观察者模式');
```

> 观察者模式最主要的作用是解决类或对象之间的耦合，解耦两个相互依赖的对象，使其依赖于观察者的消息机制，这样对于任意一个订阅者来说，其他订阅者对象的改变不会影响到自身，对于每一个订阅者来说，其自身既可以是消息的发布者，也可以是消息的执行者。这取决于观察者对象的三种方法。

## 状态模式（超级玛丽）

> 状态模式：当有一个对象的内部状态发生改变时，会导致其行为的改变，这看起来像是改变了对象

 以超级玛丽为例：超级玛丽拥有这多种动作，包括跳跃，开枪，蹲下，奔跑等，如果对他的状态进行单个判断，必然会导致代码非常糟糕，所以要使用状态管理

```js
//    创建超级玛丽类
let MarryState = function (){
    //玛丽当前的状态
    let _currentState = {}
    //动作与状态方式映射
    let states = {
        jump : function (){
            console.log('jump')
        },
        move : function (){
            console.log('move')
        },
        shoot : function (){
            console.log('shoot')
        },
        squat : function (){
            console.log('squat')
        }
    };
    //  动作控制类
    let Action = {
        //  改变状态方法
        changeState : function (){
            //  组合动作通过传递多个参数传递
            let arg = arguments;
            // 重置内部的状态
            _currentState = {}
            if(arg.length){
                for(let i = 0 ; i<arg.length;i++){
                    _currentState[arg[i]] = true
                }
            }
            return this;
        },
        //  执行动作
        goes : function (){
            console.log('执行一次动作');
            for(let i in _currentState){
                states[i] && states[i]()
            }
            return this
        }
    }
    return {
        //外观模式
        change:Action.changeState,
        goes:Action.goes
    }
}

//    运行
let marry = new MarryState();
marry
    .change('jump','shoot')
    .goes()
    .change('squat','move')
    .goes()
```

> 状态模式既是解决程序中臃肿的分支判断语句问题，将每个分支转换为一种状态出来，方便每种状态的管理，而不至于每次执行是遍历所有分支，但是如果是比较简单的功能，状态模式会造成额外的代码量。

## 策略模式（活诸葛）

> 策略模式：将定义的一组算法封装起来，使其相互之间可以替换，封装的算法具有一些独立性，不会随客户端变化而变化。

```JS
//   价格策略对象
   let PriceStrategy = function (){
   //  内部算法对象
     let stragtegy = {
       //  100返30
       return30 : function (price){
         return +price + parseInt( price / 100) * 30;
       },
       //  100返50
       return50 : function (price){
         return +price + parseInt( price / 100) * 50;
       },
       //  9折
       percent90 : function (price){
         return price + 100 * 90 / 10000;
       },
       //  八折
       percent80 : function (price){
         return price + 100 * 80 / 10000;
       },
       //  五折
       percent50 : function (price){
         return price + 100 * 50 / 10000;
       },
     }
   //  策略算法调用接口
     return function (algorithm,price) {
       // 如果算法存在，则调用算法，否则返回
       return stragtegy[algorithm] && stragtegy[algorithm](price)
     }
   }()

   let price = PriceStrategy('return50','314.67')
   console.log(price)
```

> 策略模式最主要的特色是创建一系列策略算法，每组算法处理的业务都是相同的，但是处理的过程或者是结果是不同的，所以他们都是可以互相转换的。这样就解决了算法之间的耦合。
>
> 此外，策略模式和状态模式的格式很像，都是建立一个对象放在闭包中，给出一个接口修改添加删除对象
>
> 策略模式有三个有点
>
> 1. 策略模式封装了一系列代码簇，并且封装的代码相互之间独立，便于算法的重复引用，提高了算法的复用率
> 2. 策略模式与继承相比，在类的继承中继承的方法是被封装在类中，因此当需求很多算法时，就不得不创建多种类，这样会导致算法和算法的使用者耦合在一起。
> 3. 同状态模式一样，策略模式也是一种优化分支判断语句的模式，采用策略模式对算法封装使得算法更利于维护。

## 职责链模式（有序车站）

> 职责链模式：解决请求的发送者与请求的接收者之间的耦合，通过职责链上的多个对象分卸请求流程，实现请求在多个对象之间的传递。直到最后一个对象完成请求的处理。

```JS

/***
     * 异步请求对象
     * 参数 data  请求数据
     * 参数 dealType 响应数据处理对象
     * 参数 dom   事件源
     ***/

let sendData = function (data,dealType,dom){
    //  XHR 对象
    let xhr = new XMLHttpRequest();
    let url = 'xxxxxxxx';
    //  请求返回事件
    xhr.onload = function (event){
        //  请求成功
        if((xhr.status >= 200 && xhr.status < 300 ) || xhr.status === 304){
            dealData(xhr.responseText,dealType,dom);
        }else {
            //  请求失败
        }
        //  拼接请求字符串
        for(let i in data ){
            url += '&' + i + '=' + data[i];
        }
        //  发送异步请求
        xhr.open('get',url,true);
        xhr.send(null)
    }
}

/***
     * 处理响应数据
     * 参数 data  请求数据
     * 参数 dealType 响应数据处理对象
     * 参数 dom   事件源
     ***/
let dealData = function (data,dealType,dom) {
    //  对象toString 方法的简化应用
    let dataTpye = Object.prototype.toString.call(data);
    //  判断响应数据处理对象
    switch (dealType){
            //  输入框提示功能
        case 'sug':
            //      如果数据是数组
            if(dataTpye === '[object Array]'){
                //  创建提示框组件
                return createSug(data,dom)
            }
            //   将响应的对象数据转化为数组
            if(dataTpye === '[object Object]'){
                let newData = [];
                for(var i in data){
                    newData.push(data[i]);
                }
                return createSug(newData,dom)
            }
            break;
        case 'validate':
            //    创建校验组件
            return createValidataResult(data,dom);
            break;
    }
}

/***
     * 创建提示框组件
     * 参数 data  请求数据
     * 参数 dom   事件源
     ***/
let createSug = function (data,dom){
    let html = '';
    //    拼接每一条提示语句
    for(let i = 0;i<data.length;i++){
        html += '<li>' + data[i] + '</li>>'
    }
    //  显示提示框
    dom.parentNode.getElementsByTagName('ul')[0].innerHTML = html ;
}

/***
     * 创建提示框组件
     * 参数 data  请求数据
     * 参数 dom   事件源
     ***/
let createValidataResult = function (data,dom){
    dom.parentNode.getElementsByTagName('span')[0].innerHTML = data ;
}


//    对象实例化
let input = document.getElementsByTagName('input');
input[0].onchange = function (e){
    sendData({value:input[0].value},'validate',input[0])
},
    input[1].onchange = function (e){
    sendData({value:input[1].value},'sug',input[1])
}
```

代码一共分了三部，请求数据，处理数据，根据数据建立组件模块

调用时，只需要调用第一步的函数即可，每一步的函数都只完成各自的任务，其他任务一律不管。

> 职责链模式定义了请求的传递方向，通过多个对象对请求的传递，实现了一个复杂的逻辑操作，因此职责链模式将负责的需求颗粒化逐一实现每个对象分内的需求，并将请求顺序的传递。

## 命令模式

> 命令模式：将请求与实现解耦并封装成独立对象，从而实现不同请求对客户端的实现参数化。

```JS
//模块实现模块
let viewCommand = (function () {
    let tpl =
        {
            // 展示图片结构模板
            product: [
                '<div>',
                '<img sre="(#src#}"/>',
                '<p>{#text#}</p>',
                '</div>'
            ].join(''),
            //展示标题结构模板
            title: [
                '<div class="title">',
                '<div class="main">',
                '<h2>(#title#}</h2>',
                '<p>(#tips#}</p>',
                '</div>'
            ].join('')
        },
        //格式化字符串缓存字符串
        html = '';
    function formateString(str,obj){
        return str.replace(/\{#(\W+)#\}/g,function (match,key){
            return obj[key];
        })
    }
    let Action = {
        create : function(data,view){
            if (data.length) {
                for(let i = 0; i < data.length; i++){
                    html += formateString (tpl[view], data[i]);
                }
            }else{
                html += formateString(tpq[view],data)
            }
        },
        display :function (container,data,view){
            if(data){
                this.create(data,view);
            }
            document.getElementById(container).innerHTML = html;
            html = '';
        }
    }
    //命令接口
    return function excute(){}
})()
```

实例化

```JS 
let productData = [
    {
        src:'command/02.jpg',
        text:'绽放的桃花'
    },
    {
        src:'command/03.jpg',
        text:'阳光下的温馨'
    },
    {
        src:'command/04.jpg',
        text:'镜头前的绿色'
    },
]
let titleData = {
    title:'夏日里的一片温馨',
    tips : '暖暖的温情带给人们家的感受'
}

viewCommand({
    command : 'display',
    params : ['title',titleData,'title' +
              '']
})
```

> 命令模式就是将执行的命令封装，解决命令的发起者和命令的执行者之间的耦合，每一天命令实质上是一个操作，命令的使用者不必要了解命令的执行者的接口是如何实现的，命令是如何接受的。所有的命令都被存储在命令对象里

## 访问者模式（驻华大使）

> 访问者模式：针对对象结构中的元素，定义在不改变该对象的前提下访问结构中元素的新方法

```js
  //  访问器
let Visitor =(function () {
    return {
        //截取数据方法
        splice:function () {
            //  splice 方法参数，从原参数的第二个参数开始算起
            let args = Array.prototype.splice.call(arguments,1);
            //  对第一个参数对象执行splice方法
            return Array.prototype.splice.apply(arguments[0],args);
        },
        //  追加数据方法
        push:function () {
            //  强化类数组对象，使他拥有length属性
            let len = arguments[0].length | 0;
            //  添加的数据从第二个开始算起
            let args = this.splice(arguments,1);
            //  校正length属性
            arguments[0].length = len + arguments.length - 1;
            //  对第一个参数对象执行push方法
            return Array.prototype.push.apply(arguments[0],args);
        },
        //  弹出最后一次添加的元素
        pop:function (){
            //  对第一个参数对象执行pop方法
            return Array.prototype.pop.apply(arguments[0])
        }
    }
})()
```

```js
//  例子
let a = new Object();
console.log(a.length);
Visitor.push(a,1,2,3,4,5);
console.log(a);
Visitor.pop(a)
console.log(a)
Visitor.splice(a,2)
console.log(a)
```

> 访问者模式解决数据与数据的操作方式的耦合，将数据的操作方法独立于数据，使其可以自由化演变。因此访问者更适合那些数据稳定，但是数据的操作方法易变的环境下，因此当操作环境发生改变时，可以自由修改操作方法以适应新的操作环境而不用去修改原数据。

## 中介者模式（媒婆）

> 中介者模式：通过中介者对象封装一系列对象之间的交互，使对象之间不在相互引用，降低他们之间的耦合，有时中介者对象也可以改变对象之间的交互。

```js
//  中介者对象
let Mediator = (function () {
    //  消息对象
    let _msg = {};
    return {
        /***
       *
       * 订阅消息方法
       * 参数 type 消息名称
       * 参数 action 消息回调函数
       *
       ***/
        register : function (type,action){
            //  如果消息存在
            if(_msg[type]){
                //  存入回调函数
                _msg[type].push(action);
            }else{
                //  不存在就新建一个消息列表
                _msg[type] = [];
                //存入新消息回调函数
                _msg[type].push(action)
            }
        },
        /***
       *
       * 发布消息方法
       * 参数 type 消息名称
       *
       ***/
        send:function (type) {
            //  如果消息已经被订阅
            if(_msg[type]){
                //  遍历已存储的消息回调函数
                for(let i = 0;i<_msg[type].length;i++){
                    //执行回调函数
                    _msg[type][i]()
                }
            }
        }
    }
})()
```

```JS
//  浅浅测试一下
//  订阅demo消息
  Mediator.register('demo',function () {
    console.log('订阅');
  })
//  订阅demo消息
  Mediator.register('demo',function (){
    console.log('又订阅')
  })
//  发布demo消息
  Mediator.send('demo')
```

> 同观察者模式一样，中介者模式的主要业务的主要业务也是通过模块间或者对象间的复杂通信，来解决模块间或者对象间耦合，对于中介者对象的本质是分装多个对象的交互。并且这些对象的交互一般都是在中介者内部实现的。
>
> 与外观模式的封装特性相比，中介者模式对多个对象交互地封装，且这些对象一般处于同一层面上，并且封装的交互在中介者内部，而外观模式封装的目的是为了提供更简单易用的接口，而不会添加其他功能。

## 备忘录模式（做好笔录）

> 备忘录模式：在不破坏对象封装性的前提下，在对象之外捕获并保存该对象内部的状态以便日后对对象使用或者对象恢复到以前的状态。

```js
//  下一页按钮点击事件
$('#next_page').click(function (){
    //  获取新闻内容元素
    let $news = $('#news_content'),
        page = $news.data('page');
    //  获取并展示新闻
    getPageData(page,function () {
        $news.data('page',page+1)
    })
});
$('#pre_page').click(function (){
    //  显示上一页
})

//  Page备忘录类
let Page = function (){
    //  信息缓存对象
    let cache = {};
    /***
     *主函数
     * 参数 page 页码
     * 参数 fn 成功回调函数
     ***/
    return function (page,fn) {
        //  判断该页数据是否在缓存中
        if(cache[page]){
            //  恢复到该页状态，显示该页内容
            showPage(page,cache[page]);
            //  执行成功回调函数
            fn&&fn();
        }else{
            //  若缓存cache中无该页数据
            //  请求数据
            showPage(page,res.data);
            //  将该页数据放入缓存中
            cache[page] = res.data;
            //  执行成功回调函数
            fn&fn()
        }
    }
}
```

> 备忘录模式最主要的任务就对现有数据或状态做缓存，为将来的某个时刻使用或恢复提供备份,在JS中，备忘录模式常常用于对数据的缓存备份，浏览器端获取的数据往往是从陪你过服务器端请求获取到的，而请求往往非常耗时，对数据进行缓存是一种非常有效提升性能的方式。
>
> 在备忘录模式中，数据常常存储在备忘录对象的缓存器中，这样对于数据的读取必定要调用备忘录提供的方法，因此备忘录对象也是对数据存储器的一次保护封装，

## 迭代器模式（点钞器）

> 迭代器：在不暴露对象内部结构的同时，可以顺序地访问聚合内部的元素。

通过迭代器，我们可以顺序地访问一个聚合对象的每一个元素

```js
 //迭代器
let Iterator = function (item,container1) {
    //  获取父容器，若container参数存在，并且可以获取到该元素则获取，否则获取document
    let container = container1 && document.getElementById(container1) || document,
        //获取元素
        items = container.getElementsByTagName(item),
        //索引值
        index = 0,
        //获取元素长度
        length = items.length;
    let splice = [].splice;
    return {
        //  获取第一个元素
        first:function (){
            index = 0;
            return items[index];
        },
        //  获取最后一个元素
        second:function (){
            index = length - 1;
            return items[index];
        },
        //  获取前一个元素
        pre:function (){
            if(--index > 0){
                return items[index];
            }else{
                index = 0;
                return null;
            }
        },
        //  获取后一个元素
        next:function (){
            if(++index < length){
                return items[index];
            }else{
                index = length - 1;
                return null;
            }
        },
        //  获取某一个元素
        get:function (num){
            index = num >= 0 ? num % length : num % length + length;
            return items[index]
        },
        //  对每一个元素执行某一个方法
        dealEach:function (fn){
            //  第二个参数开始为回调函数中参数
            let args = splice.call(arguments,1)
            //  遍历元素
            for(let i = 0;i < length ; i++){
                //  对元素进行回调函数
                fn.apply(items[i],args)
            }
        },
        //  对某一个元素执行某一个方法
        dealItem:function (num,fn){
            fn.apply(this.get(num),splice.call(arguments,2))
        },
        //  排他方式处理某一个元素
        exclusive:function (num,allFn,numFn){
            //  对所有元素执行回调函数
            this.dealEach(allFn);
            //  如果num是数组类型的
            if(Object.prototype.toString.call(num) === '[object Array]'){
                //  遍历num数组
                for(let i = 0 ; i < num.length ; i++){
                    //  分别处理每一个元素
                    this.dealItem(num[i],numFn);
                }
            }else {
                this.dealItem(num,numFn)
            }

        }
    }
}
```

浅浅测试一下

```js
//测试一下
let demo = new Iterator('li','container')
console.log(demo.first())
console.log(demo.pre())
console.log(demo.next())
console.log(demo.get(2000))
demo.dealEach(function (){
    this.innerHTML = 'hello';
    this.style.background = 'red';
})
demo.exclusive([3],function (){
    this.innerHTML = 'hello';
    this.style.background = 'blue';
},function (){
    this.innerHTML = 'hi';
    this.style.background = 'white';
}
)
```

实现一下数组的迭代器

```JS
/***
   * 数组迭代器：对于JS中的数组，有一些低版本的IE并没有实现，需要我们去定义一下
   ***/
let eachArray = function (arr,fn){
    for(let i = 0 ; i < arr.length;i++){
        if( fn.call(arr[i],i,arr[i]) === false ){
            break;
        }
    }
}

//测试
array = [1,2,3,4,5]
eachArray(array,function(i,data){
    console.log(i,data)
})
```

实现一下对象的迭代器

```js
/***
 * 对象迭代器
 ***/
let eachObject = function (obj,fn){
  for(let i in obj){
    if( fn.call(obj[i],i,obj[i]) === false ){
      break;
    }
  }
}
//测试
OBJECT = {
  1:'A',
  2:'B',
  3:'C',
  4:'D',
  5:'E'
}
eachObject(OBJECT,function(i,data){
  console.log(i,data)
})
```

在项目中，时常会出现对象套对象的问题，这时是不可以通过点运算符拿到数据的，所以需要进行拍平数组，对象的操作。

```js
/***
   * 同步迭代器
   * 对象通常会有很深的迭代结构
   ***/
let A = {
    common: {},
    client:{
        user:{
            username:'yjm',
            uid:'123',
        }
    },
    server:{}
}

//  同步变量迭代器取值器
  Getter = function (obj,key){
      //  如果不存在A则返回未定义
      if(!obj){
          return undefined
      }
      //获取同步变量A对象
      let result = obj;
      //解析属性层次序列
      key = key.split('.')
      //迭代同步变量A对象属性
      for(let i = 0 ;i<key.length;i++){
          //   如果第i层属性存在对应的值则迭代该属性值
          if(result[key[i]] !== undefined){
              result = result[key[i]];
              //  如果不存在则返回未定义
          }else{
              return undefined
          }
      }
      return result
  }

console.log(Getter(A,'client.user.username'))
console.log(Getter(A,'client.user.undefined'))
```

> 通过迭代器我们可以顺序访问一个聚合对象中的每一个元素

## 解释器模式（语言翻译）

> 解释器模式：对于一种语言，给出其文法表示形式，定义一种解释器，通过使用这种解释器来解释语言中定义的句子

# 第五篇：技巧型设计模式

## 链模式（永无尽头）

> 链模式：通过在对象方法中将当前对象返回，实现对同一个对象多个方法的链式调用，从而简化对该对象的对个方法的多次调用时，对该对象的多次引用。

## 委托模式（未来预言家）

> 委托模式：多个对象接受并处理同一请求，他们将请求委托给另一个对象处理请求，将子元素的事件委托给父元素，然后通过事件冒泡传递到父元素，然后判断事件源的特性来执行某一业务逻辑
>
> 应用场景：
>
> * 1、子元素拥有者相同的事件，可以将事件提取到父元素里
> * 2、要给未出现的子元素绑定事件，可以先把事件绑定到父元素里

## 数据访问对象模式（数据管理器）

> 数据访问对象模式（DAO）：因为在前端，一般使用的是localStorage，但是在localStorage里面是没有限制的，这意味着每个人都可以随意操作它，所以我们需要一个类来限制他.

```js

  /***
   * 本地存储类
   * 参数：preId   本地存储数据库前缀
   * 参数：timeSign    时间戳与存储数据之间的拼接符
   ***/
  let BaseLocalStorage = function (preId,timeSign){
  //  定义本地数据库前缀
    this.preId = preId;
  //  定义本地时间戳与存储数据之间的拼接符
    this.timeSign = timeSign;
  }
//  本地存储类原型
  BaseLocalStorage.prototype = {
  //  操作状态
    status:{
      SUCCESS:0,//成功
      FAILURE:1,//失败
      OVERFLOW:2,//溢出
      TIMEOUT:3,//过期
    },
  //  保存本地存储链接
    storage: localStorage || window.localStorage,
  //  获取本地存储数据数据真实字段
    getKey :function (key) {
      return this.preId + key;
    },
    /***
     * 添加（修改数据）
     * 参数：key   数据字段标识
     * 参数：value   数据值
     * 参数：callback   回调函数
     * 参数：time   添加时间
     ***/
    set : function (key,value,callback,time){
      //  默认操作状态成功
      let status = this.status.SUCCESS;
      //    获取真实字段
      let currentKey = this.getKey(key);
      try{
        //  参数时间参数时获取时间戳
        time = new Date(time).getTime() || time.getTime();
      }catch (e){
        //  如果出现错误，默认参数有效时间为一个月
        time = new Date().getTime() + 1000 * 60 * 24 * 31;
      }
      try{
        //  向数据库中添加数据
        this.storage.setItem(currentKey,time+this.timeSign + value);
      }catch (e){
        //  溢出失败
        status = this.status.OVERFLOW;
      }
      //  有回调函数就执行回调函数并传入参数操作状态，真实数据字段标识以及存储数据值
      callback && callback.call(this,status,currentKey,value);
    },
    /***
     * 获取数据
     * 参数：key   数据字段标识
     * 参数：callback   回调函数
     ***/
    get : function (key,callback) {
        //默认操作状态成功
        let status = this.status.SUCCESS;
        //获取当前数据的字段
        let currentKey = this.getKey();
        let value = null;
        //时间戳与存储数据之间的拼接符长度
        let timeSignLen = this.timeSign.length;
        //缓存当前对象
        let that = this;
        //时间戳与存储数据之间的拼接符起始位置
        let index;
        //时间戳
        let time ;
        //最后得到的数据
        let result;
      try{
      //  获取当前字段对应的数据字符串
        value = that.storage.getItem(currentKey);
      }catch (e){
      //  获取失败则返回失败状态，数据结果为null
        result = {
          status : that.status.FAILURE,
          value : null
        };
      //  执行回调并返回
        callback && callback.call(this,result.status,result.value)
        return result;
      }
    //  如果成果获取数据字符串
      if(value){
      //  获取时间戳与存储数据之间的拼接符起始位置
        index = value.indexOf(that.timeSign);
      //  获取时间戳
        time = +value.slice(0,index);
      //  如果时间为过期
        if(new Date(time).getTime() > new Date().getTime() || time === 0){
          value = value.slice(index + timeSignLen);
        }else{
        //  过期则结果为null
          value = null;
        //  设置状态为过期状态
          status = that.status.TIMEOUT;
        //  删除该字段
          that.remove(key);
        }
      }else{
      //  未获取数据字符串状态为失败状态
        status = that.status.FAILURE;
      }
    //设置结果
      result = {
        status : status,
        value : value
      }
      //  执行回调并返回
      callback && callback.call(this,result.status,result.value)
      return result;
    },
    /***
     * 删除数据
     * 参数：key   数据字段标识
     * 参数：callback   回调函数
     ***/
    remove : function (key,callback) {
      //  默认操作状态成功
      let status = this.status.FAILURE;
      //    获取真实字段
      let currentKey = this.getKey(key);
      //设置默认数据为空
      let value = null;
      try{
        //  获取字段对应的数据
        value = this.storage.getItem(currentKey);
      }catch (e){}
      if(value){
        try{
        //  删除数据
          this.storage.removeItem(currentKey)
        //  设置操作成功
          status = this.status.SUCCESS
        }catch (e){}
      }
      //  有回调函数就执行回调函数并传入参数操作状态，真实数据字段标识以及存储数据值
      callback && callback.call(
        this,
        status,
        status > 0 ? null : value.slice(value.indexOf(this.timeSign) + this.timeSign.length)
      );
    }
  }
```

新建实例对象

```js
let LS = new BaseLocalStorage('LS__','###')
LS.set('a','yjm',function (){
    console.log(arguments);
})
LS.get('a',function (){
    console.log(arguments)
})
LS.remove('a',function (){
    console.log(arguments)
})
LS.get('a',function (){
    console.log(arguments)
})
```

> 数据范文对象模式是一种对数据库的操作（如简单的CRUD创建，读取，更新，删除），进行封装，用户不必为操作数据库感到烦恼，DAO已经为我们准备好了简单而统一的操作接口。

## 节流模式（执行控制）

> 节流模式：对重复的业务逻辑进行节流控制，执行最后一次操作并取消其他操作，以提高性能

节流器做了两件事情

1、清除将要执行的函数，如果传入的第一个参数是TRUE，则表示要清除将要执行的函数，同时会判断第二个参数（执行函数）有没有计时器句柄，有就清除

2、延迟执行函数，要传入两个参数，执行函数，相关参数，首相要对执行函数绑定一个计时器句柄，保存该执行函数的计时器，第二个参数包含：执行作用域，执行函数的参数，执行函数延迟执行的时间

```js
/***
   * 节流器
   * 参数 boolean 控制是否清楚计数器
   * 参数 函数  需要节流的函数
  ***/
let throttle = function (){
    let isClear = arguments[0];
    let fn = undefined;
    if(typeof isClear === 'boolean'){
      //第二个参数则为函数
      fn = arguments[1];
      //函数的计时器句柄存在，则清除该计时器
      fn._throttleID && clearTimeout(fn._throttleID);
    //  通过计时器眼熟函数的执行
    }else{
      fn = arguments[0];
      params = arguments[1];
    //  为执行时的参数适配默认值
      let p = extend({
        context:null,
        args:[],
        time:300,
      },params)
    //清除执行计数器句柄
      arguments.callee(true,fn);
    //  为函数绑定计时器句柄，延迟执行函数
      fn._throttleID = setTimeout(function (){
      //  执行函数
        fn.apply(p.context,p.args)
      },p.time)
    }
  }
```

> 节流模式的核心思想是创建计时器，延迟程序的执行，这也使得计时器中回调函数的操作是异步执行的。

## 简单模板模式（卡片拼图）

> 简单模板模式：通过格式化字符串拼接出视图避免创建视图时大量节点操作，优化内存开销。

一个模板

```js
//    模板渲染方法
A.formatString = function (str , data){
    return str.replace(/\{#(\w+)#\}/g,function (match,key) {
        return typeof data[key] === undefined ? '' : data[key]});
}
//命名空间 单体对象
let A= A|| {};
// 主体展示区容器
A.root = document.getElementById('container');
//创建视图方法集合
A.strategy = {
    //    文字列表展示
    'listPart' : function  (data) {
        let s = document.createElement('div'),
            ul = '',
            ldata = data.data.li,
            //    模块模板
            tpl = [
                '<h2>{#h2#}</h2>',
                '<p>{#p#}</p>',
                '<ul>{#ul#}</ul>',
            ].join('');
        //  列表项模板
        liTpl = [
            '<li>',
            '<strong>{#strong#}</strong>',
            '<span>{#span#}</span>',
            '</li>'
        ].join('');
        //  有id设置模块id
        data.id && (s.id = data.id)
        //  遍历列表数据
        for(let i= 0 ; i < ldata.length ; i ++ ){
            //  如果有列表项数据
            if(ldata[i].em || ldata[i].span){
                //  列表字符串追加一项列表项
                ul += A.formatString(liTpl,ldata[i]);
            }
        }
        //  装饰列表数据
        data.data.ul = ul;
        //  渲染模块并插入模块中
        s.innerHTML = A.formatString(tpl,data.data);
        //渲染模块
        A.root.appendChild(s)
    },
    'codePart' : function () {},
    'onlyTitle' : function () {},
    'guide' : function () {},
}
//    模板生成器
A.view = function (name){
    //  模板库
    let v = {
        //  代码模板
        code : '<pre><code>{#code#}</code></pre>',
        //  图片模板
        img : '<img src = "{#src#}" alt="{#alt#}" title="{#title#}"/>',
        //  组合模板
        theme : [
            '<div>',
            '<h1>{#title#}</h1>',
            '{#content#}',
            '</div>'
        ].join('')
    }
    //  如果参数是一个数组，则返回多行模板
    if(Object.prototype.toString.call(name) === "[object Array]"){
        //  模板缓存器
        let tpl = '';
        //  遍历标识
        for(let i = 0 ; i < name.length;i++){
            //  模板缓存追加模板
            tpl += arguments.callee(name[i])
        }
        //  返回最终模板
        return tpl;
    }else{
        //  如果模板库中有该模板就返回该模板，否则返回简易模板
        return v[name] ? v[name] : `<${name}>{#${name}#}</${name}>`
    }
}
```

实例化

```js
//命名空间 单体对象
let A= A|| {};
// 主体展示区容器
A.root = document.getElementById('container');
//创建视图方法集合
A.strategy = {
    //    文字列表展示
    'listPart' : function  (data) {
        tpl = A.view(['h2','p','ul']);
        //  列表项模板
        liTpl = A.formatString(A.view('li'),{li:A.view(['strong','span'])})
    },
    'codePart' : function () {},
    'onlyTitle' : function () {},
    'guide' : function () {},
}	
```

> 简单模板模式一般存在三部分：字符串模板库，格式化方法，字符串拼接操作。
>
> 然而前者在不同需求的视线中，视同往往是不一致的，一次字符串模板常常是多变的。

## 惰性模式（机器学习）

> 惰性模式：减少每次代码执行时的重复性的分支判断，通过对对象的重定义来屏蔽原对象中的分支判断

先看一个错误示范

```JS
//  单体模式定义命名空间
  let A = {};
//  添加绑定事件方式on
  A.on = function (dom,fype,fn){
    if(dom.addEventListener){
      dom.addEventListener(tpye,fn,false);
    }else if(dom.attachEvent){
      dom.attachEvent('on' + type,fn);
    }else{
      dom['on' + type] = fn;
    }
  }
```

上面的这个代码是根据不同的浏览器进行不同的适配，但是一般情况下这种代码只执行一次就可以了，后续的执行都是无效的，所以需要对它进行优化。

### 第一种方式

```JS
/***
 * 加载即执行
 ***/
A.on = function (dom,fype,fn){
  if(dom.addEventListener){
    return function (dom,fype,fn){
      dom.addEventListener(tpye,fn,false);
    }
  }else if(dom.attachEvent){
    return function (dom,fype,fn){
      dom.attachEvent('on' + type,fn);
    }
  }else{
    return function (dom,fype,fn){
      dom['on' + type] = fn;
    }
  }
}()

```

### 第二种方式

```JS

/***
 * 惰性执行（重定义函数）
 ***/
A.on = function (dom,fype,fn){
  if(dom.addEventListener){
    A.on = function (dom,fype,fn){
      dom.addEventListener(tpye,fn,false);
    }
  }else if(dom.attachEvent){
    A.on = function (dom,fype,fn){
      dom.attachEvent('on' + type,fn);
    }
  }else{
    A.on = function (dom,fype,fn){
      dom['on' + type] = fn;
    }
  }
//  执行重定义on方法
  A.on(dom,type,fn)
}
```

> 惰性模式是一种拖延模式，由于对象的创建或数据的计算会花费高昂的代价，因此页面会演出对这一类对象的创建

## 参与者模式（异国战场）

> 参与者模式：在特定的作用域中执行给定的函数，并将参数原封不动的传递

## 等待者模式（入场仪式）

> 等待者模式：通过对多个异步进程监听，来触发未来发生的动作

```js
  /***
   * 下面手写一个简单的promise
   ***/
//  等待者对象
let Waiter = function () {
    //注册了的等待对象容器
    let dfd = [],
        //成果回调方法容器
        doneArr = [],
        //失败回调方法容器
        failArr =[],
        //缓存Array方法
        slice = Array.prototype.slice,
        //缓存this
        that = this;
    //监控对象类
    let Primise = function (){
        this.resolved = false;
        this.rejected = false;
    }
    //监控对象类原型
    Primise.prototype = {
        resolve : function (){
            //  设置当前监控对象成功
            this.resolved = true;
            //如果没有监控对象则取消执行
            if(dfd.length){
                return;
            }
            for(let i = dfd.length - 1 ; i >= 0;i--){
                //  如果在任意一个监控对象没有被解决或者解决失败时返回
                if(dfd[i] && !dfd[i].resolve || dfd[i].reject){
                    return;
                }
                //  清除监控对象
                dfd.splice(i,1);
            }
            //  执行解决成功回调方法
            _exec(doneArr);
        },
        reject: function (){
            //  设置当前监控对象解决失败
            this.rejectd = true;
            //  如果没有监控对象则取消执行
            if(!dfd.length){
                return;
                //  清除所有监控对象
                dfd.splice(0);
                //  执行解决成功回调方法
                _exec(failArr);
            }
        }
    }
    //创建监控对象
    that.Deferred = function () {
        return new Primise();
    }
    //回调执行方法
    function _exec(arr){
        for(let i = 0;i<arr.length;i++){
            try{
                arr[i] && arr[i]();
            }catch (e){}
        }
    }
    //监控异步方法 参数，监控对象
    that.when = function (){
        //  设置监控对象
        dfd = slice.call(arguments);
        //  获取监控对象数组长度
        let i = dfd.length;
        //  向前遍历对象，最后一个对象的索引值是length - 1
        for(--i;i >= 0 ;i--){
            //  如果不存在监控对象，或者监控对象已经解决或者不是监控对象
            if(!dfd[i] || dfd[i].resolved || dfd[i].rejected || !dfd[i] instanceof Primise){
                //  清理内存，清除当前监控对象
                dfd.splice(i,1)
            }
        }
        //返回等待者对象
        return that;
    };
    //解决成果回调函数添加方法
    that.done = function (){};
    //解决失败回调函数添加方法
    that.fail = function (){};
}
```

> 对于一些耗时较长的操作，同步的做法会导致阻塞，而异步加等待者的模式可以很好的解决这一问题
