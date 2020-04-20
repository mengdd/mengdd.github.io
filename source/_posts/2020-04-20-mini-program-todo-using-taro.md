---
title: 用Taro做个微信小程序Todo, 小白工作记录
date: 2020-04-20 17:01:58
tags: [Taro, Mini Program]
categories: [Mini Program]
---

## 微信小程序框架: Taro
做微信小程序的框架, 几个比较主流的:
* 官方的`WePY`: https://tencent.github.io/wepy/document.html#/
* 美团的`mpvue`: http://mpvue.com/mpvue/#-html
* 京东的`Taro`: https://taro.aotu.io/

前两者都是Vue风格的, Taro是React的.

本篇本着学习的目的, 用Taro做一个简单的小程序.
代码在这里: https://github.com/mengdd/mini-program-todo

<!-- more -->

背景: 
一直做Android开发, 最近想把其他各种技术都撸一撸, 拓展一下视野.
之前练过微信小程序原生开发的例子, 基本上只知道个大概, 翻书马冬梅, 合书马什么梅?
所以这次用框架了, 觉得还是要有个输出, 这样印象深刻一些.

所以本文是从一个小白的角度, 手把手做练习的.

## Taro程序环境
这部分参考:
* 官方文档: https://nervjs.github.io/taro/docs/README.html
* Getting Started:
https://nervjs.github.io/taro/docs/GETTING-STARTED.html

要有node环境, 推荐用nvm来管理.

需要安装CLI工具:
```
npm install -g @tarojs/cli
```

## 创建项目并运行
### 创建项目
创建模板项目:
```
taro init myApp
```
在这个阶段会有一些问题要回答.
```
* 请输入项目介绍！ My awesome project blablabla.
* 是否需要使用 TypeScript ？ Yes
* 请选择 CSS 预处理器（Sass/Less/Stylus） Sass
* 请选择模板 默认模板
```

完了之后会自动安装依赖:
```
npm install
```
然后就成功啦!

### 运行
Taro需要运行不同的命令, 将Taro代码编译成不同端的代码, 然后在对应的开发工具中查看效果.

这里仅说微信小程序. 还是需要微信开发者工具:
https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html

import这个项目就行了.

开发前微信开发者工具的项目设置要注意:
```
需要设置关闭ES6转ES5功能, 开启可能报错.
需要设置关闭上传代码时样式自动补全, 开启可能报错.
需要设置关闭代码压缩上传, 开启可能报错.
```
这些默认都是关闭的.

微信小程序的编译预览及打包:

npm script
```
$ npm run dev:weapp
$ npm run build:weapp
```

或者:
```
$ taro build --type weapp --watch
$ taro build --type weapp
```

加上`--watch`会监听文件修改, 去掉会对代码进行压缩打包.

运行编译命令之后就可以在微信开发者工具中预览了. 是显示一个Hello World.

### 项目结构
因为创建项目的时候用了TypeScript, 所以项目结构是这样的:
```
├── dist                   编译结果目录
├── config                 配置目录
|   ├── dev.js             开发时配置
|   ├── index.js           默认配置
|   └── prod.js            打包时配置
├── src                    源码目录
|   ├── pages              页面文件目录
|   |   ├── index          index页面目录
|   |   |   ├── index.scss index页面样式
|   |   |   └── index.tsx  index页面逻辑
|   ├── app.scss           项目总通用样式
|   ├── app.tsx            项目入口文件
|   └── index.html          
└── package.json
```
如果需要创建组件, 可以在`src`下创建`component`目录来统一存放组件.

"Hello world!"就在`index.tsx`中.

## IDE
关于IDE:
我发现用微信开发者工具是打不开这些后缀名为`scss`和`tsx`的文件的. 
微信开发者工具只能用来预览.


后来用了Intellij, 是能查看文件了, 
但是开始写代码之后, 发现没有自动提示, 也没有自动格式化工具. 

又下了个VS CODE:
```
brew cask install visual-studio-code
```

格式化的快捷键是:
`Shift + Alt + F`.

后来我在Text Editor设置里勾选了`Format On Save`.

## 编写代码

### Step 1: 静态页面显示
先弄几个数据静态显示一下:
```
export default class Index extends Component {
  config: Config = {
    navigationBarTitleText: '首页'
  }

  constructor(props) {
    super(props)
    this.state = {
      list: ['get up', 'eat breakfast', 'study',]
    }
    inputVal: ''
  }

  render() {
    let { list, inputVal } = this.state

    return (
      <View className='index'>
        <View className='list_wrap'>
          <Text>Todo List</Text>
          {
            list.map((item, index) => {
              return <View>
                <Text>{index + 1}.{item}</Text>
              </View>
            })
          }
        </View>
      </View>
    )
  }
}

```
然后命令行运行:
```
taro build --type weapp --watch
```
就会直接显示出来结果. 之后的修改也是随时呈现.


这里IDE报告两个TypeScript的问题:
```
Property 'list' does not exist on type 'Readonly<{}>'
```
和:
```
Property 'inputVal' does not exist on type 'Index'.
```
按照[这里](https://stackoverflow.com/questions/47561848/property-value-does-not-exist-on-type-readonly)的方法修改的:

```
interface MyProps {
}

interface MyState {
  list: Array<string>;
  inputVal: string;
}

export default class Index extends Component<MyProps, MyState> {
//...
}
```

代码中设置值的时候调用`this.setState()`.


### Step 2: 添加和删除项目
加上输入框, 添加和删除按钮的对应方法.
完整代码:
```
interface MyProps {
}

interface MyState {
  list: Array<string>;
  inputVal: string;
}

export default class Index extends Component<MyProps, MyState> {
  config: Config = {
    navigationBarTitleText: '首页'
  }

  constructor(props) {
    super(props)
    this.state = {
      list: ['get up', 'eat breakfast', 'study',],
      inputVal: ''
    }

  }

  addItem() {
    let { list } = this.state
    const inputVal = this.state.inputVal

    if (inputVal == '') return
    else {
      list.push(inputVal)
    }
    this.setState({
      list,
      inputVal: ''
    })
  }

  removeItem(index) {
    let { list } = this.state
    list.splice(index, 1)
    this.setState({
      list
    })
  }

  inputHandler(e) {
    this.setState({ inputVal: e.target.value })
  }

  render() {
    let { list, inputVal } = this.state

    return (
      <View className='index'>
        <Input className='input' type='text' value={inputVal} onInput={this.inputHandler.bind(this)} />
        <Text className='add' onClick={this.addItem.bind(this)}>添加</Text>
        <View className='list_wrap'>
          <Text>Todo List</Text>
          {
            list.map((item, index) => {
              return <View className='list_item'>
                <Text>{index + 1}.{item}</Text>
                <Text className='del' onClick={this.removeItem.bind(this, index)}>删除</Text>
              </View>
            })
          }
        </View>
      </View>
    )
  }
}
```

### Step 3: 样式调整
用了这里面的样式: https://juejin.im/book/5b73a131f265da28065fb1cd

然后企图调整button的对齐. (CSS不太擅长, 请轻拍, 请指教.)
```
.input {
    display: inline-block;
    margin: 32px;
    border: 1px solid #666;
    width: 65%;
    vertical-align: middle;
}
.list_wrap {
    padding: 32px;
}
.list_item {
    margin: 20px 0;
}
.add,
.del {
    display: inline-block;
    width: 120px;
    height: 60px;
    margin: 0 10px;
    padding: 0 10px;
    font-size: 22px;
    line-height: 60px;
    text-align: center;
    border-radius: 10px;
    box-sizing: border-box;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    justify-content: center;
    vertical-align: middle;
}
.add {
    background-color: #5c89e4;
    color: #fff;
    border: 1px solid #5c89e4;
}
.del {
    background-color: #fff;
    color: #5c89e4;
    border: 1px solid #5c89e4;
    position: absolute;
    right: 32px;
}

```

## Taro命令
快速生成新的页面文件: 
```
Taro create --name [页面名称] 
```


## 参考
* [taro github](https://github.com/NervJS/taro)
* [awesome taro](https://github.com/NervJS/awesome-taro)
* [微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/page.html)
* [掘金小册: Taro 多端开发实现原理与项目实战](https://juejin.im/book/5b73a131f265da28065fb1cd)
