# 概要
ロボットのシミュレーションプラットフォーム「genesis」について調べたことなどをここに示す

# 参考
- [genesis 入門](https://note.com/npaka/n/n07b448c74613)
- [genesis日本語docs](https://genesis-world.readthedocs.io/ja/latest/user_guide/index.html)
- [genesis日本語README](https://github.com/Genesis-Embodied-AI/Genesis/blob/main/README_JA.md)

# install
- [genesis install](https://note.com/npaka/n/n07b448c74613)
- macだとpipじゃできなかった。conda推奨
- ubuntuでもcondaが必要？

```bash:ubuntuにinstall
# minicondaをinstall
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh # x86
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh -O ~/miniconda3/miniconda.sh # aarch64
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh

# 仮想環境にデフォルトで入らないようにする設定
conda config --set auto_activate_base False

source ~/miniconda3/bin/activate

# genesis用conda環境の作成
conda create --name genesis_env python=3.10 -y
conda activate genesis_env
conda install torch torchvision torchaudio -c pytorch
pip install genesis-world
conda config --add channels conda-forge
conda install pyrender
pip install PyOpenGL==3.1.5

# genesisのサンプルをclone
git clone https://github.com/Genesis-Embodied-AI/Genesis
```

# モーションプランニング
- [インバースキネマティクスとモーションプランニング](https://note.com/npaka/n/n3b06df2458c1)
- genesisのモーションプランニングには「OMPL」を使う
```bash:mac x86でOMPLをinstall
wget https://github.com/ompl/ompl/releases/download/prerelease/ompl-1.6.0-cp310-cp310-macosx_13_0_x86_64.whl
pip install ./ompl-1.6.0-cp310-cp310-macosx_13_0_x86_64.whl
```