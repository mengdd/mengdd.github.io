---
title: Kotlin Android项目静态检查工具的使用
date: 2020-02-08 15:17:33
tags: [Kotlin, Android, Lint, Tools]
categories: [Kotlin, Android]
---

# Kotlin Android项目静态检查工具的使用
Kotlin Android项目可用的静态检查工具: Android官方的Lint, 第三方的ktlint和detekt.

<!-- more -->

## 静态检查工具
静态检查工具, 指不需要运行代码, 对代码进行检查的工具.

不止代码风格, 还可以检查代码的正确性, 是否有安全问题, 是否有性能问题等.

静态检查工具一般都具备可扩展性, 方便使用者制定和添加自己的规则.

比较流行的Java静态检查工具有CheckStyle, FindBugs, PMD等.

Android项目, 用Kotlin语言, 可用的静态检查工具: 官方的Android Lint, ktlint和detekt.

## Android Lint
Android官方提供了代码扫描工具, 叫lint.

在Android Studio运行lint, 在菜单中选:
`Analyze -> Inspect Code...`, 选择范围, 点`OK`即可运行.

也可以在命令行:
```
 ./gradlew lint
```
会分类报告出各种各样的错误.

### 查看所有的Lint规则
点击菜单中的`Analyze -> Inspect Code...`, 在弹框的`Inspection profile`部分点击表示更多的`...`按钮, 会弹出框显示当前的所有lint规则.

可以选择是否包括规则, 编辑它们的优先级和应用范围等.

![android lint](/images/android-studio-lint.png)

### Android Lint配置
全局的配置除了用IDE, 还可以通过`lint.xml`文件.

还可以在gradle中配置, 比如:
```groovy
android {
  ...
  lintOptions {
    // Turns off checks for the issue IDs you specify.
    disable 'TypographyFractions','TypographyQuotes'
    // Turns on checks for the issue IDs you specify. These checks are in
    // addition to the default lint checks.
    enable 'RtlHardcoded','RtlCompat', 'RtlEnabled'
    // To enable checks for only a subset of issue IDs and ignore all others,
    // list the issue IDs with the 'check' property instead. This property overrides
    // any issue IDs you enable or disable using the properties above.
    check 'NewApi', 'InlinedApi'
    // If set to true, turns off analysis progress reporting by lint.
    quiet true
    // if set to true (default), stops the build if errors are found.
    abortOnError false
    // if true, only report errors.
    ignoreWarnings true
  }
}
```

特定代码可以利用`@SuprressLint`注解.
比如: `@SuppressLint("NewApi")`, `@SuppressLint("all")`.

特定xml文件中可以用`tools:ignore`, 比如`tools:ignore="UnusedResources"`, `tools:ignore="NewApi,StringFormatInvalid"`, `tools:ignore="all"`等.

### Android Lint现状基准快照
可以在gradle中进行设置一个baseline:
```groovy
android {
  lintOptions {
    baseline file("lint-baseline.xml")
  }
}
```
第一次添加这个, 运行lint之后, 会自动创建一个`lint-baseline.xml`文件, 所有当前的问题都会被写入这个文件.

当再次运行lint, 就只会读这个文件, 作为基准, 只报告新的错误或警告. 并且提示其他原有问题在baseline文件中, 被排除.

如果想重新生成baseline可以手动删除这个文件, 重新进行.

## ktlint
[ktlint](https://github.com/pinterest/ktlint)

代码风格检查, 遵循的是: [Kotlin style guide](https://developer.android.com/kotlin/style-guide)

所有的规则代码:
[github: ktlint rule set](https://github.com/pinterest/ktlint/tree/master/ktlint-ruleset-standard/src/main/kotlin/com/pinterest/ktlint/ruleset/standard)
### ktlint安装
在`app/build.gradle`中添加:
```groovy
configurations {
    ktlint
}

dependencies {
    ktlint "com.pinterest:ktlint:0.36.0"
    ...
}
task ktlint(type: JavaExec, group: "verification") {
    description = "Check Kotlin code style."
    classpath = configurations.ktlint
    main = "com.pinterest.ktlint.Main"
    args "src/**/*.kt"
    // to generate report in checkstyle format prepend following args:
    // "--reporter=plain", "--reporter=checkstyle,output=${buildDir}/ktlint.xml"
    // see https://github.com/pinterest/ktlint#usage for more
}
check.dependsOn ktlint

task ktlintFormat(type: JavaExec, group: "formatting") {
    description = "Fix Kotlin code style deviations."
    classpath = configurations.ktlint
    main = "com.pinterest.ktlint.Main"
    args "-F", "src/**/*.kt"
}
```

### ktlint使用 
检查:
```
./gradlew ktlint
```

自动修改:
```
./gradlew ktlintFormat
```
并不是所有问题都可以自动修改, 有的问题需要手动改.

配置中的这句:
```
check.dependsOn ktlint
```
把运行检查加到了check task中. 通过:
```
./gradlew check --dry-run
```
可以查看运行`check` task都会做哪些事情(dry run表示不用实际执行这些任务).

### ktlint源码
看ktlint的源码, 程序的入口是`Main.kt`文件中的`main`.

使用的规则集是:
```kotlin
class StandardRuleSetProvider : RuleSetProvider {

    // Note: some of these rules may be disabled by default. See the default .editorconfig.
    override fun get(): RuleSet = RuleSet(
        "standard",
        ChainWrappingRule(),
        CommentSpacingRule(),
        FilenameRule(),
        FinalNewlineRule(),
        ImportOrderingRule(),
        IndentationRule(),
        MaxLineLengthRule(),
        ModifierOrderRule(),
        NoBlankLineBeforeRbraceRule(),
        NoConsecutiveBlankLinesRule(),
        NoEmptyClassBodyRule(),
        NoLineBreakAfterElseRule(),
        NoLineBreakBeforeAssignmentRule(),
        NoMultipleSpacesRule(),
        NoSemicolonsRule(),
        NoTrailingSpacesRule(),
        NoUnitReturnRule(),
        NoUnusedImportsRule(),
        NoWildcardImportsRule(),
        ParameterListWrappingRule(),
        SpacingAroundColonRule(),
        SpacingAroundCommaRule(),
        SpacingAroundCurlyRule(),
        SpacingAroundDotRule(),
        SpacingAroundKeywordRule(),
        SpacingAroundOperatorsRule(),
        SpacingAroundParensRule(),
        SpacingAroundRangeOperatorRule(),
        StringTemplateRule()
    )
}
```
可以看到都是代码格式相关的规则.

在线查看: [ktlint ruleset standard](https://github.com/pinterest/ktlint/tree/master/ktlint-ruleset-standard/src/main/kotlin/com/pinterest/ktlint/ruleset/standard)

## detekt
Github: [detekt](https://github.com/arturbosch/detekt)也是一个Kotlin的静态分析工具.

官方网站: [detekt](https://arturbosch.github.io/detekt/index.html).

### detekt安装
在项目根目录的`build.gradle`文件中添加:
```groovy
plugins {
    id("io.gitlab.arturbosch.detekt").version("1.5.0")
}

allprojects {
    repositories {
        google()
        jcenter()
    }
    apply plugin: 'io.gitlab.arturbosch.detekt'
}

detekt {
    failFast = true // fail build on any finding
    buildUponDefaultConfig = true // preconfigure defaults
    config = files("$projectDir/config/detekt.yml")
    // point to your custom config defining rules to run, overwriting default behavior
    baseline = file("$projectDir/config/baseline.xml")
    // a way of suppressing issues before introducing detekt

    reports {
        html.enabled = true // observe findings in your browser with structure and code snippets
        xml.enabled = true // checkstyle like format mainly for integrations like Jenkins
        txt.enabled = true
        // similar to the console output, contains issue signature to manually edit baseline files
    }
}
```

运行`./gradlew tasks`可以看到相关任务被加进去了:
```
detekt
detektBaseline - Creates a detekt baseline on the given --baseline path.
detektGenerateConfig - Generate a detekt configuration file inside your project.
detektIdeaFormat - Uses an external idea installation to format your code.
detektIdeaInspect - Uses an external idea installation to inspect your code.
```
### detekt使用
运行:
```
./gradlew detekt
```
进行检测.

```
detekt {
}
```
块是用来进行自定义的属性设置的.

如果开启了文件输出, 结果在:
`app/build/reports/detekt`目录下可以查看.

并且运行`./gradlew check --dry-run`可以发现, 运行`check`的时候也会执行`detekt`.

可以利用:
```
./gradlew detektGenerateConfig
```
生成配置文件: `config/detekt/detekt.yml`. 

在配置文件中可以看到对各种规则的开关状态, 编辑它可以进行定制.


### detekt源码
程序的入口在`detekt-cli`module的`Main.kt`.
根据参数的不同返回不同的`Runner`.

主要用于检测的是这个[Runner](https://github.com/arturbosch/detekt/blob/master/detekt-cli/src/main/kotlin/io/gitlab/arturbosch/detekt/cli/runners/Runner.kt), 它的执行方法:
```kotlin
override fun execute() {
    createSettings().use { settings ->
        val (checkConfigTime) = measure { checkConfiguration(settings) }
        settings.debug { "Checking config took $checkConfigTime ms" }
        val (serviceLoadingTime, facade) = measure { DetektFacade.create(settings) }
        settings.debug { "Loading services took $serviceLoadingTime ms" }
        var (engineRunTime, result) = measure { facade.run() }
        settings.debug { "Running core engine took $engineRunTime ms" }
        checkBaselineCreation(result)
        result = transformResult(result)
        val (outputResultsTime) = measure { OutputFacade(arguments, result, settings).run() }
        settings.debug { "Writing results took $outputResultsTime ms" }
        if (!arguments.createBaseline) {
            checkBuildFailureThreshold(result, settings)
        }
    }
}
```
读取配置, 创建服务, 执行检测, 转化输出结果.

当没有用户自己声明的配置文件时, 使用的默认配置文件是[default-detekt-config.yml](https://github.com/arturbosch/detekt/blob/master/detekt-cli/src/main/resources/default-detekt-config.yml).

在这里可以看到对所有规则的默认开关状态.

detekt的规则代码放在`detekt-rules`这个module下:
[detekt rules](https://github.com/arturbosch/detekt/tree/master/detekt-rules/src/main/kotlin/io/gitlab/arturbosch/detekt/rules).
![detekt rules](/images/detekt-rules.png)

官方文档上也有说明, 比如这是performance分类下的rules: [detekt performance](https://arturbosch.github.io/detekt/performance.html).


### 包含ktlint的规则
detekt的规则种类更多, 并且它包含了ktlint的代码检查规则.
所有的`ktlint`规则被包装在`detekt-formatting`这个module下, 见: [detekt formatting rules](https://github.com/arturbosch/detekt/tree/master/detekt-formatting/src/main/kotlin/io/gitlab/arturbosch/detekt/formatting/wrappers).

但是并不是所有规则都默认开启.

在配置文件的`formatting`块可以查看具体的规则开关状态.
![detekt config formatting](/images/detekt-config-formatting-settings.png)

如果需要修改定制, 需要生成自己的配置文件.

## 和Git hook结合
Lint工具的运行可以放在CI上, 也可以每次本地手动跑, 这样在push之前就能发现错误并改正.

一种避免忘记的方法就是使用Git hook.

项目的`.git/hooks`文件夹下自带一些有意思的sample.

### 如何使用git hook
只要在`.git/hooks`路径下添加对应名字的文件, 如`pre-push`, `pre-commit`, 写入想要的代码即可.

hook文件返回0表示通过, 会进行下一步操作.
否则后面的操作会被放弃.

注意文件要有执行权限:
`chmod +x 文件名`

如果不想跑hook可以加上 `--no-verify`
比如
```
git commit --no-verify
```

### 在提交之前运行静态检查
以ktlint为例, 我们希望每次commit的时候先运行`./gradlew ktlint`, 如果有issue则放弃提交.

在`.git/hooks`中添加`pre-commit`文件, 其中:
```
#!/bin/bash
echo "Running ktlint"

./gradlew ktlint
result=$?
if [ "$result" = 0 ] ; then    
   echo "ktlint found no problems"     
   exit 0
else
   echo "Problems found, files will not be committed."     
   exit 1
fi
```
detekt的添加也是类似, 官网上给了一个例子:
[detekt/git-pre-commit-hook](https://arturbosch.github.io/detekt/git-pre-commit-hook.html).


### 如何track, 共享Git hooks
因为hooks文件在`.git`目录里, 只是在本地, 那么在团队成员间如何共享呢?

根据这个问题:
https://stackoverflow.com/questions/427207/can-git-hook-scripts-be-managed-along-with-the-repository

从git 2.9开始, 可以设置:`core.hooksPath`了.

可以在repo里面添加一个目录hooks, 然后把git hooks文件放进去track.

在命令行跑:
```
git config core.hooksPath hooks
```
把找hook文件的目录设置成指定目录就好了.

## 总结
本文介绍了三种静态检查的工具:
* Android Lint: Android自带的Lint检查. 检查的内容比较丰富.
* ktlint: Kotlin代码样式规范检查. 特色: 有自动修正命令.
* detekt: 除了code style还有performance, exceptions, bugs等方面的检查. formatting模块包含了ktlint的所有style检查(虽然有的具体的规则默认不开启, 但是可以配置), 是前者的超集.

这些工具都可以扩展, 添加自定义的检查规则.

结合Git hook把静态检查在本地进行, 可以实现问题的尽早暴露和修改.

### 示例代码
[AndroidLintDemo](https://github.com/mengdd/AndroidLintDemo)
* ktlint集成: ktlint分支.
* detekt集成: detekt分支.

## 参考
* [Android Lint: 官方文档](https://developer.android.com/studio/write/lint)
* [ktlint](https://github.com/pinterest/ktlint)
* [detekt](https://github.com/arturbosch/detekt)
* [Kotlin静态代码检测方案分析](http://mouxuejie.com/blog/2019-04-26/kotlin-static-code-analysis/)
