# NewsNow API 文档

## 概述

NewsNow 提供两个 HTTP 接口用于获取各平台热榜/最新新闻数据。所有接口均返回 JSON，无需鉴权（ProductHunt 源除外，需服务端配置 API Token）。

---

## 接口一：获取单个源数据

```
GET /api/s?id={sourceId}
```

### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 数据源 ID，见下方源列表 |
| `latest` | string | 否 | 传任意非 `false` 值时，强制跳过缓存拉取最新数据（需登录） |

### 响应结构

```json
{
  "status": "success",
  "id": "zhihu",
  "updatedTime": 1710000000000,
  "items": [
    {
      "id": "123456",
      "title": "新闻标题",
      "url": "https://example.com/article/123456",
      "mobileUrl": "https://m.example.com/article/123456",
      "pubDate": 1710000000000,
      "extra": {
        "hover": "摘要或副标题",
        "info": "附加信息，如热度、作者",
        "date": 1710000000000,
        "icon": "https://example.com/icon.png"
      }
    }
  ]
}
```

### 响应字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `status` | `"success"` \| `"cache"` | `success` 为实时数据，`cache` 为缓存数据 |
| `id` | string | 实际使用的源 ID（redirect 后的） |
| `updatedTime` | number | 数据更新时间戳（毫秒） |
| `items` | NewsItem[] | 新闻列表，最多 30 条 |

### NewsItem 字段说明

| 字段 | 类型 | 必有 | 说明 |
|------|------|------|------|
| `id` | string \| number | 是 | 条目唯一标识 |
| `title` | string | 是 | 标题 |
| `url` | string | 是 | 跳转链接 |
| `mobileUrl` | string | 否 | 移动端链接 |
| `pubDate` | number \| string | 否 | 发布时间，毫秒时间戳或日期字符串 |
| `extra.hover` | string | 否 | 鼠标悬停显示的摘要/描述 |
| `extra.info` | string \| false | 否 | 显示在标题旁的附加信息（热度、作者、分数等） |
| `extra.date` | number \| string | 否 | 显示用的时间（部分源用此字段替代 pubDate） |
| `extra.icon` | string \| object \| false | 否 | 图标 URL，或 `{ url, scale }` 对象 |

---

## 接口二：批量获取多个源缓存数据

```
POST /api/s/entire
```

### 请求体

```json
{
  "sources": ["zhihu", "weibo", "bilibili-hot-search"]
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sources` | string[] | 是 | 源 ID 数组 |

### 响应结构

返回数组，每个元素结构与 `/api/s` 相同，`status` 固定为 `"cache"`。只返回已有缓存的源，未缓存的源不会出现在结果中。

```json
[
  {
    "status": "cache",
    "id": "zhihu",
    "updatedTime": 1710000000000,
    "items": [...]
  },
  {
    "status": "cache",
    "id": "weibo",
    "updatedTime": 1710000000000,
    "items": [...]
  }
]
```

> 此接口仅读缓存，不会触发实时抓取，适合前端首屏批量加载。

---

## 数据源列表

`redirect` 表示该 ID 会自动跳转到另一个 ID，两者等价。`interval` 为缓存刷新间隔（毫秒）。

### 中国热榜

| ID | 名称 | 类型 | interval | 备注 |
|----|------|------|----------|------|
| `zhihu` | 知乎热榜 | hottest | 600000 | |
| `weibo` | 微博实时热搜 | hottest | 120000 | |
| `baidu` | 百度热搜 | hottest | 600000 | |
| `toutiao` | 今日头条热榜 | hottest | 600000 | |
| `tieba` | 百度贴吧热议 | hottest | 600000 | |
| `douyin` | 抖音热搜 | hottest | 600000 | |
| `bilibili` → `bilibili-hot-search` | B站热搜 | hottest | 600000 | |
| `bilibili-hot-video` | B站热门视频 | hottest | 600000 | Cloudflare 环境禁用 |
| `bilibili-ranking` | B站排行榜 | hottest | 1800000 | Cloudflare 环境禁用 |
| `kuaishou` | 快手热榜 | hottest | 600000 | Cloudflare 环境禁用 |
| `douban` | 豆瓣热门电影 | hottest | 600000 | |
| `hupu` | 虎扑主干道热帖 | hottest | 600000 | |
| `nowcoder` | 牛客热搜 | hottest | 600000 | |
| `ifeng` | 凤凰网热点资讯 | hottest | 600000 | |
| `tencent` → `tencent-hot` | 腾讯新闻综合早报 | hottest | 1800000 | |
| `thepaper` | 澎湃新闻热榜 | hottest | 1800000 | |
| `qqvideo` → `qqvideo-tv-hotsearch` | 腾讯视频热搜榜 | hottest | 1800000 | |
| `iqiyi` → `iqiyi-hot-ranklist` | 爱奇艺热播榜 | hottest | 1800000 | |
| `chongbuluo` → `chongbuluo-latest` | 虫部落最新 | realtime | 1800000 | |
| `chongbuluo-hot` | 虫部落最热 | hottest | 1800000 | |
| `freebuf` | Freebuf 网络安全 | hottest | 600000 | |

### 科技

| ID | 名称 | 类型 | interval | 备注 |
|----|------|------|----------|------|
| `ithome` | IT之家 | realtime | 600000 | |
| `36kr` → `36kr-quick` | 36氪快讯 | realtime | 600000 | |
| `36kr-renqi` | 36氪人气榜 | hottest | 600000 | |
| `v2ex` → `v2ex-share` | V2EX 最新分享 | realtime | 600000 | |
| `juejin` | 稀土掘金热榜 | hottest | 600000 | |
| `sspai` | 少数派热门文章 | hottest | 600000 | |
| `coolapk` | 酷安今日最热 | hottest | 600000 | |
| `github` → `github-trending-today` | GitHub Trending | hottest | 600000 | |
| `hackernews` | Hacker News | hottest | 600000 | |
| `producthunt` | Product Hunt | hottest | 600000 | 需配置 `PRODUCTHUNT_API_TOKEN` |
| `solidot` | Solidot | realtime | 3600000 | |
| `linuxdo` → `linuxdo-latest` | Linux.do 最新 | realtime | 600000 | |
| `linuxdo-hot` | Linux.do 热门 | hottest | 600000 | |
| `ghxi` | 果核剥壳 | realtime | 600000 | |
| `pcbeta` → `pcbeta-windows11` | 远景论坛 Win11 | realtime | 300000 | RSS |
| `pcbeta-windows` | 远景论坛 Windows | realtime | 300000 | RSS |

### 财经

| ID | 名称 | 类型 | interval | 备注 |
|----|------|------|----------|------|
| `wallstreetcn` → `wallstreetcn-quick` | 华尔街见闻快讯 | realtime | 300000 | |
| `wallstreetcn-news` | 华尔街见闻最新 | realtime | 1800000 | |
| `wallstreetcn-hot` | 华尔街见闻最热 | hottest | 1800000 | |
| `cls` → `cls-telegraph` | 财联社电报 | realtime | 300000 | |
| `cls-depth` | 财联社深度 | realtime | 600000 | |
| `cls-hot` | 财联社热门 | hottest | 600000 | |
| `xueqiu` → `xueqiu-hotstock` | 雪球热门股票 | hottest | 120000 | |
| `jin10` | 金十数据 | realtime | 600000 | |
| `gelonghui` | 格隆汇事件 | realtime | 120000 | |
| `fastbull` → `fastbull-express` | 法布财经快讯 | realtime | 120000 | |
| `fastbull-news` | 法布财经头条 | realtime | 1800000 | |
| `mktnews` → `mktnews-flash` | MKTNews 快讯 | realtime | 120000 | |

### 国际

| ID | 名称 | 类型 | interval | 备注 |
|----|------|------|----------|------|
| `zaobao` | 联合早报 | realtime | 1800000 | 第三方早晨报站点 |
| `cankaoxiaoxi` | 参考消息 | realtime | 1800000 | |
| `sputniknewscn` | 卫星通讯社中文 | realtime | 600000 | 走代理中转 |
| `kaopu` | 靠谱新闻 | realtime | 1800000 | |
| `steam` | Steam 在线人数 | hottest | 600000 | |

---

## 各源 extra 字段详情

不同源返回的 `extra` 内容不同，以下列出各源实际会填充的字段：

| 源 | `extra.info` | `extra.hover` | `extra.icon` | `extra.date` |
|----|-------------|---------------|--------------|--------------|
| `zhihu` | 热度文本（如 "1234万热度"） | 问题摘要 | - | - |
| `weibo` | - | - | 新/热/爆 标志图片 URL | - |
| `bilibili-hot-search` | - | - | 词条图标 URL | - |
| `bilibili-hot-video` / `bilibili-ranking` | `作者 · N万观看 · N点赞` | 视频简介 | 封面图 URL | - |
| `hackernews` | 分数（如 "123 points"） | - | - | - |
| `github-trending-today` | `✰ 星数` | 仓库描述 | - | - |
| `producthunt` | `△ 投票数` | 产品 tagline | - | - |
| `36kr-renqi` | `作者 \| 热度` | 文章描述 | - | - |
| `36kr-quick` | - | - | - | 发布时间戳 |
| `wallstreetcn` | - | - | - | 发布时间戳 |
| `cls-telegraph` / `cls-depth` | - | - | - | - |
| `jin10` | `✰`（重要标记） | 正文内容 | - | - |
| `mktnews-flash` | `"Important"`（重要时） | 正文内容 | - | - |
| `xueqiu-hotstock` | `涨跌幅% 交易所` | - | - | - |
| `gelonghui` | 来源媒体 | - | - | 发布时间戳 |
| `douban` | 年份/导演/主演（前3项） | 完整副标题 | - | - |
| `steam` | 当前在线人数 | - | - | - |
| `coolapk` | 热度（如 "374.4万热度"） | - | - | - |
| `iqiyi-hot-ranklist` | 描述文本 | 详细描述 | - | - |
| `qqvideo-tv-hotsearch` | - | 副标题 | - | - |
| `toutiao` | - | - | 标签图标 URL | - |
| `freebuf` | - | 文章摘要 | - | - |
| `kuaishou` | - | - | 热词图标 URL | - |

---

## MCP 工具

NewsNow 同时暴露了一个 MCP（Model Context Protocol）工具，供 AI 助手调用。

### 工具名：`get_hotest_latest_news`

**描述：** 通过源 ID 获取热榜或最新新闻。

**参数：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `id` | string | 必填 | 源 ID，同上方源列表 |
| `count` | number | 10 | 返回条数 |

**返回：** 文本列表，每条格式为 `[标题](URL)`。

**示例调用：**
```json
{
  "id": "zhihu",
  "count": 5
}
```

**示例返回：**
```
[为什么年轻人越来越不愿意结婚？](https://www.zhihu.com/question/123456)
[如何看待...](https://www.zhihu.com/question/234567)
...
```

---

## 缓存机制说明

- 每个源有独立的 `interval`（刷新间隔）和 TTL（缓存有效期，默认更长）
- `interval` 内命中缓存直接返回，`status` 为 `"success"`，`updatedTime` 为当前时间
- TTL 内但超过 `interval`，未登录用户返回缓存，`status` 为 `"cache"`，`updatedTime` 为缓存写入时间
- 登录用户带 `latest` 参数可强制刷新
- `/api/s/entire` 只读缓存，不触发刷新
