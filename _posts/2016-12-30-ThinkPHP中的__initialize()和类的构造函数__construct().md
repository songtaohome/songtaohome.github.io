---
layout: post
title: ThinkPHP中的__initialize()和类的构造函数__construct()
categories: PHP
description: ThinkPHP中的__initialize()和类的构造函数__construct()
keywords: PHP, ThinkPHP
---

# ThinkPHP中的`__initialize()`和类的构造函数`__construct()`
网上有很多关于`__initialize()`的说法和用法，总感觉不对头，所以自己测试了一下。将结果和大家分享。不对请更正。
首先，我要说的是
1、`__initialize()`不是php类中的函数，php类的构造函数只有`__construct()`.
2、类的初始化：子类如果有自己的构造函数`__construct()`,则调用自己的进行初始化，如果没有，则调用父类的构造函数进行自己的初始化。
3、当子类和父类都有`__construct()`函数的时候，如果要在初始化子类的时候同时调用父类的`__constrcut()`，则可以在子类中使用`parent::__construct()`.
如果我们写两个类，如下：

```php
class Action{  
    public function __construct()  
    {  
        echo 'hello Action';  
    }  
}  
class IndexAction extends Action{  
    public function __construct()  
    {  
        echo 'hello IndexAction';  
    }  
}  
$test = new IndexAction;  
//output --- hello IndexAction
```

很明显初始化子类`IndexAction`的时候会调用自己的构造器，所以输出是'hello IndexAction'。
但是将子类修改为

```php
class IndexAction extends Action{  
    public function __initialize()  
    {  
        echo 'hello IndexAction';  
    }  
}
```

那么输出的是'hello Action'。因为子类IndexAction没有自己的构造器。
如果我想在初始化子类的时候，同时调用父类的构造器呢?

```php
class IndexAction extends Action{  
    public function __construct()  
    {  
        parent::__construct();  
        echo 'hello IndexAction';  
    }  
}  
```

这样就可以将两句话同时输出。
当然还有一种办法就是在父类中调用子类的方法。

```php
class Action{  
    public function __construct()  
    {
        if(method_exists($this,'hello'))  
        {  
            $this -> hello();  
        }  
        echo 'hello Action';  
    }  
}  
class IndexAction extends Action{  
    public function hello()  
    {  
        echo 'hello IndexAction';  
    }  
}
```

这样也可以将两句话同时输出。
而，这里子类中的方法`hello()`就类似于ThinkPHP中`__initialize()`。
所以，ThinkPHP中的`__initialize()`的出现只是方便程序员在写子类的时候避免频繁的使用`parent::__construct()`，同时正确的调用框架内父类的构造器，所以，我们在ThnikPHP中初始化子类的时候要用`__initialize()`,而不用`__construct()`,当然你也可以通过修改框架将`__initialize()`函数修改为你喜欢的函数名。 







