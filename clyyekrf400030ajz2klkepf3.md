---
title: "평생 무료 서버를 구축해보자!"
datePublished: Tue Jul 23 2024 12:39:38 GMT+0000 (Coordinated Universal Time)
cuid: clyyekrf400030ajz2klkepf3
slug: 7yj7iodioustoujjcdshjzrsotrpbwg6rws7lav7zw067o07j6qiq
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/xbEVM6oJ1Fs/upload/34f11af65d64da7e1150cb03af018668.jpeg
tags: ubuntu, oracle, nginx

---

# 오라클

오라클 클라우드는 무료로 사용할 수 있는 다양한 클라우드 서비스를 제공하고 있습니다. 이 글에서는 오라클 클라우드의 무료 서버를 설정하고 세팅하는 방법을 단계별로 설명하겠습니다.

## 회원가입

오라클 클라우드를 사용하기 위해서는 먼저 계정을 생성해야 합니다. 다음 단계에 따라 계정을 생성하세요.

1\. [오라클 클라우드 홈페이지](https://www.oracle.com/kr/cloud/free/)에 접속합니다.

2\. “무료로 시작하기” 버튼을 클릭합니다.

3\. 이름, 이메일 주소 등 필요한 정보를 입력하고 계정을 생성합니다. (신용 카트 확인 필요)

4\. 이메일 인증을 완료한 후 로그인합니다.

## VM 설정

로그인하고 처음보게 되는 화면입니다

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721737207000/91516a77-c9f3-4081-997a-0e57023bf322.png align="center")

다양한 선택지를 제공하는데 우리가 필요한것은 VM 서버!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721737229569/185f82e3-c358-4088-99ce-e2563de6976c.png align="center")

리소스 실행 -&gt; 'VM 인스턴스 생성 메뉴'를 클릭합니다

아래 단계에 맞게 설정값만 지정하면 끝

SSH 키를 저장하는 것을 까먹지 말자

1. VM 이름 입력
    
2. OS 선택 (Oracle Linux 기본, Ubuntu 선택 가능)
    
3. SSH 키 저장
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721737354117/7dd95b2f-be8e-467a-9271-0f1596532794.png align="center")

## ssh 접속

VM이 만들어지면 아래 화면이 보인다

다양한 정보와 함께 우리가 확인해야할 부분은 ip 주소

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721737412755/f0f785bd-591e-4e43-afca-c82430f33ece.png align="center")

미리 저장해둔 key를 이용하여 해당 ip주소에 접근한다

```bash
ssh -i [path_to_private_key] [username]@[instance_ip]
```

정상적으로 접근한 것을 확인할 수 있을 것이다

```bash
ubuntu@instance:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           958Mi       468Mi        98Mi       748Ki       560Mi       489Mi
Swap:          4.0Gi        11Mi       4.0Gi

ubuntu@instance:~$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
tmpfs              98184    1120     97064   2% /run
efivarfs             256      17       235   7% /sys/firmware/efi/efivars
/dev/sda1       46212176 6542200  39653592  15% /
tmpfs             490908       0    490908   0% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
/dev/sda16        901520  146568    691824  18% /boot
/dev/sda15        106832    6246    100586   6% /boot/efi
tmpfs              98180      12     98168   1% /run/user/1001
```

## nginx 및 방화벽 설정

이제 서버와 통신을 nginx를 통해 열어주도록 하자

```bash
sudo apt-get update
sudo apt-get install nginx -y
```

nginx가 정상적으로 설치가 되었다

```bash
ubuntu@instance:~$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-07-23 12:28:54 UTC; 2min 17s ago
       Docs: man:nginx(8)
    Process: 14587 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 14591 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 14597 (nginx)
      Tasks: 3 (limit: 1089)
     Memory: 2.6M (peak: 2.8M)
        CPU: 183ms
     CGroup: /system.slice/nginx.service
             ├─14597 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─14599 "nginx: worker process"
             └─14600 "nginx: worker process"

Jul 23 12:28:54 instance-20240723-2036 systemd[1]: Starting nginx.service - A high performance web server and a reverse>
Jul 23 12:28:54 instance-20240723-2036 systemd[1]: Started nginx.service - A high performance web server and a reverse >
lines 1-17/17 (END)
```

대시보드에서도 포트를 열어줘야한다

그래야 웹에서 해당 서버 IP로 접근이 가능하다

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721738004658/4fb30ace-a5df-4b3e-a429-78729e7a0254.png align="center")

서브넷에 접근한다

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721738044275/378a22ce-318a-4b35-b25b-1c6f2849de40.png align="center")

보안목록에 이름 접근 -&gt; 신규 규칙 추가

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721738081309/db87ba87-eb92-4e61-bb4c-cc5b683df426.png align="center")

위와 같이 80 포트를 열어준다

그리고 주소창에 해당 VM의 ip주소를 입력하면?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721738109107/7225831e-e244-4694-baad-8e63054a33ee.png align="center")

짜잔~

## 결론

오라클 클라우드의 무료 서버를 활용하면 기본적인 웹 서버 세팅뿐만 아니라 다양한 사이드 프로젝트를 손쉽게 시작할 수 있다.

이 가이드를 따라 서버를 설정하고, 웹 서버를 세팅하며, 간단한 애플리케이션을 배포할 수 있다.

무려 오라클인 만큼 다양한 기능도 있고 프로젝트를 수행하면서 클라우드 환경에서의 경험을 쌓을 수 있다.

무료 너무 좋아하면 큰일나지만 이 정도 자원이면 적극적으로 활용하는 것을 추천한다!