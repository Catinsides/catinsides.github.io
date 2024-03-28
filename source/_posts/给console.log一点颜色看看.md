---
title: 给console.log一点颜色看看
date: 2017-07-03 23:44:15
tags: Javascript
description: 开发中少不了使用console.log进行调试，那么如何使打印出的内容具有颜色呢？
---
开发中少不了使用console.log进行调试，那么如何使打印出的内容具有颜色呢？

首先要说明的是，这里指的打印是在命令行中进行输出的，而不是在chrome控制台中。

答案是，把console.log()第一个参数设置为ANSI转义码即可, 第二个参数为需要打印的内容。

```javascript
// 像这样
console.log("\x1b[31m", "I'm green.");

// 当然，聪明而又懒惰的我们不会一个一个去敲这些字符，真是光看看就觉得麻烦啊，So...
const FgRed = "\x1b[31m";
console.log(FgRed, "I'm red.");

// 真是雕虫小技啊:P
// 这里还有有更多的颜色
Reset = "\x1b[0m",
Bright = "\x1b[1m",
Dim = "\x1b[2m",
Underscore = "\x1b[4m",
Blink = "\x1b[5m",
Reverse = "\x1b[7m",
Hidden = "\x1b[8m",
FgBlack = "\x1b[30m",
FgRed = "\x1b[31m",
FgGreen = "\x1b[32m",
FgYellow = "\x1b[33m",
FgBlue = "\x1b[34m",
FgMagenta = "\x1b[35m",
FgCyan = "\x1b[36m",
FgWhite = "\x1b[37m",
BgBlack = "\x1b[40m",
BgRed = "\x1b[41m",
BgGreen = "\x1b[42m",
BgYellow = "\x1b[43m",
BgBlue = "\x1b[44m",
BgMagenta = "\x1b[45m",
BgCyan = "\x1b[46m",
BgWhite = "\x1b[47m";
```

拓展阅读:[ANSI escape code](https://en.wikipedia.org/wiki/ANSI_escape_code)
