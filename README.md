# build-note-jetson-nano-2GB
**Jetson Nano** 2GB 用ビルドノート（build211014、**随時更新中**）<br>
NVIDIAのOSイメージはJetpack4.5を選択（最新は4.6.1）※2021年10月時点<br>
ベースのOSはUbuntu、デスクトップはLXDE（2GB) / Unity（4GB)であった<br>
SSH、DockerはImageからの立ち上げでデフォルトで使用可能になっている。<br>
但しDockerはUser権限無し<br>
PWは10桁以上を推奨<br>
初期立ち上げはデスクトップからの指示で一通り行う。その後下記の操作をCLIから実施<br>

### ・まずはアップデート
	sudo apt update && sudo apt upgrade -y
アップグレードでエラーが出たら<br>
→[dpkgやapt-get関連でエラーが出た時の対処法](https://qiita.com/yukari-n/items/d1b17bd37036f120153c)
### ・swapの追加
	sudo dd if=/dev/zero of=/var/swapfile bs=1G count=4（4GB追加の場合）
	sudo mkswap /var/swapfile
	sudo chmod 600 /var/swapfile
	sudo vi /etc/fstab
→末尾に追加<br>

	/var/swapfile        none                  swap           swap                                         0 0
	sudo swapon /var/swapfile<br>

	free -m  でswap容量確認

### ・Jetson statsのインストール
	sudo apt install python-pip
	sudo -H pip install -U jetson-stats

	sudo reboot

	jtop
→jtop実行後に<br>
[WARN] jetson-stats not supported for [L4T 32.5.2]　の<br>
メッセージが出る件については調査中<br>

### ・tightvcnserverのインストール
プリインストールのVNCサーバーが遅いための解決策<br>

	sudo apt install tightvncserver -y
	sudo apt autoremove　（必須でない）
VNCを起動<br>

	tightvncserver -geometry 1920x1080
パスワードを求められるので入力（8文字まで）<br>

	view only　〜　はnoでいい
VNC viewerから

	<username>@<host>.local:1 で接続できることを確認
この時点ではデスクトップが立ち上がらず、NVIDIAマークが表示される状態で止まる<br>
VNC自動起動用設定<br>

	sudo crontab -e
末尾に以下のコマンドを追加（解像度は適宜調整）<br>

	@reboot su - <user> -c '/usr/bin/tightvncserver -geometry 1920x1080'
デスクトップを表示される設定を下記ファイル<br>

	~/.vnc/xstartup　に
	startlxde &
を追加し、再起動　[(参考記事)](https://forums.developer.nvidia.com/t/tightvnc-with-desktop-environment-on-jetson-nano-2gb-in-headless-mode/163593)

デスクトップからスクリーンセーバーをOFFにする<br>

### ・セキュリティ対策
ルートパスワード設定<br>

	sudo passwd root

→　パスワード設定<br>

### ・パスワード無しでsudo実行可能とする
	sudo visudo　→エディタ開く
一番下の行に以下のように入力<br>

	<username> ALL=(ALL) NOPASSWD: ALL

### ・Dockerをsudoなしで立ち上げられる設定にする
	sudo gpasswd -a $USER docker
（確認方法）<br>

	groups <username>

### ・Jetson GPIO Libraryパッケージの導入
	git clone https://github.com/NVIDIA/jetson-gpio.git
	cd jetson-gpio
	sudo python3 setup.py install
	sudo groupadd -f -r gpio
	sudo usermod -a -G gpio <user>
	sudo cp lib/python/Jetson/GPIO/99-gpio.rules /etc/udev/rules.d/
	sudo udevadm control —reload-rules && sudo udevadm trigger
反映のため再起動<br>

### ・nanoエディタインストール（必須でない）
sudo apt install nano<br>

### ・sublimeインストール（ライセンス保持者向け、必須でない）
linux版は下記の手順でインストール(公式より）<br>
・GPGキーのインストール<br>

	wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add - 
・apt-getでhttpsを使用できるようにする。<br>

	sudo apt-get install apt-transport-https
・使用するチャンネル？を選択<br>
・stable<br>

	echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
・dev<br>

	echo "deb https://download.sublimetext.com/ apt/dev/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
・インストール<br>

	sudo apt-get update
	sudo apt-get install sublime-text
・起動するとライセンス情報を要求されるため、ライセンス購入時に送られたメールからコピーしたライセンス（テキスト）情報をペースト（ライセンス情報はテキストファイルにコピペし、sftpで送付（下記のsubl_licenseがテキストファイル名になる）<br>

	(ファイルがある端末からsftp接続後）sftp put subl_license
