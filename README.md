# Bright-Data-Browser-API-in-Action-Comparing-Scraping-of-Booking.com-Dynamic-Pages-blog
亮数据 | Browser API 实战：Booking.com 动态页面抓取对比 | 博客


> 视频讲解：
> (https://v-blog.csdnimg.cn/asset/41f7b7fda50cfc770b19bf97ccbaf049/play_video/f5ff017de77a44e53ced64d520905493.m3u8)(title-JS网站太难爬？Browser API处理动态网站全流程实战)]

# 前言

在做旅游、电商、社交平台这类网页采集时，很多页面已经不是简单的静态 HTML。页面内容需要 JS 渲染，还可能包含验证码、弹窗、搜索表单、日期选择器等交互流程。

这次我选择 Booking.com 作为测试目标，搜索地点为 `New York`，流程包括：

- 打开 Booking.com 首页
- 输入搜索地点
- 选择入住和退房日期
- 提交搜索
- 抓取酒店名称、价格、评分和评价数量

这个场景比较适合验证 Browser API，因为它不是单纯请求一个 URL 就能拿到结构化数据。

# 普通爬虫遇到的问题

我先用普通 `requests` 方式请求 Booking.com 首页，即使加了浏览器 UA、Cookie 等请求头，返回状态码仍然是 `202`。

简化后的测试逻辑如下：

```python
import requests

url = "https://www.booking.com/"
headers = {
    "user-agent": "Mozilla/5.0 ..."
}

response = requests.get(url, headers=headers)
print(response.status_code)
```

这里的 `202` 并不代表已经拿到了正常页面内容。对爬虫来说，它更像是网站把请求放进了验证或风控流程里。普通 HTTP 请求无法执行页面里的 JS，也无法像真实用户一样完成点击、等待、表单提交等动作，所以后续很难继续解析有效数据。

# Bright Data Browser API 的配置

这次我改用 Bright Data Browser API，通过 Playwright 连接云端浏览器。配置都放在代码前面，核心配置如下：

```python
AUTH = "用户名:密码"
BROWSER_WS = f"wss://{AUTH}@brd.superproxy.io:9222"

URL = "https://www.booking.com/"
SEARCH_TEXT = "New York"
CHECK_IN_AFTER_DAYS = 1
CHECK_OUT_AFTER_DAYS = 2
MAX_RESULTS = 20
```

运行逻辑也比较清晰：

```python
browser = playwright.chromium.connect_over_cdp(BROWSER_WS)
page = browser.new_page()
response = page.goto(URL, wait_until="domcontentloaded")
```

相比普通请求，Browser API 的关键区别在于：它使用的是真实浏览器环境，可以执行 JS、等待页面加载、处理弹窗、填写搜索框、点击日期和提交按钮。

# 实际运行结果

本次运行时，Booking.com 首页初始状态码仍然出现了 `202`，但 Bright Data Browser API 可以继续处理验证流程，并进入最终搜索结果页。

终端输出结果显示：

```text
正在连接 Bright Data Browser API...
连接成功，正在打开 Booking.com...
页面已打开，初始状态码：202
验证处理状态：not_detected
已进入搜索结果页
抓取完成，共获取 20 条结果
JSON 结果：booking_results.json
HTML 页面：booking_result.html
页面截图：booking_result.png
```

部分抓取结果如下：

| 酒店名称                                                     | 价格 | 评分 | 评价数 |
| ------------------------------------------------------------ | ---- | ---- | ------ |
| The Cloud One New York-Downtown, part of the Motel One Group | $357 | 8.8  | 8113   |
| Staypineapple, An Artful Hotel, Midtown New York             | $304 | 8.7  | 1145   |
| Renaissance New York Midtown Hotel                           | $499 | 8.8  | 1692   |
| Archer Hotel New York                                        | $499 | 8.9  | 1093   |
| Hotel Indigo NYC Financial District by IHG                   | $359 | 9.0  | 1866   |

从结果来看，普通请求卡在 `202`，而 Browser API 完整走完了“打开页面 - 搜索 - 等待结果 - 解析卡片”的流程，最终拿到了可用数据。

# 体验总结

这次测试最大的感受是：动态网站不能只看请求是否发出成功，还要看能不能拿到最终渲染后的页面。

普通 `requests` 更适合结构简单、无强交互、无复杂 JS 渲染的网站；而像 Booking.com 这种带搜索表单、日期选择、验证流程的网站，就更适合用 Browser API 这种真实浏览器方案。

对于爬虫开发者来说，Bright Data Browser API 的价值主要体现在：

- 可以处理 JS 重度渲染页面
- 支持点击、输入、等待、截图等浏览器操作
- 能应对普通请求遇到的验证和风控流程
- 抓取结果可以直接保存为 JSON、HTML 和截图，方便复盘

这也是我这次选择 Browser API 做实战演示的原因：它解决的不是“怎么请求网页”，而是“怎么像真实用户一样完成网页操作并拿到最终数据”。

> 通过以下专属链接注册，在结账时输入`brd09` 即可抵扣 30 美金
> 福利入口：[https://www.bright.cn/products/scraping-browser?utm_source=brand&utm_campaign=brnd-mkt_cn_csdn_qidian202609](https://www.bright.cn/products/scraping-browser?utm_source=brand&utm_campaign=brnd-mkt_cn_csdn_qidian202609)
> 视频讲解：见文首
