---
title: PHP实现无限级分类 -- path标识
date: 2018-05-16 13:30:34
tags:
  - 无限级分类
category:
  - PHP
---

在实际项目中经常要用到无限级分类，如多级分类、导航表等。PHP 实现无限级分类通常有两种实现方式，一种是利用`path`字段（pid+id）标识当前层级；另一种是利用递归循环`pid`的方式。此处介绍前种方式。

<!--more-->

# PHP 实现无限级分类 -- `path`标识

## 1、数据库设计

```sql
--创建分类表
create table `b_category`(
`id` int primary key not null auto_increment,
`cat_name` varchar(20) not null default '',
`cat_description` text default '',
`level` int not null default 0 comment '等级',
`pid` int comment '父级id',
`path` varchar(10) comment 'pid+,+id标识，用于无限级分类'
);
```

## 2、PHP 代码实现

```php
$data = $m->field("*, concat(path,',',id) as paths ")->order('paths')->select();

foreach($data as $k=>$v ){
    $data[$k]['name'] = str_repeat("   ", $v['level']) . $v['name'];
}
```

## 3、视图层显示

```php
<div class="row cl">
  <label class="dorm-label col-2">描述：</lable>
  <div class="formControls col-5">
      <span class="select-box">
         <select class="select" size="1" name="pid">
            <option value="0" selected>顶级分类</option>
            {foreach $data as $item}
               <option value="{$item.id}">{$item.name}</option>
            {/foreach}
         </select>
      </span>
   </div>
  </div>
```
