# 概要
stm32f446reでhalを使用した開発メモ

# 環境構築
[STM32をVSCode&PlatformIO(C++)で開発する](https://iroiro-craft.hatenablog.com/entry/2022/06/04/172802)
1. cubemxでいろいろ設定する
2. コードジェネレイト
3. stm32pioを使って生成したプログラムをplatformioで使える形式にする
```bash
stm32pio new -b nucleo_f446re
```
4. vscodeで開いてビルド