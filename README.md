# ubuntu_gpu_driver

ubuntuにGPUドライバーをインストールすることに若干苦労したため, 備忘録としてメモを示す.

# 環境

**ubuntu 22.04LTS**で実施.

# 手順

## Nouveauの無効化

`sudo gedit /etc/modprobe.d/blacklist-nouveau.conf`でnouveauの設定ファイルを新規作成し, 以下を記述し保存.

```
blacklist nouveau
options nouveau modeset=0
```

保存後, `sudo update-initramfs -u`を実行.

## 現在のCUDA, nvidia-driverの確認・削除

現在使用しているCUDA・nvidia-driverを確認し, 削除する(無い場合はskip).</br>
以下で確認
```
dpkg -l | grep nvidia
dpkg -l | grep cuda
```

以下コマンドで削除

```
sudo apt-get --purge remove nvidia-*
sudo apt-get --purge remove cuda-*
```

## ディスプレイサーバーをX11に変更

`sudo vim /etc/gdm3/custom.conf`でファイルを開き, 以下のように変更

```
# GDM configuration storage
#
# See /usr/share/gdm/gdm.schemas for a list of available options.

[daemon]
AutomaticLoginEnable=True
AutomaticLogin=kaito

# Uncomment the line below to force the login screen to use Xorg
WaylandEnable=false

# Enabling automatic login

# Enabling timed login
#  TimedLoginEnable = true
#  TimedLogin = user1
#  TimedLoginDelay = 10

[security]

[xdmcp]

[chooser]

[debug]
# Uncomment the line below to turn on debugging
# More verbose logs
# Additionally lets the X server dump core if it crashes
#Enable=true
```

## .runファイルでNVDIAドライバーをインストール

念の為以下コマンドを実施

```
sudo apt remove --purge nvidia*
```

[NVIDIA公式最新ドライバーのダウンロード](https://www.nvidia.com/ja-jp/drivers/)から.runファイルをダウンロード.</br>

以下コマンドを実施.

```
sudo chmod 755 NVIDIA-Linux-x86_64-550.107.02.run 
```

`[Ctrl] + [Alt] + [F6]`でコンソールモードに切り替え, ログイン.

`sudo service lightdm stop` と `sudo service gdm stop`でGUIサービスを停止.

runファイルでのインストールには`gcc-12`と`make`が必要であるため下記コマンドでインストール.

```
sudo apt install make

sudo apt install gcc-12 gcc-12-base libgcc-12-dev
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 20
sudo update-alternatives --config gcc
```

入力後, `gcc --version`でgccのバージョンを確認.</br>
確認後, 以下コマンドを実施.

```
sudo ./NVIDIA-Linux-x86_64-525.60.11.run
reboot
```

再起動後, `nvidia-smi`で動作確認.

# CUDAのインストール

[CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)からインストールコマンドを取得し, 実行.</br>
Installer Typeは**deb(network)**を選択.</br>
下記は取得したコマンドの例.

```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-6
```

終了後`reboot`で再起動.</br>
再起動後, `sudo vim ~/.bashrc`でファイルを開き以下を記述.

```
export PATH=/usr/local/cuda:/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/lib:/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

記入後, `source ~/.bashrc`を実行. 以下コマンドで確認.

```
nvcc -V
```

バージョンが確認できれば, インストール終了.

# 参考文献

1. [ubuntuにCUDA、nvidiaドライバをインストールするメモ](https://qiita.com/porizou1/items/74d8264d6381ee2941bd)
2. [[ubuntu22.04].runファイルでNVIDIA ドライバをインストールする手順](https://qiita.com/porizou1/items/09270f2b4d3196b00aee)
3. [Ubuntu で Wayland と Xorg を切り替える方法](https://jp.moyens.net/how-to/220737/)
4. [ubuntu18.04にnvidiaドライバを入れるの苦労した](https://qiita.com/kawazu191128/items/8a46308be6949f5bda57)
5. [Ubuntu17.10にNVIDIAドライバをインストールしてみた。](https://qiita.com/tatsuya11bbs/items/be6ceb85eee074c57b71)
6. [Ubuntu に最新の NVIDIA Driver をインストールする。](https://zondeel.hateblo.jp/entry/2014/08/29/202919)
7. [nvidiaのdriverをインストール(ubuntu)](https://mikemoke.hatenablog.com/entry/2015/06/23/004739)
8. [UbuntuでCUDA，NVIDIAドライバ，cudnnをインストールし，PyTorchでGPU環境を使えるようにするまで](https://qiita.com/tf63/items/0c6da72fe749319423b4)
9. [Ubuntu Linux 22.04 設定メモ](https://qiita.com/j0306043/items/fcc9546056eeca5b025a)
10. [Driver install fails with the error [An error occurred while performing the step: “building kernel modules”. See /var/log/nvidia-installer.log…]](https://forums.developer.nvidia.com/t/driver-install-fails-with-the-error-an-error-occurred-while-performing-the-step-building-kernel-modules-see-var-log-nvidia-installer-log/280385/1)