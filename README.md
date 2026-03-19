# TechFlow API 接入文档

本文档面向第三方调用方，说明 TechFlow 对外开放的内容列表 API。

## Base URL

固定地址：

```text
https://api.techflowpost.com/
```

完整请求示例：

```text
GET https://api.techflowpost.com/api/v1/external/articles
GET https://api.techflowpost.com/api/v1/external/newsflashes
```

## 频率限制

两个接口共用同一个 rate limit 配额：

- 规则：每个 IP 每分钟最多 60 次请求
- 超限状态码：`429 Too Many Requests`
- 返回标准 `RateLimit-*` 响应头

这意味着：

- 同一个 IP 请求文章列表和快讯列表，会共同消耗额度
- 例如连续请求 40 次文章列表、20 次快讯列表后，再发第 61 次请求会被限流

## 通用分页参数

两个接口都支持以下参数：

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `page` | number | 否 | `1` | 页码，从 1 开始 |
| `page_size` | number | 否 | `10` | 每页数量，最大 `100` |
| `keyword` | string | 否 | - | 按标题模糊搜索 |

补充说明：

- `keyword` 当前只匹配标题，不匹配正文
- 返回内容固定按 `created_at desc` 排序
- 仅返回 `is_visible = true` 且 `is_deleted = false` 的数据

## 文章列表

### 请求

```http
GET /api/v1/external/articles?page=1&page_size=10&keyword=bitcoin
```

### 响应字段

```json
{
  "data": [
    {
      "id": 1,
      "title": "文章标题",
      "abstract": "文章摘要",
      "content": "文章正文",
      "cover": "https://cdn.example.com/cover.jpg",
      "created_at": "2026-03-19T00:00:00.000Z",
      "category": {
        "id": 3,
        "name": "市场"
      },
      "author": {
        "id": 9,
        "name": "作者名",
        "avatar": "https://cdn.example.com/avatar.jpg"
      }
    }
  ],
  "total": 128,
  "page": 1,
  "page_size": 10
}
```

### 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | number | 文章 ID |
| `title` | string \| null | 标题 |
| `abstract` | string \| null | 摘要 |
| `content` | string \| null | 正文内容 |
| `cover` | string \| null | 封面图 URL |
| `created_at` | string | 创建时间，ISO 8601 格式 |
| `category.id` | number | 分类 ID |
| `category.name` | string \| null | 分类名称 |
| `author.id` | number | 作者 ID |
| `author.name` | string | 作者名称 |
| `author.avatar` | string \| null | 作者头像 |

## 快讯列表

### 请求

```http
GET /api/v1/external/newsflashes?page=1&page_size=10&keyword=ethereum
```

### 响应字段

```json
{
  "data": [
    {
      "id": 101,
      "title": "快讯标题",
      "abstract": "快讯摘要",
      "content": "快讯正文",
      "created_at": "2026-03-19T00:00:00.000Z",
      "category": {
        "id": 2,
        "name": "快讯分类"
      }
    }
  ],
  "total": 56,
  "page": 1,
  "page_size": 10
}
```

### 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | number | 快讯 ID |
| `title` | string \| null | 标题 |
| `abstract` | string \| null | 摘要 |
| `content` | string \| null | 正文内容 |
| `created_at` | string | 创建时间，ISO 8601 格式 |
| `category.id` | number | 分类 ID |
| `category.name` | string \| null | 分类名称 |

## Curl 示例

### 获取文章列表

```bash
curl --request GET \
  --url 'https://api.techflowpost.com/api/v1/external/articles?page=1&page_size=10&keyword=bitcoin'
```

### 获取快讯列表

```bash
curl --request GET \
  --url 'https://api.techflowpost.com/api/v1/external/newsflashes?page=1&page_size=10&keyword=ethereum'
```

## 错误响应

### 400 Bad Request

请求参数不合法时返回，例如：

- `page=0`
- `page_size=101`
- `category_id=1`

### 429 Too Many Requests

超过频率限制时返回，例如：

```json
{
  "message": "Too many external requests, please try again later"
}
```

## 对接建议

- 接入方应缓存列表结果，避免频繁请求
- 建议优先用 `page` 和 `page_size` 做增量拉取
- 若需要按正文搜索、详情接口或更高频率额度，建议单独沟通扩展接口
