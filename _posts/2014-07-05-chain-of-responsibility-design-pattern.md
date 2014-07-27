---
layout: post
title: 责任链模式（Chain of responsibility pattern）
date: 2014-07-05 8:35:37
disqus: y
---

责任链模式是一种行为型模式，它包含了一些命令对象和一系列的处理对象。每一个处理对象决定它能处理哪些命令对象，它也知道如何将它不能处理的命令对象传递给该链中的下一个处理对象。该模式还描述了往该处理链的末尾添加新的处理对象的方法。

主要角色

抽象责任(Responsibility）角色：定义所有责任支持的公共方法。
具体责任(Concrete Responsibility)角色：以抽象责任接口实现的具体责任
责任链(Chain of responsibility)角色：设定责任的调用规则


实例:


{% highlight PHP %}

<?php
abstract class Responsibility { // 抽象责任角色
    protected $next; // 下一个责任角色

    public function setNext(Responsibility $l) {
        $this->next = $l;
        return $this;
    }
    abstract public function operate(); // 操作方法
}

class ResponsibilityA extends Responsibility {
    public function __construct() {}
    public function operate(){
        if (false == is_null($this->next)) {
            $this->next->operate();
        }
    };
}

class ResponsibilityB extends Responsibility {
    public function __construct() {}
    public function operate(){
        if (false == is_null($this->next)) {
            $this->next->operate();
        }
    };
}

$res_a = new ResponsibilityA();
$res_b = new ResponsibilityB();
$res_a->setNext($res_b);
?>

{% endhighlight %}