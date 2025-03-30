# 概要
halを使用したstm32g474re開発方法についてここに示す

[stm32g474reデータシート](https://www.st.com/resource/en/datasheet/stm32g474re.pdf)
[stm32g4シリーズのhalドキュメント](https://www.stmcu.jp/download/?dlid=669290_jp)
[HAL関数まとめ](https://tattakaaqua.hatenablog.com/entry/2017/12/10/220854)
[stm32マイコンboot関係のデータシート](https://www.st.com/resource/ja/application_note/an2606-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf)
[stm32 usb](https://zeptoelecdesign.com/stm32-usb/)
[stm32 hal タイマー割り込み](https://rt-net.jp/mobility/archives/20315)
[stm32 hal pwm](https://moons.link/post-632/)

# vscodeで開発する
[STM32CubeMXとPlatformIOでSTM32開発環境構築](https://qiita.com/desertfox_i/items/c6391e7d38148cfd4041)
1. 必要なpythonパッケージをインストール
```bash
pip install stm32pio platformio
```
2. cubeMXで作成したディレクトリでプロジェクトをplatformioで使える形式に変換する
```bash:cubeをplatformio形式に変換するコマンド
stm32pio new  -b nucleo_g474re
```
3. 
```ini:platformio.ini
[env:nucleo_g474re]
platform = ststm32
monitor_speed = 115200
upload_protocol = custom
upload_command = STM32_Programmer_CLI -c port=SWD -w $BUILD_DIR/firmware.elf 0x08000000 -v -s 0x08000000
board = nucleo_g474re
framework = stm32cube
board_build.stm32cube.custom_config_header = yes
build_flags =
    -mcpu=cortex-m4
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=softfp
    ; -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/include
    ; -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2
    ; -I$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F
build_src_filter = 
    +<*>
    ; FreeRTOS 関連のソース（既存設定）
    ; +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/*.c>
    ; +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2/*.c>
    ; +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/MemMang/heap_4.c>
    ; +<$PROJECT_DIR/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F/*.c>

[platformio]
include_dir = Inc
src_dir = Src
```
STM32CubeProgrammerでBOOT関係のレジスタを設定する必要があるかも。


# FDCAN
- クロック何もわからん
- Prescaler: クロックの分周比
- PLL: 位相同期回路。入力の周波数を整数倍した出力を得ることができる。
- FDCANのクロックはSYSCLKから派生する
- [stm32のクロック系統を理解する](https://moons.link/post-1326/)
- [stm32g4 can classic](http://blog.livedoor.jp/tec_kanpaku/archives/40427025.html)

# FreeRTOS
[公式のチュートリアル](https://www.stmcu.jp/download/?dlid=51214_jp)
[STM32で始めるFreeRTOSとLwIP](https://qiita.com/tom_S/items/b152bfc8cf572c1a6f04)
[STM32のFreeRTOSの使い方まとめ](https://yukblog.net/stm32-freertos/)

# Platformioで書き込みはできるけど動作しない
- `platformio.ini`でSTM32CubeProgrammer CLIのコマンドを使う
[STM32CubeProgrammer data sheet](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://www.st.com/resource/en/user_manual/um2237-stm32cubeprogrammer-software-description-stmicroelectronics.pdf&ved=2ahUKEwjYoYrfov2LAxXer1YBHRKoO8gQFnoECBsQAQ&usg=AOvVaw1CNQXxF1ef1d-yOhKnUOYA)
STM32_Programmer_CLI -c port=SWD reset=HWrst mode=UR ap=0 -w $BUILD_DIR/firmware.bin 0x08000000 -v --start 0x08000000
```

