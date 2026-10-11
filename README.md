# WordSteps Voices

词阶 WordSteps 的**离线语音模型**集合 —— 用于 App 内置的 sherpa-onnx 离线神经 TTS。

配套主仓库：**https://github.com/dove-cyber/wordsteps**

---

## 一、可用音色

| 音色 | 语言 | 采样率 | 大小 | 说明 |
| --- | --- | --- | --- | --- |
| [`anon-medium`](anon-medium/) | 英语 en_US | 22050Hz | 60.6MB | 单说话人，由角色配音素材克隆训练 |

---

## 二、怎么用

### 方式 A：Piper 直接使用

```bash
# 需要 piper-tts
pip install piper-tts
echo "Hello world" | piper -m anon-medium/anon.onnx -f out.wav
```
> Piper 要求配置文件名为 `<模型名>.json`，本仓库已按 `anon.onnx` + `anon.onnx.json` 配对。

### 方式 B：sherpa-onnx 使用

```python
import sherpa_onnx
tts = sherpa_onnx.OfflineTts(sherpa_onnx.OfflineTtsConfig(
    model=sherpa_onnx.OfflineTtsModelConfig(
        vits=sherpa_onnx.OfflineTtsVitsModelConfig(
            model="anon-medium/anon.onnx",
            tokens="anon-medium/tokens.txt",
            data_dir="espeak-ng-data",   # 需自备 espeak-ng 数据
        )
    )
))
audio = tts.generate("Hello world", sid=0, speed=1.0)
```

### 方式 C：装进词阶 WordSteps 的 WebView（WASM）

主仓库的 App 用的是 sherpa-onnx **WASM 运行时**，模型需要重打包进 `.data`：

```bash
node scripts/repack-sherpa-model.mjs \
  web/js/vendor/sherpa/sherpa-onnx-wasm-main-tts.js \
  web/js/vendor/sherpa/sherpa-onnx-wasm-main-tts.data \
  anon-medium/anon.onnx \
  out-package \
  --tokens anon-medium/tokens.txt
```

⚠️ **两个必须注意的点**（踩过的坑）：
1. **`tokens.txt` 必须与该模型配套**，不能沿用别的模型的。音素→ID 映射不一致会让某些音素查不到 ID 被丢弃，输出变成电流噪声。
2. sherpa 的 `tokens.txt` 解析器**只接受单个 Unicode 字符**的音素；像 `aɪ`/`aʊ`/`eɪ`/`oʊ`/`ɔɪ` 这类合并双元音必须剔除（espeak 音素化默认本来就是拆开的，所以剔除无影响）。

---

## 三、模型详情

### anon-medium

| 项 | 值 |
| --- | --- |
| 架构 | VITS (Piper medium) |
| 采样率 | 22050 Hz |
| 说话人 | 1 |
| 音素数 | 256（实际用到 161 个单字符音素）|
| 训练方式 | 从 `en_US-lessac-medium` 官方检查点（1,355,540 步）**微调 600 轮** |
| 最佳 val_mel | 0.3474 |
| 训练数据 | 539 句英语合成音频（23.5 分钟），由 GPT-SoVITS 以目标音色零样本克隆生成 |
| 训练硬件 | RTX 5060 Ti 16G |

详见 [`anon-medium/MODEL_CARD.md`](anon-medium/MODEL_CARD.md)。

---

## 四、⚠️ 来源与免责声明（重要，请务必阅读）

本仓库中的模型权重是**衍生作品**，其来源存在明确的许可与权利限制：

1. **上游基座**：微调起点是 Piper 官方 `en_US-lessac-medium` 检查点，其训练数据为 **Blizzard 2013 Lessac 数据集**。该数据集的许可为 **Research Licence Agreement**，明确声明音频权利与版权归 Voice Factory International Inc. 与 Lessac Technologies Inc. 所有，且**仅授权给具名个人/机构、仅限研究用途、不可转让**。
2. **音色来源**：`anon-medium` 的音色由**受版权保护的动画角色配音素材**克隆而来，可能涉及配音演员的声音权益及相关版权方的权益。

**因此：**

- 本仓库对模型权重**不作任何权利主张**，也**不授予任何许可**。
- 使用者需**自行评估并承担**因使用/分发这些权重而产生的全部法律风险。
- 请勿用于任何商业用途。
- 若你是相关权利方并认为本仓库内容侵犯了你的权益，请提 Issue，我会立即移除。

仓库中的**脚本与文档**（如有）由 **鸽浅.strandove** 以 MIT 许可提供。

---

## 五、作者

**鸽浅.strandove** · MIT License © 2026（仅适用于脚本与文档，不适用于模型权重）
