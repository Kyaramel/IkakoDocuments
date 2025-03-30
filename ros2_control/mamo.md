# 概要
`ros2_control`に関する情報をまとめる

# 公式資料
[ros2_control docs](https://control.ros.org/humble/index.html)
[ros2_control - humble Documentation](https://control.ros.org/humble/doc/api/index.html)
[ROS Japan UG #41 ROS/ROS2 Control勉強会資料](https://rosjp.connpass.com/event/200588/presentation/)
[ros-control roadmap for ROS2](https://github.com/ros-controls/roadmap)


# githubリポジトリ
[ros2_control](https://github.com/ros-controls/ros2_control.git)
[ros2_control_demos](https://github.com/ros-controls/ros2_control_demos.git)

# 実装メモ
`ros2_control docs - Getting Started`参考
```bash
mkdir -p control/src
cd control
vcs import --input https://raw.githubusercontent.com/ros-controls/ros2_control_ci/master/ros_controls.$ROS_DISTRO.repos src
rosdep update --rosdistro=$ROS_DISTRO
sudo apt update
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --continue-on-error --packages-skip-build-finished # 何をしてもgazebo_ros2_controlのビルドが通らなかったからその場しのぎ
colcon build --symlink-install --packages-skip gazebo_ros2_control ign_ros2_control # armプロセッサでgazebo_ros, ign_rosは動かない
```

```bash:サンプルを動かしてみた
# うごかん。。。
ros2 launch ros2_control_demo_example_1 view_robot.launch.py

sudo apt purge ros-humble-joint-state-publisher ros-humble-joint-state-publisher-gui
sudo apt install ros-humble-joint-state-publisher ros-humble-joint-state-publisher-gui
```

ここでエラー
```bash:gazeboエラー
--- stderr: gazebo_ros2_control                                                                                    
CMake Error at CMakeLists.txt:9 (find_package):
  By not providing "Findgazebo_ros.cmake" in CMAKE_MODULE_PATH this project
  has asked CMake to find a package configuration file provided by
  "gazebo_ros", but CMake did not find one.

  Could not find a package configuration file provided by "gazebo_ros" with
  any of the following names:

    gazebo_rosConfig.cmake
    gazebo_ros-config.cmake

  Add the installation prefix of "gazebo_ros" to CMAKE_PREFIX_PATH or set
  "gazebo_ros_DIR" to a directory containing one of the above files.  If
  "gazebo_ros" provides a separate development package or SDK, be sure it has
  been installed.

sudo apt install libgazebo-dev
source /opt/ros/humble/setup.bash  
```

```bash:ignition gazeboエラー
--- stderr: ign_ros2_control                                                                                       
CMake Error at CMakeLists.txt:43 (find_package):
  By not providing "Findignition-gazebo6.cmake" in CMAKE_MODULE_PATH this
  project has asked CMake to find a package configuration file provided by
  "ignition-gazebo6", but CMake did not find one.

  Could not find a package configuration file provided by "ignition-gazebo6"
  with any of the following names:

    ignition-gazebo6Config.cmake
    ignition-gazebo6-config.cmake

  Add the installation prefix of "ignition-gazebo6" to CMAKE_PREFIX_PATH or
  set "ignition-gazebo6_DIR" to a directory containing one of the above
  files.  If "ignition-gazebo6" provides a separate development package or
  SDK, be sure it has been installed.


sudo apt install libignition-gazebo6-dev
source /opt/ros/humble/setup.bash
```

どうやら`ros-humble-gazebo-ros-pkgs`はARMベースのシステムに対応してないらしい
rock5bでやったのが間違いかも。gazeboのconfigファイルはは`/usr/lib/aarch64-linux-gnu/cmake/gazebo/`にあった。
```bash:CMAKE_PREFIX_PATHの追加
export CMAKE_PREFIX_PATH=/usr/lib/aarch64-linux-gnu/cmake/gazebo:$CMAKE_PREFIX_PATH
```
```bash:ros-humble-gazebo-ros-pkgsをソースビルド
mkdir -p gazebo/src/
cd gazebo/src/
git clone https://github.com/ros-simulation/gazebo_ros_pkgs.git -b ros2
rosdep update
rosdep install --from-paths src --ignore-src -r -y
colcon build
```
↑できなかった。めっちゃ`/usr/local/include/google/protobuf/`でエラーが出てるみたい
```bash:protobufをreinstall
sudo apt remove --purge protobuf-compiler libprotobuf-dev libprotoc-dev
sudo apt install protobuf-compiler libprotobuf-dev libprotoc-dev
```


```bash
# sudo apt autoremoveしたら必要なやつも消しちゃったみたい

fatal error: sys/capability.h: No such file or directory
   33 | #include <sys/capability.h>
      |          ^~~~~~~~~~~~~~~~~~

sudo apt install libcap-dev # これで解決
```


# 実装メモ2
またやる気が出てきたからもう一回やってみる。

### 参考
- [demoを動かす]