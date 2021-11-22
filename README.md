# Lab手順

## メモ
・jq install
・/etc/hosts 10.1.1.8 backend1 backend2 backend3 backend4


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

## 対象のホストにログイン
実行ユーザの確認
```
whoami

出力結果がcentosであることを確認してください。
webshell を利用してrootで操作している場合には、su - centos でユーザを切り替えてください
```

## 1. NGINX Plusのインストール
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

#### NGINX パッケージのインストール
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

## 2. NGINXの基礎
### 1. Directive / Block
P32

### 2. Configの階層構造
P42,41

## 3. 基本的な動作の確認
### 0. 事前ファイルの取得
ラボで必要なファイルをGitHubから取得
```
sudo su - 
cd ~/
git clone https://github.com/hiropo20/back-to-basic_plus/


適切にURLを変更

```
### 1. 設定のテスト、設定の反映

```
# cd /etc/nginx/conf.d/
cp ~/back-to-basic_plus/lab/m1-1_demo.conf default.conf

# cp ~/back-to-basic_plus/lab/m1-1_demo.conf default.conf
# cat default.conf
server {
    # you need to add ; at end of listen directive.
    listen       81
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

設定のテスト結果を確認してください。
-tと-T二つのオプションを実行し、違いを確認してください
```
# nginx -h
nginx version: nginx/1.21.3 (nginx-plus-r25)
Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
             [-e filename] [-c filename] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /etc/nginx/)
  -e filename   : set error log file (default: /var/log/nginx/error.log)
  -c filename   : set configuration file (default: /etc/nginx/nginx.conf)
  -g directives : set global directives out of configuration file

# nginx -t
nginx: [emerg] invalid parameter "server_name" in /etc/nginx/conf.d/default.conf:4
nginx: configuration file /etc/nginx/nginx.conf test failed
```

"server_name" directive でエラーとなっていることがわかります。
これは、その一つ前の行が正しく「；(セミコロン)」で終わっていないことが問題となります。
エディタで設定ファイルを開き修正してください
```
vi default.conf

# listen directiveの文末に ; を追加
---
[変更前]    listen       81
[変更後]    listen       81;
```

再度テストを実行してください
```
# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
# nginx -T
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
# configuration file /etc/nginx/nginx.conf:

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}


※省略※
# configuration file /etc/nginx/conf.d/default.conf:
server {
    # you need to add ; at end of listen directive.
    listen       81;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}

```

設定の読み込み、動作確認
正しく Port 81 でListenしていることを確認してください
```
# nginx -s reload
# ss -anp | grep nginx | grep LISTEN
tcp    LISTEN  0       511                                              0.0.0.0:81                                                0.0.0.0:*                      users:(("nginx",pid=9341,fd=12),("nginx",pid=9340,fd=12),("nginx",pid=9147,fd=12))

# curl -s localhost:81 | grep title
<title>Welcome to nginx!</title>

```

### 2. 設定の継承
ラボで使用するファイルをコピー
```
cp -r ~/back-to-basic_plus/html .
# cp ~/back-to-basic_plus/lab/m2-1_demo.conf default.conf
```

設定ファイルの確認してください。
本設定では、indexがポイントとなります。

listen 80では、indexを個別に記述をしていません。
listen 8080では、indexとして main.html を指定しています。
また、それぞれ root の記述方法が異なっています。
```
# cat default.conf
index index.html;
server {
        listen 80;
        root conf.d/html;
}
server {
        listen 8080;
        root /etc/nginx/conf.d/html;
        index main.html;
}
```
設定を反映し、これらがどのように動作するのか見てみましょう。
```
# nginx -s reload
# ss -anp | grep nginx | grep LISTEN
tcp    LISTEN  0       511                                              0.0.0.0:8080                                              0.0.0.0:*                      users:(("nginx",pid=9392,fd=9),("nginx",pid=9391,fd=9),("nginx",pid=9147,fd=9))
tcp    LISTEN  0       511                                              0.0.0.0:80                                                0.0.0.0:*                      users:(("nginx",pid=9392,fd=8),("nginx",pid=9391,fd=8),("nginx",pid=9147,fd=8))


# curl -s localhost:80 | grep path
        <h2>path: html/index.html</h2>
# curl -s localhost:8080 | grep path
        <h2>path: html/main.html</h2>

```


### 3. Server Directive
ラボで使用するファイルをコピーします
```
# cp ~/back-to-basic_plus/lab/m3-1_demo.conf default.conf
```

設定内容を確認し、反映します
```
# cat default.conf
server {

}

# nginx -s reload
# ss -anp | grep nginx | grep LISTEN
tcp    LISTEN  0       511                                              0.0.0.0:80                                                0.0.0.0:*                      users:(("nginx",pid=9445,fd=8),("nginx",pid=9444,fd=8),("nginx",pid=9147,fd=8))

```
設定が反映され、80でListenしていることが確認できます
curlコマンドで結果を確認してみます
```
# curl localhost:80
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.21.3</center>
</body>

```
404エラーとなりました。これはどこを参照しているのでしょうか。
各directiveのdefaultパラメータを確認してください

root
http://nginx.org/en/docs/http/ngx_http_core_module.html#root
index
http://nginx.org/en/docs/http/ngx_http_index_module.html#index
listen
http://nginx.org/en/docs/http/ngx_http_core_module.html#listen

これらの内容より、server directiveに設定を記述しない場合にも、defaultのパラメータで動作していることが確認できます。

それでは対象となるディレクトリにファイルをコピーします

```
# mkdir ../html
# cp ~/back-to-basic_plus/html/m3-1_index.html ../html/index.html
```

htmlファイルを配置しました。
設定ファイルに変更は加えておりませんので、再度curlコマンドで結果を確認します
```
# curl -s localhost:80 | grep default
        <h2>This is default html file path</h2>

```
今度は正しく結果が表示されました
このようにdefeaultパラメータの動作を確認できました



### 4. 複数のListen Directiveを指定
ラボで使用するファイルをコピーします
```
# cp ~/back-to-basic_plus/lab/m3-2_demo.conf default.conf
```

設定内容を確認し、反映します
```
# cat default.conf
# server {
#    ## no listen directive
# }

server {
    listen 127.0.0.1:8080;
}

server {
    listen 127.0.0.2;
}

server {
    listen 8081;
}

server {
    listen unix:/var/run/nginx.sock;
}


# service nginx restart

```
設定で指定したポート番号やソケットでListenしていることを確認してください。
（正しく設定が読み込めない場合は、再度上記コマンドにて設定を読み込んでください)

```
# ls /var/run/nginx.sock
/var/run/nginx.sock

# ss -anp | grep nginx | grep LISTEN
u_str LISTEN    0      511                                  /var/run/nginx.sock 60394                                                   * 0                      users:(("nginx",pid=9947,fd=9),("nginx",pid=9946,fd=9),("nginx",pid=9945,fd=9))
tcp   LISTEN    0      511                                            127.0.0.2:80                                                0.0.0.0:*                      users:(("nginx",pid=9947,fd=7),("nginx",pid=9946,fd=7),("nginx",pid=9945,fd=7))
tcp   LISTEN    0      511                                            127.0.0.1:8080                                              0.0.0.0:*                      users:(("nginx",pid=9947,fd=6),("nginx",pid=9946,fd=6),("nginx",pid=9945,fd=6))
tcp   LISTEN    0      511                                              0.0.0.0:8081                                              0.0.0.0:*                      users:(("nginx",pid=9947,fd=8),("nginx",pid=9946,fd=8),("nginx",pid=9945,fd=8))
```

それぞれ Listen している内容に対して接続できることを確認してください


```
# curl -s 127.0.0.1:8080 | grep default
        <h2>This is default html file path</h2>
# curl -s 127.0.0.2:80 | grep default
        <h2>This is default html file path</h2>
# curl -s 127.0.0.1:8081 | grep default
        <h2>This is default html file path</h2>

# curl -s --unix-socket /var/run/nginx.sock http: | grep default
        <h2>This is default html file path</h2>

```

socketを削除し、NGINXが起動することを確認します

```
rm /var/run/nginx/sock
rm default.conf
service nginx restart
```

### 5.  複数のserver_nameを指定
ラボで使用するファイルをコピーします
```
# cp ~/back-to-basic_plus/lab/m3-3_demo.conf default.conf
```

設定内容を確認し、反映します
```
# cat default.conf

server {
    server_name example.com;
    return 200 "example.com\n";
}

server {
    server_name host1.example.com;
    return 200 "host1.example.com\n";
}

server {
        server_name www.example.*;
    return 200 "www.example.*\n";
}
server{
        server_name *.org;
    return 200 "*.org\n";
}
server {
        server_name *.example.org;
    return 200 "*.example.org\n";
}

server {
        listen 80;
        server_name ~^(www2|host2).*\.example\.com$;
   return 200 "~^(www2|host2).*\.example\.com\n";
}
server {
        listen 80;
        server_name ~^.*\.example\..*$;
    return 200 "~^.*\.example\..*\n";
}
server {
        listen 80;
        server_name ~^(host2|host3).*\.example\.com$;
    return 200 "~^(host2|host3).*\.example\.com\n";
}
# nginx -s reload
```
server_nameの処理順序は以下です
1. 文字列の完全一致
1. Wild Cardを用いた文字列の前方一致
1. Wild Cardを用いた文字列の後方一致
1. 正規表現のはじめの一致

以下のコマンドを実行し結果を確認します。
どのような処理が行われているか確認してください。
```
・完全一致する結果を確認
# curl localhost -H 'Host:host1.example.com'
host1.example.com

・Wild Cardの前方一致する結果を確認
# curl localhost -H 'Host:www.example.co.jp'
www.example.*

・正規表現のはじめに一致する結果を確認
# curl localhost -H 'Host:host2.example.co.jp'
~^.*\.example\..*

```


### 6. 複数のlocationを指定
ラボで使用するファイルをコピーします
```
# cp ~/back-to-basic_plus/lab/m4-1_demo.conf default.conf
```

設定内容を確認し、反映します
```
# cat default.conf
server {
   listen 80;
   location / {
      return 200 "LOCATION: / , URI: $request_uri, PORT: $server_port\n";
   }
   location ~* \.(php|html)$ {
      return 200 "LOCATION: ~* \.(php|html), URI: $request_uri, PORT: $server_port\n";
   }
   location ^~ /app1 {
      return 200 "LOCATION: ^~ /app1, URI: $request_uri, PORT: $server_port\n";
   }
   location ~* /app1/.*\.(php|html)$ {
      return 200 "LOCATION: ~* /app1/.*\.(php|html), URI: $request_uri, PORT: $server_port\n";
   }
   location = /app1/index.php {
           return 200 "LOCATION: = /app1/index.php, URI: $request_uri, PORT: $server_port\n";
   }
   location  /app2 {
      return 200 "LOCATION: /app2, URI: $request_uri, PORT: $server_port\n";
   }
   location ~* /app2/.*\.(php|html)$ {
      return 200 "LOCATION: ~* /app2/.*\.(php|html), URI: $request_uri, PORT: $server_port\n";
   }

}

# nginx -s reload
```

locationの処理順序は以下となります。
P18,19

期待した結果となることを確認してください。
```
# curl http://localhost/app1/index.html
LOCATION: ^~ /app1, URI: /app1/index.html, PORT: 80


# curl http://localhost/app2/index.html
LOCATION: ~* \.(php|html), URI: /app2/index.html, PORT: 80

```

### 7. Proxy
ラボで使用するファイルをコピーします
```
# cp ~/back-to-basic_plus/lab/m5-1_demo.conf default.conf
```

設定内容を確認し、反映します
```
# cat default.conf

server {
    listen 80;
    location /app1 {
        proxy_pass http://backend1:81/otherapp;
    }
    location /app2 {
        proxy_pass http://backend1:81;
    }

}

# nginx -s reload
```

以下のコマンドを実行し結果を確認します。
どのような処理が行われているか確認してください。
```
# curl -s localhost/app1/usr1/index.php | jq .
{
  "request_uri": "/otherapp/usr1/index.php",
  "server_addr": "10.1.1.8",
  "server_port": "81"
}
# curl -s localhost/app2/usr1/index.php | jq .
{
  "request_uri": "/app2/usr1/index.php",
  "server_addr": "10.1.1.8",
  "server_port": "81"
}

```


### 8. Load Balancing
ラボで使用するファイルをコピーします
```
cp -r ~/back-to-basic_plus/html .
# cp ~/back-to-basic_plus/lab/m6-1_demo.conf default.conf
# cp ~/back-to-basic_plus/lab/m6-1_plus_api.conf plus_api.conf
```

設定内容を確認し、反映します
```
# cat default.conf

upstream server_group {
    zone backend 64k;
    server backend1:81 weight=1;
    server backend2:82 weight=2;
}
server {
    listen 80;
    location / {
        proxy_pass http://server_group;
    }
}
# cat plus_api.conf
server {
    listen 8888;
    access_log /var/log/nginx/mng_access.log;

    location /api {
        api write=on;
        # directives limiting access to the API
    }

    location = /dashboard.html {
        root   /usr/share/nginx/html;
    }

}
# nginx -s reload
```
ブラウザでNGINX Plus Dashboardを開きます
（ブラウザでubuntu01のDashboardを開きます)

以下コマンドを実行し、適切に分散されることを確認します。
```
# for i in {1..9}; do echo "==$i==" ; curl -s localhost | jq . ; sleep 1 ; done

==1==
{
  "request_uri": "/",
  "server_addr": "10.1.1.8",
  "server_port": "82"
}
※省略※
==9==
{
  "request_uri": "/",
  "server_addr": "10.1.1.8",
  "server_port": "82"
}
```
Dashboardの結果が適切なweightで分散されていることを確認してください。


### 9. トラフィックの暗号化
ラボで使用するファイルをコピーします
```
# cp -r ~/back-to-basic_plus/ssl .
# cp ~/back-to-basic_plus/lab/m8-1_demo.conf default.conf
```

設定内容を確認し、反映します
```
# cat default.conf
server {
    listen 80;
        listen 443 ssl;
        ssl_certificate_key conf.d/ssl/nginx-ecc-p256.key;
        ssl_certificate conf.d/ssl/nginx-ecc-p256.pem;
        location / {
                proxy_pass http://backend1:81;
        }
}
# nginx -s reload
```

以下のコマンドを実行し結果を確認します。

```
# curl -v http://localhost
*   Trying 127.0.0.1:80...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.21.3
< Date: Mon, 22 Nov 2021 15:05:35 GMT
< Content-Type: application/octet-stream
< Content-Length: 65
< Connection: keep-alive
<
* Connection #0 to host localhost left intact
{ "request_uri": "/","server_addr":"10.1.1.8","server_port":"81"}


# curl -kv https://localhost
*   Trying 127.0.0.1:443...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-ECDSA-AES256-GCM-SHA384
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: CN=localhost
*  start date: Mar 24 01:04:24 2021 GMT
*  expire date: Apr 23 01:04:24 2021 GMT
*  issuer: CN=localhost
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
> GET / HTTP/1.1
> Host: localhost
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.21.3
< Date: Mon, 22 Nov 2021 15:05:49 GMT
< Content-Type: application/octet-stream
< Content-Length: 65
< Connection: keep-alive
<
* Connection #0 to host localhost left intact
{ "request_uri": "/","server_addr":"10.1.1.8","server_port":"81"}

```
