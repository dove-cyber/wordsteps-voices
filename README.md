# MyGO 千早爱音 英文声库

> 适配平台与技术：**Piper / sherpa-onnx / ONNX · Android · Web(WebView) · Windows / Linux**

一个可直接使用的**离线英文语音模型**（声库），由《BanG Dream! It's MyGO!!!!!》**千早爱音**（Chihaya Anon）的配音素材克隆训练而成。

- 🔌 **完全离线**：不联网、不依赖系统语音包
- 🧩 **多平台**：Piper 原生 / sherpa-onnx（含 Android、WebAssembly）/ 任意支持 ONNX 的推理框架
- 📦 **开箱即用**：单个 60.6MB ONNX 模型 + 配套音素表

---

## 一、文件说明

```
anon-medium/
├─ anon.onnx          60.6MB   VITS 模型（ONNX，含 sherpa 所需元数据）
├─ anon.onnx.json              Piper 配置文件（Inference 参数 / 音素表）
└─ tokens.txt                  sherpa-onnx 音素→ID 映射表 ⚠️ 必须与该模型配套使用
```

---

## 二、怎么用

### 1. Piper 原生

```bash
pip install piper-tts
echo "Hello world" | piper -m anon-medium/anon.onnx -f out.wav
```

> Piper 要求配置文件名与模型同名（`anon.onnx` ↔ `anon.onnx.json`），本仓库已按此配对。

### 2. sherpa-onnx（Python / C++ / C# / Go / Java …）

```python
import sherpa_onnx

tts = sherpa_onnx.OfflineTts(sherpa_onnx.OfflineTtsConfig(
    model=sherpa_onnx.OfflineTtsModelConfig(
        vits=sherpa_onnx.OfflineTtsVitsModelConfig(
            model="anon-medium/anon.onnx",
            tokens="anon-medium/tokens.txt",
            data_dir="espeak-ng-data",     # 自备 espeak-ng 数据目录
        ),
        num_threads=2,
        provider="cpu",
    )
))

audio = tts.generate("Hello world. Welcome!", sid=0, speed=1.0)
print(audio.sample_rate, len(audio.samples))
```

`speed` 参数支持 **0.5x ~ 2.0x** 语速调节。

### 3. Android / Web(WebView) —— sherpa-onnx WASM

把模型打包进 Emscripten FS 的 `.data` 即可在 WebView 内离线运行：

```
.data 内的文件布局：
  /model.onnx        ← 本仓库的 anon.onnx 改名
  /tokens.txt        ← 本仓库的 tokens.txt
  /espeak-ng-data/…  ← espeak-ng 数据
```

⚠️ **两个极易踩的坑**（都实际遇到过，会导致「电流噪声」而不是报错）：

| 坑 | 说明 |
| --- | --- |
| **`tokens.txt` 未配套替换** | 若沿用别的模型的 tokens，音素→ID 会对不上，某些音素被丢弃 → 输出变成电流噪声 |
| **sherpa 只接受单字符音素** | sherpa 的 `ReadTokens` 要求每个音素经 UTF-8 解码后长度为 1；`aɪ`/`aʊ`/`eɪ`/`oʊ`/`ɔɪ` 这类合并双元音会被**直接拒绝并退出**。espeak 音素化默认本就把它拆成单字符，所以剔除它们无影响 |

另外，sherpa 读取 ONNX 的 `metadata_props` 获取 `sample_rate` 等，缺失会报
`'sample_rate' does not exist in the metadata`。

---

## 三、模型详情

| 项 | 值 |
| --- | --- |
| 架构 | VITS（Piper **medium** 配置）|
| 采样率 | 22050 Hz |
| 说话人 | 1（单音色）|
| 语言 | 英语 en_US |
| 音素数 | 256（`phoneme_id_map` 166 项，实际使用 161 个单字符音素）|
| 训练方式 | 从 `en_US-lessac-medium` 官方检查点（1,355,540 步）**微调** |
| 最佳 val_mel | **0.3474** |
| 训练数据 | 539 句英语合成音频（23.5 分钟），由 GPT-SoVITS 以目标音色零样本克隆生成 |
| 硬件 | RTX 5060 Ti 16G |

完整训练记录与复现经验见 [`anon-medium/MODEL_CARD.md`](anon-medium/MODEL_CARD.md)。

---

## 四、⚠️ 来源与免责声明（重要）

**本仓库的模型权重是衍生作品，附有第三方权利：**

1. **上游基座**：微调起点为 Piper 官方 `en_US-lessac-medium` 检查点，其训练数据是 **Blizzard 2013 Lessac 数据集**。该数据集采用 **Research Licence Agreement**，声明音频权利与版权归 **Voice Factory International Inc.** 与 **Lessac Technologies Inc.** 所有，且**仅授权给具名个人/机构、仅限研究用途、不可转让**。
2. **音色来源**：音色由**受版权保护的动画角色配音素材**克隆而来，可能涉及配音演员的声音权益及相关版权方的权益。

**因此：**
- 本仓库对模型权重**不作任何权利主张，也不授予任何许可**；
- 请**勿用于任何商业用途**；
- 使用者需**自行评估并承担**全部法律风险；
- 若你是相关权利方并认为本仓库内容侵权，请提交 Issue，我会**立即移除**。

完整声明见 [`NOTICE.md`](NOTICE.md)。仓库中的**脚本与文档**以 [MIT License](LICENSE) 提供。

---

## 五、作者

**鸽浅.strandove**
