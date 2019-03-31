---
title: 推荐：超好用的 Laravel Repository 包
tags:
  - laravel
  - PHP
  - Repository
url: 122.html
id: 122
categories:
  - PHP
date: 2018-07-04 22:22:44
---

什么是Repository模式，如何使用Repository模式
--------------------------------

这里就不再啰嗦了，请参见以下几个链接 [如何使用 Repository 模式?](http://oomusou.io/laravel/repository/) [关于 Repository 的设计模式](https://laravel-china.org/articles/6232/about-the-design-patterns-of-repository) [laravel-china.org搜索](https://laravel-china.org/search?q=repository)

我的使用历程
------

### 原由

`MVC`在如今仍然是流行趋势，但多数框架都只提供基础的MVC架构。 几年前在开发中我们经常会遇到问题Model过于臃肿，写着写着就会变成类似于万能类，最后面的人就真成了接盘侠了。 很不幸我就是其中之一。后来我就一直在思考如何才能让`Model`看起来清爽，功能更加单一简洁。（当时并不知道`Repository`），终于开始重构。一把心酸泪。。。。。最多的是组合和Trait

### 使用Laravel

最开始接触Laravel就是感觉它的文档清爽，以为是个简单的框架，结果不小心一入坑，才发现被它的外表给欺骗了。 但却也为此深深爱上了它，是啊，这不就是我一直追求的吗？无限的灵活性，可替换，越研究代码越发现处处都是精髓。 但在`Laravel`中也不可避免的基础`MVC`模式，上述问题仍然存在。

### 初期使用

一直以为我都遵循一个核心：以仓库层为处理数据基础，为`Serivce`和`Controller`等提供数据供给，仓库需要的原始数据则通过`Model`中获取。这样可以完全分离`Model`和`Controller`的依赖。 最开始在`Laravel`中使用是通过定义大量的`RepositoryInterface`来注入，`bind`，实现具体的`Repository`工作类。 这是理想的使用方法可替换性很强。

### 遇到的问题

*   实际开发过程中`Repository`基本不会被替换，无数的Interface带来的规范，也带来了开发的麻烦。
*   在使用Repository模式中我们不断的注入`Model`，每个方法都需要直接Model来进行一次次的查询数据集，却失去了在外层链式调用的便捷性（这其实并不合理，但存在即有原由）。

### 中间的折中

后来索性在开发中我们去掉了`Interface`的约束，直接作用功能类来注入使用，此时简洁性和便捷性大大的提高，如果非要替换仍然`bind`可以解决问题。这样的开始一直持续很长时间。但是像链接调用仍然没有解决，为些我们开发出了新的仓库包。[https://github.com/crcms/repository](https://github.com/crcms/repository)

### 再次轮回

开始玩微服务，开始分离代码，当然就离不开`RPC`，十分庆幸我们使用了`Repository`模式，通过开启对应的`Rpc Repository`，我们可以很快进行本地`Repository`切换，以`Interface`来约束。

便捷的Repository包
--------------

### 基础示例

    class TestRepository extends AbstractRepository
    {
        /**
         * @var array
         */
        protected $guard = [
            'id', 'title','other'
        ];
    
        /**
         * @return TestModel
         */
        public function newModel(): TestModel
        {
            return app(TestModel::class);
        }
    
        /**
         * @param int $perPage
         * @return LengthAwarePaginator
         */
        public function paginate(AbstractMagic $magic = null, int $perPage = 15): LengthAwarePaginator
        {
            $query = $this->where('built_in', 1);
    
            if ($magic) {
                $query->magic($magic);
            }
    
            return $query->orderBy($this->getModel()->getKeyName(), 'desc')->paginate($perPage);
        }
    
        /**
         * @param int $name
         * @param int $title
         */
        public function updateName(string $name, string $title)
        {
            $this->getModel()->where('name', $name)->update(['title' => $title]);
        }
    
    }
    

### 超好用的Magic方法

在多条件搜索中，肯定会存在大量的判断，优雅度太低，如：

    if($request->input('username')) {
        $query->where('username',$username)
    }
    
    if($request->input('email')) {
        $query->where('email',$email)
    }
    
    .......
    

但通过`QueryMagic`方法我们可以轻松优雅解决这些问题，示例： 创建Magic类

    use CrCms\Repository\AbstractMagic;
    use CrCms\Repository\Contracts\QueryRelate;
    
    class TestMagic extends AbstractMagic
    {
        /**
         * @param QueryRelate $queryRelate
         * @param int $id
         * @return QueryRelate
         */
        protected function byName(QueryRelate $queryRelate, string $name)
        {
            return $queryRelate->where('name', $name);
        }
    
        /**
         * @param QueryRelate $queryRelate
         * @param string $title
         * @return QueryRelate
         */
        protected function byTitle(QueryRelate $queryRelate, string $title)
        {
            return $queryRelate->where('title', 'like', "%{$title}%");
        }
    
        /**
         * @param QueryRelate $queryRelate
         * @param int $id
         * @return QueryRelate
         */
        protected function byId(QueryRelate $queryRelate, int $id)
        {
            return $queryRelate->where('id', $id);
        }
    }
    

使用`Magic`（这里只是简单示例）： `public function paginate(array $condition, int $perPage = 15): LengthAwarePaginator { return $query->magic(new TestMagic($condition))->orderBy($this->getModel()->getKeyName(), 'desc')->paginate($perPage); }`

### 更多

开发此包的原因是在这之前我并示找到我想要的（适合我的）兼具Model的灵活性以及数据仓库的分离模式，所以为此开发了这个仓库包。目前此包已经使用在好几个项目中目前运行良好。 后面还打算兼容`TP`以及`Yii`等使用率高的框架，暂时只支持`Laravel` 更多详情，请移步github：[https://github.com/crcms/repository](https://github.com/crcms/repository)

最后
--

哈哈，请原谅我着急的文本描述，希望对需要的人以及面临和我曾经一样困惑的人有所帮助。