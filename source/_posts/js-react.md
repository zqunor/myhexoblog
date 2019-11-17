---
title: 【React入门】实现todolist功能
date: 2018-08-08 22:07:21
tags:
  - web
  - React
category:
  - 【前端相关】
---

作为一名 PHP 初级的程序员，目前尚且处于学习 PHP 业务逻辑实现到日常工作中的阶段，但是由于现在想要搭建一个满意的个人博客，并且尝试过很多 hexo 主题后总是会对主题的某些或某个部分不太满意，
所以为了以后可以自己实现相应页面的开发，所以还是想着能够学点前端框架的知识，为以后成为全栈工程师做准备。目前比较流行的前端框架主要有`React.js`和`Vue.js`，因为当前公司使用的是`React.js`开发的，所以也选择`React`作为学习对象。

<!--more-->

## 开发环境准备

### 安装 node.js

建议在 React 中使用 CommonJS 模块系统，比如 browserify 或 webpack(推荐使用)。

使用淘宝定制的 cpm 命令行工具代替默认的 npm

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
npm config set registry https://registry.npm.taobao.org
```

这样就可以使用 cnpm 命令来代替 npm 进行安装模块了：

```bash
cnpm install [name]
```

### 创建 react 应用

```bash
cnpm install -g create-react-app
cd todolist-react/
npm start
```

### 目录结构

- 原始结构

```info
todolist-react/
    node_modules/
    public/
        favicon.ico
        index.html
        manifest.json
    src/
        App.css
        App.js
        App.test.js
        index.css
        index.js
        logo.svg
    README.md
    package-lock.json
    package.json
    .gitignore
```

## 代码实现

### 准备工作

（1）编写 todolist 功能时，对默认项目的文件进行删减，只需要保留主要的文件即可。

将`App.js`重命名为`TodoList.js`，创建文件`TodoItem.js`。

- 精简后的文件结构

```info
todolist-react/
    node_modules/
    public/
        favicon.ico
        index.html
        manifest.json
    src/
        TodoList.js
        TodoItem.js
        index.js
    README.md
    package-lock.json
    package.json
    .gitignore
```

（2）组件拆分

`React` 的组件开发意思是将页面上每个部分作为一个组件，然后通过每个组件之间的通信，进行数据交互，实现完整页面的渲染。

`TodoList` 功能包括用户输入部分和 `List` 清单部分，所以将整个功能拆分为两部分。用户输入部分为`TodoList`; `List`清单部分为`TodoItem`。

（3）`index.js`代码实现

```javascript
import React from "react";
import ReactDOM from "react-dom";

// 引入TodoList组件
import TodoList from "./TodoList";

// 获取TodoList返回的数据，并渲染到id为root的元素节点中
ReactDOM.render(<TodoList />, document.getElementById("root"));
```

（4）`TodoList.js`代码实现

```javascript
// 引入React类，引入React.Component组件类
import React, { Component} from 'react';
// 从当前目录下的TodoItem.js中引入TodoItem子组件
import TodoItem from './TodoItem';

// 组件定义和实现
class TodoList extends Component {
  // 通过render() 方法渲染页面数据
  render() {
    return (
      <div>
        <div>
          <input value={this.state.inputValue} onChange={this.handleInputChange}/>
          <button onClick={this.handleBtnClick}>add</button>
        </div>
        <ul>
        {
            return (
                this.state.list.map((item, index) => {
                    return <TodoItem key={index} content={item} index={index} delete={this.handleDelete}/>
                })
            )
        }
        </ul>
      </div>
    );
  }
}
```

### 未完待续
