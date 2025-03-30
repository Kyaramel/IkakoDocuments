# 概要
RaspberryPi zero 2wについてのメモを示す

# ROS 2 humbleをinstall
[githubリポ](https://github.com/Ar-Ray-code/rpi-bullseye-ros2.git)
- README見ればいける

# sshが途切れる問題
- https://suzu-ha.com/entry/2023/12/25/000000
- どうやらsshが途切れるんじゃなくてラズパイ自体が固まってるみたい
- [スワップ容量を変更すれば治る](https://qiita.com/nnn112358/items/c5bbea50fc710b5c3f72)

# クロスコンパイル

# エラー
- `sudo apt upgrade`したら起動しなくなった
  `dphys-swapfile`を変更してるときに止まってる気がする