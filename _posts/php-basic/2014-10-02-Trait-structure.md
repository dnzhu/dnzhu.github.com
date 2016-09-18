---
layout: post
title: "php5.4中trait语法结构"
description: "有点多继承的意思"
category: php-basic
tags: []
---

> 众所周知，在php的语法结构中，没有多继承这个概念。那么在php5.4以后的版本中，新增了一种结构trait。

### 使用trait的好处：

    1.增加了代码的复用性。trait就是为了减少单继承语言的限制，使开发人员能够自由的在不同的层次结构的类中复用method。
    
    2.避免了传统多继承类臃肿的问题。在一定程度上，减少了多继承的复杂性。

    3.能够水平扩展功能。增加的扩展功能的灵活性。
---

### 优先级

优先顺序是当前类中的方法会覆盖 trait 方法，而 trait 方法又覆盖了基类中的方法。 

官方demo：
```
<?php
    class Base {
        public function sayHello() {
            echo 'Hello ';
        }
    }

    trait SayWorld {
        public function sayHello() {
            parent::sayHello();
            echo 'World!';
        }
    }

    class MyHelloWorld extends Base {
        use SayWorld;
    }

    $o = new MyHelloWorld();
    $o->sayHello();
?>


输出结果：
    Hello World!

```
----

### 多个 trait

通过逗号分隔，在 use 声明列出多个 trait，可以都插入到一个类中。

```
<?php
    trait Hello {
        public function sayHello() {
            echo 'Hello ';
        }
    }

    trait World {
        public function sayWorld() {
            echo 'World';
        }
    }

    class MyHelloWorld {
        use Hello, World;
        public function sayExclamationMark() {
            echo '!';
        }
    }

    $o = new MyHelloWorld();
    $o->sayHello();
    $o->sayWorld();
    $o->sayExclamationMark();
?>


输出结果：

Hello World!

```
----

###　冲突的解决

如果两个 trait 都插入了一个同名的方法，如果没有明确解决冲突将会产生一个致命错误。

为了解决多个 trait 在同一个类中的命名冲突，需要使用 insteadof 操作符来明确指定使用冲突方法中的哪一个。

以上方式仅允许排除掉其它方法，as 操作符可以将其中一个冲突的方法以另一个名称来引入。 

```

<?php
trait A {
    public function smallTalk() {
        echo 'a';
    }
    public function bigTalk() {
        echo 'A';
    }
}

trait B {
    public function smallTalk() {
        echo 'b';
    }
    public function bigTalk() {
        echo 'B';
    }
}

class Talker {
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
    }
}

class Aliased_Talker {
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
        B::bigTalk as talk;
    }
}
?>

```
----

### 修改方法的访问控制

使用 as 语法还可以用来调整方法的访问控制。 

```
<?php
    trait HelloWorld {
        public function sayHello() {
            echo 'Hello World!';
        }
    }

    // 修改 sayHello 的访问控制
    class MyClass1 {
        use HelloWorld { sayHello as protected; }
    }

    // 给方法一个改变了访问控制的别名
    // 原版 sayHello 的访问控制则没有发生变化
    class MyClass2 {
        use HelloWorld { sayHello as private myPrivateHello; }
    }
?>

```




