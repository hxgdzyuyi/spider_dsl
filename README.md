# SpiderDSL（构思中）

一个构思中的，用来快速爬取网页内容并转换成 JSON 的类 css 风格的 DSL 语言。

## 适用需求举例

- 使用 Node.js 从一个网页生成一个 RSS。
- 使用 Swift 为任意一个网页在 iOS 客户端生成 Widget （比如爬取 APOD 每日一图）。

## 设计目的

- 能快速的在不同语言下使用，或转换成不同语言
- 简单，好学，能用

## 简单举例

```sass
---
request: http://www.bjp.org.cn/APOD/list.shtml
---

.td[align=left] b a:first
  url: attr("href")
```

对应转换后的 JavaScript 代码

```javascript
const cheerio = require('cheerio');
const got = require('@/utils/got');

const res = await got("http://www.bjp.org.cn/APOD/list.shtml");
const $ = cheerio.load(res.data);

var result = {}
result.url = $('td[align=left] b').first().attr("href")
return result
```

## 简单规则
- css 选择器代表了当前选择的节点
- css 嵌套选择器可以识别上下文进行当前节点的选择
- 可以通过 @request 关键字在上下文中进行第二个页面的请求并形成上下文
- 原 css 语法中的 `属性:值` 代表了返回的 json 的属性和值
- 属性 `[items].[tags].title: text()` 可以用来代表返回值中具有属性 `{ items: [tags: {text: "当前节点的 text 值"} ] }`
- jQuery 的扩展语法 `:first` 代表了只选取第一个元素， 如果没有，就进行 `.each` 迭代
- 值中的链式调用相当于管道，仅由函数或者字符串构成
- 函数来自于各个语言的内建函数

## 复杂举例

```txt
---
request: http://www.bjp.org.cn/APOD/list.shtml
---

.td[align=left] b a:first
  url: attr("href")
  title: text()
  @request attr("href")
    .juzhong img
      cover: attr("src")

```

对应转换后的 JavaScript 代码

```javascript
const cheerio = require('cheerio');
const got = require('@/utils/got');
const { parseDate } = require('@/utils/parse-date');
const timezone = require('@/utils/timezone');

module.exports = async (ctx) => {
    const baseUrl = 'https://www.bjp.org.cn';
    const listUrl = `${baseUrl}/APOD/list.shtml`;

    const res = await got(listUrl);
    var result = {}

    const $ = cheerio.load(res.data);

    const e = $($('td[align=left] b').first())
    result["title"] = e.find('a').attr('title')
    result["url"] = e.find('a').attr('href')
    result["time"] = timezone(parseDate(e.find('span').text().replace('：', ''), 'YYYY-MM-DD'), 8)

    ctx.cache.tryGet(e.link, async () => {
        const { data } = await got.get(e.link);
        const $ = cheerio.load(data);

        result["cover"] = $('.juzhong img').attr("src");

        return e;
    })

    return reuslt
};
```
