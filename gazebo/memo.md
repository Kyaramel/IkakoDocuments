# 概要
gazeboに関する内容をここに示す

# dockerをつかう
- これが最適解
```bash
sudo apt install xorg
sudo docker pull arm64v8/gazebo:latest
sudo docker run -it arm64v8/gazebo:latest
```
```bash:dockerでgui
xhost +local:docker
sudo docker run -it -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix arm64v8/gazebo:latest
```

# arm64にgazeboを入れる
gazeboはarm64プロセッサシステムに対応しておらず`apt install`で入れることができない。この場合はソースビルドする必要がある

```bash:sdformat9の9.8をソースビルド
sudo apt install ignition-tools
wget http://osrf-distributions.s3.amazonaws.com/sdformat/releases/sdformat-9.8.0.tar.bz2
tar -xf sdformat-9.8.0.tar.bz2
cd sdformat-9.8.0
mkdir build
cd build
cmake ..
make -j4
sudo maek install 
```

```bash:gazeboをソースビルド
git clone https://github.com/osrf/gazebo.git
cd gazebo
git checkout gazebo11
mkdir build
cd build
cmake .. # ここでエラー1
make -j$(nproc)
sudo make install

export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
echo 'export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
```

```bash:頑張ればaptで入る？
sudo add-apt-repository ppa:openrobotics/gazebo11-non-amd64
sudo apt update
sudo apt install gazebo
```


# protobufをソースビルドする
まじでこいつはなんなんだ。ビルドエラーを吐き出す時にのみ現れる。googleはくそ
[多分これを参考にビルドすればいい](https://github.com/protocolbuffers/protobuf/blob/main/src/README.md)
```bash:protobufをソースビルド
# ubuntu22.04にbazelをaptインストール
sudo su
curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor > /etc/apt/trusted.gpg.d/bazel.gpg
cat << 'EOF' > /etc/apt/sources.list.d/bazel.list
deb [arch=amd64] https://storage.googleapis.com/bazel-apt stable jdk1.8
EOF
exit

sudo apt update
sudo apt install -y autoconf automake libtool curl make g++ unzip git ca-certificates gnupg lsb-release bazel
git clone https://github.com/protocolbuffers/protobuf.git
cd protobuf
# git checkout 3.6.x バージョンを指定する場合
git submodule update --init --recursive

# arm64
bazel build :protoc :protobufocp bazel-bin/protoc /usr/local/bin

# ./autogen.sh
# ./configure --prefix=/usr/local
# make -j$(nproc)
# sudo make install
# sudo ldconfig
# protoc --version
```

bazelはarm64に対応してない
```bash:代替案
wget https://github.com/bazelbuild/bazelisk/releases/download/v1.10.1/bazelisk-linux-arm64
chmod +x bazelisk-linux-arm64
```

[参考](https://sig9.org/blog/2022/05/23/)

```bash:メモ
ikako@ikako-rock5b:/usr/local$ sudo unzip protoc-29.2-linux-aarch_64.zip 
Archive:  protoc-29.2-linux-aarch_64.zip
  inflating: bin/protoc              
  inflating: include/google/protobuf/any.proto  
  inflating: include/google/protobuf/api.proto  
   creating: include/google/protobuf/compiler/
  inflating: include/google/protobuf/compiler/plugin.proto  
  inflating: include/google/protobuf/cpp_features.proto  
  inflating: include/google/protobuf/descriptor.proto  
  inflating: include/google/protobuf/duration.proto  
  inflating: include/google/protobuf/empty.proto  
  inflating: include/google/protobuf/field_mask.proto  
  inflating: include/google/protobuf/go_features.proto  
  inflating: include/google/protobuf/java_features.proto  
  inflating: include/google/protobuf/source_context.proto  
  inflating: include/google/protobuf/struct.proto  
  inflating: include/google/protobuf/timestamp.proto  
  inflating: include/google/protobuf/type.proto  
  inflating: include/google/protobuf/wrappers.proto  
  inflating: readme.txt 
```


# protobufをaptでinstallしてlocalにmvする
```bash
# この時点でpcないからprotobufを完全に抹消していること！！
sudo apt install protobuf-compiler
sudo mv /usr/bin/protoc /usr/local/bin/protoc
sudo mkdir -p /usr/local/include/google/
sudo mv /usr/include/google/protobuf/ /usr/local/include/google/
```


# ソースビルドしたprotobufを削除
```bash:./configure --prefix=/usr/localの場合
sudo rm -rf /usr/local/include/google
sudo rm -rf /usr/local/lib/libprotobuf*
sudo rm -rf /usr/local/bin/protoc
sudo rm -rf /usr/local/share/man/man1/protoc.1
```


# protobufをバージョンごとにソースビルドして環境変数で選択する
- この方法でやるとパッケージごとに`CMakeLists.txt`内に手動でprotobufのパスをsetしなきゃかも
  ```cmake
  set(Protobuf_INCLUDE_DIR /usr/local/protobuf-3.6.1/include)
  ```

```bash:protobuf3.6.1のビルド
wget -q https://github.com/protocolbuffers/protobuf/archive/v3.6.1.tar.gz
tar -xzf v3.6.1.tar.gz
cd protobuf-3.6.1
./autogen.sh
./configure --prefix=/usr/local/protobuf-3.6.1 --enable-static=no # インストール先を指定
make -j4
sudo make install
```
```bash:protobuf3.6.1を使用する場合
PATH=$(echo "$PATH" | tr ':' '\n' | sed '/\/usr\/local\/protobuf-3.12.4\/bin/d' | tr '\n' ':')
LD_LIBRARY_PATH=$(echo "$LD_LIBRARY_PATH" | tr ':' '\n' | sed '/\/usr\/local\/protobuf-3.12.4\/lib/d' | tr '\n' ':')
export PATH LD_LIBRARY_PATH
export PATH="$PATH:/usr/local/protobuf-3.6.1/bin"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/protobuf-3.6.1/lib"
```

```bash:protobuf3.12.4のビルド
wget -q https://github.com/protocolbuffers/protobuf/archive/v3.12.4.tar.gz
tar -xzf v3.12.4.tar.gz
cd protobuf-3.12.4
./autogen.sh
./configure --prefix=/usr/local/protobuf-3.12.4 --enable-static=no # インストール先を指定
make -j4
sudo make install
```
```bash:protobuf3.12.4を使用する場合
PATH=$(echo "$PATH" | tr ':' '\n' | sed '/\/usr\/local\/protobuf-3.6.1\/bin/d' | tr '\n' ':')
LD_LIBRARY_PATH=$(echo "$LD_LIBRARY_PATH" | tr ':' '\n' | sed '/\/usr\/local\/protobuf-3.6.1\/lib/d' | tr '\n' ':')
export PATH LD_LIBRARY_PATH
export PATH="$PATH:/usr/local/protobuf-3.12.4/bin"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/protobuf-3.12.4/lib"
```

```bash:環境変数から特定のパスのみ削除するコマンド
PATH=$(echo "$PATH" | awk -F':' '$0 !~ "/不要なパス/" {printf "%s%s", $0, (NR>1 ? ":" : "")}')
export PATH
```


# エラー一覧
### エラー1
protobuf大っ嫌い
```bash
CMake Error at /usr/share/cmake-3.22/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find Protobuf (missing: Protobuf_INCLUDE_DIR)
Call Stack (most recent call first):
  /usr/share/cmake-3.22/Modules/FindPackageHandleStandardArgs.cmake:594 (_FPHSA_FAILURE_MESSAGE)
  /usr/share/cmake-3.22/Modules/FindProtobuf.cmake:650 (FIND_PACKAGE_HANDLE_STANDARD_ARGS)
  cmake/SearchForStuff.cmake:47 (find_package)
  CMakeLists.txt:163 (include)
```