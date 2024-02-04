# WSL2でDockerをsystemd経由で起動する環境の構築

## Dockerのインストール
```bash
# パッケージ更新
sudo apt -y update

# 必要なパッケージの追加
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# 証明書ダウンロード
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 証明書登録
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# パッケージ更新
sudo apt -y update

# Dockerのインストール
sudo apt -y install docker-ce docker-ce-cli containerd.io

# 操作ユーザーをグループに追加
sudo gpasswd -a $USER docker

# 一旦ターミナルを閉じる
exit
```

## ネットワーク設定
```bash
# iptable(v6含め)の利用するバージョンを落とす
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```


## Dockerを自動起動
```bash
# systemdを起動
sudo /usr/libexec/wsl-systemd
# rootユーザーに変更
sudo /usr/libexec/nslogin
# dockerの起動
systemctl start docker
# dockerを自動起動設定
systemctl enable docker
```

## vimインストール
```bash
sudo apt install -y vim
```

## 起動時に systemd を起動する
```bash
sudo vi /etc/profile.d/00-wsl-systemd.sh
```
###### ※編集内容
```bash:/etc/profile.d/00-wsl-systemd.sh
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# ↓追加
SYSTEMD_PID=$(ps -fe | grep '/lib/systemd/systemd --unit=multi-user.target$' | grep -v unshare | awk '{print $2}')

if [ -z "$SYSTEMD_PID" ]; then
    sudo /usr/libexec/wsl-systemd
fi
# ↑追加
```

## systemd を sudo なしで実行できるようにする
```bash
sudo vi /etc/sudoers.d/wsl-systemd
```
###### ※編集内容
```bash:/etc/sudoers.d/wsl-systemd
%adm ALL=(ALL:ALL) NOPASSWD: /usr/libexec/wsl-systemd
```

# nslogin を自動で実行
```bash
vi ~/.bashrc
```
###### ※編集内容(末尾に追加)
```bash:~/.bashrc
SYSTEMD_PID=$(ps -fe | grep '/lib/systemd/systemd --unit=multi-user.target$' | grep -v unshare | awk '{print $2}')
if [ -n "$SYSTEMD_PID" ] && [ "$SYSTEMD_PID" != "1" ]; then
    if [ -z "$1" ]; then
        /usr/libexec/nslogin
    exit
    fi
fi
```



## docker compose のインストール
```bash
# フォルダ作成＆移動
sudo mkdir -p /usr/local/libexec/docker/cli-plugins
cd /usr/local/libexec/docker/cli-plugins

# https://github.com/docker/compose/releases にアクセスして最新を調べて、バージョンを調査
DOCKER_COMPOSE_VERSION=v2.24.5

# compose の実行ファイルを取得
sudo curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-linux-x86_64 -o docker-compose

# 権限変更
sudo chmod +x docker-compose
```
