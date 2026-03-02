# suryaOCR

surya-env の再アクティベート方法
今回問題になった通り、ターミナルを閉じるとvenvは無効化されます。次回使うときは、PowerShellで以下のコマンドを実行してください：

& "$env:USERPROFILE\surya-env\Scripts\Activate.ps1"

& "$env:USERPROFILE\surya-env\Scripts\Activate.ps1"
アクティベート成功すると、プロンプトの先頭に (surya-env) と表示されます。その状態で python コマンドを使えばPython 3.10.11が使われます。

使用例（毎回のパターン）

# 1. venvをアクティベート
& "$env:USERPROFILE\surya-env\Scripts\Activate.ps1"

# 2. PDFにOCRテキストを重ねる
python "$env:USERPROFILE\Desktop\pdf_ocr_overlay.py" --input "入力.pdf" --output "出力.pdf"

# 3. 画像フォルダをOCR
python "$env:USERPROFILE\Desktop\surya_batch_ocr.py" --input_dir "画像フォルダ"


# 1. venvをアクティベート& "$env:USERPROFILE\surya-env\Scripts\Activate.ps1"# 2. PDFにOCRテキストを重ねるpython "$env:USERPROFILE\Desktop\pdf_ocr_overlay.py" --input "入力.pdf" --output "出力.pdf"# 3. 画像フォルダをOCRpython "$env:USERPROFILE\Desktop\surya_batch_ocr.py" --input_dir "画像フォルダ"
⚠️ 注意点
WSLからは使えません（パッケージがWindows側にしかない）
PowerShell（Windows）で実行してください
venvを有効にしないとシステムのPython 2.7（MGLTools）が使われてエラーになります
