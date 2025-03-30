# 概要
stm32f767ziでhalを使用した開発方法について示す

# メモ
- 1.17.2だとビルドがとおらない〜〜〜


# 環境構築
[STM32をVSCode&PlatformIO(C++)で開発する](https://iroiro-craft.hatenablog.com/entry/2022/06/04/172802)
1. cubemxでいろいろ設定する
2. コードジェネレイト
3. stm32pioを使って生成したプログラムをplatformioで使える形式にする
```bash
stm32pio new -b nucleo_f767zi
```
4. vscodeで開いてビルド
5. いっぱいエラーが出る
platformio.iniの内容を変えたらビルドいけた
```ini:platformio.ini
[env:nucleo_f767zi]
platform = ststm32
board = nucleo_f767zi
framework = stm32cube
board_build.stm32cube.custom_config_header = yes
build_flags =
    -mcpu=cortex-m7
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=softfp
    -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/include
    -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2
    -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM7/r0p1
platform_packages = framework-stm32cubef7@^1.17.2
; FreeRTOSソースをコンパイル対象に含める例（必要なファイルのみ追加）
build_src_filter = 
    +<*>
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/MemMang/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM7/r0p1/*.c>


[platformio]
include_dir = Inc
src_dir = Src
```

lwIPを追加したらこんな感じになった
```ini:platformio.ini
[env:nucleo_f767zi]
platform = ststm32
board = nucleo_f767zi
framework = stm32cube
monitor_speed = 115200
board_build.stm32cube.custom_config_header = yes
build_flags =
    -mcpu=cortex-m7
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=softfp
    -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/include
    -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2
    -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM7/r0p1
    -I$PROJECT_DIR/Middlewares/Third_Party/LwIP/src/include
    -I$PROJECT_DIR/Middlewares/Third_Party/LwIP/system
platform_packages = framework-stm32cubef7@^1.17.2
; FreeRTOSソースをコンパイル対象に含める例（必要なファイルのみ追加）
build_src_filter = 
    +<*>
    ; FreeRTOS 関連のソース（既存設定）
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/MemMang/heap_4.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM7/r0p1/*.c>
    ; lwIP のコア部分
    +<$PROJECT_DIR/Middlewares/Third_Party/LwIP/src/core/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/LwIP/src/core/ipv4/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/LwIP/src/core/ipv6/*.c>
    ; lwIP の API とネットインターフェース
    +<$PROJECT_DIR/Middlewares/Third_Party/LwIP/src/api/*.c>
    +<$PROJECT_DIR/Middlewares/Third_Party/LwIP/src/netif/*.c>
    ; lwIP の OS 抽象化レイヤー
    +<$PROJECT_DIR/Middlewares/Third_Party/LwIP/system/OS/*.c>


[platformio]
include_dir = Inc
src_dir = Src
```
デフォルトのファイルパス以外自分で設定しなきゃいけないのだるすぎる。ぜったいどっかで環境構築ミスってる。

# PlatformIOで外部のhalドライバをつかう
こんなんしなくてもよかった

stm32f767ziの環境を作ったらcubemx側のhal driverとplatformio側のhal driverのバージョンに相違が生じて、ビルドができない。
しょうがないから最新版のhal driverをgit cloneしてplatformioで使う

1. git clone
```bash
git clone --recursive --depth 1 --branch v1.17.2 https://github.com/STMicroelectronics/STM32CubeF7.git # 1.17.2をclone
```

2. `package.json`を作成する
```bash:STM32CubeF7/package.json
{
  "name": "framework-stm32cube",
  "version": "v1.17.2",
  "description": "Custom STM32Cube HAL for STM32F7",
  "keywords": ["framework", "hal", "stm32", "stm32cube"],
  "homepage": "https://github.com/STMicroelectronics/STM32CubeF7"
}
```

3. `platformio.ini`でカスタムhalを使うように設定
```ini
[env:nucleo_f767zi]
platform = ststm32
board = nucleo_f767zi
framework = stm32cube
; 以下のパスはカスタムHALドライバのパスに合わせて変更してください
platform_packages =
    framework-stm32cube@file:///Users/ikako/ikako_ws/cubemx/hal_drivers/STM32CubeF7
```