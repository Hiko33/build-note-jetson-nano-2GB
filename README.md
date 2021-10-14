# build-note-jetson-nano-2GB
**Jetson Nano** 2GB 用ビルドノート（build211014、**随時更新中**）<br>
NVIDIAのOSイメージはJetpack4.5を選択（最新は4.6.1）※2021年10月時点<br>
ベースのOSはUbuntu、デスクトップはLXDE（2GB) / Unity（4GB)であった<br>
SSH、DockerはImageからの立ち上げでデフォルトで使用可能になっている。<br>
但しDockerはUser権限無し<br>
PWは10桁以上を推奨<br>
初期立ち上げはデスクトップからの指示で一通り行う。その後下記の操作をCLIから実施<br>

### ・まずはアップデート
sudo apt update && sudo apt upgrade -y<br>
アップグレードでエラーが出たら<br>
→dpkgやapt-get関連でエラーが出た時の対処法<br>
https://qiita.com/yukari-n/items/d1b17bd37036f120153c<br>
#### ・swapの追加
sudo dd if=/dev/zero of=/var/swapfile bs=1G count=4（4GB追加の場合）<br>
sudo mkswap /var/swapfile<br>
sudo chmod 600 /var/swapfile<br>
sudo vi /etc/fstab<br>
→末尾に追加<br>
/var/swapfile        none                  swap           swap                                         0 0<br>
sudo swapon /var/swapfile<br>
free -m  でswap容量確認<br>

### ・Jetson statsのインストール
sudo apt install python-pip<br>
sudo -H pip install -U jetson-stats<br>
sudo reboot<br>
→jtop実行後に<br>
[WARN] jetson-stats not supported for [L4T 32.5.2]　の<br>
メッセージが出る件については調査中<br>

### ・tightvcnserverのインストール
プリインストールのVNCサーバーが遅いための解決策<br>
sudo apt install tightvncserver -y<br>
sudo apt autoremove　（必須でない）<br>
VNCを起動<br>
tightvncserver -geometry 1920x1080<br>
パスワードを求められるので入力（8文字まで）
　view only　〜　はnoでいい
　VNC viewerから　<user>@<host>.local:1 で接続できることを確認
　この時点ではデスクトップが立ち上がらず、NVIDIAマークが表示される状態で止まる
　VNC自動起動用設定
　　sudo crontab -e
　　末尾に以下のコマンドを追加（解像度は適宜調整）
　　　@reboot su - <user> -c '/usr/bin/tightvncserver -geometry 1920x1080'
　デスクトップを表示される設定を下記ファイル
　　~/.vnc/xstartup　に
　　　startlxde &　　を追加し、再起動
　参考URL
　https://forums.developer.nvidia.com/t/tightvnc-with-desktop-environment-on-jetson-nano-2gb-in-headless-mode/163593
　デスクトップからスクリーンセーバーをOFFにする

### ・セキュリティ対策
　ルートパスワード設定
　　sudo passwd root　→　パスワード設定

### ・パスワード無しでsudo実行可能とする
　　sudo visudo　→エディタ開く
　　一番下の行に以下のように入力
　　{ユーザー名} ALL=(ALL) NOPASSWD: ALL

### ・Dockerをsudoなしで立ち上げられる設定にする
　　sudo gpasswd -a $USER docker
　　（確認方法）groups <user>

### ・Jetson GPIO Libraryパッケージの導入
　git clone https://github.com/NVIDIA/jetson-gpio.git
　cd jetson-gpio
　sudo python3 setup.py install
　sudo groupadd -f -r gpio
　sudo usermod -a -G gpio <user>
　sudo cp lib/python/Jetson/GPIO/99-gpio.rules /etc/udev/rules.d/
　sudo udevadm control —reload-rules && sudo udevadm trigger
　反映のため再起動

### ・nanoエディタインストール（必須でない）
　sudo apt install nano

### ・sublimeインストール
　linux版は下記の手順でインストール
　・GPGキーのインストール
　　wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
　・apt-getでhttpsを使用できるようにする。
　　sudo apt-get install apt-transport-https
　・使用するチャンネル？を選択
　　・stable
　　　echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
　　・dev
　　　echo "deb https://download.sublimetext.com/ apt/dev/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
　・インストール
　　sudo apt-get update
　　sudo apt-get install sublime-text
　・起動するとライセンス情報を要求されるため、macからコピーしたテキスト情報を全部ペースト
　　(macから）sftp put subl_license
