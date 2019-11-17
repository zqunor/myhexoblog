---
title: 【Laravel】Command 脚本任务实现
date: 2019-04-23 22:21:01
tags:
  - laravel
category:
  - 【PHP框架】
---


大多数项目在业务发展过程中，都需要修复历史数据和定时任务来完成一些业务逻辑，这部分通常都需要通过脚本来完成，一般的框架爱也都提供这部分的功能，学习并使用是工作中的基本要求。

<!--more-->

## 基本流程

**commands模式**运行脚本定时任务基本流程：
1. 在 `app/Console/Commands/` 目录下创建脚本任务文件
2. 在`app/Console/Kernel.php` `$commands`数组中添加新建的脚本类
3. 在`app/Console/Kernel.php` `schedule()`方法中添加脚本定时任务命令

## 具体实现

### 创建脚本文件
`app/Console/Commands/QingShan/commandQingshan.php`

```php
<?php
namespace App\Console\Commands\QingShan;

use Illuminate\Console\Command;

class commandQingshan extends Command
{
    // 自定义脚本命令签名
    protected $signature = 'qingshan:commandQingshan';

    // 自定义脚本命令描述
    protected $description = '这里是脚本命令的描述qingshan';

    // 创建一个新的命令实例
    public function __construct()
    {
        parent::__construct();
    }

    //具体执行的业务内容
    public function handle()
    {
    }
}
```

### 注册脚本

在`app/Console/Kernel.php`  `$commands`数组中追加新建的脚本类

```php
protected $commands = [
    'BasicIT\LumenVendorPublish\VendorPublishCommand',
    Commands\QingShan\commandQingshan::class
]
```

### 执行脚本

####  查看脚本命令调用方式
1. 在项目目录下执行下面的命令，查看当前可以执行的命令
```bash
> php artisan list
```

在`Available commands`下会有一列：
```bash
qingshan
    qingshan:commandQingshan     这里是脚本命令的描述qingshan
```

2. 执行脚本命令
```bash
> php artisan qingshan:commandQingshan
```

### 添加到定时任务

在`app/Console/Kernel.php` `schedule()`方法中添加脚本定时任务命令

```php
// 设置commandQingshan脚本为每天15：00自动执行
protected function schedule(Schedule $schedule)
{
    $schedule->command('qingshan:commandQingshan')->dailyAt('15:00');

}
```

参考资料：

[Larave5.8中文文档--Artisan 命令行](https://learnku.com/docs/laravel/5.8/artisan/3913)
