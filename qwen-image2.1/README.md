# Qwen-Image-2.1-Turbo "去审查版" ComfyUI 工作流

8G 显存本地可跑的 Qwen-Image-2.1-Turbo 工作流（ComfyUI API 格式 JSON，拖入画布即用）。

**实测**：RTX 5060（8GB 显存）真机跑通 —— 首次出图约 2 分钟（含模型初始化加载），之后每次出图 1 分钟内。

> 📖 完整安装教程（含 4 个模型文件的下载地址、存放位置、5 个真机排坑实录）请看：
> **https://x.com/hiwise_aigc/status/2109255757845914013**

## 本仓库内容

| 文件 | 说明 |
| --- | --- |
| `Qwen-Image-2.1-Turbo去审查版_ComfyUI工作流.json` | 文生图工作流（11 个节点）：euler + Turbo 官方 9 步 sigmas 调度，CFG=1.0，CR Aspect Ratio 控制图片比例 |
| `Qwen-Image-2.1-Turbo去审查版_ComfyUI图生图工作流.json` | 图生图/编辑工作流（12 个节点）：`TextEncodeQwenImage21` 原生编辑路径，上传参考图 + 自然语言指令即可改图 |

## 快速上手

1. 按置顶推文把 4 个模型文件下载放到对应目录（`diffusion_models/`、`text_encoders/`、`vae/`）
2. 装好前置插件与补丁（见下）
3. 把 JSON 拖入 ComfyUI 画布，点 Queue

## 前置条件

- ComfyUI 更新到最新版（Qwen-Image-2.1 原生支持需 0.37+）
- Manager 安装：`ComfyUI-GGUF`、`ComfyUI-KJNodes`、`Comfyroll Studio`（搜 Comfyroll，认准描述里带 "Aspect Ratio" 的卡）
- 社区补丁（修 Qwen3-VL 编码器被误认的报错）：在 `custom_nodes` 目录执行
  `git clone https://github.com/pottokao-dotcom/comfyui-gguf-qwen3vl-te`
  完全重启 ComfyUI，控制台看到 `[GGUF-Qwen3VL-TE] ComfyUI-GGUF patched` 即生效

## 模型文件（4 个）

| 文件 | 大小 | 存放位置 |
| --- | --- | --- |
| `Qwen-Image-2.1-Turbo-AD-Q4_K.gguf`（去噪器） | 4.20GB | `models/diffusion_models/` |
| `Qwen-Image-2.1-Turbo-Abliterated-Uncensored-Q8_0.gguf`（编码器） | 8.71GB | `models/text_encoders/` |
| `mmproj-Qwen-Image-2.1-Turbo-Abliterated-Uncensored-F16.gguf`（视觉投影器） | 1.16GB | `models/text_encoders/`（与编码器同目录，勿改名） |
| `qwen_image_2.1_vae_bf16.safetensors`（VAE） | 644MB | `models/vae/` |

下载直链见置顶推文。注意：文件名带 `Abliterated-Uncensored` 的是**编码器**（进 CLIPLoader），不带的是**去噪器**（进 Unet Loader），拿混直接报错。

## 实测排坑

1. **CustomSigmas 报"无法转换为 INT"** —— 本仓库 JSON 已修复，直接用
2. **Unet Loader 报 `Unexpected architecture type: 'qwen3vl'`** —— 去噪器/编码器拿混了，看上表区分
3. **采样时维度爆炸 `[4096]` vs `[1, 512, 12288]`** —— 装上面的社区补丁
4. **补丁装完报 `Missing vision tower`** —— 下载 mmproj 放到 `text_encoders`，勿改名
5. **CR Aspect Ratio 连线被吞** —— 本仓库 JSON 已改成 `empty_latent` 直连采样器，无需 EmptyLatentImage

## 关于"去审查"

先说清楚：官方实测 45 个敏感提示词，换这个编码器前后成图 **0 翻转** —— 它改的是编码器作为语言模型时的拒绝倾向（有害请求拒绝率 88.9% → 1.2%），不是图像管线的安全过滤器（原生管线本来就没有）。本地跑，没人管你画什么，仅此而已。

本地跑不代表无法无天。官方模型卡明确写了：别做未成年人相关内容，别做真人非自愿的亲密 / 贬损图像。技术无罪，用法有界。

## 许可

Qwen-Image-2.1 官方权重使用 **Qwen Research License**（非 Apache 2.0），个人学习实测没问题，商用前请阅读官方 LICENSE。

---

欢迎关注，一起玩本地 AI 画图 🤝
