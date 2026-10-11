# Model Card — 千早爱音 英文声库 (anon-medium)

## 基本信息

| 项 | 值 |
| --- | --- |
| 名称 | anon-medium（MyGO 千早爱音 英文声库）|
| 角色 | 千早爱音 / Chihaya Anon（《BanG Dream! It's MyGO!!!!!》）|
| 语言 | 英语 en_US |
| 架构 | VITS（Piper medium 配置）|
| 采样率 | 22050 Hz |
| 说话人数 | 1 |
| 音素数 | 256（`phoneme_id_map` 共 166 项，实际使用 161 个单字符音素）|
| 文件 | `anon.onnx`(60.6MB) + `anon.onnx.json` + `tokens.txt` |

## 训练流程

```
角色配音素材 (45.5 分钟, m4s/AAC)
   ↓ PyAV 解码 + 自动挑选参考片段(语音连续、电平稳定、无削波)
5 个 6 秒参考音频
   ↓ GPT-SoVITS 零样本克隆 (ref_free 免参考文本模式)
539 句英语训练集 (23.5 分钟, 22050Hz 单声道)
   ↓ Piper VITS 微调 (起点: 官方 en_US-lessac-medium, 1,355,540 步)
600 轮微调 → epoch=2740-val_mel=0.3474
   ↓ 导出 ONNX + 补 sherpa 元数据(sample_rate / n_speakers / has_espeak 等)
anon.onnx
```

## 训练参数

| 参数 | 值 |
| --- | --- |
| 微调起点 | `en_US-lessac-medium/epoch=2164-step=1355540.ckpt` |
| 训练轮数 | 2164 → 2764（600 轮）|
| batch size | 32 |
| 学习率 | 继承检查点（生成器 2e-4 / 判别器 1e-4）|
| 最佳 val_mel | **0.3474** |
| 硬件 | RTX 5060 Ti 16G（sm_120 / CUDA 12.8）|
| 耗时 | 约 2.5 小时 |

## 质量自检（客观指标）

| 音频 | 谱平坦度（越低越干净）| 高频(>3kHz)占比 |
| --- | --- | --- |
| 训练集（干净基准）| 0.0053 | 0.0163 |
| **本模型** | **0.0048** | **0.0146** |

## 复现经验（重要，能省很多时间）

1. **训练步数是决定性的。**
   官方 lessac 训练了 **135 万步**。仅训约 2.7 万步的模型，`val_mel` 看着正常（0.44），但输出有**明显宽带噪声**——VITS 的 mel 损失很早就饱和，**噪声靠对抗(GAN)训练慢慢磨掉**。
   → **从官方检查点微调**是性价比最高的做法；从零训练需要数万轮。

2. **学习率调度别设太短。**
   `max_epochs` 会让学习率在全程衰减到 5%（`lr_final_ratio`）。若设成 300，后期几乎停止学习。

3. **`tokens.txt` 必须与模型配套**，且 sherpa 只接受单字符音素（合并双元音需剔除）。

4. **ONNX 需补元数据**（`sample_rate` / `n_speakers` / `has_espeak` 等），否则 sherpa 报
   `'sample_rate' does not exist in the metadata`。

5. **检查点跨平台**：Linux 上训练的 `.ckpt` 含 `pathlib.PosixPath`，在 Windows 上需转换
   （PyTorch 2.6+ 还默认 `weights_only=True`）。

6. **诊断用频谱图，别只看指标**。`val_mel` 正常但听感有问题时，画频谱图能一眼看出宽带噪声。

## ⚠️ 许可与来源

本模型是衍生作品：

- 微调起点 `en_US-lessac-medium` 的训练数据为 **Blizzard 2013 Lessac 数据集**，
  许可为 Research Licence Agreement（音频版权归 Voice Factory International Inc. /
  Lessac Technologies Inc.，仅限具名者研究用途、不可转让）。
- 音色由**受版权保护的动画角色配音素材**克隆而来。

本模型**不作任何权利主张、不授予任何许可**，**禁止商用**；使用者自行承担全部法律风险。
相关权利方若认为侵权请联系移除。
