---
title: PHP协程
url: 22.html
id: 22
categories:
  - PHP
tags:
---

<?php

//生成器和协程



class Test implements Iterator
{
    protected $data;

    protected $cursor = 0;

    public function __construct(array $data)
    {
        $this->data = $data;
    }

    public function current()
    {
        return $this->data\[$this->cursor\];
    }

    public function next()
    {
        return $this->cursor += 1;
    }

    public function key()
    {
        return $this->cursor;
    }

    public function valid()
    {
        return isset($this->data\[$this->cursor\]);
    }

    public function rewind()
    {
        $this->cursor = 0;
    }


}

//
//foreach (new Test(\[1,2,3\]) as $key => $value) {
//    echo strval($key) , '===', $value,'<br>';
//}


//$test = new Test(\[1,2,3\]);

//echo $test->current(),'<br>';
//$test->next();
//echo $test->current(),'<br>';
//$test->next();
//echo $test->current(),'<br>';


//function x() {
//    yield new Test(\[1,2,3\]);
//}


//function generate()
//{
//    yield '123';
//    yield '456';
//}
//
////返回的是一个生成器对象，生成器是属于迭代器接口
//$generate = generate();
//
//echo $generate->current();
//$generate->next();
//echo '<br>';
//echo $generate->current();
//$generate->next();
//echo '<br>';
//echo $generate->current();


//发送数据至生成器
//function generate()
//{
//    yield ('yield '. yield);
//}
//
//$gen = generate();//$gen迭代器被创建的时候一个renwind()方法已经被隐式调用
//var_dump($gen->current());//第一次yield后面并没有任何值，所以为Null
//$gen->send('123');//当发送123时yield接收了值，所以值为 'yield . 123'，并且指针自动下移
//echo $gen->current();//再次执行的时候遇到了yield (xxx) ，所以返回后面的值

function generate()
{
    yield '123';
    yield '456';
}
$gen = generate();
//var_dump($gen->current());
$gen->send('xxx');
var_dump($gen->current());

//$gen迭代器被创建的时候一个rewind()方法已经被隐式调用
//相当于$gen->rewind();,并且忽略了他的返回值，可以直接使用$gen->current()获得
//当再执行send的时候，指针+1所以获取的是下一个yield 即 456