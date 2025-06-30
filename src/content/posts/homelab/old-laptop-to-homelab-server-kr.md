---
title: 오래된 노트북 홈랩 서버로 사용하기
published: 2025-06-29
description: '잠자던 씽크패드가 이세계에선 24시간 꺼지지 않는 서버...?'
image: ''
tags: [homelab, ubuntu-server]
category: 'homelab'
draft: false 
lang: 'kr'
---

> 잠자던 씽크패드가 이세계에선 24시간 꺼지지 않는 서버...?

# 배경

![](images/2025-06-29-14-09-46.png)

오래된 씽크패드 노트북이 있습니다. 살 때 꽤나 비싸게 샀는데 시간이 지나다보니 맥북만 사용하고 씽크패드는 전혀 사용하지 않게 되더라고요.
그동안 홈랩에 대한 로망이 있기도 했고, 시간적 여유도 생겨 이 씽크패드를 홈랩 서버로 사용할 예정입니다.
어떤 서비스를 호스팅 해야 할지는 모르겠지만 뭐 일단 만들어 놓으면 어떻게든 사용하지 않을까요?

# 운영체제

지금 당장의 이 씽크패드는 윈도우 os를 사용하고 있습니다. 
아무리 생각해도 홈랩 서버로써 윈도우 보다는 리눅스가 당연하다는 생각이 들었어요.

제가 생각하는 리눅스가 나은 큰 이유들은:
1. 오픈소스 프로젝트가 주는 왠지 모를 편안한 느낌
2. 효율적인 컴퓨팅 리소스 활용
    - 불필요한 리소스 사용을 줄여 전기요금을 최대한으로 절약하고 싶습니다
3. 높은 안정성
    - 윈도우 강제 업데이트로 많은 상처를 받았던 저는, 서버가 강제 업데이트에 들어가는 상황을 피하고 싶었습니다
4. 개발자 경험
    - 개인적으로 윈도우 CLI보다 리눅스 CLI 환경이 훨씬 익숙합니다. 원격 접속이나 서비스 배포 역시 리눅스 환경에서 진행하는 것이 훨씬 편리하게 느껴졌습니다

이러한 이유로 리눅스 배포판을 찾아보았고, 가장 접근성이 좋고 참고 자료가 많은 **우분투 서버(Ubuntu Server)** 를 최종적으로 선택했습니다.

:::note
**윈도우 정품 인증에 대한 걱정**

제 씽크패드는 윈도우가 기본으로 설치된 제품이었습니다. 그래서 리눅스를 설치하기 위해 윈도우를 삭제할 경우, 나중에 다시 돌아가고 싶을 때 정품 인증은 어떻게 해야 할지 걱정되었습니다.
찾아보니 요즘 노트북은 대부분 마더보드에 윈도우 정품 키가 내장되어 있을 확률이 높다는 하더라고요.
따라서 추후 윈도우를 다시 설치하기만 하면 자동으로 정품 인증이 완료되므로, 이 부분에 대한 걱정은 할 필요가 없어 보였습니다.
:::

우분투 서버 버전은 가장 최신 LTS버전인 `24.04.2 LTS`로 결정하고 다운로드를 하다가, 제가 USB 드라이브가 없다는걸 깨달았습니다.
평소에는 굴러 다니는게 USB인데 왜 하필 지금 못찾겠는지... 
그래서 카메라용 마이크로SD를 SD카드 리더에 물리고, 여기에 [Balena Etcher](https://etcher.balena.io/)로 USB 부팅 드라이브를 만들었습니다.
이미지를 굽는 과정은 [여기](https://ubuntu.com/tutorials/create-a-usb-stick-on-macos)에 매우 자세하게 적혀 있었습니다.

부팅 USB를 준비 한 후, [우분투 서버 설치 가이드](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview)와 [씽크패드 공식 리눅스 설치 가이드](https://download.lenovo.com/pccbbs/mobiles_pdf/tp_p1_gen2_ubuntu_18.04_lts_installation_v1.0.pdf_)를 참고해 우분투 서버를 설치했습니다. 
생각보다도 더 간단하더라구요. 이렇게 자세하게 잘 적힌 문서들이 더 많아졌으면 좋겠네요.

그리고 제 맥으로 돌아와 

## 우분투 서버 설치 후 세팅

우분투 서버 설치 후 첫번째로 마주친 문제는 콘솔 글자가 너무 작다는거 였습니다.
너무 작아서 도저히 뭘 읽을 수가 없더라구요. 
글자가 이렇게 작은 이유는 제 씽크패드가 4K 모니터인데 콘솔 환경의 기본 폰트가 픽셀 기반 고정 크기라 매우 작게 보이는 것으로 의심 됬습니다.

그래서 `console-setup`도구로 아래와 같이 폰트 사이즈를 조정 했습니다.

1. `sudo dpkg-reconfigure console-setup`
2. 인코딩은 UTF-8로 유지
3. 문자 집합도 "Guess optimal character set"으로 유지
4. 제가 개인적으로 좋아하는 Terminus 폰트로 변경했습니다
5. Font size를 가장 큰 `16x32`로 변경 했습니다

이제 드디어 읽을 수 있는 크기가 되었네요.

그 후 우분투 서버에서 `ip a`로 IP 주소를 가져온 후, 맥에서 아래 포맷으로 ssh 연결을 시도해보니 무한 로딩 증상이 있네요...
```bash frame="none"
ssh <USER>@<SERVER IP>
```
그래서 `ping <SERVER IP>`을 해보니 성공하는 것을 확인했습니다.
이건 서버 방화벽 문제라 확신하고, 서버에서 방화벽 확인을 아래 커맨드로 해봤습니다. 
```bash frame="none"
sudo ufw status
```
SSH 규칙이 없는 것을 확인하고, 방화벽 활성화 후 ssh 접속을 허용하기 위해 22번 포트를 열어줬습니다.
```bash frame="none"
sudo ufw enable && sudo ufw allow ssh
```
그리고 맥에서 ssh 연결을 시도하니 문제 없이 작동하네요!
```bash frame="none"
ssh junsu@<SERVER IP>
The authenticity of host '<SERVER IP> (<SERVER IP>)' can't be established.
ED25519 key fingerprint is SHA256:F+2M6uBdFrj9gRdsX/vSSRngAYGcehtsyDCe4koXC2o.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '<SERVER IP>' (ED25519) to the list of known hosts.
junsu@<SERVER IP>'s password: 
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Jun 30 01:25:28 AM UTC 2025

  System load:  0.16              Temperature:                36.0 C
  Usage of /:   6.6% of 97.87GB   Processes:                  181
  Memory usage: 2%                Users logged in:            1
  Swap usage:   0%                IPv4 address for wlp0s20f3: <SERVER IP>


Expanded Security Maintenance for Applications is not enabled.

57 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status
```
그럼 이제 노트북 서버인 만큼 노트북을 닫아도 대기 상태에 들어가지 않게 다음 설정들을 입력해 줍시다.

설정 파일을 열고
```bash frame="none"
# 설정 파일 열기
sudo vim /etc/systemd/logind.conf
```
다음 설정들을 아래와 같이 변경합니다.
```bash frame="none"
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```
하나씩 살펴보면:
- `HandleLidSwitch=ignore`: 노트북이 배터리 전원으로 작동하고 있을때 노트북 닫는걸 무시하기
- `HandleLidSwitchExternalPower=ignore`: 노트북이 전원 연결되어 있을 때 노트북 닫는걸 무시하기
- `HandleLidSwitchDocked=ignore`: 노트북이 도킹 스테이션에 연결되어 있트 때 (저는 없지만...) 노트북 닫는걸 무시하기

이제 로그인 서비스를 재시작하여 새로운 설정을 적용해줍니다.
```bash frame="none"
sudo systemctl restart systemd-logind.service
```

그리고 추가로, 노트북에 아무런 키보드 입력이 없으면 정해진 시간 후에 자동으로 화면을 끄는 설정을 추가해야 합니다. 

다음 파일을 열고
```bash frame="none"
sudo vim /etc/default/grub
```

`GRUB_CMDLINE_LINUX_DEFAULT`를 찾아 아래의 설정을 추가합니다:
```bash frame="none"
GRUB_CMDLINE_LINUX_DEFAULT="consoleblank=180"
```
- 180초(3분) 동안 키보드 입력이 없으면 화면을 끄기로 설정

그 후 파일 저장 후 GRUB 설정을 시스템에 적용하고 재부팅을 하면 자동 화면(만) 커짐 모드가 활성화 됩니다.
```bash frame="none"
sudo update-grub
```
그 후 화면이 저절로 꺼지는지, 노트북이 덮힌 상태로 원격 접속이 가능한지 확인해 보니 정상 작동 하는것을 확인했습니다.

# 마무리
이렇게 먼지 쌓인 씽크패드에 우분투 서버를 설치하여 할 일을 주었습니다. 
운영체제를 선택하는 고민부터 시작해, 작은 글씨와 방화벽이라는 예상치 못한 장애물을 해결하고, 덮개를 닫거나 화면이 꺼져도 24시간 동작하는 진짜 서버의 모습을 갖추게 되었습니다. 이제 제 맥북 터미널에서 SSH로 접속할 때마다, 구석에서 조용히 제 역할을 다하고 있을 씽크패드를 생각하면 왠지 모를 든든함이 느껴집니다.

물론 아직은 텅 빈 깡통에 불과합니다. 이 서버의 진짜 가치는 앞으로 어떤 서비스를 올리고 어떻게 활용하는지에 달려 있겠죠.

다음 목표는 이 서버에 실질적인 쓸모를 부여하는 **'네트워크 구성과 서비스 배포'**입니다. 내부 네트워크에서만 맴도는 서버를 넘어, 외부에서도 제가 만든 서비스에 안전하게 접근할 수 있는 환경을 만드는 것이 최종 목표입니다. 이 과정에서 아래와 같은 기술들을 탐색해 볼 예정입니다.

- 컨테이너 기술을 이용해 여러 서비스를 쉽고 깔끔하게 관리하기
- 리버스 프록시 (Reverse Proxy): 여러 웹 서비스를 도메인 주소로 깔끔하게 연결하기
- DDNS와 SSL 인증서: 유동적인 집 IP 환경에서 도메인을 유지하고, 안전한 HTTPS 통신 환경 구축하기

꾸준히 해보겠습니다. 감사합니다.
