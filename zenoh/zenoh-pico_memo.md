# 概要
zenoh-picoに関する情報をここに示す

# メモ
- [zenoh-pico github](https://github.com/eclipse-zenoh/zenoh-pico.git)
- [zenoh-pico api reference](https://zenoh-pico.readthedocs.io/en/1.0.4/api.html)

# とりあえず使ってみる
```bash:intall
git clone -b 1.0.4 https://github.com/eclipse-zenoh/zenoh-pico.git
```

# error
```bash
lib/zenoh-pico/src/collections/refcount.c:118:2: error: #error "Multi-thread refcount in C99 only exists for GCC, use GCC or C11 or deactivate multi-thread"
 #error "Multi-thread refcount in C99 only exists for GCC, use GCC or C11 or deactivate multi-thread"
  ^~~~~
```
- この行を消す
- 一番上に`#define ZENOH_C_STANDARD 11`を書く