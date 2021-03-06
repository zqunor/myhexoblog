---
title: 【总结】一年的工作任务总结
date: 2019-04-01 20:00:00
tags:
  - 工作总结
category:
  - 【工作总结】
---

从2018年9月到2019年4月，经历了杭州互联网创业公司的996，对程序员这个职业的热情大大消磨掉了，加上没有精力总结整理，以及核心竞争力没有像样的突破，所以在这个大环境不好的前提下，依然选择了裸辞。半年多的高压工作，却没有抽出时间来整理，也实在是精力跟不上，在这段休息的时间里，准备停下来，把积累的能力总结整理一下，为了更好的出发。

<!-- more -->

## 功能模块介绍
### 1.云端工作/云端账单模块

#### 视图/表设计

- i_job 云端工作表
  - 增加审核人员、用户来源字段
- i_job_period 云端工作账单表
- v_periods（i_job + i_job_period + i_user_login + i_job_note_log）
  - i_user_login 开发者和企业方信息
  - i_job_note_log 云端工作备注（截取账单的备注信息）
- i_job_deposit_log 云端工作押金变更日志
  - 状态：支付、抵用、退还

## 功能列表

### 1.账单模块

#### 后台

1）账单详情

- 未托管费用（100%结算比例，费率实时取工猫配置|网站配置）
- 托管费用但未结算（企业方费率取数据库、开发者费率实时取）
- 已结算（直接取数据库值）

> 难点：
>- 细项费用计算公式
>- 费用固化阶段（什么状态时写入，什么状态时动态获取）

2）核定工资

- 确定云端工作信息（押金、核定工资、试用期）
- 确定账单信息（押金、核定工资、试用期、开发票、发薪|结薪时间、服务费率）

> 难点：
> - 数据固化
>   - 固化设置的`企业方`**费率**信息，支付时直接使用数据库值（托管费用时固化企业方**费用**）

3）账单结算

- 固化开发者费用
- 判断是否可以结算？
  - 已托管账单费用 / 有押金可以抵用
- 判断是否可以使用押金结算？
  - 有押金
  - 云端工作结束合作
  - 账单未托管费用

> 难点：
>- 细项费用计算公式
>- 动态取值 or 取数据库值？
>- 使用押金结算

- 退还押金
- 判断是否可以退换押金
  - 有押金【难点】
  - 云端工作结束合作

5）结算综合表

> 难点：
>- 建试图简化数据查询
>   - 次月托管（当前账单的下一周期账单，及其托管费用情况）
>   - 账单备注（备注只关联了云端工作，现在需要和账单关联）
>   - 审核人员和用户来源（后续添加云端表字段，不用再从备注表中截取）
>- 导出(csv)

#### 前台

1）账单详情

> 难点：
>- 细项费用计算公式
>- 不同用户身份展示不用费用

### 2.面试评价模块

#### 表设计
- i_outsource_rating
  - rating_type （对应RatingType枚举类）
  - impressions_desc 面试评价关键词（保存文案、避免后期关键词变更，对应关系出错）
  - interview_status 面试评价结果 （面试通过/面试不通过）

#### 功能列表
1）整合云端工作和整包项目的面试评价

- 生成动态
- 消息通知

#### 难点整理

整合云端工作和整包项目的面试评价，构建统一的面试评价模块

### 3.付费会员模块

#### 表/视图设计

- i_vip_type 会员类型表（开发者会员、企业会员）
  - 会员优惠信息（原价、现价）
  - 会员类型、会员等级
  - 购买方式（月、季、年）
- i_user_vip 用户会员表
  - 用户会员状态、到期时间
  - 会员类型
- i_vip_order 会员订单表
  - 购买的会员类型
  - 购买价格、优惠信息
  - 续购/首购
- i_vip_ compensate_record 补偿的优质开发者表（赠送会员）
- v_user（增加会员信息）
  - 会员类型
  - 会员到期时间

#### 功能列表

##### 后台

1）会员信息更新

2）会员用户列表

3)会员订单列表

##### 前台

1）会员中心

2）会员支付

难点：

- 支付逻辑

#### 难点介绍
主要是：

- 前期会员类型表的设计，如何存储会员优惠信息，及会员类型的界定（会员类型？、会员等级？、会员类型要不要和会员等级关联？、会员优惠怎么存？）

- 会员优惠信息
  - 和云端工作费率等相关联

### 4.产品认证模块

#### 表/视图设计

- i_certified_product 认证产品表
  - 状态（开放申请、暂停认证、下架隐藏）
  - 价格（会员价、现价、原价）
  - 基本信息（名称、描述、简介、证书文案）
- i_certified_product_log 认证产品表变更日志
- i_user_certified 用户认证表
  - 审核状态（AuditStatus枚举类）
  - 证书状态（CertStatus枚举类 未生成、正常、过期、撤销）
  - 认证类型（CertType枚举类、AuditType枚举类）
  - 审核信息（审核失败时间|操作人员、申请时间、审核失败次数）
  - 证书基本信息（证书编号、有效时间）
- i_user_cert_log 用户认证变更日志表
- i_common_audit_log 通用的审核日志表
  - 认证详情中的审核记录
- v_user（增加认证信息）
  - 认证类型
  - 认证到期时间

#### 功能介绍

##### 后台

1）认证产品列表：列出所有认证产品

2）编辑认证产品

- 富文本

3）认证用户列表：列出所有认证的用户及相应的认证信息

4）用户认证详情

- 添加备注
- 审核（申请、通过、拒绝、撤销）
- 发送邮件（http://api.sendcloud.net/apiv2/mail/send）

##### 前台

1）认证产品列表

> 难点：
>- 按钮文案
>- 排序（已认证/申请中的排在上方）

2）认证产品申请详情

> 难点：
>- 按钮文案
>- 认证达标条件

3）认证证书详情

> 难点：
>- 证书二维码(http://github.com/Bacon/BaconQrCode)
>- 生成证书图片

#### 难点介绍

认证和审核的逻辑拆分。

### 5.协作群组模块

#### 表/视图设计
- i_group 用户群组表
  - 群组人数
  - 管理员
  - 云端工作
- i_group_log 群组变更日志表
- i_group_job 群组-云端工作关联表
- i_group_manager 群组-管理员关联表
- v_job_report：云端工作日报统计表
  - 上一周期工时
  - 本期周期工时
  - 本周工时 YEARWEEK(date)
  - 近7天工时 datediff()
  - 今天工时 fron_unixtime(date+28800,'%Y%m%d') (mysql终端的时区为格林威治时区)

#### 功能介绍

#####  后台

1). 群组列表：列出所有协作群组

> 难点：
>- 群组-云端工作-开发者 关系关联

##### 前台

1). 群组列表：列出当前用户创建的、管理的、参与的协作群组

> 难点：
>- 群组-云端工作-开发者 关系关联

2). 创建群组：关联管理员、云端工作创建管理群组

3). 更新群组：关联管理员、云端工作更新管理群组

> 难点：
>- 当前用户创建的云端工作列表（排序）
>   - 将已勾选的云端工作放在第一页

4). 删除群组

5). 群组详情：工时情况、日报情况、群组成员情况

- 群组成员
- 工时统计表【难点】
- 日报列表
- 角色权限判断

#### 难点介绍

多对多关系映射，中间关联表在查询时的作用，先查主群组表，再查关联表，再通过关联表查成员信息。直接通过关联表查询的话，会导致没有成员的群组展示不出来。直接通过成员信息表查询也是同理。

## 自我反思

## 经验总结

## 项目展示

### 云端工作/账单模块

账单详情-后台
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1woab12ouj31li1047ad.jpg)

账单详情-前台
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wql0qb3kj312m12qq89.jpg)

账单结算
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wobhupntj325e18anat.jpg)

结算表
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wqhwyopcj32la16uneh.jpg)

核定工资
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wtvc0qjbj312c0outbe.jpg)

### 认证模块

认证列表-前台
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wny82faej31w21dmk5d.jpg)

认证申请
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wn41p907j31600qsjut.jpg)

认证证书
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wn524i8xj31fe0zy4gs.jpg)

认证审核详情-后台
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wuzr4vicj31m215y11m.jpg)

### 付费会员模块

会员中心页
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wmx9578mj31jk166ama.jpg)

会员设置-后台
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wuxgvy33j31bw0zs42y.jpg)

会员订单-后台
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wuz37x1uj32e816u4fr.jpg)

### 协作群组模块

协作群组列表
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wnuadcvuj31lg1667fk.jpg)

编辑协作群组
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wnwejct3j3188188gr8.jpg)

协作群组详情
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wnsj0sqsj31wo1d67ez.jpg)

协作群组-后台
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wuy427dij32cq19san9.jpg)

### 面试评价

整包/云端面试评价页
![](https://ws1.sinaimg.cn/mw690/005EgYNMly1g1wo7qw2k9j30tk0ycgp0.jpg)