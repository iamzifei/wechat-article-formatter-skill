---
name: wechat-article-formatter
description: Format markdown and publish to WeChat Official Account via bm.md rendering + WeChat official API
---

# WeChat Article Formatter & Publisher

Format markdown articles with bm.md styling and publish directly to WeChat Official Account drafts via the official API.

## Overview

1. Read markdown file (typically `wechat.md` from article directory)
2. Upload local images to WeChat CDN via `media/uploadimg` API
3. Replace local image paths with WeChat CDN URLs in markdown
4. Render markdown to styled HTML via bm.md API with custom CSS
5. Publish to WeChat draft via official `draft/add` API

**CRITICAL**: Do NOT use third-party publishing APIs (e.g. 微绿 `wechat-publish`). They strip inline CSS styles. Always use WeChat's official `draft/add` API directly.

## Credentials

Read from `~/.env`:

| Variable | Purpose |
|----------|---------|
| `WECHAT_APP_ID` | WeChat Official Account AppID |
| `WECHAT_APP_SECRET` | WeChat Official Account AppSecret |

If not found, prompt the user to provide them and save to `~/.env`.

## Workflow

### Step 1: Identify Input

The user provides a markdown file path (e.g. `wechat.md` in an article directory).

Detect the article directory to find:
- `promotion.md` → extract digest/summary
- `_attachments/` → local images
- Brand config from the conversation context → author name

### Step 2: Get WeChat Access Token

```python
params = urlencode({
    'grant_type': 'client_credential',
    'appid': WECHAT_APP_ID,
    'secret': WECHAT_APP_SECRET
})
# GET https://api.weixin.qq.com/cgi-bin/token?{params}
```

If error `40164` (IP not in whitelist), instruct user to add the IP at: mp.weixin.qq.com → 设置与开发 → 基本配置 → IP白名单.

### Step 3: Upload Images to WeChat CDN

**CRITICAL**: Upload ALL images in the SAME session. WeChat CDN URLs from previous sessions may expire.

Use `media/uploadimg` API for inline content images:

```python
# POST https://api.weixin.qq.com/cgi-bin/media/uploadimg?access_token={token}
# Content-Type: multipart/form-data
# Body: media=@image_file
# Response: {"url": "http://mmbiz.qpic.cn/..."}
```

Use `material/add_material` API for cover/thumb image:

```python
# POST https://api.weixin.qq.com/cgi-bin/material/add_material?access_token={token}&type=image
# Content-Type: multipart/form-data
# Body: media=@cover_image
# Response: {"media_id": "...", "url": "..."}
```

For remote images (e.g. `https://i.ibb.co/...`), download first, then upload to WeChat CDN.

After uploading, replace all local/remote image paths in the markdown with WeChat CDN URLs before rendering.

### Step 4: Render with bm.md

Read custom CSS from `{{SKILL_DIR}}/styles/custom.css`.

**IMPORTANT**: Write the JSON payload to a temp file and use `curl -d @file` to avoid shell escaping issues.

```python
payload = json.dumps({
    "markdown": md_content,
    "markdownStyle": "green-simple",
    "codeTheme": "kimbie-light",
    "customCss": css_content,
    "enableFootnoteLinks": True,
    "openLinksInNewWindow": True,
    "platform": "wechat"
}, ensure_ascii=False)

# Write to /tmp/bm-payload.json, then:
# curl -s -X POST https://bm.md/api/markdown/render \
#   -H "Content-Type: application/json" \
#   -d @/tmp/bm-payload.json
```

The `ensure_ascii=False` is critical — without it, Chinese characters become `\uXXXX` escape sequences.

### Step 5: Publish to WeChat Draft

```python
draft_payload = json.dumps({
    "articles": [{
        "title": title,
        "author": author,                    # e.g. "在悉尼和稀泥"
        "thumb_media_id": thumb_media_id,     # from Step 3 material upload
        "content": html,                      # from Step 4 bm.md render
        "digest": digest,                     # from promotion.md, max ~60 Chinese chars
        "need_open_comment": 1,               # enable comments
        "only_fans_can_comment": 0,           # allow all users to comment
        "original_article_type": 1,           # declare as original content
        "reward_wording": "喜欢这篇文章，请我喝杯咖啡",  # enable tipping
        "creation_source_type": 1             # 个人观点，仅供参考
    }]
}, ensure_ascii=False).encode('utf-8')

# POST https://api.weixin.qq.com/cgi-bin/draft/add?access_token={token}
```

**Default publish settings**:

| Field | Value | Description |
|-------|-------|-------------|
| `need_open_comment` | `1` | Enable comments |
| `only_fans_can_comment` | `0` | All users can comment |
| `original_article_type` | `1` | Declare as original |
| `reward_wording` | `"喜欢这篇文章，请我喝杯咖啡"` | Enable tipping |
| `creation_source_type` | `1` | 个人观点，仅供参考 |

**Digest extraction from promotion.md**:

Look for `## 文章摘要` section, extract text, truncate to ~60 Chinese characters (WeChat limit is 120 bytes).

**Author resolution** (first match wins):
1. User explicitly specified
2. Brand config `author` field
3. Brand config `品牌名称` field

### Step 6: Report Result

```
WeChat Publishing Complete!

Title: {title}
Author: {author}
Digest: {digest}
Original: ✓ declared
Tipping: ✓ enabled
Comments: ✓ open to all
Images: {N} uploaded to WeChat CDN
media_id: {media_id}

→ Preview at: https://mp.weixin.qq.com (内容管理 → 草稿箱)
```

## Markdown Content Order for WeChat

When building the WeChat markdown, elements should appear in this order:

```
1. 关注引导文字 (e.g. 👆 「关注」加「星标」...)
2. 封面图 (cover image)
3. --- 分隔线
4. 正文内容 (article body with inline images)
5. --- 分隔线
6. 作者介绍 (about author section)
7. --- 分隔线
8. 推广区块 (promo blockquote with image)
9. --- 分隔线
10. 互动引导 (engagement CTA)
```

The H1 title should be REMOVED from markdown — WeChat displays its own title from the `title` field.

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Chinese shows as `\uXXXX` | Use `ensure_ascii=False` in ALL `json.dumps()` calls |
| Images not displaying | Upload in same session; don't reuse old CDN URLs |
| Styles stripped (plain text) | Use WeChat official `draft/add` API, NOT third-party APIs |
| `40164` IP whitelist error | Add current IP at mp.weixin.qq.com → 基本配置 → IP白名单 |
| `45004` digest too long | Keep digest under 60 Chinese characters |
| `40007` invalid media_id | Upload cover via `material/add_material` first to get `thumb_media_id` |
| `> [!NOTE]` renders wrong | Convert GitHub alerts to regular `>` blockquotes for WeChat |

## bm.md API Reference

**Render**: `POST https://bm.md/api/markdown/render`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `markdown` | string | required | Markdown content |
| `markdownStyle` | string | `ayu-light` | Style ID (use `green-simple`) |
| `codeTheme` | string | `kimbie-light` | Code highlight theme |
| `customCss` | string | `""` | Custom CSS scoped to `#bm-md` |
| `enableFootnoteLinks` | boolean | `true` | Convert links to footnotes |
| `openLinksInNewWindow` | boolean | `true` | Links open in new window |
| `platform` | string | `html` | Target: `html`, `wechat`, `zhihu`, `juejin` |

Available styles: `ayu-light`, `bauhaus`, `blueprint`, `botanical`, `green-simple`, `maximalism`, `neo-brutalism`, `newsprint`, `organic`, `playful-geometric`, `professional`, `retro`, `sketch`, `terminal`

## Styling Features

The custom CSS (`styles/custom.css`) provides:
- Optima/Microsoft YaHei fonts
- Green accent color (rgb(53, 179, 120))
- H2 headings: white text on black background
- Bold text: green color
- Blockquotes: green left border
- Responsive tables with alternating row colors
- Code blocks with dark background

## Dependencies

- WeChat Official Account API credentials (`~/.env`)
- bm.md rendering service (no auth required)
- Custom CSS at `{{SKILL_DIR}}/styles/custom.css`
