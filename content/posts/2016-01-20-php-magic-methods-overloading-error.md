---
title: "使用 PHP 的 __get 魔术方法时遇到的一个问题"
date: 2016-01-20T11:24:00+08:00
categories: ["PHP"]
tags: ["PHP"]
---
PHP官方的定义：

> PHP所提供的"重载"（overloading）是指动态地"创建"类属性和方法。
>  
> 当调用当前环境下未定义或不可见的类属性或方法时，重载方法会被调用。

<!--more-->

定义一个测试类`Foo`：

```php
class Foo
{

    public function getId()
    {
        return 'id';
    }

    public function getFoo()
    {
        return 'foo->' . $this->id;
    }

    public function __get($name)
    {
        $getter = 'get' . $name;
        if (method_exists($this, $getter)) {
            return $this->$getter();
        } else {
            throw new InvalidArgumentException("unknown property {$name}.");
        }
    }

}
```

调用：

```php
$foo = new Foo();
var_dump($foo->foo);
```

输出：

```php
foo->id
```

## 但是

如果我们将`getId`方法修改为：

```php
public function getId()
{
    return 'id->' . $this->foo;
}
```

这段代码并不会触发循环调用，而是报`Notice`错误：`Notice: Undefined property: Foo::$foo`。

上面这段代码调用的顺序应该是：`Foo::__get('foo')` -> `Foo::getFoo()` -> `Foo::__get('id')`，接下来这里又需要`Foo::__get('foo')`，但是PHP检查到已经在`Foo::__get('foo')`作用域中，因此直接调用了`Foo::$foo`报错。
