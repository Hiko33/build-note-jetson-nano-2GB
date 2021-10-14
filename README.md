# build-note-jetson-nano-2GB
Jetson Nano 2GB 用ビルドノート（build211014、随時更新中）
NVIDIAのOSイメージはJetpack4.5を選択（最新は4.6.1）※2021年10月時点
ベースのOSはUbuntu、デスクトップはLXDE（2GB) / Unity（4GB)であった
SSH、DockerはImageからの立ち上げでデフォルトで使用可能になっている。
但しDockerはUser権限無し
PWは10桁以上を推奨

初期立ち上げはデスクトップからの指示で一通り行う。その後下記の操作をCLIから実施

# ・まずはアップデート
　sudo apt update && sudo apt upgrade -y
　アップグレードでエラーが出たら
　→dpkgやapt-get関連でエラーが出た時の対処法
　　https://qiita.com/yukari-n/items/d1b17bd37036f120153c

・swapの追加
　sudo dd if=/dev/zero of=/var/swapfile bs=1G count=4（4GB追加の場合）
　sudo mkswap /var/swapfile
　sudo chmod 600 /var/swapfile
　sudo vi /etc/fstab
　→末尾に追加
　　/var/swapfile        none                  swap           swap                                         0 0
　　sudo swapon /var/swapfile
　　free -m  でswap容量確認

# ・Jetson statsのインストール
　sudo apt install python-pip
　sudo -H pip install -U jetson-stats
　sudo reboot
　→jtop実行後に
　　[WARN] jetson-stats not supported for [L4T 32.5.2]　の
　　メッセージが出る件については調査中

# ・プリインストールのVNCサーバーが遅いので、tightvcnserverを入れる
　sudo apt install tightvncserver -y
　sudo apt autoremove　（必須でない）
　VNCを起動。
　　tightvncserver -geometry 1920x1080
　パスワードを求められるので入力（8文字まで）monster4
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

# ・セキュリティ対策
　ルートパスワード設定
　　sudo passwd root　→　パスワード設定

# ・パスワード無しでsudo実行可能とする
　　sudo visudo　→エディタ開く
　　一番下の行に以下のように入力
　　{ユーザー名} ALL=(ALL) NOPASSWD: ALL

# ・Dockerをsudoなしで立ち上げられる設定にする
　　sudo gpasswd -a $USER docker
　　（確認方法）groups <user>

# ・Jetson GPIO Libraryパッケージの導入
　git clone https://github.com/NVIDIA/jetson-gpio.git
　cd jetson-gpio
　sudo python3 setup.py install
　sudo groupadd -f -r gpio
　sudo usermod -a -G gpio <user>
　sudo cp lib/python/Jetson/GPIO/99-gpio.rules /etc/udev/rules.d/
　sudo udevadm control —reload-rules && sudo udevadm trigger
　反映のため再起動

# ・nanoエディタインストール（必須でない）
　sudo apt install nano

# ・sublimeインストール
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

　
