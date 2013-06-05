---
layout: post
title: "说说 Objective-C 里的 @()"
date: 2013-06-06 00:39
comments: true
categories: [iOS, Objective-C]
---

## Intro

在 Objective-C 中我们可以用 `@"foo"` 来创建一个 `NSString` 常量，看起来似乎平淡无奇。

但它背后其实比想象的精彩，`@` 可以被理解成一个特殊的宏，其接受一个 C 字符串作为参数，也可写作 `@("foo")`。

之所以说 `@` 是一个特殊的宏，是因为其能根据传入的 C 字符串类型不同——C 字符串常量或 C 字符串——在运行时构建返回不同类型的 `NSString`，参见下面的代码：

``` objective-c
char* obtain_c_string(void)
{
	return "c_string";
}

NSLog(@"%@", @"foo".class);
NSLog(@"%@", @("bar").class);
NSLog(@"%@", @(obtain_c_string()).class);
```

输出结果如下：

``` objective-c
2013-06-05 01:14:15.097 Sandbox[45804:c07] __NSCFConstantString
2013-06-05 01:14:15.098 Sandbox[45804:c07] __NSCFConstantString
2013-06-05 01:14:15.098 Sandbox[45804:c07] __NSCFString
```

可见，如果传入的是 C 字符串常量，运行时构建的则为 `NSConstantString`；如果传入的是 C 字符串则创建的是 `NSString`。

## Then?

你可能会问这么理解了又怎样？

众所周知，Objective-C 代码里有很多地方需要我们把代码中的一些文法串写成字符串再作为传入参数，比如 KVO 中的 `keyPath` 参数往往就要传入形如 `propertyA.propertyB` 的字符串，从实用角度出发这有两个弊端：

- 写字符串的时候没有代码提示，很容易写错
- 即便一开始写对了，如果后来相关类重构了，`keyPath` 的参数便失效了，而 Xcode Refactor 无法扫描字符串

当我们理解了 `@()`，再加上自定义的宏，上述两个问题便可迎刃而解。

``` c
/**
 * # 将宏的参数字符串化，C 函数 strchr 返回字符串中第一个 '.' 字符的位置
 */
#define Keypath(keypath) (strchr(#keypath, '.') + 1)

[objA addObserver:objB
       forKeyPath:@Keypath(ObjA.property1.property2) // 有代码提示，可以被重构扫描到
          options:nil
          context:nil];
```

这个简单实现只算是抛砖引玉，除了 `@()` 配合自定义宏来字符串化代码中的文法串，更多的用法就有待在开发中不断发掘了。

*PS: 在即将完成这篇文章的时候我发现已有国外开发者利用 `@()` 特性配合自定义宏，全面系统的解决了上述问题，详情参见 [libextobjc/EXTKeyPathCoding.h](https://github.com/jspahrsummers/libextobjc/blob/master/extobjc/EXTKeyPathCoding.h)。*

## Extra

此外，`@()` 还可以接受 int 字面量或 int 变量作为参数，有兴趣的读者可以自行感受下。
