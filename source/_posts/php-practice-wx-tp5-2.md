---
title: 【实战】tp5+小程序(二)--接口编写
date: 2018-07-5 16:26:58
tags:
  - thinkphp5
  - 微信小程序
category:
  - 【PHP框架】
---

ThinkPHP5 从入门到深入学习，结合实战项目深入理解 ThinkPHP5 的特性和使用方法。编写完成简单的基于 RESTFul 接口，实现相应功能，掌握控制器、模型、异常处理、数据校验的使用。

<!--more-->

## 8-1 Banner 相关表分析（数据表关系分析）

- banner 位的数据表`banner`

`banner(id, name, description, delete_time, update_time)`

- 每个 banner 位图片的数据表`banner_item`

`banner_item(id, img_id, key_word, type, delete_time, banner_id, update_time)`

- 图片表`image`

`image(id, url, from, delete_time, update_time)`

    banner <=> banner_item 一对多关系
    image <=> banner_item 一对一关系

## 8-2 模型关联--定义关联与查询关联

> model/Banner.php

```php
// 创建关联方法
public function items()
{
    // 参数1：关联模型的模型名
    // 参数2：关联模型的外键
    // 参数3：当前模型的主键
    // hasMany：表示是一对多的关系
    return $this->hasMany('BannerItem', 'banner_id', 'id');
    // 【需要创建BannerItem模型类文件】
}
```

> controller/Banner.php

```php
//with()方法，设置关联模型
$banner = BannerModel::with('items')->find($id);
//等价于 $banner = model('banner')->with('items')->find($id);
```

执行结果时会自动附件关联信息。

```json
{
  "id": 1,
  "name": "首页置顶",
  "description": "首页轮播图",
  "delete_time": null,
  "update_time": "1970-01-01 08:00:00",
  "items": [
    {
      "id": 1,
      "img_id": 65,
      "key_word": "6",
      "type": 1,
      "delete_time": null,
      "banner_id": 1,
      "update_time": "1970-01-01 08:00:00"
    }
  ]
}
```

## 8-3 模型关联 --嵌套关联查询

1.多个关联表

    with(['items','item2'])

2.命令行创建模型（自动完成模板）

    php think make:model api/Image

3.banner 嵌套 items，现在需要给 items 嵌套 img 相关信息

多层嵌套使用方法：

    with(['items', 'items.img'])

4.具体实现：

- `model/BannerItem.php`

```php
public function img()
{
    // BannerItem和Image是一对一的关系，使用的方法是belongsTo
    return $this->belongsTo('Image', 'img_id', 'id');
    //【需要创建Image模型类文件】
}
```

也可以在`model/Image.php`中定义,实现的效果是一样的。

- `controller/Banner.php`

```php
// 多层嵌套的使用
$banner = BannerModel::with(['items', 'items.img'])->find($id);
```

5.实现效果如下：

```json
{
  "id": 1,
  "name": "首页置顶",
  "description": "首页轮播图",
  "delete_time": null,
  "update_time": "1970-01-01 08:00:00",
  "items": [
    {
      "id": 1,
      "img_id": 65,
      "key_word": "6",
      "type": 1,
      "delete_time": null,
      "banner_id": 1,
      "update_time": "1970-01-01 08:00:00",
      "img": {
        "id": 65,
        "url": "/banner-4a.png",
        "from": 1,
        "delete_time": null,
        "update_time": "1970-01-01 08:00:00"
      }
    },
    {...}
  ]
}
```

## 8-4 隐藏模型字段

- 方法 1：将对象转化为数组`toArray()`，再将该字段 unset

```php
$banner = $banner->toArray();
unset($banner['delete_time']);
```

- 方法 2：使用对象的 `hidden()` 方法

```php
$banner->hidden(['update_time', 'delete_time']);
```

- 方法 3：只显示指定字段`visible()`

```php
$banner->visible(['id', 'name']);
```

## 8-5 在模型内部隐藏字段

1.对嵌套的数据字段隐藏

最好的办法：在相应的模型类中定义相应的属性。

- 想要隐藏 `banner` 的字段信息

```php
// model/Banner.php
// 隐藏的字段
protected $hidden = ['id'];
// 只显示的字段
protected $visibale = ['name','update_time'];
```

- 想要隐藏 `banner.items` 下的字段信息：

```php
// model/BannerItem.php
protected $hidden = ['id'];
```

- 想要隐藏 `banner.items.img` 的字段信息

```php
// model/Image.php
protected $hidden = ['from'];
```

## 8-6 图片资源 URL 配置

1.数据库存放的图片 url 是相对地址，所以获取的数据也是相对地址，不能直接获取到图片的具体资源位置。

    具体路径 = 服务器域名+路径配置+相对地址

2.定义自己项目相关的配置 =》 自定义配置文件

TP5 扩展配置目录 =》自动加载该目录下的配置文件

默认位置：`application/extra`

3.定义配置项:

```php
// application/extra/setting.php
return [
  'img_prefix' => 'http://mypro.com/static/images'
];
```

4.tp5 中只有 public 目录是对外公开，可以访问的，所以图片资源应当放在 public 目录下

5.读取配置文件：

```php
// 配置文件自动被加载，直接读取配置项即可
config('setting.img_prefix');
```

6.**【注】：如果自定义了`CONF_PATH`目录，则自动加载的配置文件目录应该在`config/extra`目录下**

```php
// public/index.php
// 自定义CONF_PATH目录
define('CONF_PATH', __DIR__ . '/../config/');
```

## 8-7 模型读取器的巧妙应用

1.读取器的命名：`get+字段名+Attr`

如对 url 处理则定义为`getUrlAttr`

2.读取器的特性：

- 模型具有的性质
- 使用模型时自动调用的方法（访问该属性时调用）
- AOP 思想的一个实现

  3.接收器参数说明：

      参数1：需要处理的字段的值
      参数2：当前记录的完整信息(包括隐藏未显示的字段)

  4.使用方法：

```php
// 定义读取器（框架自动调用）
public function getUrlAttr($value)
{
    // $value为获取到的url值。
    $prefix = config('setting.img_prefix');
    return $prefix.$value;
}
```

url 字段被自动拼接成：`"url": "http://mypro.com/static/images/banner-4a.png"`形式

5.根据业务逻辑进行调整

image 数据表中的`from`字段标识当前图片的来源。

    from=1 =》 图片来自当前项目，存储的是 相对路径
    from=2 =》 图片来自网络，存储的是 绝对路径

即：当 from=1 时，才需要对 url 进行相关操作。

此时需要访问到`from`的值，要用到第二个参数。

6.调整代码实现

```php
// 定义读取器（框架自动调用）
public function getUrlAttr($value, $data)
{
    // $value 获取到的url值。
    // $data 当前记录的完整信息(包括隐藏未显示的字段)

    $finalUrl = $value;
    if ($data['from'] == 1) {
        $prefix = config('setting.img_prefix');
        $finalUrl = $prefix . $value;
    }

    return $finalUrl;
}
```

通过关联模型访问 Image 模型并获取 url 字段信息时调用该方法。

## 8-8 自定义模型基类

1.对于多个模型处理 url 字段时，为增强代码的复用性，可将该处理方法封装到模型类基类`model/BaseModel.php`中。

2.其他的模型类不再直接继承`model`类，而是直接继承`BaseModel`类。

3.又考虑到当前使用的 url 表示的是 img 路径，而其他数据表中的 url 可能并非 img 路径，所以需要再次调整。将`getUrlAttr`功能的具体实现进行拆分。

(1) `model/BaseModel.php`，定义成一个普通的方法

```php
public function prefixImgUrl($value, $data)
{
    $finalUrl = $value;
    if ($data['from'] == 1) {
        $prefix = config('setting.img_prefix');
        $finalUrl = $prefix . $value;
    }

    return $finalUrl;
}
```

(2) `model/Image.php`，读取器中调用基类的方法。

```php
public function getUrlAttr($value, $data)
{
    return $this->prefixImgUrl($value, $data);
}
```

(3)分析：将业务逻辑的具体实现集中到一起，简化业务变动时的频繁修改。提高了项目的**扩展性**。

## 8-9 定义 API 版本号

1.为什么要实现多版本？

由于业务调整，实现的功能需要进行变更，（处理同一个问题需要使用不同解决方式），并且之前的功能还需要兼容，此时如果通过判断条件进行判断，再执行相应的功能会使得代码冗余，违背代码的**开闭原则**。应该将代码分离出来，每一个版本做一个单独的代码模块。

> 开闭原则：对扩展是开放的，对修改是封闭的。（以扩展的形式修改代码）

2.如何实现多版本？

- 目录设置:

```info
application
    |__ api
        |__ controller
            |__ v1
                |__ Banner.php
            |__ v2
                |__ Banner.php
```

- 路由设置：

```php
// 动态参数 :version 动态访问相应版本
Route::get('api/:version/banner/:id', 'api/:version.Banner/getBanner');
```

## 8-10 专题接口模型分析

- theme 专题表
  `theme(id,name,description,topic_img_id,delete_time,head_img_id,update_time)`

      `topic_img_id` 首页主题入口的img图片
      `head_img_id` 进入相应主题显示的head图片

- product 产品表
  `product(id,name,price,stock,delete_time,category_id,main_img_url,from,create_time,update_time,summary,img_id)`

      `main_img_url`
      `img_id`

- theme_product 专题-产品关联表
  `theme_product(theme_id,product_id)`

      theme <=> product 多对多关系
      theme_product 多对多关系表中需要一个关联表连接两者关系

## 8-11 一对一关系解析

    theme <=> image 一对一关系

1.一对一关系的表示方法（有主从关系）：

    hasOne()
    belongsTo()

外键存储在其中一张表里，所以需要使用`hasOne`和`belongsTo`来区分。

    有外键的表`belongsTo`无外键的表
    无外键的表`hasOne`有外键的表

theme -- (topic_img_id, head_img_id) -- 表中有外键 (对应 image 表中的 id 主键)
=》 theme topicImg belongsTo image

image -- 表中没有外键
=》 image hasOne theme

## 8-12 Theme 接口验证与重构

1.Theme 接口实现的不同方法对比：

（1）客户端只负责调用接口，由接口确定需要返回的主题 theme 的 id 号（2）由客户端传入具体需要的主题 Theme 的 id 号（前端有更大的灵活性）

2.方法实现

步骤：

(1)定义控制器方法名

```php
// api/controller/v1/Theme.php
getSimpleList();
```

(2)路由文件定义路由

```php
// config/route.php
Route::get('api/:version/theme', 'api/:version.Theme/getSimpleList');
```

(3)控制器方法具体实现业务功能（一） --- 参数要求

```php
/**
 * 获取需要展示的主题theme
 * @Location api/controller/v1/Theme.php
 * @param string $ids
 * @return string $theme
 */
```

(4)验证器验证

```php
// api/validate/IDCollection.php

// 1.验证规则
protected $rule = [
    'ids' => 'require|checkIDs'
];

// 2.验证不通过的提示信息
protected $message = [
    'ids' => 'ids必须是以逗号隔开的多个正整数'
];

// 3.自定义验证方法(验证器)
/**
 * 验证ids
 * @param string $values = id1,id2,id3,...
 * @return bool false/true
 */
protected function checkIDs($values)
{
    $ids = explode(',', $values);

    if (empty($ids)) {
        return false;
    }
    foreach ($ids as $id) {
        // 每个id必须是正整数
        $res = $this->isPositiveInteger($id);
        if (!$res) {
            return false;
        }

        return true;
    }
}
```

3.**扩展**:

IDCollection 和 IDMustPositiveInt 都用到对 id 是正整数的验证，为提高代码的复用性，可以：

    （1）将isPositiveInteger提取到公共方法中（没有内聚性）

    （2）将方法重新定义到验证器基类中供所有验证器之类调用。（优化的选择）

```php
// api/validate/BaseValidate.php
/**
 * 验证是否是正整数
 *
 * @param int $value
 * @return boolean false/true
 */
protected function isPositiveInteger($value)
{
    if (is_numeric($value) && is_int($value+0) && ($value +0)>0) {
        return true;
    } else {
        return false;
    }
}
```

4.调用验证器

```php
// api/controller/v1/Theme.php
(new IDCollection())->goCheck();
```

5.测试 url

```url
http://mypro.com/api/v1/theme?ids=0.1,2,3
http://mypro.com/api/v1/theme?ids=1,s,3
```

## 8-13 完成 Theme 简要信息接口

1.完成获取信息接口

```php
// api/controller/v1/Theme.php
public function getSimpleList($ids='')
{
    // 验证用户传递的参数
    (new IDCollection())->goCheck();

    $ids = explode(',', $ids);

    // 关联Image表获取相应信息
    $theme = model('theme')->with(['topicImg', 'headImg'])->select($ids);

    // 无查询结果时，进行异常处理
    if (!$theme) {
        throw new ThemeMissException();
    }

    // 对数组格式的返回数据进行json格式化
    return json($theme);
}
```

2.完成异常处理类

```php
// application/lib/exception/ThemeMissException.php
public $code = 404;
public $msg = '请求的主题不存在';
public $errorCode = 30000;
```

3.在相应的模型中隐藏部分字段

(1)隐藏 Theme 表的部分字段

```php
// api/model/v1/Theme.php
protected $hidden = ['delete_time', 'update_time', 'topic_img_id', 'head_img_id'];
```

(2)隐藏 Image 表的部分字段(只显示部分字段)

```php
// api/model/v1/Image.php
protected $visible = ['url'];
```

4.补充说明：

对于复杂的业务处理，应该将相应的代码写到 Service 层(Model 层之上) -- 特别是涉及到多个模型之间的关联的时候

## 8-14 开启路由完整匹配

1.功能需求说明

```info
点击专题图片进入到专题后需要显示相应的产品图片、

=》获取属于该专题的产品信息

（一个产品可以属于一个专题，也可以属于多个专题； 一个专题会包含多个产品） ==》多对多关系[Theme <=> Product]

多对多关系的数据表有一个中间关联表
```

2.模型关联获取关联的数据

```php
// api/model/Theme.php
public function products()
{
    // 参数1： 对应数据表的模型名
    // 参数2： 关联表的模型名
    // 参数3： 关联表中的外键名(和参数1模型关联)
    // 参数4： 关联表的外键(关联当前模型)
    return $this->belongsToMany('Product', 'theme_product', 'product_id'. 'theme_id');
}
```

3.编写控制器方法(定义方法名和需要接收的参数)

```php
// api/v1/controller/Theme.php
public function getProducts($id){}
```

4.定义路由

```php
Route::get('api/:version/theme/:id', 'api/:version.Theme/getProducts/:id');
```

【注意】：

默认情况下 TP5 的配置项是关闭路由完整匹配的，这种情况下访问当前路由接口时，由于先匹配到`api/:version/theme`路由，便不会再继续向下匹配路由，从而会调用该路由对应的接口。

==》**解决办法**：`开启路由完整匹配`

```php
// application/config.php默认配置文件路径
// 路由使用完整匹配（设置为true时开启）
'route_complete_match'   => false,  // =>true
```

## 8-15 完成 Theme 详情接口

### 1.参数校验

```php
// api/v1/controller/Theme.php
(new IDMustPositiveInt)->check();
```

### 2.在模型中编写方法实现数据获取

```php
// api/model/Theme.php
public function getThemeWithProducts($id)
{
    $theme = self::with('products,topicImg,headImg')->find($id);
    return $theme;
}
```

【注】REST 是面向资源的请求方式，即将相关的数据全部返回给客户端，不管客户端目前需不需要用得上，但这种方式返回的资源应该有一个限度，

### 3.在控制器中调用

```php
// api/v1/controller/Theme.php
$theme = model('theme')->getThemeWithProducts($id);
if(!$theme) {
    throw new ThemeMissException();
}
return $theme;
```

### 4.编写异常处理类

```php
// api/lib/exception/ThemeMissException.php
class ThemeMissException extends BaseException
{
    /**
     * 覆盖父类的相应属性
     */
    public $code = 404;
    public $msg = '请求的主题不存在';
    public $errorCode = 30000;
}
```

## 8-16 数据库字段冗余的合理利用

多对多关系的数据表关联查询时会自动多一个`pivot`字段的信息，存储关联字段。但关联信息不是我们需要显示的信息，所以将该字段隐藏掉。

`products`中`main_img_url`和`img_id`都是用来关联 image 表，记录图片信息。属于数据冗余。

但此处是数据冗余的合理应用范围，因为需要在多处使用到，并且数据量和业务并不是太复杂。

## 8-17 REST 的合理利用

1.数据冗余之后对数据的完整性和一致性的维护变得困难。

2.数据更新时需要对多处数据进行修改，否则就会出现数据不一致的现象。

3.完成方法编写(对 product 相关字段的 url 进行处理---添加前缀)

```php
// api/model/Product.php
public function getMainImgUrlAttr($value, $data)
{
    return $this->prefixImgUrl($value, $data);
}
}
```

4.REST 设计原则

(1)REST 是基于资源的，凡是和业务相关的数据都应该返回，不管当前的业务是否需要使用相应的数据。

好处在于后期业务变更需要相应的数据的时候，可以直接调用即可，不用更改服务器的接口程序，可以用来保证客户端的稳定性。

(2)但也不能一味的将所有相关的数据返回，会消耗数据库的性能。

## 8-18 最近新品接口编写

1.TP5 框架自带时间更新操作,使用模型操作数据库时，当插入记录时，自动带上`create_time`; 更新操作时自动带上`updated_time`;删除时自动带上`delete_time`

2.删除操作不是真实的物理删除，而是通过判断`delete_time`的值来确定该条记录的状态

3.实现步骤

(1)定义控制器方法 [方法名|传递参数]

```php
public function getRecent($count=15){}
```

(2)定义路由

```php
Route::get('api/:version/product/recent', 'api/:version.Product/getRecent');
```

(3)定义模型方法

- limit()方法的使用

```php
public function getMostRecent($count)
{
    $products = self::limit($count)->order('create_time desc')->select();
    return $products;
}
```

(4)编写验证器

```php
//需要对传递的count值进行验证
// application/validate/Count.php
protected $rule = [
    'count' => 'isPositiveInteger|between:1,15'
];
```

(5)完成控制器方法

```php
public function getRecent($count=15)
{
    (new Count())->goCheck();
    $products = model('product')->getMostRecent($count);
    if ($products) {
        throw new ProductMissException();
    }
    return $products;
}
```

(6)完成异常处理类方法

```php
class ProductMissException extends BaseException
{
    public $code = '404';
    public $msg = '请求的product不存在';
    public $errorCode = 20000;

}
```

[注]：`app_debug`设置为 true 时，在`ExceptionHandler.php`中会调用父类的`render()`方法，导致框架的异常处理类找不到程序中自定义的异常处理类，从而会有报错提示。

!!!出现 500 系统内部错误!

- 原因=>config.php 设置`default_return_type`的值为`html`, 而 Product 的 controller 中 return 的结果值为 array，导致系统内部错误。

- 解决=>将`default_return_type`的值为`json`。或者将 Product 的 controller 中 return 的结果进行 json 格式化。

### **【警告】学会查看 log 日志信息，提高错误排查能力！**

## 8-19 使用数据集还是数组？

1.问题 1：验证方法中，`$rule`属性数组的键值对中， 值`'isPositiveInteger|between:1,15'`中`|`符两端不能有空格，否则会被视为验证错误。

2.问题 2：对某些当前不需要用到，但后期会用到的字段信息（特殊情况不用，大多数情况要用），既不能直接显示，也不能直接隐藏，如何处理？

=》 在`api/v1/Product/recent`接口中临时隐藏`summary`字段。

3.**collection()方法**：临时隐藏某个或某些字段

【使用方法】：

```php
// 使用数据集，临时隐藏某些字段
$productCollection = collection($products);
$products = $productCollection->hidden(['summary']);
```

4.一个 product 是一个对象，一组 product 也可是是一个对象(数据集)。

5.使用对象的方式，可读性好，内聚性好。

6.TP5 调用模型自动返回一个数据集的形式：`resultset_type` [database.php]

默认是`array`，设置成`collection`后，模型返回的数据自动就是`collection`形式，不需要再转换一次。

```php
// 在database.php中配置之后，不需要手动转换为collection
$products = $products->hidden(['summary']);
```

【扩展】：

但是这样使用之后，控制器中调用模型返回数据后，返回的是对象，即使没有数据，也不是空，所以直接使用`!`判断是不能实现效果的。

=》解决方法：使用数据集对象的`isEmpty()`方法进行判空。

## 8-20 分类列表接口

1.模型类的`all`方法使用。

- 参数 1：主键列表或者查询条件（闭包） `mixed`

- 参数 2：关联预查询 `array | string`

```php
$categories = model('Category')->with('img')->select();
// 等价于
$categories = model('Category')->all([], 'img');
```

## 8-21 扩展：接口粒度与接口分层

1.减少首页 http 请求(api)的次数，从而减轻服务器的压力

2.接口粒度： 太粗=》代码复用性不好，不够灵活；太细=》需要发送的请求太多，不方便

3.架构师 =》 Api 接口设计 =》 底层设计力度比较小、灵活性比较高的 api 接口；越往上粒度逐渐变粗。

4.如果确实调用的接口比较多，应该在 api 基础数据层上建立业务层，再在业务层调用基础数据层相关的接口，再进行封装。
