# LTX 2.3 PromptRelay Shot Board

**A visual shot board UI for multi-segment video generation with LTX 2.3 + ComfyUI PromptRelay**

> 由 [vplab 虛擬製作研習社](https://www.instagram.com/ltu_vplab/) 開發

<img width="941" height="530" alt="syu0519_ltx-promptrelay-shotboard" src="https://github.com/user-attachments/assets/f455a9e1-bfd8-401f-9826-e6bfa6739d79" />


## 什麼是這個工具？

把原本需要**手動編輯 JSON、手動計算幀數**的 LTX 2.3 多分鏡工作流，變成一個**瀏覽器直接用的視覺化介面**。

### 完整製作流程

```
① ChatGPT（腳本生成）
        ↓  貼上 [SHOT_xx] 格式腳本
② GPT Image 2.0 / FLUX（分鏡圖生成）
        ↓  拖曳圖片上傳
③ Shot Board GUI（本工具）← 你在這裡
        ↓  一鍵組合 Workflow JSON
④ ComfyUI + LTX 2.3 + PromptRelay（影片生成）
```

### 解決了什麼問題？

- ❌ 以前：手動數幀數、手動改 JSON 裡每個 `index_N`、算 `segment_lengths`
- ✅ 現在：貼腳本 → 上傳圖 → 按一個鍵

---

## 功能

- 📋 **腳本解析** — 貼入 GPT 生成的 `[SHOT_xx]` 格式腳本，自動切分分鏡
- 🖼️ **Grid 縮圖板** — 每個分鏡卡片顯示縮圖、中文描述、英文 prompt 預覽、對白
- 🖱️ **拖曳上傳圖片** — 點擊或拖放圖片到縮圖區
- ⚙️ **格率選擇** — 24 / 25 / 29.97 / 30 / 60 fps，幀數自動重算
- 🎛️ **參數面板** — 解析度、Steps、LoRA Strength、img_compression、Noise Seed
- 🖥️ **單卡 / 雙卡模式切換** — 自動切換 loader nodes 和對應解析度選單
- 📡 **直接送 ComfyUI** — 自動上傳圖片 + 送出 workflow，一鍵完成
- 💾 **下載 JSON** — 匯出可直接丟進 ComfyUI 的 workflow 檔案

---

## 快速開始

### 1. 下載工具

直接下載 [`zibai_pipeline_v5.html`](zibai_pipeline_v5.html)，**不需要安裝任何東西**，瀏覽器直接打開。

### 2. 安裝 ComfyUI 必要節點

```bash
cd ComfyUI/custom_nodes

# ComfyUI PromptRelay（必要）
git clone https://github.com/kijai/ComfyUI-PromptRelay

# ComfyUI-KJNodes（必要）
git clone https://github.com/kijai/ComfyUI-KJNodes

# ComfyUI-VideoHelperSuite（影片輸出）
git clone https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite
```

### 3. 需要的模型

放到 `ComfyUI/models/` 對應資料夾：

| 檔案 | 位置 |
|------|------|
| `ltx-2.3-22b-dev-fp8.safetensors` | `checkpoints/` |
| `ltx-2.3-22b-distilled-lora-384.safetensors` | `loras/` |
| `gemma_3_12B_it_fp4_mixed.safetensors` | `text_encoders/` |

### 4. 啟動 ComfyUI（需要開 CORS）

```bash
python main.py --enable-cors-header
```

---

## 腳本格式（GPT System Prompt）

把以下 System Prompt 設定進你的 ChatGPT 助理，之後說「**使用LTXV3的格式**」就會輸出可以直接貼進工具的腳本：

```
收到用戶提到「使用LTXV3的格式」時，
請將故事概念直接輸出以下格式，不要輸出任何其他說明。

茲白固定視覺（每個 prompt 必須包含）：
- silver-white high ponytail, golden-blonde center streak in bangs
- jade-teal geometric earrings, golden-amber eyes
- black techwear-lite outfit, white chunky platform boots
- cinematic photorealism, shallow depth of field

格式規則：
- [SHOT_xx] 編號從 01 開始，補零
- duration 單位秒，1.5~5.0 之間
- 總時長 15~40 秒，分鏡 7~14 個
- prompt 必須英文，包含：景別、運鏡、角色動作、光線
- 有對白寫入 prompt：speaking in Mandarin Chinese, she says: "..."
- dialogue 欄填中文對白，無對白填（無）
- 每個 shot 之間空一行
- 不輸出標題、說明、表格，只輸出分鏡區塊

輸出格式：

[SHOT_01] duration:2.0
desc: 中文場景描述（20字內）
prompt: English LTX 2.3 cinematic prompt here
dialogue: （無）

[SHOT_02] duration:3.0
desc: ...
prompt: ...
dialogue: 中文對白
```

---

## 關鍵參數說明

### img_compression（最影響動態）

| 數值 | 效果 |
|------|------|
| 5–10 | 錨定太強 → 人物幾乎不動、畫面凍結 |
| **25–35** | **甜蜜點** → 外觀保持 + 動態自然 |
| 40+ | 錨定太弱 → 人物外觀飄移 |

預設 **30**，從參考 workflow 驗證的設定。

### Steps（Distilled LoRA 專用）

`ltx-2.3-22b-distilled-lora-384` 是 consistency distillation 訓練：

| Steps | 效果 |
|-------|------|
| 4 | 細節不足 |
| **8** | **最佳**（預設） |
| 12+ | 過採樣 → 動作僵硬、對白消失 |

### LTX 幀數規則

LTX 2.3 的幀數必須符合 `floor(秒數 × fps / 8) × 8 + 1`，工具已自動套用，不需要手動計算。

---

## 已知問題與解法

### KJNodes preview_rate crash（NoneType 錯誤）

**症狀：**
```
AttributeError: 'NoneType' object has no attribute 'encode'
File: comfyui-kjnodes/nodes/ltxv_nodes.py line 654
```

**原因：** KJNodes 的 race condition bug，preview callback 在 server 尚未記錄 node ID 時觸發。

**解法：** 在 Windows 建立以下 Python 腳本並執行：

```python
# 存成 fix_kjnodes.py，用記事本建立
f = r'你的ComfyUI路徑\custom_nodes\comfyui-kjnodes\nodes\ltxv_nodes.py'
with open(f, 'r', encoding='utf-8') as fh:
    lines = fh.readlines()

for i, line in enumerate(lines):
    if 'if serv.last_node_id is None:' in line:
        indent = '            '
        lines[i]   = indent + 'if serv.last_node_id is None:\n'
        lines[i+1] = indent + '    return\n'
        lines[i+2] = indent + "message.write(struct.pack('16p', serv.last_node_id.encode('ascii')))\n"
        break

with open(f, 'w', encoding='utf-8') as fh:
    fh.writelines(lines)
print('Done')
```

```powershell
python fix_kjnodes.py
```

> ⚠️ 不要用 PowerShell 的 pipe（`|`）直接寫入 .py 檔案，會清空檔案。請用記事本建立再執行。

### Text Encoder 建議設定

`LTXAVTextEncoderLoader` 的 `device` 必須設為 `"cpu"`，設成 `"default"` 會與 preview callback 產生 VRAM 衝突。本工具已自動套用此設定。

---

## Workflow 檔案說明

| 檔案 | 說明 |
|------|------|
| `prompt_relay_ltx23_test_02.json` | 單卡參考 workflow（640×640，測試用） |
| `zibai_14shots_mgpu_1280.json` | 雙卡 1280×720，14 分鏡，Multi-GPU |

Multi-GPU 版使用 `LTXV2CheckpointLoaderMultiGPU` 和 `LTXV2AudioVAELoaderMultiGPU`，需要雙卡環境（如 dual RTX 3090）。

---

## 關於茲白（Zibai）

茲白是 vplab 的原創虛擬角色：

- 銀白短髮，中間一縷金色挑染
- 玉色幾何耳環、琥珀金色眼睛  
- 黑色 Techwear 裝束、白色厚底靴

Global Prompt 欄位可以換成任何角色描述，**工具本身完全通用**。

---

## 相關資源

- [ComfyUI-PromptRelay](https://github.com/kijai/ComfyUI-PromptRelay) — kijai
- [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes) — kijai
- [LTX-Video](https://github.com/Lightricks/LTX-Video) — Lightricks
- [vplab Instagram](https://www.instagram.com/ltu_vplab/)

---

*vplab 虛擬製作研習社 · 嶺東科技大學數位媒體設計系*
