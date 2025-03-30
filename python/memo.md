# 概要
python関連の情報をここに示す

# vscodeでコード整形の設定
`autopep8`を使用する方法
1. vscodeの拡張機能で`autopep8`をインストールする
2. 今自分が作業しているpython環境にautopep8を入れる
```bash
pip install autopep8
```
3. vscode画面右下のpythonインタープリタに自分のpython環境を認識させる
4. `settings.json`をちょっと編集する
```json:settings.json
~~~
"[python]": {
    "editor.defaultFormatter": "ms-python.autopep8",
    "editor.formatOnSave": true
},
~~~
```
5. linuxなら`ctrl + shift + i`でコード整形

# pythonを使った開発を快適にしたい
- logをファイルで出力させる
- デバッグ機能を使う
- プログラムファイルを小分けに作って関数一つで実行できるようにする。jupyterで呼び出す

# 小技
pythonは小技が多すぎてわかりにくい
- `print(f"operater={operater=}")` `{変数名=}`でその変数の中身をprintしてくれる
