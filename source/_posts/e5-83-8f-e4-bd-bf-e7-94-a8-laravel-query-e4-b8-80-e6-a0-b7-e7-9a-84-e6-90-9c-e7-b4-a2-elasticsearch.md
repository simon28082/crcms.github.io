---
title: 像使用 Laravel Query 一样的搜索 Elasticsearch
tags:
  - elasticsearch
  - laravel
  - orm
url: 26.html
id: 26
categories:
  - PHP
date: 2018-05-29 21:56:01
---

需要使用到ES大数据引擎，但无奈，不管是官方包还是Github上都没有找到自己想要的，那还说什么呢，自己操刀来一份吧。 Github地址：[https://github.com/crcms/elasticsearch](https://github.com/crcms/elasticsearch) Composer和安装和Laravel下的加载

Version Matrix
--------------

Elasticsearch Version

crcms/elasticsearch Branch

>= 6.0

1.*

>= 5.0, < 6.0

0.*

Install
-------

    composer require crcms/elasticsearch
    

Laravel
-------

Modify `config / app.php`

    'providers' => [
        CrCms\ElasticSearch\LaravelServiceProvider::class,
    ]
    
    

Pubish

    php artisan vendor:publish --provider="CrCms\ElasticSearch\LaravelServiceProvider"
    

直接开始，示例如下：

Quickstart
----------

### Create

    Route::get('test/create',function(\CrCms\ElasticSearch\Builder $builder){
        $result = $builder->index('index')->type('type')->create([
            'key' => 'value',
        ]);
        dump($result);
    });
    
    

### Update

    Route::get('test/update',function(\CrCms\ElasticSearch\Builder $builder){
        $result = $builder->index('index')->type('type')->update('id',[
            'key' => 'value2',
        ]);
        dump($result);
    });
    
    

### Delete

    Route::get('test/delete',function(\CrCms\ElasticSearch\Builder $builder){
        $result = $builder->index('index')->type('type')->delete('id');
        dump($result);
    });
    
    

### Select

    Route::get('test/select',function(\CrCms\ElasticSearch\Builder $builder){
        $builder = $builder->index('index')->type('type');
    
        //SQL:select ... where id = 1 limit 1;
        $result = $builder->whereTerm('id',1)->first();
    
        //SQL:select ... where (key=1 or key=2) and key1=1
        $result = $builder->where(function (Builder $inQuery) {
            $inQuery->whereTerm('key',1)->orWhereTerm('key',2)
        })->whereTerm('key1',1)->get();
    
    });
    
    

### More

skip / take

    $builder->take(10)->get(); // or limit(10)
    $builder->offset(10)->take(10)->get(); // or skip(10)
    

term query

    $builder->whereTerm('key',value)->first();
    

match query

    $builder->whereMatch('key',value)->first();
    

range query

    $builder->whereBetween('key',[value1,value2])->first();
    

where in query

    $builder->whereIn('key',[value1,value2])->first();
    

logic query

    $builder->whereTerm('key',value)->orWhereTerm('key2',value)->first();
    

nested query

    $result = $builder->where(function (Builder $inQuery) {
        $inQuery->whereTerm('key',1)->orWhereTerm('key',2)
    })->whereTerm('key1',1)->get();
    

更多的使用方法详见[Github](https://github.com/crcms/elasticsearch) 最后：如果对您有用请给个Star吧，更多的是欢迎拍砖，支持开源。