# 概要
Unitree Go1に関する情報をここに示す。

# wifi
- デフォルトでアクセスポイントになっている。
```bash
SSID: Unitree_Go290205A
PASS: 00000000
IP: 192.168.12.1
```

- 基本的には`Unitree_Go290205A`に接続して↑のipにsshで入る
- 外部wifiに接続するにはetherか内部をいじってwifiに接続する
- wifiでやる方法
  1. 普通にunitree wifiかethernetでssh
  2. `sudo wpa_supplicant -c /etc/wpa_supplicant/wpa_supplicant.conf -i wlan0`を実行する。-iはインターフェース
     `/var/run/wpa_supplicant/`にインターフェースのファイルがある場合は削除する。
     外部wifiに接続するとデフォのwifiは無くなるので注意（再起動すると元通りになる）
  3. このままじゃ、wifiには繋がってもネットには繋がらないので`netstat -nr`でデフォルトゲートウェイを確認する
    ```bash:出力例
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    0.0.0.0         192.168.123.1   0.0.0.0         UG        0 0          0 eth0
    0.0.0.0         192.168.12.1    0.0.0.0         UG        0 0          0 wlan1
    192.168.2.0     0.0.0.0         255.255.255.0   U         0 0          0 eth0
    192.168.12.0    0.0.0.0         255.255.255.0   U         0 0          0 wlan1
    192.168.123.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
    224.0.0.0       0.0.0.0         240.0.0.0       U         0 0          0 lo
    ```
  4. `sudo route add default gw 192.168.1.1`でデフォルトゲートウェイを追加
  5. `ping 8.8.8.8`で通信できているか確認