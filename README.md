# 泽龙AI 中转站 API 完整手册

> 一站式 AI 模型调用服务，支持文本对话、图片生成、图片编辑、视频生成。

## 🔑 配置信息

| 配置项 | 值 |
|--------|-----|
| **Base URL** | `https://api.zelong.vip/v1` |
| **API Key** | `sk-57f98031c2c2b0da97f8c9930d73d6d552ec578ca849035b` |

---

## 📋 当前可用模型（14个）

| 模型 ID | 类型 | 说明 |
|------|------|------|
| `gpt-5.5` | 文本 | GPT 最新 | — |
| `gpt-5.4` | 文本 | GPT 主力 | ✅ 稳定 |
| `gpt-5.4-high` | 文本 | GPT 高精度 | — |
| `gpt-5.4-mini` | 文本 | GPT 轻量快速 | ✅ 稳定 |
| `claude-opus-4-7` | 文本 | Claude 最强 | ✅ 稳定 |
| `claude-opus-4-6` | 文本 | Claude | ✅ 稳定 |
| `claude-sonnet-4-6` | 文本 | Claude 均衡 | ✅ 稳定 |
| `claude-sonnet-4-6-thinking` | 文本 | Claude 深度思考 | ❌ 不可用 |
| `claude-sonnet-4-5` | 文本 | Claude | ❌ 不可用 |
| `claude-sonnet-4-5-thinking` | 文本 | Claude 思考 | ❌ 不可用 |
| `claude-haiku-4-5` | 文本 | Claude 极速 | ✅ 稳定 |
| `claude-haiku-4-5-thinking` | 文本 | Claude 极速思考 | ❌ 不可用 |
| `gpt-image-2` | 🖼️ 图片 | 文生图 + 图片编辑 | ✅ 稳定 |
| `grok-video` | 🎬 视频 | 文生视频 + 图生视频 | ✅ 稳定 |

> ✅ = 实测通过 · ❌ = 不可用 · — = 未测试

---

## 快速开始

```bash
# 测试连通性
curl https://api.zelong.vip/v1/models \
  -H "Authorization: Bearer sk-your-api-key"
```

**认证方式**：所有请求需携带 `Authorization: Bearer sk-xxx`

---

## 一、文本对话（Chat Completions）

### 端点

```
POST /v1/chat/completions
```

### 格式

`application/json`（支持流式 SSE）

### 调用示例

**cURL**：
```bash
curl https://api.zelong.vip/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-your-api-key" \
  -d '{
    "model": "gpt-5.4",
    "messages": [{"role": "user", "content": "你好"}],
    "max_tokens": 100
  }'
```

**Python (OpenAI SDK)**：
```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-your-api-key",
    base_url="https://api.zelong.vip/v1",
)

response = client.chat.completions.create(
    model="gpt-5.4",
    messages=[{"role": "user", "content": "你好!"}],
)
print(response.choices[0].message.content)
```

**Python (Anthropic SDK)**：
```python
from anthropic import Anthropic

client = Anthropic(
    api_key="sk-your-api-key",
    base_url="https://api.zelong.vip",
)

msg = client.messages.create(
    model="claude-haiku-4-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "你好"}],
)
print(msg.content[0].text)
```

**流式输出**：
```python
response = client.chat.completions.create(
    model="gpt-5.4",
    messages=[{"role": "user", "content": "讲个故事"}],
    stream=True,
)
for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### 可用文本模型

| 模型 | 类型 |
|------|------|
| `gpt-5.5` | GPT 最新 |
| `gpt-5.4` | GPT |
| `gpt-5.4-high` | GPT 高精度 |
| `gpt-5.4-mini` | GPT 轻量 |
| `claude-opus-4-7` | Claude 最强 |
| `claude-opus-4-6` | Claude |
| `claude-sonnet-4-6` | Claude 均衡 |
| `claude-sonnet-4-6-thinking` | Claude 思考 |
| `claude-sonnet-4-5` | Claude |
| `claude-sonnet-4-5-thinking` | Claude 思考 |
| `claude-haiku-4-5` | Claude 快速 |
| `claude-haiku-4-5-thinking` | Claude 快速思考 |

---

## 二、文生图（Image Generation）

### 端点

```
POST /v1/image/created    （推荐）
POST /v1/images/generations
```

### 格式

`application/json`

### 调用示例

**cURL**：
```bash
curl --noproxy "*" https://api.zelong.vip/v1/image/created \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-your-api-key" \
  -d '{"model":"gpt-image-2","prompt":"一只可爱的橘猫坐在窗台上看夕阳，日系动漫风格"}'
```

**Python**：
```python
import requests, base64

r = requests.post(
    "https://api.zelong.vip/v1/image/created",
    headers={"Authorization": "Bearer sk-your-api-key"},
    json={"model": "gpt-image-2", "prompt": "一只猫"},
    timeout=180,
)
b64 = r.json()["data"][0]["b64_json"]
with open("output.png", "wb") as f:
    f.write(base64.b64decode(b64))
print("✅ 图片已保存")
```

### 必填参数

| 参数 | 说明 |
|------|------|
| `model` | `gpt-image-2` |
| `prompt` | 图片描述文本 |

### 可选参数

| 参数 | 说明 |
|------|------|
| `image_size` | `512` / `1K` / `2K` / `4K` |
| `aspect_ratio` | `1:1` / `16:9` / `9:16` |
| `n` | 生成数量，1-4，默认 1 |
| `response_format` | 仅支持 `b64_json`（默认） |

### ⚠️ 注意

- **不要传** `size`、`quality`、`style` 等 OpenAI 标准参数，会导致报错
- 返回永远是 base64 编码，需自行解码保存为 PNG
- 费用：1.0 图片积分/张（1K 分辨率）

### 返回格式

```json
{
  "created": 1710000000,
  "model": "gpt-image-2",
  "data": [
    {
      "b64_json": "iVBORw0KGgoAAAANSUhEUgAA...",
      "mime_type": "image/png"
    }
  ],
  "usage": {
    "image_count": 1,
    "image_size": "1K",
    "billing_type": "image_credit",
    "image_credits_charged": 1.0
  }
}
```

---

## 三、图片编辑（Image Edit）

### 端点

```
POST /v1/image/edit
POST /v1/images/edits
```

### 格式

**`multipart/form-data`**（不是 JSON！）

### 调用示例

**cURL**：
```bash
curl --noproxy "*" https://api.zelong.vip/v1/image/edit \
  -H "Authorization: Bearer sk-your-api-key" \
  -F "model=gpt-image-2" \
  -F "prompt=给这只猫戴上墨镜" \
  -F "image=@your_image.png"
```

**Python**：
```python
import requests, base64

with open("original.png", "rb") as f:
    r = requests.post(
        "https://api.zelong.vip/v1/image/edit",
        headers={"Authorization": "Bearer sk-your-api-key"},
        files=[("image", ("original.png", f, "image/png"))],
        data={"model": "gpt-image-2", "prompt": "给这只猫戴上墨镜"},
        timeout=120,
    )
b64 = r.json()["data"][0]["b64_json"]
with open("edited.png", "wb") as f:
    f.write(base64.b64decode(b64))
```

### 必填参数

| 参数 | 说明 |
|------|------|
| `model` | `gpt-image-2` |
| `prompt` | 编辑指令文本 |
| `image` | 原图文件（multipart 上传），可重复传多个 |

### ⚠️ 注意

- **`image` 必须是文件上传**，不能用 JSON body 里的 base64 或 URL
- 支持上传多张 `image` 进行组合编辑
- 费用：0.5 图片积分/次

---

## 四、视频生成（Video Generation）

### 端点

```
POST /v1/created/video    （推荐：创建任务并轮询至完成，返回 content_url）
POST /v1/videos           （异步：先创建任务，再查询和下载）
```

### 格式

**`multipart/form-data`**

### 模型

`grok-video`

### 调用示例

**文生视频（cURL）**：
```bash
curl --noproxy "*" https://api.zelong.vip/v1/created/video \
  -H "Authorization: Bearer sk-your-api-key" \
  -F "model=grok-video" \
  -F "prompt=霓虹雨夜街头，电影感慢镜头追拍" \
  -F "seconds=10" \
  -F "resolution_name=720p"
```

**图生视频（cURL）**：
```bash
curl --noproxy "*" https://api.zelong.vip/v1/created/video \
  -H "Authorization: Bearer sk-your-api-key" \
  -F "model=grok-video" \
  -F "prompt=让参考图中的主体自然运动，电影感镜头推进" \
  -F "seconds=10" \
  -F "resolution_name=720p" \
  -F "input_reference[]=@reference.png"
```

**Python（文生视频）**：
```python
import requests

r = requests.post(
    "https://api.zelong.vip/v1/created/video",
    headers={"Authorization": "Bearer sk-your-api-key"},
    data={
        "model": "grok-video",
        "prompt": "一只猫在海边散步，夕阳暖色调",
        "seconds": "6",
        "resolution_name": "720p",
    },
    timeout=300,
)
result = r.json()
# 下载视频
content_url = result["content_url"]
video = requests.get(
    f"https://api.zelong.vip{content_url}",
    headers={"Authorization": "Bearer sk-your-api-key"},
)
with open("output.mp4", "wb") as f:
    f.write(video.content)
```

**Python（图生视频）**：
```python
import requests

r = requests.post(
    "https://api.zelong.vip/v1/created/video",
    headers={"Authorization": "Bearer sk-your-api-key"},
    data={
        "model": "grok-video",
        "prompt": "让参考图中的主体自然运动，电影感",
        "seconds": "6",
        "resolution_name": "720p",
    },
    files=[("input_reference[]", open("reference.png", "rb"))],
    timeout=300,
)
result = r.json()
print(f"视频: {result['content_url']}")
```

**异步模式（先创建后查询）**：
```bash
# 1. 创建任务
curl --noproxy "*" https://api.zelong.vip/v1/videos \
  -H "Authorization: Bearer sk-your-api-key" \
  -F "model=grok-video" \
  -F "prompt=未来城市航拍" \
  -F "seconds=10"

# 返回 {"id": "video_xxx", "status": "processing", ...}

# 2. 查询任务状态
curl https://api.zelong.vip/v1/videos/video_xxx \
  -H "Authorization: Bearer sk-your-api-key"

# 3. 下载视频
curl -L https://api.zelong.vip/v1/videos/video_xxx/content \
  -H "Authorization: Bearer sk-your-api-key" \
  -o result.mp4
```

### 参数说明

| 参数 | 必填 | 说明 |
|------|------|------|
| `model` | 是 | `grok-video` |
| `prompt` | 是 | 视频描述文本 |
| `seconds` | 否 | 时长：6 / 10 / 12 / 16 / 20，默认 6 |
| `size` | 否 | 尺寸：720x1280 / 1280x720 / 1024x1024 / 1024x1792 / 1792x1024 |
| `resolution_name` | 否 | 清晰度：480p / 720p |
| `preset` | 否 | 预设：fun / normal / spicy / custom |
| `input_reference[]` | 否 | 图生视频参考图，可重复，最多 7 张。不传 = 文生视频 |

### 返回格式

```json
{
  "id": "video_xxx",
  "status": "completed",
  "model": "grok-imagine-video",
  "prompt": "一只猫在海边散步...",
  "seconds": "6",
  "size": "720x1280",
  "content_url": "/v1/videos/video_xxx/content",
  "usage": {
    "billing_type": "image_credit",
    "image_credits_charged": 3.0,
    "seconds": 6
  }
}
```

### ⚠️ 注意

- 视频生成耗时较长（30秒~几分钟），`/created/video` 会等待完成
- 费用：0.5 积分/秒（6秒=3积分，10秒=5积分）
- 下载视频用 `content_url`，需拼接完整 URL
- 如果本地有代理，cURL 需加 `--noproxy "*"`

---

## 五、通用规则

### 所有接口

| 规则 | 说明 |
|------|------|
| 认证头 | `Authorization: Bearer sk-xxx` 或 `X-API-Key: sk-xxx` |
| 模型查询 | `GET /v1/models` |
| 不要加的参数 | `size`(OpenAI格式)、`quality`、`style` — 全部不支持 |

### 代理问题

如果本地有 HTTP 代理，cURL 需加：
```bash
--noproxy "*"
```

Python 需禁用代理：
```python
import os, requests
os.environ.pop('HTTP_PROXY', None)
os.environ.pop('HTTPS_PROXY', None)

# 方式一（推荐）：禁用 trust_env
session = requests.Session()
session.trust_env = False
r = session.post(url, ..., timeout=180)

# 方式二：手动设空代理
r = requests.post(url, ..., proxies={"http": None, "https": None}, timeout=180)
```

### 客户端配置

**Claude Code**：
```bash
export ANTHROPIC_BASE_URL="https://api.zelong.vip"
export ANTHROPIC_API_KEY="sk-your-api-key"
```

**Cursor / Continue / Cline**：
```
API Key: sk-your-api-key
Base URL: https://api.zelong.vip/v1
```

**OpenClaw**：
```json
{
  "api": "anthropic-messages",
  "baseUrl": "https://api.zelong.vip",
  "apiKey": "sk-your-api-key"
}
```

---

## 六、错误码速查

| 错误码 | 含义 | 解决方法 |
|--------|------|---------|
| `UNAUTHORIZED` | Key 无效 | 检查 Key 是否正确/激活 |
| `INSUFFICIENT_IMAGE_CREDITS` | 图片积分不足 | 联系管理员充值 |
| `IMAGE_RESPONSE_FORMAT_NOT_SUPPORTED` | 用了 url 格式 | 只能用 b64_json |
| `INVALID_IMAGE_FILE` | 图片编辑缺文件 | image 必须 multipart 上传 |
| `VIDEO_MODEL_NOT_FOUND` | 视频模型未配置 | 确认传了 model=grok-video |
| `MODEL_NOT_FOUND` | 模型不存在 | 查 /v1/models 确认可用模型 |
| `UPSTREAM_INVALID_REQUEST` | 上游渠道参数不兼容 | 换模型或参数；Claude thinking 系列暂不可用 |
| `NO_CHANNEL` | 该参数组合无可用渠道 | 换 image_size 或不传该参数 |
| `VIDEO_GENERATION_FAILED` | 视频渠道临时异常 | 稍后重试 |

---

## 七、获取帮助

- 模型列表：`GET https://api.zelong.vip/v1/models`
- Key 申请：联系 Tang
- 余额查询：登录 https://zelong.vip 后台

---

> 🍂 最后更新：2026-06-07 · 全部端点实测通过
> 实测：小秋 · 反馈：小婷 · Agent 协作系统
