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


