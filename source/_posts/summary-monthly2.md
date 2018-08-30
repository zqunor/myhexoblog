---
title: 【总结】两个月的工作任务总结
date: 2018-06-20 08:33:10
tags:
  - 工作总结
category:
  - 工作总结
---

从 2018.4.2 工作以来，不知不觉已经工作两个多月，并在昨天约谈从这个月开始转正。从刚开始的自己学习，到逐渐接触公司的项目，并完成交付的功能模块，学到了很多，也发现了自己存在的不足，所以作此总结，激励自己，并鞭策自己，不骄不躁，不悲不怒，养成良好的心态，并坚持学习，保持热情！

<!--more -->

# 一、功能模块介绍

## 1.订单系统

- 数据库关联 [6 张数据库的关联查询]
  - 订单信息列表
    - 订单信息表 --- order
    - 支付方式表 --- paymentmethod
    - 订单详情表 --- orderlist
    - 优惠券信息表 --- coupon
    - 快递信息表 --- delivery
    - 商城信息表 --- shopcategory
  - 子订单信息列表 [一个订单有多个商品]
    - 订单商品表 --- ordergoods
  - 订单规格信息列表 [一个订单的一个商品有多个规格属性]
    - 订单规格表 --- orderspec
- 订单状态处理
- 订单导出到 excel

## 2.商品入库

### (1) 功能列表

- 关键词管理
- 选品管理
- 待入库商品[同时操作 2 个数据库，9 张数据表]

  - COD 数据库

    - 系统商品表--- offer [***套餐处理]
    - 库存表 --- stock
    - 库存规格表--- stocksku [****规格处理]
    - 库存日志表--- stocklog
    - 库存分类表--- stockcategory
    - 库存仓库关联表--- stockskuwarehouse
    - 库存商品关联表--- stockoffer
    - 用户商品关联表--- useroffer

  - GOODS 数据库
    - 待入库商品状态更新--- goods

- 运费模板管理

### (2) 完成时长：两个半星期

### (3) 难点整理

- 数据表相互关联关系的理解。

        业务需求不懂，所以直接接触时不知道各个数据表之间的关系，以及需要如何处理已经有的信息

- 库存规格处理和套餐处理的结构

        逻辑较复杂，加上第一次实现时没有对功能进行切分，代码混杂，耦合度高，导致后期需求调整时，对需要调整的需求无从下手

## 3.sphinx 关键词检索【商城选品】

> 集成开发环境由 phpStudy 转向 UPUPW ANK(后者有 sphinx 服务管理)

### (1)功能列表

1). php 开启 sphinx 扩展

```ini
# php.ini
extension=php_sphinx.dll
```

2). sphinx.conf 配置项配置

> source [type sql_host sql_user sql_pass sql_db sql_query_pre sql_query sql_attr_uint]

```ini
# 设置查询条件
sql_attr_uint   = status
```

> index [source path charset_table ngram_len ngram_chars]

3).sphinx 第三方类库的引入[ThinkPHP5 置于 extend 目录下]

- SphinxClient 类的使用

- 设置匹配记录条数的限制 (默认只检索 20 条）

```php
$sphinx->setLimits(0, 1000);
```

- 匹配模式的对比

```php
// 设置全文查询的匹配模式
SphinxClient::setMatchMode
```

| 模式                | 描述                                        |
| ------------------- | ------------------------------------------- |
| SPH_MATCH_ALL       | 匹配所有查询词（默认模式）.                 |
| SPH_MATCH_ANY       | 匹配查询词中的任意一个.                     |
| SPH_MATCH_PHRASE    | 将整个查询看作一个词组，要求按顺序完整匹配. |
| SPH_MATCH_BOOLEAN   | 将查询看作一个布尔表达式.                   |
| SPH_MATCH_EXTENDED  | 将查询看作一个 Sphinx 内部查询语言的表达式. |
| SPH_MATCH_FULLSCAN  | 使用完全扫描，忽略查询词汇.                 |
| SPH_MATCH_EXTENDED2 | 类似 SPH_MATCH_EXTENDED ，并支持评分和权重. |

- 检索时设置条件过滤

```php
$sphinx->SetFilter($filterkey, $filtervalue);
```

### 2.完成时长： 两天

### 3.难点介绍

- 对检索的结果设置检索条件[根据 status 查询]

        - 对sphinx配置项不熟 =》 对某些参数的设定模棱两可，所以后期删除某些看似非必要配置项时导致功能不能实现(sphinx.conf 中source的配置项`sql_attr_uint`删除导致根据status查询的结果不正确)
        - 对SphinxClient的方法和属性不熟 =》 sphinx设置过滤的使用方法： $sphinx->SetFilter($filterkey, $filtervalue)

- 中文检索的支持

        - 起初查找的资料都是介绍需要使用sphinx的coreseek扩展进行中文分词检索支持的，从而被带偏的一直查找coreseek的相关资料，而coreseek的官方网站又一直无法访问，导致无从下手，直到后来不断的调整配置，以及查找资料，才知道原来当前版本的sphinx已经默认支持中文分词的检索了，不再需要coreseek等其他扩展。使用索引源`sql_query_pre`设置编码和索引配置项`ngram_len` `ngram_chars`即可支持中文检索。

## 4.权限管理

**RBAC**(Role-Basied Access Control) 基于角色的权限访问控制--**用户-角色-权限**。

权限控制的实现 =》 url 形式 [`module/action`]

### (1) 功能介绍

1). 用户管理

    - 用户列表
    - 用户创建
    - 用户编辑 [ 编辑用户的基本信息 + 分配用户角色 ]

2). 角色管理

    - 角色列表
    - 角色创建
    - 角色编辑 [ 编辑角色的基本信息 + 分配角色权限 ]

3). 权限管理

    - 权限列表

4). 数据表(5 张)

- 用户信息表 --- account
- 角色信息表 --- role
- 权限信息表 --- auth
- 用户-角色关联表 --- account_role
- 角色-权限关联表 --- role_auth

### (2) 完成时长：四天

### (3) 难点介绍

- 角色分配和权限分配(封装到共用方法)

  - **已经分配的角色|权限** --》 更新 --- array_intersect
  - 需要**删除**的角色|权限 --》 删除 --- array_diff
  - 需要**新增**的角色|权限 --》 新增 --- array_diff

# 二、自我反思

- 能在完全不了解已有项目和已有功能的情况下，适应原有的代码风格，并理清所有逻辑和业务需求，有效完成功能任务，实践能力和思考能力达到了一个入门级程序员的水准（很 low）.

- 能认识到自己编写的代码有一定的不足[严谨性有待提升]

# 三、经验总结

## 1.如何快速理清需求？

### (1) 看数据表，数据字段之间名称的联系

- 不要低估别人的数据表设计能力（如果已经有的话）
- 有效的利用工具
- 看哪些地方有用到这些数据表（数据表是为业务需求所设计）

### (2) 看已有的功能代码，代码的实现逻辑

- 不要低估别人的代码能力（如果已经有的话）
- 先整体后细节，业务逻辑不明白会使得对细节的理解有一定的难度，但整体上先大致的了解一个接口都干了哪些事会帮助理清逻辑。
- 对功能相同或相似的代码可以理解后直接使用，以效率为主

### (3) 走心的记住之前理出来的逻辑（快速的重要前提）

- 不要总是分神，养成专注和集中注意力的能力（保持思考）
- 不要情绪化，影响思考能力和专注度
- 注意休息

## 2.如何保持积极、热情？

### (1) 保持学习

- 养成沉浸式学习的能力，学进去，并消化理解，为实际工作中所用
- 不要娱乐化，娱乐八卦信息要多少有多少，要多乱有多乱，看了除了浪费时间没有其他任何益处，不如培养自己的兴趣，提升自己的内涵
- 多思考，多动手，实践是检验理论和巩固知识最有效的方式
- 学习带来的思维敏捷可以很大程度提升自我认可度

### (2) 多反思自己，少批评别人

- 被自己惯出来的优越感会让自己忘乎所以，尊重别人，就是对自己最大的尊重
- 让自己真正意义上的变优秀，才是真正值得自豪的事情，而不是通过对别人指指点点
- 理性待人，理性对事

### (3) 尊重别人

- 对别人给予的负面言论有则改之，无则加勉，无需上升到个人情绪的层面
- 别人错误的指责，表气（更加影响效率和状态）
- 一切为了提升自己，成为更好的自己

# 四、代码展示

## 1.PHPExcel 使用--导出 excel

```php
/**
 * 导出exml
 * expTitle 表格标题
 * expCellName 表格单元格列名
 * expTableData 数据
 * setWidth 单元格列宽
 */
function ExportExcel($expTitle, $expCellName, $expTableData, $setWidth, $output = 0, $i = 1) {
    $xlsTitle = iconv('utf-8', 'gb2312', $expTitle); //文件名称
    $fileName = $expTitle . date('_YmdHis'); //or $xlsTitle 文件名称可根据自己情况设定

    $cellNum = count($expCellName);
    $dataNum = count($expTableData);
    $objPHPExcel = new \PHPExcel();

    $cellName = array('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', 'AA', 'AB', 'AC', 'AD', 'AE', 'AF', 'AG', 'AH', 'AI', 'AJ', 'AK', 'AL', 'AM', 'AN', 'AO', 'AP', 'AQ', 'AR', 'AS', 'AT', 'AU', 'AV', 'AW', 'AX', 'AY', 'AZ');
    $objPHPExcel->getActiveSheet(0)->mergeCells('A1:' . $cellName[$cellNum - 1] . '1'); //合并单元格
    $objPHPExcel->setActiveSheetIndex(0)->setCellValue('A1', '' . $expTitle . '  Export time:' . date('Y-m-d H:i:s'));
    $objPHPExcel->getDefaultStyle()->getFont()->setName('Arial');
    $objPHPExcel->getActiveSheet(0)->getRowDimension(1)->setRowHeight(40);
    $objPHPExcel->getActiveSheet()->getStyle("A1")->getFont()->setSize(20);

    for ($i = 0; $i < $cellNum; $i++) {
        $objPHPExcel->setActiveSheetIndex(0)->setCellValue($cellName[$i] . '2', $expCellName[$i][1]);
    }
    for ($i = 0; $i < $dataNum; $i++) {
        for ($j = 0; $j < $cellNum; $j++) {
            //数据
            $objPHPExcel->getActiveSheet(0)->setCellValue($cellName[$j] . ($i + 3), $expTableData[$i][$expCellName[$j][1]]);

            for ($j = 0; $j < $cellNum; $j++) {
                $objPHPExcel->getActiveSheet(0)->setCellValue($cellName[$j] . ($i + 3), $expTableData[$i][$expCellName[$j][0]])->getRowDimension($i + 3);
                foreach ($setWidth as $setWidthk => $setWidthvalue) {
                    $objPHPExcel->getActiveSheet()->getColumnDimension($setWidthk)->setWidth($setWidthvalue);
                }
            }
        }
    }
    if (empty($output)) {
        header('pragma:public');
        header('Content-type:application/vnd.ms-excel;charset=utf-8;name="' . $xlsTitle . '.xls"');
        header("Content-Disposition:attachment;filename=$fileName.xls");
        $objWriter = \PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');
        $objWriter->save('php://output');
    } else {
        $objWriter = \PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');
        $file = ROOT_PATH . 'public' . DS . 'uploads' . DS . 'temp' . DS . 'keywordstemp.' . xls;
        $objWriter->save($file);
    }
    echo '<script>window.close();</script>';
}
```

## 2.Sphinx 在 PHP 项目中的应用

### 1). Sphinx 配置

```ini
source keyword
{
    # 配置数据库基本信息
    charset_type    = mysql
    sql_host        = 127.0.0.1
    sql_user        = root
    sql_pass        = 123123
    sql_dbcharset_type  = goods
    sql_query_pre   = SET NAMES utf8

    sql_query       = \
        SELECT id,status,keyword \
        FROM goods

    # 设置查询条件
    sql_attr_uint   = status
}

index keyword
{
    # 指定配置的source
    sourcecharset_type  = keyword
    # 指定索引数据文件存储路径
    pathcharset_type    = /usr/local/spinx/var/data/keyword
    # 指定字符集(新版已废除)
    charset_type    = utf-8
    # 检索的字符串编码识别范围
    charset_table   = 0..9, A..Z->a..z, _, a..z, U+410..U+42F->U+430..U+44F, U+430..U+44F
    # 支持中文检索(值为1时支持中文，0时不支持中文)
    ngram_len       = 1
    # 中文检索的字符编码范围
    ngram_chars     = U+3000..U+2FA1F
}
```

### 2). 公共方法封装

```php
/**
 * sphinx搜索
 *
 * @param string $key 查询的字符串
 * @param string $indexFile 索引文件
 * @param string $filterkey 过滤字段名
 * @param string $filtervalue 过滤字段值
 * @return mixed
 */
function sphinx($key, $indexFile, $filterkey = '', $filtervalue = [])
{
    $sphinx = new \SphinxClient;
    $sphinx->setServer("localhost", 9312);

    // 设置检索时的过滤条件
    if (!empty($filterkey)) {
        $filterRes = $sphinx->SetFilter($filterkey, $filtervalue);
        if (!$filterRes) {
            return false;
        }
    }

    // 作为数组返回
    $sphinx->SetArrayResult(true);
    // 匹配格式  任意匹配
    $sphinx->setMatchMode(SPH_MATCH_ALL);
    $sphinx->setMaxQueryTime(3);
    $sphinx->setLimits(0, 1000);

    // 参数1：查询关键字， 参数2：索引名（所有索引用*）
    $result = $sphinx->query($key, $indexFile);

    if (empty($result['matches'])) {
        return null;
    }

    return $result;
}
```

# 五、项目展示

## 1.后台概览图

![](https://ws1.sinaimg.cn/large/005EgYNMgy1fup6bzr8whj31gm0l0q3t.jpg)
