# 废弃的语法设计

## jslt style

请求 `http://www.bjp.org.cn/APOD/list.shtml`

```javascript
--today-link-selector: "td[align=left] b a"

{
  url: css(var(--today-link-selector)).attr("href"),
  title: css(var(--today-link-selector)).text(),
  time: css("span").replace("：", "").parse_date("YYYY-MM-DD").timezone(8),
  cover: css(var(--today-link-selector))
    .attr("href")
    .fetch()
    .css(".juzhong img")
    .attr("src")
}
```
