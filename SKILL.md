---
name: antigravity-commander
description: Call Antigravity Tools (or any OpenAI-compatible image model) to generate images. Support Base64 decoding and local saving.
author: YourName
version: 1.2.0
---

# Antigravity Commander

通过指令 `谷歌画图` 或 `画图` 调用外部绘图服务。支持 Antigravity Tools、OneAPI、NewAPI 等 OpenAI 兼容接口。

## Setup (如何配置)

你有两种方式配置此插件：

1.  **直接修改文件 (推荐)**：在下方的 `USER_CONFIG` 区域填入你的 API URL 和 Key。
2.  **环境变量**: 在系统环境中设置 `ANTIGRAVITY_API_URL` 和 `ANTIGRAVITY_API_KEY`。

## Tools

### `generate_image_command`

Execute the image generation command.

- **prompt** (string, required): The image description.
- **size_override** (string, optional): Size override (e.g., "16:9").

```python
import requests
import json
import base64
import os
import time

# ================= USER_CONFIG (用户配置区) =================
# 请在此处填入你的服务地址和 Key
# Please fill in your service URL and Key here
USER_CONFIG = {
    "url": "",  # e.g., "https://域名.com/v1/chat/completions"
    "key": "",  # e.g., "sk-xxxxxxxxxxxxxxxx"
    "model": "gemini-3-pro-image",
    "default_size": "1024x1024"
}
# ===========================================================

def get_config():
    """优先读取环境变量，否则使用 USER_CONFIG"""
    return {
        "url": os.getenv("ANTIGRAVITY_API_URL", USER_CONFIG["url"]),
        "key": os.getenv("ANTIGRAVITY_API_KEY", USER_CONFIG["key"]),
        "model": os.getenv("ANTIGRAVITY_MODEL", USER_CONFIG["model"]),
        "default_size": USER_CONFIG["default_size"]
    }

def generate_image_command(prompt, size_override=None):
    cfg = get_config()
    
    # 检查配置是否已填写
    if not cfg["url"] or not cfg["key"]:
        return "❌ 插件未配置！请打开 `SKILL.md` 填写 `USER_CONFIG` 中的 URL 和 Key，或者设置环境变量 `ANTIGRAVITY_API_URL` / `ANTIGRAVITY_API_KEY`。"

    # 尺寸映射逻辑
    target_size = cfg["default_size"]
    if size_override:
        if "16:9" in size_override: target_size = "1280x720"
        elif "9:16" in size_override: target_size = "720x1280"
        elif "4:3" in size_override: target_size = "1216x896"
        elif "1:1" in size_override: target_size = "1024x1024"

    # 清洗 Prompt
    clean_prompt = prompt
    for prefix in ["谷歌画图：", "谷歌画图:", "谷歌画图", "画图：", "画图:", "画图"]:
        clean_prompt = clean_prompt.replace(prefix, "")
    
    clean_prompt = clean_prompt.strip()
    if not clean_prompt:
        return "❌ 请提供绘图描述，例如: 谷歌画图 一只猫"

    print(f"🎨 Drawing: '{clean_prompt}' (Size: {target_size})")

    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {cfg['key']}"
    }
    
    # 构造请求 (伪装成 Chat)
    payload = {
        "model": cfg["model"],
        "messages": [
            {"role": "user", "content": clean_prompt}
        ],
        "extra_body": {
            "size": target_size
        }
    }

    try:
        # 设置 180s 超时以适应慢速服务
        response = requests.post(cfg["url"], headers=headers, json=payload, timeout=180)
        
        if response.status_code != 200:
            return f"❌ 绘图失败 (HTTP {response.status_code}): {response.text}"
            
        result = response.json()
        
        if "choices" in result and len(result["choices"]) > 0:
            content = result["choices"][0]["message"]["content"]
            
            # 情况1: URL
            if content.startswith("http") or "![" in content:
                return f"🎨 **绘图完成**\n\n{content}"
            
            # 情况2: Base64 自动保存
            try:
                img_data = content
                if "base64," in content:
                    img_data = content.split("base64,")[1]
                elif "](" in content: # Markdown 包装
                    start = content.find("](") + 2
                    end = content.find(")", start)
                    url_part = content[start:end]
                    if "base64," in url_part:
                        img_data = url_part.split("base64,")[1]
                
                img_bytes = base64.b64decode(img_data)
                
                # 保存路径
                save_dir = "/home/openclaw/.openclaw/workspace/media/generated"
                os.makedirs(save_dir, exist_ok=True)
                filename = f"img_{int(time.time())}.png"
                file_path = os.path.join(save_dir, filename)
                
                with open(file_path, "wb") as f:
                    f.write(img_bytes)
                
                return f"🎨 **绘图完成**\n\n![Generated Image]({file_path})"
                
            except Exception:
                # 兜底：如果不是图片，直接返回文本
                return f"⚠️ 接口返回非标准内容:\n{content[:200]}..."

        else:
            return f"❌ 接口返回异常: {json.dumps(result)}"

    except Exception as e:
        return f"❌ 执行出错: {str(e)}"
```
