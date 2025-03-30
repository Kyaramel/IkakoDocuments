# 概要
cybergearについて調べたことをここにまとめる

# メモ
- [cybergear をブラウザから操作するコード？](https://github.com/Tony607/Cybergear.git)
  動かし方がよくわからない

- [cybergear_ros2](https://github.com/NaokiTakahashi12/cybergear_ros2.git)
  本当に動く？

- [cybergear資料類](https://drive.google.com/drive/folders/1DhB4O10Yd8j_8nWrY-7yux-uzQbmLRMH)

- [cybergear docs datasheet en](https://github.com/belovictor/cybergear-docs/blob/main/instructionmanual/instructionmanual.md)

- [OpenELAB Cybergear wiki](https://wiki.openelab.io/motors/cybergear-micromotor-instruction-manual)
  一番わかりやすくまとめられている

# `cybergear_ros2`の使い方
- `ros2 run`の時に`--ros-args -p device_id:=`でcybergearのidを設定する
- nodeを起動したらserviceで`~/enable_torque`に`{"data":true}`を送信すると動作開始。`{"data":false}`とすると停止
- 指令値は`/joint_trajectory (trajectory_msgs/msg/JointTrajectory)`に送信する
```yaml:3.14[rad]に位置制御する時のyaml
{
  "header": {
    "stamp": {
      "sec": 0,
      "nanosec": 0
    },
    "frame_id": "cybergear"
  },
  "joint_names": [
    "cybergear"
  ],
  "points": [
    {
      "positions": [
        3.14
      ],
      "velocities": [
        0
      ],
      "accelerations": [
        0
      ],
      "effort": [
        0
      ],
      "time_from_start": {
        "sec": 0,
        "nanosec": 0
      }
    }
  ]
}
```

- 位置制御 - `ros2 run cybergear_socketcan_driver cybergear_position_driver_node --ros-args -p device_id:=<id>`
- 速度制御 - `ros2 run cybergear_socketcan_driver cybergear_velocity_driver_node --ros-args -p device_id:=<id>`
- トルク制御 - `ros2 run cybergear_socketcan_driver cybergear_torque_driver_node --ros-args -p device_id:=<id>`
- `ros2_socketcan`のreceiverとsenderを起動しておく必要がある

# パラメータについて
- 位置制御に関しては「位置指定による位置制御」と「パラメータ指定による位置制御」の二つがある
  - 位置制御はcybergear側で実装されているが、なぜかパラメータで位置を指定できる
  - パラメータidは`0x7016`
  - でも、手動でデコード指令値と応答値が合わない