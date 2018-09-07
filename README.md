# closure
关于闭包的学习记录
此记录非原创，只是整合不同地方看到的觉得对理解闭包有帮助的一些笔记。

## 闭包的定义

闭包由函数及创建函数的词法环境组合而成。
```
function A(){
    function B(){
       console.log('Hello Closure!');
    }
    return B;
}
var C = A();
C();// Hello Closure!
```
这是最简单的闭包例子。
上述代码翻译成自然语言如下：
 - 定义普通函数 A
 - 在 A 中定义普通函数 B
 - 在 A 中返回 B
 - 执行 A，并把 A 的返回结果赋值给变量 C
 - 执行 C 

把这5步操作总结成一句话就是：

 > 函数A的内部函数B被函数A外的一个变量 c 引用。

把这句话再加工一下就变成了闭包的定义：

 > 当一个内部函数被其外部函数之外的变量引用时，就形成了一个闭包。

以下是从别处看到的总结：

 > JavaScript闭包的形成原理是基于函数变量作用域链的规则 和 垃圾回收机制的引用计数规则。 
 
 > JavaScript闭包的本质是内存泄漏，指定内存不释放。（不过根据内存泄漏的定义是无法使用，无法回收来说，这不是内存泄漏，由于只是无法回收，但是可以使用，为了使用，不让系统回收） 
 
 > JavaScript闭包的用处，私有变量，获取对应值等
 
其他一句话简述闭包的方式：

 > 当function里面嵌套function时，内容的function可以访问外部function里面的变量，当你return的是内部function时，就是一个闭包。

## 闭包的用途
当我们需要在模块中定义一些变量，并希望这些变量一直保存在内存中但又不会 “污染” 全局的变量时，就可以用闭包来定义这个模块。

## 闭包的高级用法
这个组件的作用是：初始化一个容器，然后可以给这个容器添加子容器，也可以移除一个容器。

```
(function (document) {
    var viewport;
    var obj = {
        init: function(id) {
           viewport = document.querySelector('#' + id);
        },
        addChild: function(child) {
            viewport.appendChild(child);
        },
        removeChild: function(child) {
            viewport.removeChild(child);
        }
    }
    window.jView = obj;
})(document);
```

## 用闭包模拟私有方法--模块模式

```
var makeCounter = function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }  
};

var Counter1 = makeCounter();
var Counter2 = makeCounter();
console.log(Counter1.value()); /* logs 0 */
Counter1.increment();
Counter1.increment();
console.log(Counter1.value()); /* logs 2 */
Counter1.decrement();
console.log(Counter1.value()); /* logs 1 */
console.log(Counter2.value()); /* logs 0 */
```
以这种方式使用闭包，提供了许多与面向对象编程相关的好处 —— 特别是数据隐藏和封装。

## 在循环中创建闭包--一个常见错误

示例如下：
```
<p id="help">Helpful notes will appear here</p>
<p>E-mail: <input type="text" id="email" name="email"></p>
<p>Name: <input type="text" id="name" name="name"></p>
<p>Age: <input type="text" id="age" name="age"></p>
```
```
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}

setupHelp();
```
运行这段代码后，您会发现它没有达到想要的效果。无论焦点在哪个input上，显示的都是关于年龄的信息。

原因是赋值给 onfocus 的是闭包。这些闭包是由他们的函数定义和在 setupHelp 作用域中捕获的环境所组成的。这三个闭包在循环中被创建，但他们共享了同一个词法作用域，在这个作用域中存在一个变量item。当onfocus的回调执行时，item.help的值被决定。由于循环在事件触发之前早已执行完毕，变量对象item（被三个闭包所共享）已经指向了helpText的最后一项。

解决这个问题的一种方案是使用更多的闭包,特别是使用前面所述的函数工厂：
```
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function makeHelpCallback(help) {
  return function() {
    showHelp(help);
  };
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
  }
}

setupHelp();
```
这段代码可以如我们所期望的那样工作。所有的回调不再共享同一个环境， makeHelpCallback 函数为每一个回调创建一个新的词法环境。在这些环境中，help 指向 helpText 数组中对应的字符串。

另一种方法使用了匿名闭包：
```
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    (function() {
       var item = helpText[i];
       document.getElementById(item.id).onfocus = function() {
         showHelp(item.help);
       }
    })(); // 马上把当前循环项的item与事件回调相关联起来
  }
}

setupHelp();
```
避免使用过多的闭包，可以用let关键词：
```
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    let item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}

setupHelp();
```
这个例子使用let而不是var，因此每个闭包都绑定了块作用域的变量，这意味着不再需要额外的闭包。

## 性能考量

如果不是特定任务需要使用闭包，在其他函数中创建函数是不明智的。

因为闭包在处理速度和内存消耗方面对脚本性能具有负面影响。

在创建新的类或对象时，不建议将方法定义到对象的构造器中，防止每次构造器被调用，方法都会被重新赋值一次，通常是关联到对象的原型上。

示例：
```
function MyObject(name, message) {
  this.name = name.toString();
  this.message = message.toString();
  this.getName = function() {
    return this.name;
  };

  this.getMessage = function() {
    return this.message;
  };
}
```
上面的代码并未利用到闭包的好处，我们可以修改成如下：
```
function MyObject(name, message) {
  this.name = name.toString();
  this.message = message.toString();
}
MyObject.prototype = {
  getName: function() {
    return this.name;
  },
  getMessage: function() {
    return this.message;
  }
};
```
但我们不建议重新定义原型。可改成如下例子：
```
function MyObject(name, message) {
  this.name = name.toString();
  this.message = message.toString();
}
MyObject.prototype.getName = function() {
  return this.name;
};
MyObject.prototype.getMessage = function() {
  return this.message;
};
```
在前面的两个示例中，继承的原型可以为所有对象共享，不必在每一次创建对象时定义方法。

