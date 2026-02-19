# OpenClaw-Antigravity-Gemini3ProImage-
### 如何让 OpenClaw 连接 Antigravity Tools 实现无限免费绘图？：ClawHub 新插件：一键调用 Gemini 3 Pro Image 生成高质量图片

  

最近我在折腾 OpenClaw 智能体时，发现如果想让它画图（Image Generation），通常需要由 DALL-E 3 或 Midjourney 这种付费服务支持。

但我手里已经部署了强大的 Antigravity Tools（利用 Google Gemini 免费额度转换的 API），能不能把这两者结合起来，让我的 OpenClaw 也能免费、无限地画图呢？

经过一番折腾，我写了一个 OpenClaw 插件 —— antigravity-commander。今天分享给大家！

### ✨ 插件亮点

1.  完美兼容 Antigravity Tools：专为 gemini-3-pro-image 模型优化。
2.  支持 Base64 图片：即使接口返回的是 Base64 编码（而非 URL），插件也会自动解码并保存为本地图片，直接发给你。
3.  伪装 Chat 模式：适配了 Antigravity 将绘图伪装成 Chat 接口的特殊协议。
4.  指令触发：在聊天中发送 谷歌画图：一只猫 即可触发，不影响主模型的对话逻辑。

### 🛠️ 准备工作

在开始之前，你需要准备：

1.  一个运行中的 OpenClaw 实例。
2.  一个可用的 Antigravity Tools 服务地址（或者 NewAPI/OneAPI 地址）。
3.  对应的 API Key。

### 📥 安装与配置

#### 第一步：获取插件代码

在你的 OpenClaw 工作区 (workspace/skills) 下创建一个新目录，例如 antigravity-commander，然后创建一个 SKILL.md 文件。

(此处你可以贴出我刚才生成的 antigravity-commander-public/SKILL.md 的完整代码)

#### 第二步：填入你的配置

打开 SKILL.md，找到文件顶部的 USER\_CONFIG 区域，填入你的服务信息：

 # ================= USER\_CONFIG (用户配置区) =================  
\# 请在此处填入你的服务地址和 Key  
\# Please fill in your service URL and Key here  
USER\_CONFIG = {  
    "url": "",  # e.g., "https://gemini.onw.cc/v1/chat/completions"  
    "key": "",  # e.g., "sk-xxxxxxxxxxxxxxxx"  
    "model": "gemini-3-pro-image",  
    "default\_size": "1024x1024"  
}  
\# ===========================================================

#### 第三步：配置 AGENTS.md (可选但推荐)

为了让 AI 更智能地调用这个工具，建议在你的 AGENTS.md 文件中加入以下规则：

****Special Commands****

-   `谷歌画图` or `画图`: When a user message starts with these commands, ****DO NOT**** call the tool directly. Instead:
-   1.  Reply to user: "正在后台为您绘图，请稍候..."
    2.  Call `sessions_spawn` with `task="使用 antigravity-commander 插件绘制：[PROMPT]"`.

这样做的好处是：绘图通常需要 1-2 分钟，使用 sessions\_spawn 可以让绘图在后台运行，不会卡住你的主聊天窗口！

#### 第四步：重启 OpenClaw

systemctl --user restart openclaw-gateway.service

#### 🎨 开始画图！

配置完成后，直接在 Telegram 或网页端发送指令：

谷歌画图：一只在赛博朋克城市里喝咖啡的猫

或者：

画图：未来的火星基地 (16:9)

稍等片刻（取决于你的 Antigravity 服务速度，通常 60 秒左右），一张高清大图就会自动推送到你的手机上！

### ❓ 常见问题

Q: 图片生成了，但是显示乱码？  
A: 插件已经内置了 Base64 解码功能。如果还乱码，请检查你的 API 是否返回了标准的 JSON 格式。

Q: 一直提示超时？  
A: Antigravity 服务有时候响应很慢。插件默认设置了 180秒 超时，如果还超时，请检查你的服务是否存活。

Q: 可以用 NewAPI 吗？  
A: 可以！只要 NewAPI 能连通 Antigravity，填入 NewAPI 的地址和 Key 即可。

  

希望这个插件能帮到大家，让 AI 绘图变得更简单、更自由！