# Lab手順

## 実施環境
* 事前にラボ環境へのInviteを行っておりますので、メールをご確認ください
* 利用するコマンド： git , docker, docker-compose , jq , sudo, curl
* NGINX Trialライセンスの取得、ラボ実施ユーザのHome Directryへ配置

## UDF コンポーネントへの接続
### Windows Jump HostへのRDP接続

指定のホストを適切にログインするように適切に変更する


Windows Jump HostからCLIの操作を行う場合、以下タブからRDP Clientファイルをダウンロードいただき接続ください
<br><img src="https://user-images.githubusercontent.com/43058573/121283531-417b5f80-c916-11eb-9ff9-8d0e7e8f1aad.png" alt="RDP" width="200"><br>

Windows Jump Hostへログインいただくと、SSHを実行するバッチファイルがありますので、そちらをダブルクリックしDocker_HOSTへ接続ください
<br><img src="https://user-images.githubusercontent.com/43058573/121170154-c8392980-c88f-11eb-9879-f7bb9c4f13a3.png" alt="ssh"><br>
RDPのUser名、パスワードはDETAILSをクリックし、GeneralのタブのCredentialsの項目を参照ください
<br><img src="https://user-images.githubusercontent.com/43058573/121283534-417b5f80-c916-11eb-88af-9f95c2ced284.png" alt="DETAILS" width="200"><br>
<img src="https://user-images.githubusercontent.com/43058573/121283535-4213f600-c916-11eb-8a89-67362d7a340b.png" alt="Generals" width="300"><br>

### Linux Hostへの接続
Docker Hostへの接続は以下メニューを開き利用ください
<br><img src="https://user-images.githubusercontent.com/43058573/121283528-404a3280-c916-11eb-8a60-bfde13129dfc.png" alt="Docker Menu" width="200"><br>
Docker HOSTへのSSH接続は、Jump Host経由　または、SSH鍵認証を用いて接続可能です。SSH鍵の登録手順は以下を参照ください
***SSH鍵を登録頂いていない場合、SSHはグレーアウトします***
<br><a href="https://github.com/hiropo20/partner_nap_workshop_secure/blob/main/UDF_SSH_Key.pdf">UDF LAB SSH鍵登録マニュアル</a> (ラボ実施時閲覧可に変更します)<br>

## ユーザの確認
実行ユーザの確認
```
whoami

出力結果がcentosであることを確認してください。
webshell を利用してrootで操作している場合には、su - centos でユーザを切り替えてください
```
## Git clone

ラボで必要なファイルをGitHubから取得
```
cd ~/
git clone https://github.com/hiropo20/nap-partner-campaign_no3.git
```

適切にURLを変更

## NGINX Plusのインストール
### 1. Install NGINX Plus 

以下の手順に従ってNGINX Plus をインストール
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#installing-nginx-plus-on-ubuntu

#### NGINX Licenseファイルのコピー
ライセンスファイルをコピーしてください
ファイルが配置されていない場合、トライアルを申請し証明書と鍵を取得してください

```
sudo mkdir -p /etc/ssl/nginx
sudo cp ~/nginx-repo.crt /etc/ssl/nginx/
sudo cp ~/nginx-repo.key /etc/ssl/nginx/
```

#### コマンドの実行
NGINX、App Protect WAF と App Protect DoS のリポジトリに利用する鍵を取得します
```
sudo wget https://cs.nginx.com/static/keys/nginx_signing.key && sudo apt-key add nginx_signing.key

sudo wget https://cs.nginx.com/static/keys/app-protect-security-updates.key && sudo apt-key add app-protect-security-updates.key

```
必要となるパッケージをインストールします
```
sudo apt-get install -y apt-transport-https lsb-release ca-certificates wget
```
レポジトリの情報を追加します
```
# NGINX Plusのレポジトリ情報
printf "deb https://pkgs.nginx.com/plus/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nginx-plus.list

# NGINX App Protectのレポジトリ情報
printf "deb https://pkgs.nginx.com/app-protect/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nginx-app-protect.list

printf "deb https://pkgs.nginx.com/app-protect-security-updates/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee -a /etc/apt/sources.list.d/nginx-app-protect.list

# NGINX App Protect DoSのレポジトリ情報
printf "deb https://pkgs.nginx.com/app-protect-dos/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nginx-app-protect-dos.list

# Mod Securityのレポジトリ情報
printf "deb https://pkgs.nginx.com/modsecurity/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nginx-modsecurity.list
```
aptコマンドの設定情報を取得します
```
sudo wget -P /etc/apt/apt.conf.d https://cs.nginx.com/static/files/90pkgs-nginx
```
パッケージ情報を更新します
```
sudo apt-get update
```

### NGINX パッケージのインストール
```
sudo apt-get install -y nginx-plus
sudo apt-get install -y app-protect app-protect-attack-signatures
sudo apt-get install -y app-protect-dos
```
インスールしたパッケージの情報の確認
参考となる記事はこちらです。K72015934: Display the NGINX software version　https://support.f5.com/csp/article/K72015934
```
nginx -v
```
NGINX App Protect のVersion
```
cat /opt/app_protect/VERSION
```
NGINX App Protect DoS のVersion
```
admd -v
```


```
ubuntu@ip-10-1-1-7:~$ dpkg-query -l | grep app-protect
ii  app-protect                        25+3.671.0-1~focal                    amd64        App-Protect package for Nginx Plus, Includes all of the default files and examples. Nginx App Protect provides web application firewall (WAF) security protection for your web applications, including OWASP Top 10 attacks.
ii  app-protect-attack-signatures      2021.11.16-1~focal                    amd64        Attack Signature Updates for App-Protect
ii  app-protect-common                 8.12.1-1~focal                        amd64        NGINX App Protect
ii  app-protect-compiler               8.12.1-1~focal                        amd64        Control-plane(aka CP) for waf-general debian
ii  app-protect-dos                    25+2.0.1-1~focal                      amd64        Nginx DoS protection
ii  app-protect-engine                 8.12.1-1~focal                        amd64        NGINX App Protect
ii  app-protect-plugin                 3.671.0-1~focal                       amd64        NGINX App Protect plugin


ubuntu@ip-10-1-1-7:~$ dpkg-query -l | grep nginx-plus
ii  nginx-plus                         25-1~focal                            amd64        NGINX Plus, provided by Nginx, Inc.
ii  nginx-plus-module-appprotect       25+3.671.0-1~focal                    amd64        NGINX Plus app protect dynamic module version 3.671.0
ii  nginx-plus-module-appprotectdos    25+2.0.1-1~focal                      amd64        NGINX Plus appprotectdos dynamic module

```
### 2. ステータスの確認
#### プロセスの確認
NGINX の停止・起動
```
sudo service nginx stop
sudo service nginx start
```
NGINX のstatus
```
sudo service nginx status
● nginx.service - NGINX Plus - high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-11-22 10:12:55 UTC; 11s ago
       Docs: https://www.nginx.com/resources/
    Process: 9126 ExecStartPre=/usr/lib/nginx-plus/check-subscription (code=exited, status=0/SUCCESS)
    Process: 9146 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
   Main PID: 9147 (nginx)
      Tasks: 3 (limit: 2327)
     Memory: 2.6M
     CGroup: /system.slice/nginx.service
             ├─9147 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
             ├─9148 nginx: worker process
             └─9149 nginx: worker process

Nov 22 10:12:55 ip-10-1-1-7 systemd[1]: Starting NGINX Plus - high performance web server...
Nov 22 10:12:55 ip-10-1-1-7 systemd[1]: nginx.service: Can't open PID file /run/nginx.pid (yet?) after start: Operation not permitted
Nov 22 10:12:55 ip-10-1-1-7 systemd[1]: Started NGINX Plus - high performance web server.
```

pidファイルの配置場所の確認
```
ubuntu@ip-10-1-1-7:~$ grep pid /etc/nginx/nginx.conf
pid        /var/run/nginx.pid;
```

pidの内容確認
```
ubuntu@ip-10-1-1-7:~$ cat /var/run/nginx.pid
9147
```

論理コア数の確認
```
ubuntu@ip-10-1-1-7:~$ grep processor /proc/cpuinfo | wc -l
2
```

NGINX Processの確認
NGINXはMaster Processと通信制御を行うWorker Processに分かれる。Worker ProcessはCPU Core数の数起動し、並列処理を行う設定となっている。
Master ProcessのPIDがPIDファイルに記載されている内容と一致していることを確認する
また、Worker ProcessがCPU Core数の数だけ起動していることを確認する
```
ubuntu@ip-10-1-1-7:~$ ps aux | grep nginx
nginx       9122  0.0  0.0   2616   608 ?        Ss   10:12   0:00 /bin/sh -c usr/share/ts/bin/bd-socket-plugin tmm_count 4 proc_cpuinfo_cpu_mhz 2000000 total_xml_memory 307200000 total_umu_max_size 3129344 sys_max_account_id 1024 no_static_config 2>&1 >> /var/log/app_protect/bd-socket-plugin.log
nginx       9123  0.3  3.0 385260 61592 ?        Sl   10:12   0:00 usr/share/ts/bin/bd-socket-plugin tmm_count 4 proc_cpuinfo_cpu_mhz 2000000 total_xml_memory 307200000 total_umu_max_size 3129344 sys_max_account_id 1024 no_static_config
nginx       9125  0.0  0.0   2616   608 ?        Ss   10:12   0:00 /bin/sh -c /usr/bin/admd -d --log info 2>&1 > /var/log/adm/admd.log
nginx       9127  0.5  2.5 799208 50732 ?        Sl   10:12   0:00 /usr/bin/admd -d --log info
root        9147  0.0  0.0   9136   892 ?        Ss   10:12   0:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx       9148  0.0  0.1   9764  3528 ?        S    10:12   0:00 nginx: worker process
nginx       9149  0.0  0.1   9764  3528 ?        S    10:12   0:00 nginx: worker process

```

#### ディレクトリの移動
```
cd ~/nap-partner-campaign/nplus-container
```
適切にパスを変更

#### Install の実施



Docker composeは複数のコンテナを利用するDockerアプリケーションを定義するツールです。Docker Composeを利用する場合には、YAMLファイルにアプリケーションとして実行したい内容を記述します。その後、コマンド実行時にYAMLファイルを指定することで、指定の通りアプリケーションを起動することが可能です
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
Docker Composeコマンドに適切な権限を付与
```
sudo chmod +x /usr/local/bin/docker-compose
```
Docker Composeコマンドの動作を確認
```
docker-compose --version
```
### 2. ContainerでNGINXを起動

Docker Container ImageをPull
```
docker pull nginx
```
NGINX Containerの起動
```
docker run --name ngx-docker -p 80:80 -d nginx
```
Web Serverの動作確認
```
curl http://localhost:80/
```
以下のコマンドを参考にContainerの動作状況などを確認
docker ps コマンドの結果を確認し、CONTAINER_IDの表示結果を参考に以下を実施
```
docker ps
docker exec -it <<CONTAINER_ID>>  bash
```
上記コマンドと同様の内容を以下コマンドで実行できることを確認
```
docker ps -aqf "name=ngx-docker"
docker exec -it `docker ps -aqf "name=ngx-docker"`  bash
```
Containerの停止
```
docker stop ngx-docker
```
Containerの削除
```
docker rm ngx-docker
```
新しいコンテナの起動
```
docker run --name ngx-docker-new -p 80:80 -d nginx
```
新しいコンテナの停止
```
docker stop ngx-docker-new
```
新しいコンテナの削除
```
docker rm ngx-docker-new
```
### 3.  Docker Composeを利用してアプリケーションを実行
Deployment file:
https://github.com/mcheo-nginx/handson_training/tree/main/basic_docker

GitHubよりファイルを取得
```
cd ~
git clone https://github.com/mcheo-nginx/handson_training.git
cd handson_training/basic_docker/
```
デプロイの実行
```
docker-compose -f docker-compose.yml up -d
```
デプロイ結果の確認
```
curl -sv localhost:8000 | head
```
設定の変更。nginx.conf ファイルの変更。(proxy_passのコメントアウト、“return” directiveの追加)
```
vi ./nginx.conf
```
設定変更内容
```
        location / {
                      #proxy_pass http://backend;
                                  return 200 "Container Lab\n";
                                          }
                                          ```
                                          デプロイしたアプリケーションの再起動
                                          ```
                                          docker-compose -f docker-compose.yml restart
                                          ```
                                          再度、デプロイ結果の確認
                                          ```
                                          curl -sv localhost:8000 | head
                                          ```


### 参考: Docker cheatsheet for beginners
Dockerコマンド参考情報。以下の内容やInternetの情報を参考にDockerコマンドを実行してください  

Docker machine commands  
* Create new: docker-machine create MACHINE  
* List all: docker-machine ls  
* Show env: docker-machine env default  
* Use: eval "$(docker-machine env default)"
* Unset: docker-machine env -u
* Unset: eval $(docker-machine env -u)

Docker image commands
* Download: docker pull IMAGE[:TAG]
* Build from local Dockerfile: docker build -t TAG .
* Build with user and tag: docker build -t USER/IMAGE:TAG .
* List: docker image ls or docker images
* List all: docker image ls -a or docker images -a
* Remove (image or tag): docker image rm IMAGE or docker rmi IMAGE
* Remove all dangling (nameless): docker image prune
* Remove all unused: docker image prune -a
* Remove all: docker rmi $(docker images -aq)
* Tag: docker tag IMAGE TAG
* Save to file:docker save IMAGE > FILE
* Load from file: docker load -i FILE

Docker container commands
* Run from image: docker run IMAGE
* Run with name: docker run --name NAME IMAGE
* Map a port: docker run -p HOST:CONTAINER IMAGE
* Map all ports: docker run -P IMAGE
* Start in background: docker run -d IMAGE
* Set hostname: docker run --hostname NAME IMAGE
* Set domain: docker run --add-host HOSTNAME:IP IMAGE
* Map local directory: docker run -v HOST:TARGET IMAGE
* Change entrypoint: docker run -it --entrypoint NAME IMAGE
* List running: docker ps or docker container ls
* List all: docker ps -a or docker container ls -a
* Stop: docker stop ID or docker container stop ID
* Start: docker start ID
* Stop all: docker stop $(docker ps -aq)
* Kill (force stop): docker kill ID or docker container kill ID
* Remove: docker rm ID or docker container rm ID
* Remove running: docker rm -f ID
* Remove all stopped: docker container prune
* Remove all: docker rm $(docker ps -aq)
* Rename: docker rename OLD NEW
* Create image from container: docker commit ID
* Show modified files: docker diff ID
* Show mapped ports: docker port ID
* Copy from container: docker cp ID:SOURCE TARGET
* Copy to container docker cp TARGET ID:SOURCE
* Show logs: docker logs ID
* Show processes: docker top ID
* Start shell: docker exec -it ID bash

Other useful Docker commands
* Log in: docker login
* Run compose file: docker-compose
* Get info about image: docker inspect IMAGE
* Show stats of running containers: docker stats
* Show version: docker version

