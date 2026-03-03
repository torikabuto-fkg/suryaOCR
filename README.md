# PDF OCR Overlay

**スキャン PDF や画像 PDF を、テキスト検索・コピー可能な PDF に変換する** Python スクリプトです。  
[Surya OCR](https://github.com/VikParuchuri/surya) で認識したテキストを **透明レイヤー** として元 PDF に重ねるため、**見た目は一切変わりません**。

---

## ✨ 特徴

| 機能 | 説明 |
|------|------|
| **見た目そのまま** | 透明テキスト（`textRenderMode=3`）を重ねるだけなので元の PDF の外観は変わらない |
| **日本語完全対応** | Surya OCR + HeiseiKakuGo-W5 CID フォントで日本語テキストを正確に埋め込み |
| **単一ファイル / フォルダ一括** | `--input` にファイルを指定すれば 1 つ、フォルダを指定すれば中の全 PDF を一括処理 |
| **ページ範囲指定** | `--pages "1-5,10,20-30"` のように処理対象ページを絞れる |
| **GPU 高速処理** | Surya OCR は CUDA 対応。バッチサイズ調整で GPU メモリを最大活用 |
| **エラー耐性** | 個別ページでエラーが出てもスキップして残りを続行 |
| **進捗表示** | ページごとに OCR 行数と処理時間をリアルタイム表示 |
| **遅延ロード** | OCR モデルは実際に必要になるまで読み込まない（`--help` 時に GPU を消費しない） |

---

## 📋 前提条件

- **Python 3.10 以上**
- **NVIDIA GPU**（CUDA 対応）推奨 — CPU でも動作するが大幅に遅い
- **CUDA Toolkit** & **cuDNN**（GPU 使用時）

### 動作確認環境

| 項目 | 値 |
|------|-----|
| OS | Windows 10/11 |
| Python | 3.10.11 |
| GPU | NVIDIA RTX 3080 (10GB VRAM) |
| PyTorch | 2.10.0+cu128 |
| Surya OCR | 0.17.1 |

---

## 🚀 セットアップ

### 1. 仮想環境を作成（推奨）

```bash
python -m venv surya-env

# Windows (PowerShell)
.\surya-env\Scripts\Activate.ps1

# Linux / macOS
source surya-env/bin/activate
```

### 2. 依存パッケージをインストール

```bash
pip install surya-ocr pikepdf reportlab pypdfium2
```

> **注意**: `surya-ocr` は PyTorch を依存に含みます。GPU 版 PyTorch を使いたい場合は先に [公式の手順](https://pytorch.org/get-started/locally/) で CUDA 対応版をインストールしてください。

---

## ▶️ 使い方

### 単一 PDF を処理

```bash
python pdf_ocr_overlay.py --input exam.pdf --output exam_searchable.pdf
```

### フォルダ内の全 PDF を一括処理

```bash
python pdf_ocr_overlay.py --input ./pdfs/ --output ./pdfs_searchable/
```

出力ファイル名は `元の名前_searchable.pdf`（接尾辞は `--suffix` で変更可能）。

### ページ範囲を指定して処理

```bash
python pdf_ocr_overlay.py --input exam.pdf --output exam_s.pdf --pages "1-5,10,20-30"
```

### GPU バッチサイズを指定（RTX 3080 向け最適値）

```bash
python pdf_ocr_overlay.py --input exam.pdf --output exam_s.pdf \
    --rec_batch 200 --det_batch 18 --dpi 200
```

---

## ⚙️ オプション一覧

| オプション | デフォルト | 説明 |
|-----------|-----------|------|
| `--input` | **必須** | 入力パス（PDF ファイル or フォルダ） |
| `--output` | **必須** | 出力パス（PDF ファイル or フォルダ） |
| `--dpi` | `200` | OCR 用レンダリング解像度。高精度なら `300`（速度とトレードオフ） |
| `--pages` | 全ページ | 処理対象ページ（例: `"1,3-5,10"`） |
| `--suffix` | `_searchable` | フォルダ一括時の出力ファイル名接尾辞 |
| `--rec_batch` | `0`（自動） | Surya 認識バッチサイズ（RTX 3080: `200`） |
| `--det_batch` | `0`（自動） | Surya 検出バッチサイズ（RTX 3080: `18`） |

### バッチサイズの目安

| GPU | VRAM | `--rec_batch` | `--det_batch` |
|-----|------|--------------|--------------|
| RTX 3080 | 10GB | 200 | 18 |
| RTX 3090 | 24GB | 400 | 36 |
| RTX 4090 | 24GB | 512 | 48 |

> VRAM 不足で `CUDA out of memory` が出る場合は値を下げてください。

---

## 🔧 処理パイプライン

```
入力 PDF
  │
  ├─ 1. pypdfium2 でページを画像にレンダリング (指定 DPI)
  │
  ├─ 2. Surya OCR でテキスト + bounding box を認識
  │
  ├─ 3. ReportLab で透明テキスト PDF レイヤーを生成
  │     ├─ 座標変換: 画像座標 (px, 左上原点) → PDF座標 (pt, 左下原点)
  │     ├─ 水平スケーリングで bbox 幅にテキストをフィット
  │     └─ textRenderMode(3) = invisible（見えないが選択可能）
  │
  ├─ 4. pikepdf で元ページの上にオーバーレイ合成
  │
  └─ 5. 検索可能 PDF として保存
```

---

## 実行ログの例

```
Loading Surya OCR models...
  Models loaded in 8.2s

  Input  : exam.pdf  (664 pages)
  Process: 664 page(s)  (DPI=200)
  Output : exam_searchable.pdf
    [1/664] Page 1 ... 42 lines  (1.8s)
    [2/664] Page 2 ... 38 lines  (0.9s)
    [3/664] Page 3 ... 55 lines  (1.1s)
    ...

=======================================================
Processed : 664 pages
OCR lines : 28503
Total time: 625.3s  (0.94s/page)
Saved     : exam_searchable.pdf
```

---

## 🛠️ 技術スタック

| パッケージ | 用途 |
|-----------|------|
| [surya-ocr](https://github.com/VikParuchuri/surya) | テキスト検出 + 認識（GPU 対応 OCR エンジン） |
| [pikepdf](https://github.com/pikepdf/pikepdf) | PDF の読み書き・ページ合成 |
| [reportlab](https://www.reportlab.com/) | 透明テキスト PDF レイヤー生成（CID フォント対応） |
| [pypdfium2](https://github.com/nicegui-dev/pypdfium2) | PDF → 画像レンダリング |
| [Pillow](https://python-pillow.org/) | 画像操作 |

---

## 📌 注意事項

- 初回実行時に Surya OCR のモデル（約 1GB）がダウンロードされます
- DPI を上げると精度が向上しますが、処理時間と VRAM 使用量が増えます
- 暗号化された PDF は事前にパスワード解除が必要です
- 非常に大きな PDF（1000 ページ超）の場合は `--pages` で分割処理を推奨します

---

## License

MIT
