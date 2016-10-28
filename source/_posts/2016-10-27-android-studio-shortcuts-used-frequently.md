---
title: 用Android Studio开发最常用到的快捷键
date: 2016-10-27 22:23:45
tags: [Android, Android Studio]
categories: [Android, Tools, IDE]
---
# Android Studio常用快捷键

Android Studio日常开发常用快捷键.
快捷键版本: Mac OS X 10.5+

<!-- more -->

## 搜索查看类
|      用途     |    Mac快捷键    |
|:-------------|:---------------|
| 搜索所有文件   | double Shift    |
| 搜索文件       | Cmd + Shift + O |
| 搜索类文件     | Cmd + O        |
| 搜索符号       | Cmd + Opt + O   |
| 打开最近的文件  | Cmd + E         |
| 打开最近编辑过的文件  | Cmd + Shift + E   |
| 在文件内搜索    | Cmd + F         |
| 全文搜索       | Cmd + Shift + F |
| 显示结构, 类中方法| Cmd + F12     |
| 跳到当前文件   | F4     |
| 从以上查找中途退出| ESC      |
| 发现引用        | Opt + F7(显示在下面)/ Opt + Cmd + F7(显示在当前)|
| 查找定义        | Cmd + B/ Cmd + 单击, 找到定义后再次点击会显示所有引用|
| 找子类/实现        | Cmd + Opt + B |
| 找基类/接口        | Cmd + U |
| 高亮Usages     | Cmd + Shift + F7|
| 查找Action     | Cmd + Shift + A |
| 显示文件在项目中的位置 | Opt + F1, 再加Enter |
| 复制当前文件的路径| Cmd + Shift + C|



## 编辑类
### 编辑
|      用途      |    Mac快捷键     |
|:-------------|:---------------|
| 复制           | Cmd + C         |
| 剪切           | Cmd + X         |
| 粘贴           | Cmd + V         |
| 从剪切板粘贴    | Cmd + Shift + V |
| 复制当前行或当前选中块 | Cmd + D     |
| 以光标位置向前, 删除一个词 | Opt + delete |
| 删除一行        | Cmd + delete    |
| 把代码包起来: try-catch等 | Cmd + Opt + T |
| 查看方法的参数信息 | Cmd + P          |


### 生成
|      用途      |    Mac快捷键   |
|:-------------|:---------------|
| 生成方法        | Cmd + N      |
| 生成未定义的方法 | Opt + Enter |
| Override方法   | Ctrl + O |
| 实现(implement)方法 | Ctrl + I |

### 自动补全
|      用途    |    Mac快捷键   |
|:-------------|:---------------|
| 加import语句   | Opt + Enter    |
| 显示Warning信息并采用快捷修复 | Opt + Enter|

### 重构
|      用途      |    Mac快捷键   |
|:-------------|:---------------|
| 重命名         | Shift + F6     |
| 更改签名(重构方法) | Cmd + F6  |
| 提取方法M,变量V,字段F,常量C,参数P | Cmd + Opt + M,V,F,C,P |
| 内联            | Cmd + Opt + N |

### 选择, 移动
|      用途    |    Mac快捷键   |
|:-------------|:---------------|
| 移动到某一行   | Cmd + L |
| 选中行         | Cmd + Shift + 方向|
| 选中词         | Opt + 上下方向|
| 按词移动光标   | Opt + 左右方向|
| 返回上/下一次光标所在的地方 | Cmd + Opt + 左右方向  |
| 移动当前行     | Cmd + Shift + 上下方向|

### 格式化
|      用途    |    Mac快捷键   |
|:-------------|:---------------|
| 格式化代码   | Cmd + Opt + L   |
| 优化imports | Ctrl + Opt + O  |

### 注释
|      用途    |    Mac快捷键   |
|:-------------|:---------------|
| 行注释         | Cmd + /    |
| 块注释         | Cmd + Opt + /  |

## 运行调试类
|      用途      |    Mac快捷键   |
|:-------------|:---------------|
| 运行           | Ctrl + R      |
| 运行...        | Ctrl + Opt + R |
| 调试           | Ctrl + D     |
| 调试...        | Ctrl + Opt + D  |
| 设置断点        | Cmd + F8    |
| 单步执行        | F8     |
| 跑到光标处     | Opt + F9        |
| 看表达式       | Opt + F8    |
| Resume        | Opt + Cmd + R   |
| 查看所有断点    | Shift + Cmd + F8|

## 测试类
|      用途    |    Mac快捷键   |
|:-------------|:---------------|
| 生成或打开测试类 | Cmd + Shift + T |
| 运行测试  | Ctrl + Shift + R|
| 调试测试     | Ctrl + Shift + D|

## 版本控制类
|      用途    |    Mac快捷键   |
|:-------------|:---------------|
| 显示版本控制窗口 | Cmd + 9   |
| 显示Diff      | Cmd + D|
| 下一个Diff    | F7           |
| 在Diff中打开文件  | F4         |

## 窗口类
|      用途    |    Mac快捷键   |
|:-------------|:---------------|
| 显示Android Monitor | Cmd + 6 |
| 代码全屏或退出 | Cmd + Shift + F12 |
| 打开Preferences | Cmd + , |
| 打开项目结构窗口 | Cmd + ; |
| 快速切换scheme | Ctrl + ` |

## Resources
- [官方文档](https://developer.android.com/studio/intro/keyboard-shortcuts.html)
- [The powerful Android Studio](http://saulmm.github.io/the-powerful-android-studio)