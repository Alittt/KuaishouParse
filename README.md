# ** KSHook API 文档**

> 快手分享链接无水印解析 API — 仅供技术学习与研究使用

Base URL: `https://parse.api-demoo.com`

# KSHook API

Base URL: `https://apihook.apifox.cn`

## 快速开始

```bash
# 最简单的调用
curl "https://parse.api-demoo.com/api/parse?url=https://v.kuaishou.com/xxx"

# POST 方式
curl -X POST "https://parse.api-demoo.com/api/parse" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://v.kuaishou.com/xxx"}'
```

**响应示例（成功）**:

```json
{
  "total": 1,
  "success": 1,
  "failed": 0,
  "results": [
    {
      "success": true,
      "url": "https://v.kuaishou.com/xxx",
      "media_type": "VIDEO",
      "title": "视频标题文案",
      "video_url": "https://v4.oskwai.com/bs2/photo-video-mz/xxx.mp4",
      "cover_url": "https://tx2.a.kwimgs.com/xxx.jpg",
      "music_url": "https://tx2.a.kwimgs.com/xxx.m4a",
      "duration_ms": 30500,
      "duration_str": "00:00:30",
      "quality": "1080p",
      "author": {
        "uid": "1234567890",
        "name": "作者昵称",
        "avatar": "https://tx2.a.kwimgs.com/xxx.jpg"
      },
      "like_count": 1234,
      "view_count": 56789,
      "comment_count": 88,
      "share_count": 12,
      "timestamp": 1750000000,
      "photo_id": "5213479715188820019",
      "user_id": "1234567890",
      "short_code": "xxx",
      "source": "ks.api-demoo.com",
      "all_video_qualities": [
        {
          "url": "https://v4.oskwai.com/bs2/photo-video-mz/xxx_hd1.mp4",
          "quality": "360p",
          "height": 360,
          "width": 640,
          "is_hevc": false,
          "bandwidth": 300000
        },
        {
          "url": "https://v4.oskwai.com/bs2/photo-video-mz/xxx_hd2.mp4",
          "quality": "720p",
          "height": 720,
          "width": 1280,
          "is_hevc": false,
          "bandwidth": 800000
        },
        {
          "url": "https://v4.oskwai.com/bs2/photo-video-mz/xxx.mp4",
          "quality": "1080p",
          "height": 1080,
          "width": 1920,
          "is_hevc": true,
          "bandwidth": 2000000
        }
      ]
    }
  ]
}
```

---

## 通用说明

### 支持的链接格式

| 格式 | 示例 |
|------|------|
| 完整分享链接 | `https://v.kuaishou.com/xxx` |
| 短链接格式 | `v.kuaishou.com/xxx` |
| 纯短码 | `KhnYZs1J` |

### 请求限制

- 批量接口最多 **100 条**链接，超出部分自动截断
- 批量请求自动**去重**
- 超时时间：15 秒（解析）、60 秒（下载/代理）

### 通用响应包装

所有解析接口返回统一格式的 `ParseResponse`：

| 字段 | 类型 | 说明 |
|------|------|------|
| `total` | int | 总条数 |
| `success` | int | 成功条数 |
| `failed` | int | 失败条数 |
| `results` | array | 结果列表，每项为 `ParseResult` 或 `ParseError` |

---

## 接口列表

---

### 1. 解析单条链接 GET

**`GET /api/parse`**

通过 Query 参数传入链接。

**请求示例**:

```
GET https://parse.api-demoo.com/api/parse?url=https://v.kuaishou.com/xxx
```

**参数**:

| 参数名 | 位置 | 类型 | 必填 | 说明 |
|--------|------|------|------|------|
| `url` | query | string | 是 | 快手分享链接或 shortcode |

---

### 2. 解析单条链接 POST

**`POST /api/parse`**

通过 JSON Body 传入链接。

**请求示例**:

```bash
curl -X POST "https://parse.api-demoo.com/api/parse" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://v.kuaishou.com/xxx"}'
```

**请求体**:

```json
{
  "url": "https://v.kuaishou.com/xxx"
}
```

---

### 3. 批量解析

**`POST /api/batch`**

一次解析多条链接。

**请求示例**:

```bash
curl -X POST "https://parse.api-demoo.com/api/batch" \
  -H "Content-Type: application/json" \
  -d '{
    "urls": [
      "https://v.kuaishou.com/xxx1",
      "v.kuaishou.com/xxx2",
      "KhnYZs1J"
    ]
  }'
```

**请求体**:

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `urls` | string[] | 是 | 链接数组，最多 100 条 |

> 链接会自动去重，顺序按首次出现的顺序保留。

---

### 4. 代理下载

**`GET /api/download`**

通过后端代理下载媒体文件。后端携带正确 Referer 请求快手 CDN，返回文件流并自动设置文件名。

**请求示例**:

```
GET https://parse.api-demoo.com/api/download?url=https://v4.oskwai.com/bs2/photo-video-mz/xxx.mp4
```

**参数**:

| 参数名 | 位置 | 类型 | 必填 | 说明 |
|--------|------|------|------|------|
| `url` | query | string | 是 | 从解析接口获取的 CDN 直链 |

**响应头**:

| 头 | 说明 |
|----|------|
| `Content-Disposition` | `attachment; filename="ks_{photo_id}.{ext}"` |
| `Content-Type` | `application/octet-stream` |
| `X-Content-Type-Options` | `nosniff` |

---

### 5. 通用媒体代理

**`GET /api/proxy`**

将媒体请求通过后端转发，以携带正确的 Referer 头访问快手 CDN。

**请求示例**:

```
GET https://parse.api-demoo.com/api/proxy?url=https://v4.oskwai.com/bs2/photo-video-mz/xxx.mp4
```

**参数**:

| 参数名 | 位置 | 类型 | 必填 | 说明 |
|--------|------|------|------|------|
| `url` | query | string | 是 | 快手 CDN 直链 |

**支持的格式**: `.mp4`、`.jpg`、`.jpeg`、`.png`、`.webp`、`.m4a`、`.mp3`

**响应头**:

| 头 | 说明 |
|----|------|
| `Content-Type` | 自动根据扩展名设置（如 `video/mp4`、`image/jpeg`） |
| `Cache-Control` | `max-age=3600` |

---

## 数据结构

### ParseResult（解析成功）

| 字段 | 类型 | 说明 |
|------|------|------|
| `success` | bool | 固定为 `true` |
| `url` | string | 原始输入链接 |
| `media_type` | string | 媒体类型，见下方说明 |
| `title` | string | 作品标题/文案 |
| `video_url` | string | 视频直链（VIDEO 类型）或图集首帧（其他类型） |
| `cover_url` | string | 封面图直链 |
| `music_url` | string | 配乐（BGM）直链 |
| `duration_ms` | int | 视频时长（毫秒） |
| `duration_str` | string | 视频时长（HH:MM:SS 格式） |
| `quality` | string | 最佳画质标签，如 `1080p` |
| `author` | object | 作者信息 |
| `like_count` | int | 点赞数 |
| `view_count` | int | 播放/浏览数 |
| `comment_count` | int | 评论数 |
| `share_count` | int | 分享数 |
| `timestamp` | int | 发布时间（Unix 时间戳，秒） |
| `photo_id` | string | 快手作品 ID |
| `user_id` | string | 作者用户 ID |
| `short_code` | string | 作品短码 |
| `source` | string | 数据来源 |
| `all_video_qualities` | array | 所有可用画质选项（VIDEO 类型时返回） |

### AuthorInfo（作者信息）

| 字段 | 类型 | 说明 |
|------|------|------|
| `uid` | string | 作者快手用户 ID |
| `name` | string | 作者昵称 |
| `avatar` | string | 作者头像直链 |

### VideoQualityOption（画质选项）

| 字段 | 类型 | 说明 |
|------|------|------|
| `url` | string | 该画质的 CDN 直链 |
| `quality` | string | 画质标签，如 `360p`、`720p`、`1080p` |
| `height` | int | 视频高度（像素） |
| `width` | int | 视频宽度（像素） |
| `is_hevc` | bool | 是否为 HEVC（H.265）编码 |
| `bandwidth` | int | 码率（bps） |

### ParseError（解析失败）

| 字段 | 类型 | 说明 |
|------|------|------|
| `success` | bool | 固定为 `false` |
| `url` | string | 原始输入链接 |
| `error` | string | 错误原因 |

错误原因可能的值：

| 值 | 说明 |
|----|------|
| `无法识别的链接格式` | 链接格式不被支持 |
| `解析错误` | 解析过程中出错 |
| `解析失败` | 网络或服务器异常 |

---

## 媒体类型说明

| media_type | 说明 | video_url 含义 |
|------------|------|---------------|
| `VIDEO` | 普通视频 | 视频直链 |
| `SINGLE_PICTURE` | 实况图片（动图） | 实况图直链 |
| `VERTICAL_ATLAS` | 竖版图集 | 图集第一张图片 |
| `HORIZONTAL_ATLAS` | 横版图集 | 图集第一张图片 |

---

## 常见问题

### Q: 视频链接播放不了？

快手 CDN 链接带有 `pkey` 参数，有有效期限制。不同画质链接有效期不同：
- **普清链接**（无 `pkey` 或过期时间较长）：较稳定
- **高清链接**（`pkey=AA...`）：有效期较短，可能 403

建议：解析成功后优先使用普清画质，或在解析结果中使用 `all_video_qualities` 字段自行选择稳定画质。

### Q: 批量解析有数量限制吗？

最多 100 条，超出自动截断。

### Q: 请求超时了怎么办？

解析接口超时 15 秒，下载/代理 60 秒。建议网络不佳时分批请求。

### Q: 可以免费使用吗？

本 API 仅供技术学习与研究使用，请勿用于商业用途或大规模爬取。

---

*© 2026 Alittt. KSHook — 仅供技术学习与研究使用*
