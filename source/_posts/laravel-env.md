---
title: Laravel env 下特殊字符的坑
date: 2019-06-15 02:09:50
tags:
  - laravel
  - env
---

## 简述

今天在部署服务器时，线上数据库使用了特殊字符，结果导致`mysql`连接一直是出错的状态。
通过`my-cli`连接却是OK，于是想到是否是帐号密码错误，打印下`env`或`config`，发现因为出现了`#`字符，后面被自动截断。

## 解决
第一反应是对于特殊字符如`# \`，应该加上`""`，测试下确实得到解决。

## 调试
首先想到肯定是在加载env，定位`Bootstrap/LoadEnvironmentVariables`：
```php
        try {
            $this->createDotenv($app)->safeLoad();
        } catch (InvalidFileException $e) {
            $this->writeErrorAndDie($e);
        }
```

继续往下，定位`safeLoad`
```php
    public function safeLoad()
    {
        try {
            return $this->loadData();
        } catch (InvalidPathException $e) {
            // suppressing exception
            return [];
        }
    }

```

再通过`loadData`往下：
```php
    public function load()
    {
        return $this->loadDirect(
            self::findAndRead($this->filePaths)
        );
    }
```

最终通过`processEntries`定位，找到处理`value`的核心方法

```php
private static function parseValue($value)
    {
        if ($value === null || trim($value) === '') {
            return $value;
        }

        return array_reduce(str_split($value), function ($data, $char) use ($value) {
            switch ($data[1]) {
                case self::INITIAL_STATE:
                    if ($char === '"' || $char === '\'') {
                        return [$data[0], self::QUOTED_STATE];
                    } elseif ($char === '#') {
                        return [$data[0], self::COMMENT_STATE];
                    } else {
                        return [$data[0].$char, self::UNQUOTED_STATE];
                    }
                case self::UNQUOTED_STATE:
                    if ($char === '#') {
                        return [$data[0], self::COMMENT_STATE];
                    } elseif (ctype_space($char)) {
                        return [$data[0], self::WHITESPACE_STATE];
                    } else {
                        return [$data[0].$char, self::UNQUOTED_STATE];
                    }
                case self::QUOTED_STATE:
                    if ($char === $value[0]) {
                        return [$data[0], self::WHITESPACE_STATE];
                    } elseif ($char === '\\') {
                        return [$data[0], self::ESCAPE_STATE];
                    } else {
                        return [$data[0].$char, self::QUOTED_STATE];
                    }
                case self::ESCAPE_STATE:
                    if ($char === $value[0] || $char === '\\') {
                        return [$data[0].$char, self::QUOTED_STATE];
                    } else {
                        throw new InvalidFileException(
                            self::getErrorMessage('an unexpected escape sequence', $value)
                        );
                    }
                case self::WHITESPACE_STATE:
                    if ($char === '#') {
                        return [$data[0], self::COMMENT_STATE];
                    } elseif (!ctype_space($char)) {
                        throw new InvalidFileException(
                            self::getErrorMessage('unexpected whitespace', $value)
                        );
                    } else {
                        return [$data[0], self::WHITESPACE_STATE];
                    }
                case self::COMMENT_STATE:
                    return [$data[0], self::COMMENT_STATE];
            }
        }, ['', self::INITIAL_STATE])[0];
    }
```

可以看到对于`#`和`\`的处理。

关于`array_reduce`函数，请[参考文档](https://www.php.net/manual/zh/function.array-reduce.php)
