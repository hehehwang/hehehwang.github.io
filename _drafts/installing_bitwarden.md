---
title: bitwarden 설치기
categories:
  - develop
  - devnote
tags:
  - installation
  - worklog
---

# 개요
raspberry pi에 [bitwarden](https://bitwarden.com/)을 [설치](https://bitwarden.com/help/install-on-premise-linux/)하면서 있었던 일들을 기록한다.

# 1일차
## Docker 설치
메뉴얼을 읽어보니 bitwarden은 docker로 돌아간다는 듯 하니 docker를 설치해보자.

* [설치 링크](https://docs.docker.com/engine/install/debian/)

근데 메뉴얼에 있는 절차를 자동으로 해주는 비공식적인 신박한 사이트를 찾았다.

```bash
curl -sSL https://get.docker.com | sh
```

[공식 설명](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)

그래.. 진작에 이런걸 지원했어야지...

그 외에 절차는 여기를 참조했다.

* [Installing Docker on the Raspberry Pi
](https://pimylifeup.com/raspberry-pi-docker/)

대충 보아하니, sudo를 안넣고도 docker를 굴릴 수 있도록 해주는 절차인듯 싶다.

```bash
sudo usermod -aG docker pi
logout
groups
```
```bash
pi adm dialout cdrom sudo audio video plugdev games users input render netdev lpadmin docker gpio i2c spi
```

ㅋㅋㅅㅂ 도커가 안깔린다

왜지

도커 애초에 태생이 리눅스 아닌가?

심지어 삭제도 깔끔하게 안돼서 고생했다.
(삭제해도 /usr/bin/docker가 남음)

dpkg exit code (1)랬나 당연히 exit code가 1이겠지 에러인데

아무튼 에러메세지에서 아무런 단서를 찾지 못했어서
```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
이걸로 다시 설치를 해봤다.

그랬더니

```
dpkg: error processing package docker-ce (--configure):
 installed docker-ce package post-installation script subprocess returned error exit status 1
 ```

 `journelctl -xe`를 쳐보라길래 쳐봤다

 ```
 ░░ The job identifier is 2933 and the job result is done.
Mar 17 22:01:18 raspberrypi systemd[1]: docker.service: Start request repeated too quickly.
Mar 17 22:01:18 raspberrypi systemd[1]: docker.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░
░░ The unit docker.service has entered the 'failed' state with result 'exit-code'.
Mar 17 22:01:18 raspberrypi systemd[1]: Failed to start Docker Application Container Engine.
░░ Subject: A start job for unit docker.service has failed
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░
░░ A start job for unit docker.service has finished with a failure.
░░
░░ The job identifier is 2933 and the job result is failed.
Mar 17 22:01:18 raspberrypi systemd[1]: docker.socket: Failed with result 'service-start-limit>
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░
░░ The unit docker.socket has entered the 'failed' state with result 'service-start-limit-hit'.
```

'service-start-limit-hit'?

`sytemsctl`쪽에선 이런다.
```
pi@raspberrypi:~ $ systemctl status docker.service
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Thu 2022-03-17 22:01:18 KST; 2min 24s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
    Process: 8433 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.>
   Main PID: 8433 (code=exited, status=1/FAILURE)
        CPU: 429ms

Mar 17 22:01:18 raspberrypi systemd[1]: docker.service: Scheduled restart job, restart counter>
Mar 17 22:01:18 raspberrypi systemd[1]: Stopped Docker Application Container Engine.
Mar 17 22:01:18 raspberrypi systemd[1]: docker.service: Start request repeated too quickly.
Mar 17 22:01:18 raspberrypi systemd[1]: docker.service: Failed with result 'exit-code'.
Mar 17 22:01:18 raspberrypi systemd[1]: Failed to start Docker Application Container Engine.
```

그러다 잘 안돼서 `sudo reboot`를 갈기고 `sudo apt-get docker-ce`를 했는데

잘 된다..

?

`upgrade`를 한 후에 재부팅도 잘 했는데?

잘 모르겠다. 아무튼 됐으니 된거지

와하하


자야겠다.


# 2일차

## vaultwarden image 올리기
다시 [가이드](https://pimylifeup.com/raspberry-pi-bitwarden/)로 돌아와서 살펴보니, 원본 bitwarden 대신 rust판으로 만들어진 vaultwarden을 이용한다고 되어있다.

docker의 gui 관리자인 Portainer을 사용하는 가이드 대신, cli를 이용해서 설치하는 쪽으로 진행

```bash
docker pull vaultwarden/server:latest
```

```bash
sudo docker run -d --name bitwarden \
    --restart=always \
    -v /bw-data/:/data/ \
    -p 127.0.0.1:8080:80 \
    -p 127.0.0.1:3012:3012 \
    vaultwarden/server:latest
```

도커 옵션 복습:
* `--name`: 컨테이너 이름
* `-v`: volume 마운트
* `-p`: port
* `restart`: docker를 실행시킬 때마다 container를 restart 할 지 여부

예전에 `docker exec -it {name} bash`로 버그찾으려 컨테이너를 헤집고 다녔던 기억이 난다.

계속 진행해보자.

## nginx 설치

사이트에 따르면 bitwarden은 ssl커넥션이 필수이기 때문에 nginx를 설치해야 한다고 한다.

난 근데 여지껏 웹서버를 계속 써오면서 느끼는거지만, 이게 하는 일이 뭔지 아직도 잘 모르겠다.\
그냥 내부 연결을 그대로 외부로 노출시키면 안되나?

안되니까 이러고 있는거겠지? (그런걸 제어하는게 웹서버의 역할일까?)

일단 지금은 내부에서 `localhost:8080`으로 접근하면 뭔가 페이지가 나타나는데, `raspberrypi:8080`으로는 접근이 안되는 상황이다. (CONNECTION_REFUSED)

[nginx를 설치해보자.](https://pimylifeup.com/raspberry-pi-nginx/)

```bash
sudo apt update
sudo apt upgrade
sudo apt install nginx
sudo systemctl start nginx
hostname -I
```

외부에서 `80`포트로 접속했을 때 nginx 웰컴페이지가 정상적으로 출력되었다.

## Lets SSL

예전에는 SSL을 하려면 돈을 내고 엄청 복잡한 절차를 거쳤어야 했는데,\
시대가 좋아져서 전자에 해당하는 부분은 해결이 된 모양이다.

공짜 SSL이라니!

그나저나 예전에 SSL의 역할중에 무슨 신뢰할 수 있는 서버 어쩌고저쩌고 했던것 같은데 이러면 피싱사이트들도 공짜 SSL깔아서 사기치면 되는거 아닌가?

뭐가 되었던건에 값싸게 MITM은 방지할 수 있게 됐으니 좋을 일이다.

* [GUIDE](https://pimylifeup.com/raspberry-pi-ssl-lets-encrypt/)

```bash
sudo apt install python3-certbot-apache
```
apache..?
아까 우리가 설치한건 nginx아닌가?

아 apache가 아닌 사람들은 그냥 `certbot`을 설치하면 된다고 한다.\
무지성으로 따라하다가 일낼뻔했다.

```bash
sudo apt install certbot
```

그럼 이제 ssl 인증서를 받아보자

certbot를 이용해 인증서를 받는 방법은 두 가지인데, 하나는 **standalone**, 다른 하나는 **webroot**방식이다.

[설명](https://elfinlas.github.io/2018/03/19/certbot-ssl/):
* standalone: 이 방법은 Certbot이 간이 웹 서버를 돌려 도메인 인증 요청을 처리하는 방식입니다.
하지만 인증용 간이 서버가 80, 443번 포트를 사용하기 때문에 운영 중인 서버가 해당 포트를 쓰게 된다면, 발급 또는 갱신 시 마다 잠시 서버를 내려야 하는 문제가 있습니다.
물론 해당 포트를 사용하지 않는다면 이 방법을 사용하셔도 무방합니다.
* webroot: 이 방법은 도메인 인증을 위해 외부에서 접근 가능한 경로를 제공하고, Let’s Encrypt 측에서 해당 경로로 접속해 인증을 하는 방식입니다.
이 방식을 사용할 경우 1번 방식의 서비스 종료는 하지 않아도 됩니다.

[document](https://eff-certbot.readthedocs.io/en/stable/using.html)


```bash
 sudo certbot certonly --standalone -d hehehwang.com -d www.hehehwang.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): hehehwang@gmail.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: n
Account registered.
Requesting a certificate for hehehwang.com and www.hehehwang.com
Performing the following challenges:
http-01 challenge for hehehwang.com
http-01 challenge for www.hehehwang.com
Cleaning up challenges
Problem binding to port 80: Could not bind to IPv4 or IPv6.
```

아 그 전에 nginx를 꺼야한다

```bash
sudo service nginx stop
```

```bash
pi@raspberrypi:~ $ sudo certbot certonly --standalone -d bw.hehehwang.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Requesting a certificate for bw.hehehwang.com
Performing the following challenges:
http-01 challenge for bw.hehehwang.com
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/bw.hehehwang.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/bw.hehehwang.com/privkey.pem
   Your certificate will expire on 2022-06-18. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

이제 nginx 설정을 바꿔보자

`/etc/nginx/sites-available/default`\
를 잘 조지면 된다는 것 같다.

```
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```
이걸

```
server {
        listen 80 default_server;
        listen [::]:80 default_server

        listen 443 ssl;

        root /usr/share/nginx/html;
        index index.html index.htm;

        server_name example.com;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

이렇게 잘 수정해보자.
